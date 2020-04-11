---
title: "区块链Fabric性能测试"
date:  2019-09-21 15:33:03
categories: 区块链
tags:
- stupid
- caliper
---

本文主要介绍区块链Fabric性能测试的2个测试工具“caliper”和“stupid”。这里假设已启动Fabric网络，如果没有启动，请参考部署Fabric网络博客。

# 性能测试工具

## stupid

### stupid概述

Stupid 是一个轻量级的 fabric 网络性能测试工具，与其他现有的 fabric 网络测试工具相比（例如 caliper），它的主要特点和优势在于： 

• 不使用任何的 SDK

• 不尝试部署 fabric 网络

• 不依赖于连接配置文件

• 不会发现节点、链码或策略

• 不监视资源的使用情况

同时，它用于执行超级简单的性能测试，具体流程如下所示：

• 直接建立大量的 gRPC 连接

• 通过 gRPC 客户端向 peer 节点发送已签署的提案

• 把经过背书的 response 装在 envelops 里 

• 将 envelop 发送给 orderer

• 观察交易提交结果

正如它的名字所代表的意思一样，这个工具非常 stupid，以至于它不会成为性能测试的瓶颈！

### 环境准备

1. 安装go

   ```shell
   wget https://dl.google.com/go/go1.13.5.linux-amd64.tar.gz
   sudo tar -C /usr/local -xzf go1.13.5.linux-amd64.tar.gz
   ```

   接下来编辑当前用户的环境变量：

   ```shell
   vi /etc/profile
   ```

   添加以下内容：

   ```shell
   export PATH=$PATH:/usr/local/go/bin 
   export GOROOT=/usr/local/go 
   export GOPATH=$HOME/gopath 
   export PATH=$PATH:$HOME/gopath/bin
   ```

   编辑保存并退出vi后，记得把这些环境载入：

   ```shell
   source /etc/profile
   ```

2. 准备stupid

   ```shell
   git clone	git@github.com:guoger/stupid.git
   cd stupid
   go build
   ```

3. 配置文件介绍

   ```
   {
     "peer_addr": "peer1.org1.example.com:8051",
     "orderer_addr": "orderer.example.com:7050",
     "channel": "mychannel",
     "chaincode": "stupid",
     "args": ["put", "a", "10"],
     "mspid": "Org1MSP",
     "private_key": "keystore/2768d5cda759d71084405ac96671193de529c084ae8ececcfb997aad5f5a6d41_sk",
     "sign_cert": "signcerts/Admin@org1.example.com-cert.pem",
     "tls_ca_certs": ["ca.crt","ca.crt"],
     "num_of_conn": 20,
     "client_per_conn": 40
   }
   ```

   

+ peer_addr: 表示 peer 节点的地址,请在/etc/hosts 中添加 peer 节点的 IP 映射，例如

  ```
  192.168.9.96	 peer0.org1.example.com
  ```

+ orderer_addr:表示 orderer 节点的地址, 请在/etc/hosts 中添加 orderer 节点的 IP 映射，例如

  ```shell
  192.168.9.96	 orderer.example.com
  ```

+ mspid:用户身份对应的MSP_ID: 与下面的私钥和身份对应的MSPID一致

  ```
  Org1MSP
  ```

+ private_key:私钥的路径。

  ```
  ./crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/2768d5cda759d71084405ac96671193de529c084ae8ececcfb997aad5f5a6d41_sk
  ```

+ sign_cert:	 用户证书的路径。

  ```
  ./crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem
  ```

+  tls_ca_certs:	peer 节点和 orderer 节点的 TLS	CA 证书。如果 tls 被禁用，则此处可不做设置。

  ```
  "./crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls/ca.crt","./crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/ca.crt"
  ```

+ channel：通道名

+ chaincode:	 调用的链码名。在 chaincodes 目录中有一个链码的例子：sample.go。这个例子中只调用一个 put 函数，设置 key:value。 

