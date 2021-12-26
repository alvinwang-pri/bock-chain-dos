- [ ] ![image-20210429162817239](./img/image-20210429162817239.png)

![image-20210429165156880](./img/image-20210429165156880.png)

### 安装Go

```shell
$ wget https://dl.google.com/go/go1.16.3.linux-amd64.tar.gz && tar -C /usr/local -xzf go1.16.3.linux-amd64.tar.gz && export PATH=$PATH:/usr/local/go/bi\n && export GOPATH=/root/go && source ~/.basr_profile
```



### 安装Fabric 

#### 安装fabric 可执行文件

```shell
$ mkdir fabric-k8s && wget https://github.com/hyperledger/fabric/releases/download/v2.3.1/hyperledger-fabric-linux-amd64-2.3.1.tar.gz && tar -C fabric-k8s -xzf hyperledger-fabric-linux-amd64-2.3.1.tar.gz && cp -r fabric-k8s/bin $GOPATH
```

检查

```shell
root@debian:~# configtxgen --version
configtxgen:
 Version: 2.3.1
 Commit SHA: 2f69b4222
 Go version: go1.14.12
 OS/Arch: linux/amd64
```

执行配置文件脚本

```shell
$ git clone https://github.com/alvinwangs/fabric-k8s.git && cd fabric-k8s && ./fabricOps.sh start
```

```
##########################################################
##### Generate certificates using cryptogen tool #########
##########################################################
org1
org2
#################################################################
### Generating channel configuration transaction 'channel.tx' ###
#################################################################
2021-04-26 03:12:41.257 UTC [common.tools.configtxgen] main -> INFO 001 Loading configuration
2021-04-26 03:12:41.276 UTC [common.tools.configtxgen.localconfig] completeInitialization -> INFO 002 orderer type: etcdraft
2021-04-26 03:12:41.276 UTC [common.tools.configtxgen.localconfig] Load -> INFO 003 Loaded configuration: configtx.yaml
2021-04-26 03:12:41.278 UTC [common.tools.configtxgen] doOutputBlock -> INFO 004 Generating genesis block
2021-04-26 03:12:41.278 UTC [common.tools.configtxgen] doOutputBlock -> INFO 005 Creating system channel genesis block
2021-04-26 03:12:41.278 UTC [common.tools.configtxgen] doOutputBlock -> INFO 006 Writing genesis block
2021-04-26 03:12:41.311 UTC [common.tools.configtxgen] main -> INFO 001 Loading configuration
2021-04-26 03:12:41.329 UTC [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: configtx.yaml
2021-04-26 03:12:41.330 UTC [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 003 Generating new channel configtx
2021-04-26 03:12:41.332 UTC [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 004 Writing new channel tx

#################################################################
#######    Generating anchor peer update for Org1MSP   ##########
#################################################################
2021-04-26 03:12:41.366 UTC [common.tools.configtxgen] main -> INFO 001 Loading configuration
2021-04-26 03:12:41.385 UTC [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: configtx.yaml
2021-04-26 03:12:41.385 UTC [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Generating anchor peer update
2021-04-26 03:12:41.387 UTC [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 004 Writing anchor peer update

#################################################################
#######    Generating anchor peer update for Org2MSP   ##########
#################################################################
2021-04-26 03:12:41.419 UTC [common.tools.configtxgen] main -> INFO 001 Loading configuration
2021-04-26 03:12:41.438 UTC [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: configtx.yaml
2021-04-26 03:12:41.438 UTC [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Generating anchor peer update
2021-04-26 03:12:41.439 UTC [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 004 Writing anchor peer update

```

#### 启动minikube

注意，minikube 集群必须要 mount 新的路径，不然pod 会挂载失败

```shell
$ minikube start --driver=docker --mount-string /home/yourname/fabric-k8s:/host --mount
```

#### 部署 fabric 网络

创建 namespace

```shell
$ kubectl get pods -n hyperledger
```

apply Deployment & service

```shell
$ kubectl create -f orderer-service/
$ kubectl create -f org1/
$ kubectl create -f org2/
```

查看 service

```shell
$ kubectl get svc -n hyperledger
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
ca-org1              ClusterIP   10.108.46.203    <none>        7054/TCP            82m
ca-org2              ClusterIP   10.100.124.82    <none>        7054/TCP            82m
orderer0             ClusterIP   10.105.236.236   <none>        7050/TCP,7053/TCP   82m
orderer0-metrics     ClusterIP   10.103.101.50    <none>        8443/TCP            82m
orderer1             ClusterIP   10.108.78.117    <none>        7050/TCP,7053/TCP   82m
orderer1-metrics     ClusterIP   10.102.82.242    <none>        8443/TCP            82m
orderer2             ClusterIP   10.105.110.157   <none>        7050/TCP,7053/TCP   82m
orderer2-metrics     ClusterIP   10.109.188.145   <none>        8443/TCP            82m
peer0-org1           ClusterIP   10.101.104.216   <none>        7051/TCP            82m
peer0-org1-metrics   ClusterIP   10.103.197.63    <none>        9443/TCP            82m
peer0-org2           ClusterIP   10.107.20.37     <none>        7051/TCP            82m
peer0-org2-metrics   ClusterIP   10.102.59.178    <none>        9443/TCP            82m
```

