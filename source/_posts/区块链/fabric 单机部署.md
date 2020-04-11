---
title: "区块链Fabric单机部署"
top: true
date:  2019-08-28 10:33:03
categories: 区块链
tags:
- fabric
---

本文主要介绍区块链Fabric单机部署流程。

## Fabric 1.4.1 版本单机部署

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

2. 安装Docker

   ```shell
   sudo yum install docker-ce
   docker version
   sudo systemctl start docker
   ```

   因为一般docker操作时都需要root用户权限，这里建议把当前用户加入docker用户组。

   ```shell
   groupadd docker
   gpasswd -a eggsy docker
   systemctl restart docker
   systemctl enable docker
   ```

   > Note: 如果普通用户执行docker命令，如果提示get …… dial unix /var/run/docker.sock权限不够，则修改/var/run/docker.sock权限 使用root用户执行sudo chmod a+rw /var/run/docker.sock

3. 安装Docker-Compose

   Docker-compose是支持通过模板脚本批量创建Docker容器的一个组件。

   ```shell
   sudo curl -L "https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose
   docker-compose version
   ```

   > Note: 如果下载比较慢，可以尝试去github上手动下载对应版本的docker-compose。https://github.com/docker/compose/releases。如果无法执行docker-compose。可将docker-compose路径放入PATH，即export PATH=$PATH:docker-compose目录

4. 准备Fabric镜像，这里介绍源码编译方式

   ```shell
   mkdir -p ~/gopath/src/github.com/hyperledger 
   cd ~/gopath/src/github.com/hyperledger 
   git clone https://github.com/hyperledger/fabric.git
   cd fabric
   git checkout v1.4.1
   make all 
   ```

5. 准备fabric-sample

   ```shell
   git clone https://github.com/hyperledger/fabric-samples.git
   cd fabric-samples/
   git checkout release-1.4
   ```

### 单机部署

#### 脚本执行

进入fabric-samples/first-network文件夹，这里提供了启动、关闭fabric网络的自动化脚本。要启动联盟链，并自动运行链码mycc的测试，执行一个命令：

```shenll
cd fabric/fabric-samples/first-network/
./byfn.sh up
```

这个做了以下操作：

1. 基于crypto-config.yaml生成公私钥和证书信息，并保存在crypto-config文件夹中。

2. 基于configtx.yaml生成创世区块和通道相关信息(指定该应用通道名为mychannel)，并保存在channel-artifacts文件夹。

3.  基于docker-compose-cli.yaml启动1Orderer+4Peer+1CLI的Fabric容器。

   在CLI启动的时候，会运行scripts/script.sh文件，这个脚本文件包含了创建通道，加入通道，安装链码，调用链码等功能。

其中，该脚本还提供其他功能，例如选择kafka、raft共识，选择couchdb作为状态数据库等。如下所示：

```shell
Usage:
  byfn.sh <mode> [-c <channel name>] [-t <timeout>] [-d <delay>] [-f <docker-compose-file>] [-s <dbtype>] [-l <language>] [-o <consensus-type>] [-i <imagetag>] [-a] [-n] [-v]
    <mode> - one of 'up', 'down', 'restart', 'generate' or 'upgrade'
      - 'up' - bring up the network with docker-compose up
      - 'down' - clear the network with docker-compose down
      - 'restart' - restart the network
      - 'generate' - generate required certificates and genesis block
      - 'upgrade'  - upgrade the network from version 1.3.x to 1.4.0
    -c <channel name> - channel name to use (defaults to "mychannel")
    -t <timeout> - CLI timeout duration in seconds (defaults to 10)
    -d <delay> - delay duration in seconds (defaults to 3)
    -f <docker-compose-file> - specify which docker-compose file use (defaults to docker-compose-cli.yaml)
    -s <dbtype> - the database backend to use: goleveldb (default) or couchdb
    -l <language> - the chaincode language: golang (default) or node
    -o <consensus-type> - the consensus-type of the ordering service: solo (default), kafka, or etcdraft
    -i <imagetag> - the tag to be used to launch the network (defaults to "latest")
    -a - launch certificate authorities (no certificate authorities are launched by default)
    -n - do not deploy chaincode (abstore chaincode is deployed by default)
    -v - verbose mode
    -b - bccsp SW/GM
  byfn.sh -h (print this message)

Typically, one would first generate the required certificates and
genesis block, then bring up the network. e.g.:

	byfn.sh generate -c mychannel
	byfn.sh up -c mychannel -s couchdb
        byfn.sh up -c mychannel -s couchdb -i 1.4.0
	byfn.sh up -l node
	byfn.sh down -c mychannel
        byfn.sh upgrade -c mychannel

Taking all defaults:
	byfn.sh generate
	byfn.sh up
	byfn.sh down
```