+ args：调用链码时需要发送的参数，需要根据链码的具体实现来设置。
+  num_of_conn: 在 client/peer、client/orderer 之间建立的 gRPC 连接数。通过增加这个数值可以增加客户端给 fabric 的压力。

+  client_per_conn:	 每个连接用于向 peer 发送提案的客户端数量。通过增加这个数值可以增加客户端给 fabric 的压力。

4. 安装stupid合约

   本文测试时安装stupid提供的sample.go智能合约。并命名为"stupid"。安装链码参考Fabric网络部署。

### 性能测试

在stupid目录下调用stupid进行性能测试，下列命令代表发送10000笔交易进行性能测试。

```
./stupid config.json 10000
```

测试结果如下所示：

```shell
Time 2m9.344959248s	Block 7000	Tx 10
Time 2m9.424999126s	Block 7001	Tx 10
Time 2m9.504853386s	Block 7002	Tx 10
Time 2m9.575175031s	Block 7003	Tx 10
Time 2m9.644561654s	Block 7004	Tx 10
Time 2m9.735391576s	Block 7005	Tx 10
tx: 10000, duration: 2m9.735438456s, tps: 77.079941
```

> Note: 注意：交易数应当设置为 batchsize 的整数倍，这是为了防止最后一个区块因为超时而被截断。例如，如果 batchsize 设置为 500，那么交易数可设置为500，1000，40000，100000 等。

## caliper

### caliper概述

Caliper是一个区块链性能基准框架，允许用户使用自定义用例测试不同的区块链解决方案，并获得一组性能测试结果。

当前支持测试的项目如下：

```
Hyperledger Besu
Hyperledger Burrow
Ethereum
Hyperledger Fabric
FISCO BCOS
Hyperledger Iroha
Hyperledger Sawtooth
```

支持的性能指标如下：

+ 事务/读取吞吐量
+ 事务/读取延迟（最小，最大，平均，百分位数）
+ 资源消耗（CPU，内存，网络IO等）

### caliper 环境准备