查看 pod

```shell
$ kubectl get pod -n hyperledger
NAME                          READY   STATUS    RESTARTS   AGE
ca-org1-7c5c5c44c6-mqrlq      1/1     Running   0          82m
ca-org2-7c9d84b44-wr6np       1/1     Running   0          82m
cli-org1-84ddf74d55-8c5jr     1/1     Running   0          82m
cli-org2-74bf968b57-cqbbx     1/1     Running   0          82m
orderer0-9bd9b8b8b-9456d      1/1     Running   0          82m
orderer1-7b6d476b77-wp7f5     1/1     Running   0          62m
orderer2-5d45c8bbf6-mdwcr     1/1     Running   0          82m
peer0-org1-7fdf98c7dc-lsjxf   1/1     Running   0          82m
peer0-org2-546d947944-v8q9n   1/1     Running   0          82m
```

Fabric 测试网络已经部署好。

### Channel 操作

#### 创建 channel

进入org1-cli 节点执行创建 channel 的命令

```shell
$kubectl  exec -n hyperledger -it cli-org1_pod_name  /bin/sh
```

```shell
# peer channel create -o orderer0:7050 -c mychannel -f ./scripts/channel-artifacts/channel.tx --tls true --cafile $ORDERER_CA
2021-04-26 09:32:22.794 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
......
2021-04-26 09:32:23.845 UTC [channelCmd] InitCmdFactory -> INFO 00d Endorser and orderer connections initialized
2021-04-26 09:32:24.048 UTC [cli.common] readBlock -> INFO 00e Received block: 0
```

#### 加入channel

在org1 的cli 节点下执行

```shell
# peer channel join -b mychannel.block
2021-04-26 09:35:05.784 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2021-04-26 09:35:05.867 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
```

查看channel 列表

```
# peer channel list
2021-04-26 09:35:09.467 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
Channels peers has joined: 
mychannel
```

切换到org2 的cli 节点将org2 加入到 channel 中

```shell
$ kubectl exec -n hyperledger -it cli-org2-pod_name /bin/sh
```

查看当前block 

```shell
# peer channel fetch 0 mychannel.block -c mychannel -o orderer0:7050 --tls --cafile $ORDERER_CA
2021-04-26 09:42:00.071 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2021-04-26 09:42:00.073 UTC [cli.common] readBlock -> INFO 002 Received block: 0
```

先看一下当前加入的channel

```shell
# peer channel list
2021-04-26 09:40:40.973 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
Channels peers has joined: 
```

还没有加入到channel

将org2 加入到mychannel 中

```sh
# peer channel join -b mychannel.block
2021-04-26 09:45:08.703 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
2021-04-26 09:45:08.768 UTC [channelCmd] executeJoin -> INFO 002 Successfully submitted proposal to join channel
/opt/gopath/src/github.com/hyperledger/fabric/peer # peer channel list
```

```shell
 # peer channel list
 2021-04-26 09:45:12.385 UTC [channelCmd] InitCmdFactory -> INFO 001 Endorser and orderer connections initialized
Channels peers has joined: 
mychannel
```

#### 部署External chaincode

https://hyperledger-fabric.readthedocs.io/en/release-2.0/cc_launcher.html#

1. 打包chaincode

   进入org1 的cli 节点 并打包

   ```
   $ kubectl  exec -n  hyperledger -it cli-org1-84ddf74d55-8c5jr /bin/sh
   $ cd /opt/gopath/src/github.com/marbles/packaging
   $ tar cfz code.tar.gz connection.json
   $ tar cfz marbles-org1.tgz code.tar.gz metadata.json
   ```

安装chaincode



```sh
$ peer lifecycle chaincode install marbles-org1.tgz

2021-04-27 08:18:06.749 UTC [cli.lifecycle.chaincode] submitInstallProposal -> INFO 001 Installed remotely: response:<status:200 payload:"\nHmarbles:5fca1cfd967ad736c799c7f77fbd955728a54b86ad6ce318f2d93cb586175ce1\022\007marbles" > 
2021-04-27 08:18:06.749 UTC [cli.lifecycle.chaincode] submitInstallProposal -> INFO 002 Chaincode code package identifier: marbles:5fca1cfd967ad736c799c7f77fbd955728a54b86ad6ce318f2d93cb586175ce1
```