#### 分步部署

本节主要是手动执行上述脚本执行的流程，相关配置文件都由样例abric-samples/first-network提供。后续操作都在该文件夹下执行，如果在手动部署之前已脚本部署，请执行./byfn.sh down清除当前脚本部署环境。

##### 生成公私钥和证书

Fabric中有两种类型的公私钥和证书，一种是给节点之间通讯安全而准备的TLS证书，另一种是用户登录和权限控制的用户证书。这些证书本来应该是由CA来颁发，但是这里是测试环境，并没有启用CA节点，所以通过Fabric提供的工具生成：cryptogen

```shell
cd fabric/fabric-samples/first-network/
cryptogen generate --config=./crypto-config.yaml
```

##### 生成创世区块和Channel配置区块

fabric-samples/first-network/configtx.yaml这个文件里面配置了由2个Org参与的Orderer共识配置TwoOrgsOrdererGenesis，以及由2个Org参与的Channel配置：TwoOrgsChannel。Orderer可以设置共识的算法是solo/kafka/etcdraft（本文样例为solo）以及共识时区块大小，超时时间等，此处使用默认值即可，不用更改。而Peer节点的配置包含了MSP的配置，锚节点的配置。如果存在有更多的Org，那么就可以根据模板进行对应的修改。

1. 生成创世区块

   ```shell
   configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
   ```

2. 生成Channel配置区块

   ```shell
   cd fabric/fabric-samples/first-network/
   configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID mychannel
   ```

3. 锚节点更新文件

   ```shell
   configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx -channelID mychannel -asOrg Org1MSP
   configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx -channelID mychannel -asOrg Org2MSP
   ```

##### 启动fabric环境

整个Fabric Docker环境的配置都放在docker-compose-cli.yaml后，只需要使用以下命令即可：

```shell
cd fabric/fabric-samples/first-network/
docker-compose -f docker-compose-cli.yaml up -d
```

后续操作要进入cli容器内部，在里面创建Channel、安装链码等。先用以下命令进入CLI：

```shell
docker exec -it cli bash
```

1. 创建通道

   ```shell
   docker exec -it cli bash
   export CHANNEL_NAME=mychannel
   export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
   peer channel create -o orderer.example.com:7050 -c $CHANNEL_NAME -f ./channel-artifacts/channel.tx --tls $CORE_PEER_TLS_ENABLED --cafile  $ORDERER_CA
   ```

2. 加入通道

   ```shell
   peer channel join -b <channel-ID.block>
   ```

   那么其他几个Peer又该怎么加入这个mychannel呢？这里就需要修改CLI的环境变量，使其指向另外的Peer。比如我们要把peer1.org1加入mychannel，那么命令是：

   ```shell
   CORE_PEER_LOCALMSPID="Org1MSP" 
   CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt 
   CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp 
   CORE_PEER_ADDRESS=peer1.org1.example.com:8051
   peer channel join -b mychannel.block
   ```

3. 安装链码

   ```shell
   peer chaincode install -n mycc -v 1.0 -p github.com/chaincode/chaincode_example02/go
   ```

4. 实例化链码

   ```shell
   peer chaincode instantiate -o orderer.example.com:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n mycc -v 1.0 -c '{"Args":["init","a", "100", "b","200"]}' -P "OR ('Org1MSP.member','Org2MSP.member')"
   ```

5. 调用链码

   ```shell
   peer chaincode invoke -o orderer.example.com:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $CHANNEL_NAME  -C $CHANNEL_NAME -n mycc -c '{"Args":["invoke","a","b","10"]}'
   ```

6. 查询链码

   ```shell
   peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
   ```