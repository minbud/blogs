---
title: kube-openmpi
date: 2019-07-03 20:45:57
tags: [k8s, mpi, openmpi]
---
## From
[Author](https://github.com/everpeace/kube-openmpi)

## Structrue
　　通过helm（k8s包管理）将mpijob以mpi-master、mpi-worker方式发布到k8s上以长期运行任务方式运行。  
　　运行mpijob时，通过*kubectl exec*在mpi-master所在的docker上运行*mpiexec*，并指定需要运行的任务数量，从而使得mpi-worker也一起工作。  
　　由于mpijob之间需要通信，主要工作是需要配置ssh。配置过程是生成*id_rsa*,*id_rsa.pub*，并将它们的值导入到相应的配置文件中，同时配置dockerfile，在docker中运行sshd。另外mpi-master所在pod中还需要运行一个hostfile-updater的容器，每隔一段时间更新mpi节点的hostfile。  

<!--more-->

## Helm
[参考](https://zhaohuabing.com/2018/04/16/using-helm-to-deploy-to-kubernetes/)  
　　相当于linux中的包管理工具，可以打包应用，管理包依赖等。  
　　工具结构主要包括helm、tiller，helm是client-cli，tiller是server端，负责与k8s交互。  
　　包主要是*Chart*，就是一个目录，底下有
```
chart/
├── Chart.yaml					pkg总体信息
├── templates					k8s manifest模板文件目录
│   ├── configmap.yaml
│   ├── _helpers.tpl
│   ├── mpi-cluster.yaml
│   ├── network-policy.yaml
│   ├── secrets.yaml
│   └── service.yaml
└── values.yaml					被templates中各个文件使用到的一些变量值
```

## Procedure
### generate ssh key
```
$ ./gen-ssh-key.sh			生成id_rsa, id_rsa.pub, 并将两个的值导入到./ssh-key.yaml中
```
### deploy
```
$ MPI_CLUSTER_NAME=__CHANGE_ME__
$ KUBE_NAMESPACE=__CHANGE_ME_
$ helm template chart --namespace $KUBE_NAMESPACE --name $MPI_CLUSTER_NAME -f values.yaml -f ssh-key.yaml | kubectl -n $KUBE_NAMESPACE create -f -
```
helm template :		Render chart templates locally and display the output  
　　也就是将包*chart*生成k8s的配置文件输出，并以*values.yaml*,*ssh-key.yaml*为值。同时把输出作为*kubectl create -f*的输入，也就是部署应用。  
　　这样mpi-master可能会启动失败，需要新[创建账户](https://github.com/everpeace/kube-openmpi/issues/24)。  
　　需要新建账户的原因是，mpi-master启动过程需要执行*hostfile-initializer*脚本，生成hostfile，生成过程中需要使用*kubectl*来与k8s的*API-SERVER*交互。要实现这个过程需要使用到*Service Account*,因为*API-SERVER*只在HTTPS安全端口443上提供服务，因而pod内的进程需要利用*Service Account*进行身份认证。
  
### run
```
kubectl -n $KUBE_NAMESPACE exec -it $MPI_CLUSTER_NAME-master -- mpiexec --allow-run-as-root \
  --hostfile /kube-openmpi/generated/hostfile \
  --display-map -n 4 -npernode 1 \
  sh -c 'echo $(hostname):hello'
```
　　通过*kubectl exec*连接到mpi-master所在docker执行*mpiexec*命令，从而在master和worker上执行mpijob。  
  
### others
　　在本地生成的*id_rsa*, *id_rsa.pub*，*authorized_keys*在k8s中生成相应的键值对，并以文件的形式挂载在master和worker的docker中,在启动的时候执行*/init.sh*将其拷贝到*/.sshd/user_key/$user*中，并对sshd启动进行相应的配置。
