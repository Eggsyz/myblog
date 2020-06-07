---
title: "区块链Fabric智能合约指南"
top: true
date:  2020-04-20 10:13:03
categories: 区块链
tags:
- fabric
- chaincode
---



## Fabric 智能合约指南



本文首先介绍了智能合约的定义、生命周期、类型及背书策略，然后介绍智能合约的整体架构以及编写智能合约需要实现的接口。最后介绍区块链智能合约编写的一些注意事项。

### 智能合约介绍

在联盟链Fabric中，智能合约也称为链码（chaincode），分为应用链码和系统链码。系统链码用来实现系统层面的功能，包括系统的配置，应用链码的部署等；应用链码用于实现用户的实际业务功能。当前链码支持的编程语言：Golang，Java，node.js。

![img](https://mmbiz.qpic.cn/mmbiz_png/eHN7rfCHfK8uHHHv3YLzkXrpGqfc37YOFo2GKLBphw4fQqaaXAup5IBhLfUNEibTciaJRhSK2MQQkUWXXgIic5Bgw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

### 链码的分类

链码分为**用户链码**和**系统链码**两部分。**系统链码**用来实现系统层面的功能，包括系统的配置，应用链码的部署等。系统链码主要分为以下几类：

- 生命周期系统链码(LSCC) ：Lifecycle System Chaincode，负责对应用链码的生命周期进行管理。
- 配置系统链码(CSCC) ：Configuration System Chaincode，负责处理 Peer 端的 Channel 配置。
- 查询系统链码(QSCC) ：Query System Chaincode，提供账本查询 API，如获取区块和交易等信息。
- 验证系统链码(VSCC) ：Validator System Chaincode，提供验证 API，如检查背书策略和读写集版本。
- 背书系统链码(ESCC)：Endorsorement System Chaincode，对提案响应进行签名，提供背书功能。

**应用链码**用于实现用户的实际业务功能，开发者开发链码应用程序并将其部署到区块链网络上，用户可以通过客户端或者提供的SDK调用链码。

### 链码的生命周期

**应用链码**运行在一个和背书节点进程隔离的一个安全的 Docker 容器中。应用链码的实例化/升级和账本状态的管理都是通过发送交易调用链码来实现的。在区块链中，引入四个命令来管理链码的生命周期：package、install、instantiate 和 upgrade。在一个链码成功地安装和实例化之后，链码处于运行状态，它可以通过 invoke 来处理交易。

![img](https://mmbiz.qpic.cn/mmbiz_png/eHN7rfCHfK8uHHHv3YLzkXrpGqfc37YOK7hsB4tUpEWAe9Uc3S4dnRVY2oBFjHyRPXASEDdb0vgmwQBiaeyuOXQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

链码的生命周期图如图所示：

1. 通过install安装链码，通过instantiate实例化链码。
2. 链码在成功install以及instantiate后，以通过invoke、query调用链码和查询链码
3. 如果需要升级链码，则需要先install安装新版本的链码，通过upgrade升级链码。
4. 在install安装链码前，可以通过package打包并签名生成打包文件，然后在通过install安装。

### 链码开发

本文主要介绍Go语言开发智能合约。使用Go语言开发链码需要定义一个struct实现接口Chaincode，定义main函数作为链码的启动入口。并且每个接口都必须实现以下两个接口：

```go
type Chaincode interface {  
	Init(stub ChaincodeStubInterface) pb.Response 
  Invoke(stub ChaincodeStubInterface) pb.Response
}
```

其中Init方法在链码实例化(instantiate)或升级(upgrade)时会被调用。当链码收到调用（invoke）或查询（query）类型的交易时，Invoke方法会被调用。

**链码样例**如下所示：

```go
package main

// 引入必要的包
import (
    "github.com/hyperledger/fabric/core/chaincode/shim"
    pb "github.com/hyperledger/fabric/protos/peer"
)

// 声明一个结构体
type SimpleChaincode struct {}

// 为结构体添加Init方法
func (t *SimpleChaincode) Init(stub shim.ChaincodeStubInterface) pb.Response {
    // 在该方法中实现链码初始化或升级时的处理逻辑
    // 编写时可灵活使用stub中的API
}

// 为结构体添加 Invoke 方法
func (t *SimpleChaincode) Invoke(stub shim.ChaincodeStubInterface) pb.Response {
    // 在该方法中实现链码运行中被调用或查询时的处理逻辑
    // 编写时可灵活使用stub中的API
}

// 主函数，需要调用 shim.Start() 方法
func main() {
    err := shim.Start(new(SimpleChaincode))
    if err != nil {
        fmt.Printf("Error starting Simple chaincode: %s", err)
    }
}
```

### 链码开发注意事项

**1. 注意链码的确定性**

在联盟链中，链码的编写需要满足确定性：相同的条件下，相同的输入得到相同的输出。因此需要谨慎使用随机数或者时间相关函数

**2. 避免从Fabric链码中访问外部资源**

在联盟链中，链码应避免访问外部文件或者url。链码背书结果不应该依赖联盟链外的数据。

 **3. 防止panic后链码挂掉**

在联盟链中，最好在Invoke函数入口处添加defer语句，捕捉panic异常，避免容器挂掉。