```shell
$ /opt/gopath/src/github.com/marbles/packaging #  peer lifecycle chaincode queryinstalled
Installed chaincodes on peer:
Package ID: marbles:5fca1cfd967ad736c799c7f77fbd955728a54b86ad6ce318f2d93cb586175ce1, Label: marbles
```

记住这个epackage id，下面调用需要用到

进入 org2 的cli 节点打包并安装 org2 的chaincode

```sh
$ rm -f code.tar.gz
$ tar cfz code.tar.gz connection.json
$ tar cfz marbles-org2.tgz code.tar.gz metadata.json
$ peer lifecycle chaincode install marbles-org2.tgz

2021-04-27 08:45:43.045 UTC [cli.lifecycle.chaincode] submitInstallProposal -> INFO 001 Installed remotely: response:<status:200 payload:"\nHmarbles:fa2be3dfaa1c606ad58d697f5ffd4c18ad33787d156fc4d2a4a771d164c5c2eb\022\007marbles" > 
2021-04-27 08:45:43.045 UTC [cli.lifecycle.chaincode] submitInstallProposal -> INFO 002 Chaincode code package identifier: marbles:fa2be3dfaa1c606ad58d697f5ffd4c18ad33787d156fc4d2a4a771d164c5c2eb
```

退出 org2 cli 节点

正常的流程需要运行 buildpack 目录下的 deteck,build 脚本 并构建 docker images，但是这里已经准备好，只需要修改deployment 文件里的packageId 再部署即可。

./chaincode/k8s 目录下修改deployment 文件

org1-chaincode-deployment.yaml 将上面org1 cli 节点 安装的chaincode 的package id替换掉 CHAINCODE_CCID

```
          env:
            - name: CHAINCODE_CCID
              value: "marbles:5fca1cfd967ad736c799c7f77fbd955728a54b86ad6ce318f2d93cb586175ce1"
            - name: CHAINCODE_ADDRESS
              value: "0.0.0.0:7052"
          ports:
            - containerPort: 7052
```



org2-chaincode-deployment.yaml 将上面org2 cli 节点 安装的chaincode 的package id替换掉 CHAINCODE_CCID

```
          env:
            - name: CHAINCODE_CCID
              value: "marbles:fa2be3dfaa1c606ad58d697f5ffd4c18ad33787d156fc4d2a4a771d164c5c2eb"
            - name: CHAINCODE_ADDRESS
              value: "0.0.0.0:7052"
          ports:
            - containerPort: 7052
```



部署 External chaincode lancher

```shell
$ kubectl create -f chaincode/k8s
$  kubectl get pods -n hyperledger
NAME                                      READY   STATUS    RESTARTS   AGE
ca-org1-7c5c5c44c6-mqrlq                  1/1     Running   0          2d19h
ca-org2-7c9d84b44-wr6np                   1/1     Running   0          2d19h
chaincode-marbles-org1-7dcd58d89-kr7qz    1/1     Running   0          41h
chaincode-marbles-org2-75cbfd8bfd-dp9tr   1/1     Running   0          41h
cli-org1-84ddf74d55-8c5jr                 1/1     Running   0          2d19h
cli-org2-74bf968b57-cqbbx                 1/1     Running   0          2d19h
orderer0-9bd9b8b8b-9456d                  1/1     Running   0          2d19h
orderer1-7b6d476b77-wp7f5                 1/1     Running   0          2d19h
orderer2-5d45c8bbf6-mdwcr                 1/1     Running   0          2d19h
peer0-org1-7fdf98c7dc-lsjxf               1/1     Running   0          2d19h
peer0-org2-546d947944-v8q9n               1/1     Running   0          2d19h
```

#### Chaicode 背书

再次进入org1 cli pod

```shell
$ peer lifecycle chaincode approveformyorg --channelID mychannel --name marbles --version 1.0 --init-required --package-id marbles:5fca1cfd967ad736c799c7f77fbd955728a54b86ad6ce318f2d93cb586175ce1 --sequence 1 -o orderer0:7050 --tls --cafile $ORDERER_CA

2021-04-29 03:40:03.587 UTC [chaincodeCmd] ClientWait -> INFO 001 txid [0a1852667619c49da275b6049fd50757ce28aaf38646f24c31ffb4ca5c8af58e] committed with status (VALID) at peer0-org1:7051
```

可以通过查询chaicode 的状态

```
$ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name marbles --version 1.0 --init-required --sequence 1 -o -orderer0:7050 --tls --cafile $ORDERER_CA

Chaincode definition for chaincode 'marbles', version '1.0', sequence '1' on channel 'mychannel' approval status by org:
org1MSP: true
org2MSP: false
```

退出org1 cli 进入 org2 cli pod