1. caliper安装

   [caliper安装](https://hyperledger.github.io/caliper/vLatest/installing-caliper/)存在很多方式，这里建议使用docker容器方式。

   ```
   docker pull hyperledger/caliper:0.1.0
   ```

   > Note: caliper也存在很多版本，需要结合fabric版本选择对应版本。

2. 配置caliper

   docker-compose.yaml

   ```
   version: '2'
   
   services:
       caliper:
           container_name: caliper
           image: hyperledger/caliper:0.1.0
           command: launch master
           environment:
           - CALIPER_BIND_SUT=fabric:1.4.0
           - CALIPER_BENCHCONFIG=benchmarks/scenario/simple/config.yaml
           - CALIPER_NETWORKCONFIG=networks/fabric/fabric-v1.4.1/2org1peergoleveldb/fabric-go.yaml
           volumes:
           - ~/caliper-benchmarks:/hyperledger/caliper/workspace
   ```

3. 配置文件介绍

   config.yaml：该文件主要是压测相关配置，一般调整一下tps即可。

   ```yaml
   ---
   test:
     name: simple
     description: This is an example benchmark for caliper, to test the backend DLT's
       performance with simple account opening & querying transactions
     clients:
       type: local
       number: 1
     rounds:
     - label: open
       description: Test description for the opening of an account through the deployed chaincode
       txNumber:
       - 100
       rateControl:
       - type: fixed-rate
         opts:
           tps: 50
       arguments:
         money: 10000
       callback: benchmark/simple/open.js
     - label: query
       description: Test description for the query performance of the deployed chaincode
       txNumber:
       - 100
       rateControl:
       - type: fixed-rate
         opts:
           tps: 100
       callback: benchmark/simple/query.js
     - label: transfer
       description: Test description for transfering money between accounts
       txNumber:
           - 100
       rateControl:
           - type: fixed-rate
             opts:
                 tps: 50
       arguments:
           money: 100
       callback: benchmark/simple/transfer.js
   monitor:
     type:
     - docker
     - process
     docker:
       name:
       - all
     process:
     - command: node
       arguments: local-client.js
       multiOutput: avg
     interval: 1
   ```

   fabric-go.yaml： 该文件是sdk使用需要的fabric网络配置文件。

   ```yaml
   name: Fabric
   version: "1.0"
   mutual-tls: false
   
   caliper:
     blockchain: fabric
     command:
       start: docker-compose -f network/fabric-v1.4.1/2org1peergoleveldb/docker-compose.yaml up -d;sleep 3s
       end: docker-compose -f network/fabric-v1.4.1/2org1peergoleveldb/docker-compose.yaml down;(test -z \"$(docker ps -aq)\") || docker rm $(docker ps -aq);(test -z \"$(docker images dev* -q)\") || docker rmi $(docker images dev* -q);rm -rf /tmp/hfc-*
   
   info:
     Version: 1.4.0
     Size: 2 Orgs with 1 Peer
     Orderer: Solo,
     Distribution: Single Host
     StateDB: GoLevelDB
   
   clients:
     client0.org1.example.com:
       client:
         organization: Org1
         credentialStore:
           path: /tmp/hfc-kvs/org1
           cryptoStore:
             path: /tmp/hfc-cvs/org1
         clientPrivateKey:
           path: network/fabric-v1.4.1/config/crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/keystore/key.pem
         clientSignedCert:
           path: network/fabric-v1.4.1/config/crypto-config/peerOrganizations/org1.example.com/users/User1@org1.example.com/msp/signcerts/User1@org1.example.com-cert.pem
   
   channels:
     mychannel:
       configBinary: network/fabric-v1.4.1/config/mychannel.tx
       created: false
       orderers:
       - orderer.example.com
       peers:
         peer0.org1.example.com:
           eventSource: true
         peer0.org2.example.com:
           eventSource: true
     chaincodes:
       - id: simple
         version: v0
         language: golang
         path: contract/fabric/simple/go
       - id: smallbank
         version: v0
         language: golang
         path: contract/fabric/smallbank
         
   organizations:
     Org1:
       mspid: Org1MSP
       peers:
       - peer0.org1.example.com
       certificateAuthorities:
       - ca.org1.example.com
       adminPrivateKey:
         path: network/fabric-v1.4.1/config/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/key.pem
       signedCert:
         path: network/fabric-v1.4.1/config/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem
   
   orderers:
     orderer.example.com:
       url: grpc://localhost:7050
       grpcOptions:
         ssl-target-name-override: orderer.example.com
   
   peers:
     peer0.org1.example.com:
       url: grpc://localhost:7051
       grpcOptions:
         ssl-target-name-override: peer0.org1.example.com
         grpc.keepalive_time_ms: 600000
   
   certificateAuthorities:
     ca.org1.example.com:
       url: http://localhost:7054
       httpOptions:
         verify: false
       registrar:
       - enrollId: admin
         enrollSecret: adminpw
   ```

### 性能测试

1. 启动caliper

   ```
   docker-compose up -f docker-compose.yaml
   ```

2. 测试结果

   会在~/caliper-benchmarks目录下生成一个html文件。如下所示：

   ### Summary

   | Test | Name     | Succ | Fail | Send Rate | Max Latency | Min Latency | Avg Latency | Throughput |
   | ---- | -------- | ---- | ---- | --------- | ----------- | ----------- | ----------- | ---------- |
   | 1    | open     | 100  | 0    | 50.7 tps  | 0.63 s      | 0.13 s      | 0.30 s      | 47.6 tps   |
   | 2    | query    | 100  | 0    | 101.9 tps | 0.04 s      | 0.01 s      | 0.02 s      | 99.2 tps   |
   | 3    | transfer | 70   | 30   | 50.6 tps  | 0.45 s      | 0.14 s      | 0.30 s      | 33.1 tps   |