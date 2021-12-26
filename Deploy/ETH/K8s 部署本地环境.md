

### 环境预备

#### 安装 docker

https://docs.docker.com/engine/install/

#### 安装Minikube

https://minikube.sigs.k8s.io/docs/start/

安装 conntrack

```
 sudo apt-get install -y conntrack
```

启动

```
 sudo -E minikube start --driver=none
```



#### 安装kubectl

https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

#### 安装Helm

helm 介绍：https://helm.sh/docs/

```shell
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

或者

```shell
$ wget https://get.helm.sh/helm-v3.5.4-linux-amd64.tar.gz && tar -zxvf helm-v3.5.4-linux-amd64.tar.gz && mv linux-amd64/helm /usr/local/bin/helm
```

安装完成之后需要将 给helm 添加repo

```shell
 $ helm repo add stable https://charts.helm.sh/stable
```



### 部署ETH 

![Image architecture 3](https://devblogs.microsoft.com/cse/wp-content/uploads/sites/55/2018/02/architecture-3.png)

部署私有的以太坊区块链网络。以太坊网络包含以下成员：

- *bootnode* 以太坊网络的节点发现，类似注册中心
- *ethstats* 以太坊的监控
- *geth-miner* 挖矿节点
- *geth-tx* 禁用了挖矿功能，主要提供交易的查询



#### Install Ethereum

```shell
$ helm install stable/ethereum --generate-name
```



```
WARNING: This chart is deprecated
NAME: ethereum-1619094113
LAST DEPLOYED: Thu Apr 22 12:21:56 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
##############################################################################
####           ERROR: Geth Account has not been configured!               ####
##############################################################################

This deployment will be incomplete until a Geth account is configured. 
See https://github.com/ethereum/go-ethereum/wiki/Managing-your-accounts for 
instructions on how to create a Geth account.

Once created, run:

    helm upgrade ethereum-1619145689 \
        --set geth.account.address=YOUR-ETHEREUM-ADDRESS-HERE
        --set geth.account.privateKey=YOUR-PRIVATE-KEY-HERE
        --set geth.account.secret=YOUR-SECRET-HERE
        stable/ethereum
```

以上信息说明节点已经部署成功，但是网络并没有部署成功，需要先创建Ethereum 账户。

执行以下命令可看到部分docker 已经运行成功，但是部分仍然在初始化

```
$ kubectl  get pod
```

```
NAME                                              READY   STATUS                  RESTARTS   AGE
ethereum-1619094113-bootnode-8cf846dcf-82hgm      2/2     Running                 0          5m44s
ethereum-1619094113-ethstats-5fdf4bff55-6nkgd     1/1     Running                 0          5m44s
ethereum-1619094113-geth-miner-597f5b6c87-4fqz5   0/1     Init:CrashLoopBackOff   5          5m44s
ethereum-1619094113-geth-miner-597f5b6c87-dpqft   0/1     Init:CrashLoopBackOff   5          5m44s
ethereum-1619094113-geth-miner-597f5b6c87-p4xwd   0/1     Init:CrashLoopBackOff   5          5m44s
ethereum-1619094113-geth-tx-67ddd4799-gqh4v       0/1     Init:CrashLoopBackOff   5          5m44s
ethereum-1619094113-geth-tx-67ddd4799-rbmmh       0/1     Init:CrashLoopBackOff   5          5m44s
```

#### 创建账户

没有pip 需要先安装

```
apt-get install python-pip
```



```
$ git clone https://github.com/vkobel/ethereum-generate-wallet
$ cd ethereum-generate-wallet
$ ./ethereum-wallet-generator.sh
```

```
Private key: ec1e9ac3cd1a3419d116783ca03b9bdbdb4d3f26c1bcb0e7c53a0db5f9b1a904
Public key:  152c9d29c7426eeccb5c4bd548e268a6a74a97947ad18fe7c9263509bf31b85617b43c52ea64d1f005da2e4a023d8b2fc7749832587dfd84c4ac9e821ab89fec
Address:     0x84b0363f76f360fe1069A4de1954B11C8a081eEC
```

#### Update Ethereum again

使用的 secret 为：eU8WPpb9Kqfkly

```shell
$ helm upgrade ethereum-1619145689 \
        --set geth.account.address=0x84b0363f76f360fe1069A4de1954B11C8a081eEC \
        --set geth.account.privateKey=ec1e9ac3cd1a3419d116783ca03b9bdbdb4d3f26c1bcb0e7c53a0db5f9b1a904 \
        --set geth.account.secret=eU8WPpb9Kqfkly \
        stable/ethereum
```

```
WARNING: This chart is deprecated
Release "ethereum-1619094113" has been upgraded. Happy Helming!
NAME: ethereum-1619094113
LAST DEPLOYED: Thu Apr 22 12:40:33 2021
NAMESPACE: default
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
1. View the EthStats dashboard at:
    export SERVICE_IP=$(kubectl get svc --namespace default ethereum-1619094113-ethstats -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo http://$SERVICE_IP

    NOTE: It may take a few minutes for the LoadBalancer IP to be available.
          You can watch the status of by running 'kubectl get svc -w ethereum-1619094113-ethstats-service'

  2. Connect to Geth transaction nodes (through RPC or WS) at the following IP:
    export POD_NAME=$(kubectl get pods --namespace default -l "app=ethereum,release=ethereum-1619094113,component=geth-tx" -o jsonpath="{.items[0].metadata.name}")
    kubectl port-forward $POD_NAME 8545:8545 8546:8546
```

执行以下命令可看到docker 全部运行起来了

```shell
$ kubectl  get pod
NAME                                              READY   STATUS    RESTARTS   AGE
ethereum-1619094113-bootnode-8cf846dcf-82hgm      2/2     Running   0          22m
ethereum-1619094113-ethstats-5fdf4bff55-6nkgd     1/1     Running   0          22m
ethereum-1619094113-geth-miner-597f5b6c87-2wmpt   1/1     Running   0          113s
ethereum-1619094113-geth-miner-597f5b6c87-dpqft   1/1     Running   0          22m
ethereum-1619094113-geth-miner-597f5b6c87-p4xwd   1/1     Running   0          22m
ethereum-1619094113-geth-tx-67ddd4799-gqh4v       1/1     Running   0          22m
ethereum-1619094113-geth-tx-67ddd4799-rbmmh       1/1     Running   0          22m
```

```shell
$ kubectl get svc
NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
ethereum-1619145689-bootnode   ClusterIP      None             <none>        30301/UDP,80/TCP    10m
ethereum-1619145689-ethstats   LoadBalancer   10.98.211.234    <pending>     80:32382/TCP        10m
ethereum-1619145689-geth-tx    ClusterIP      10.111.127.106   <none>        8545/TCP,8546/TCP   10m
kubernetes                     ClusterIP      10.96.0.1        <none>        443/TCP             15m
```



打开另外一个命令行窗口执行

```shell
$ minikube tunnel
```

```shell
$ kubectl get svc
NAME                           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
ethereum-1619145689-bootnode   ClusterIP      None             <none>        30301/UDP,80/TCP    10m
ethereum-1619145689-ethstats   LoadBalancer   10.98.211.234    127.0.0.1     80:32382/TCP        10m
ethereum-1619145689-geth-tx    ClusterIP      10.111.127.106   <none>        8545/TCP,8546/TCP   10m
kubernetes                     ClusterIP      10.96.0.1        <none>        443/TCP             15m
```

到这，整个Ethereum 网络就部署好了。

浏览器打开：https://127.0.0.1 可以看到监控工面板

