---
title: "区块链Fabric源码之交易提案的创建"
top: true
date:  2020-01-28 10:13:03
categories: 区块链
tags:
- fabric
- 
---

本文主要介绍区块链Fabric交易创建流程。

## Fabric 1.4.1源码解析——交易提案的创建

### 概念介绍

#### 交易执行模型

区块链 Fabric 引入了一种新的交易模型，称为**执行-排序-验证**。它将交易流程分为三个步骤：

- 执行一个背书提案并检查其正确性，从而给它背书
- 创建交易并通过（可插拔的）共识协议将交易排序
- 提交交易到账本前先根据特定应用程序的背书策略验证交易

通过将交易流程解耦为这三个阶段，可以提升整个系统的灵活性、可伸缩性、性能和机密性问题。

#### 交易分类

在Fabric中，将交易分为调用交易和部署交易：

- 部署交易，创建新的链码并将程序作为参数。主要指为通道实例化或者升级链码。
- 调用交易，在部署的链码环境中执行操作。主要是指调用链码对应方法。

### 源码解析

上述简单介绍了Fabric里面交易的执行模型以及分类。接下来将介绍在整个交易执行过程中，交易提案是如何创建的。

如果大家执行过单机部署，应该了解在cli容器里面执行以下命令的含义：

```shell
peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $CHANNEL_NAME  -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'
```

该命令的意思是调用mycc链码的invoke方法，使得a往b转10块钱。本文后续将从这个命令出发，介绍交易提案的构建过程。

#### 交易提案数据结构

首先，看看交易提案的数据结构是怎样的。

![提案消息数据结构](../../../../../../../Desktop/%E6%8F%90%E6%A1%88%E6%B6%88%E6%81%AF%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84.png)

交易提案SignedProposal包括提案ProposalBytes以及签名Signature信息（和提案Header的创建者creator身份对应）。提案内容ProposalBytes主要包括头部Header以及内容Payload。其中头部主要包含创建者creator身份信息以及通道、交易、链码相关信息。而Payload包含主要的调用信息。

#### 交易提案创建流程

首先，从源码看看如何生成交易提案SignedProposal的。

```go
func GetSignedProposal(prop *peer.Proposal, signer msp.SigningIdentity) (*peer.SignedProposal, error) {
	// check for nil argument
	if prop == nil || signer == nil {
		return nil, errors.New("nil arguments")
	}
	propBytes, err := GetBytesProposal(prop)
	if err != nil {
		return nil, err
	}
	signature, err := signer.Sign(propBytes)
	if err != nil {
		return nil, err
	}
	return &peer.SignedProposal{ProposalBytes: propBytes, Signature: signature}, nil
}
```

从上面的源码可以看出，交易提案SignedProposal的签名是创建该提案的创建者对ProposalBytes进行签名后的返回值。接下来，将重点关注一下Proposal的生成过程。其主要实现是在：

```go
func CreateChaincodeProposalWithTxIDNonceAndTransient(txid string, typ common.HeaderType, chainID string, cis *peer.ChaincodeInvocationSpec, nonce, creator []byte, transientMap map[string][]byte) (*peer.Proposal, string, error) {
	// 序列化ChaincodeHeaderExtension
  ccHdrExt := &peer.ChaincodeHeaderExtension{ChaincodeId: cis.ChaincodeSpec.ChaincodeId}
	ccHdrExtBytes, err := proto.Marshal(ccHdrExt)
	// 序列化ChaincodeProposalPayload
  cisBytes, err := proto.Marshal(cis)
	ccPropPayload := &peer.ChaincodeProposalPayload{Input: cisBytes, TransientMap: transientMap}
	ccPropPayloadBytes, err := proto.Marshal(ccPropPayload)
	
	// 生成Header，并序列化
	hdr := &common.Header{
		ChannelHeader: MarshalOrPanic(
			&common.ChannelHeader{
				Type:      int32(typ),  // 消息类型
				TxId:      txid,				// 交易id
				Timestamp: timestamp,		// 时间戳
				ChannelId: chainID,			// 通道id
				Extension: ccHdrExtBytes,	// 仅包含chainId信息（Path、Name、Version）
				Epoch:     epoch,					// 当前默认是0
			},
		),
		SignatureHeader: MarshalOrPanic(
			&common.SignatureHeader{
				Nonce:   nonce,	// 随机数，防止重放攻击
				Creator: creator, // 创建者身份信息
			},
		),
	}
	hdrBytes, err := proto.Marshal(hdr)
	if err != nil {
		return nil, "", err
	}
	// 生成Proposal
	prop := &peer.Proposal{
		Header:  hdrBytes,
		Payload: ccPropPayloadBytes,
	}
	return prop, txid, nil
}

```

从上述代码，我们结合Proposal数据结构可以了解整个Proposal的生成过程。