```
$ peer lifecycle chaincode approveformyorg --channelID mychannel --name marbles --version 1.0 --init-required --package-id marbles:fa2be3dfaa1c606ad58d697f5ffd4c18ad33787d156fc4d2a4a771d164c5c2eb --sequence 1 -o orderer0:7050 --tls --cafile $ORDERER_CA

2021-04-29 07:12:14.065 UTC [chaincodeCmd] ClientWait -> INFO 001 txid [259692aca24810644d1d86ec718ed81256ab81a52fe83302648f30007ecb0fb5] committed with status (VALID) at peer0-org2:7051
```

再次检查chaincode 状态，可以看到两个org 的msp 都为true 了

```
$ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name marbles --version 1.0 --init-required --sequence 1 -o -orderer0:7050 --tls --cafile $ORDERER_CA

Chaincode definition for chaincode 'marbles', version '1.0', sequence '1' on channel 'mychannel' approval status by org:
org1MSP: true
org2MSP: true
```

现在 chaicode 都得到 org1 和 org2 的背书，这个时候chaicode已经具备提交到区块链网络的条件。已经得到了证明，那么任由任何一个org 都可以提交。

在org2 cli pod 上线提交：

```shell
$ peer lifecycle chaincode commit -o orderer0:7050 --channelID mychannel --name marbles --version 1.0 --sequence 1 --init-required --tls true --cafile $ORDERER_CA --peerAddresses peer0-org1:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1/peers/peer0-org1/tls/ca.crt --peerAddresses peer0-org2:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2/peers/peer0-org2/tls/ca.crt


2021-04-29 07:15:30.503 UTC [chaincodeCmd] ClientWait -> INFO 001 txid [8486b0d766f7f695de92e3309a33cc4e06aa480e9d707382144aab5da1228574] committed with status (VALID) at peer0-org1:7051
2021-04-29 07:15:30.503 UTC [chaincodeCmd] ClientWait -> INFO 002 txid [8486b0d766f7f695de92e3309a33cc4e06aa480e9d707382144aab5da1228574] committed with status (VALID) at peer0-org2:7051
```



#### 测试Chaincode

完成上面的步骤，说明现在已经完成了chaincode的部署。

创建 marble1

```shell
$ peer chaincode invoke -o orderer0:7050 --isInit  --tls true --cafile $ORDERER_CA -C mychannel -n marbles --peerAddresses peer0-org1:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1/peers/peer0-org1/tls/ca.crt --peerAddresses peer0-org2:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2/peers/peer0-org2/tls/ca.crt -c '{"Args":["initMarble","marble1","blue","35","tom"]}' --waitForEvent

2021-04-29 07:31:15.531 UTC [chaincodeCmd] ClientWait -> INFO 001 txid [dec5f92a9568267bbe4dc8fc638bb77a3d9c37c482c495a6131dd22c58bbafc3] committed with status (VALID) at peer0-org2:7051
2021-04-29 07:31:15.534 UTC [chaincodeCmd] ClientWait -> INFO 002 txid [dec5f92a9568267bbe4dc8fc638bb77a3d9c37c482c495a6131dd22c58bbafc3] committed with status (VALID) at peer0-org1:7051
2021-04-29 07:31:15.534 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Chaincode invoke successful. result: status:200 
```

创建marble2

```shell
$ peer chaincode invoke -o orderer0:7050  --tls true --cafile $ORDERER_CA -C mychannel -n marbles --peerAddresses peer0-org1:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org1/peers/peer0-org1/tls/ca.crt --peerAddresses peer0-org2:7051 --tlsRootCertFiles /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/org2/peers/peer0-org2/tls/ca.crt -c '{"Args":["initMarble","marble2","red","50","tom"]}' --waitForEvent

2021-04-29 07:33:03.418 UTC [chaincodeCmd] ClientWait -> INFO 001 txid [49a50efc265129f6343b1667b107e430594de0218dd6353c4d349b79c626561d] committed with status (VALID) at peer0-org2:7051
2021-04-29 07:33:03.418 UTC [chaincodeCmd] ClientWait -> INFO 002 txid [49a50efc265129f6343b1667b107e430594de0218dd6353c4d349b79c626561d] committed with status (VALID) at peer0-org1:7051
2021-04-29 07:33:03.418 UTC [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 003 Chaincode invoke successful. result: status:200 
```

查询 marble1,查询marble2

```sh
$ peer chaincode query -C mychannel -n marbles -c '{"Args":["readMarble","marble1"]}'
{"docType":"marble","name":"marble1","color":"blue","size":35,"owner":"tom"}
```

```sh
$ peer chaincode query -C mychannel -n marbles -c '{"Args":["readMarble","marble2"]}'
{"docType":"marble","name":"marble2","color":"red","size":50,"owner":"tom"}

```

