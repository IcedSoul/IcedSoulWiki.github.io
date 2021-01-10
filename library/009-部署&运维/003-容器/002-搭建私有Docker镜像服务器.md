# 搭建私有Docker镜像服务器

存储Docker镜像时我们可以选用`DockerHub`，但是`DockerHub`存在以下问题：

1. 私密性。DockerHub上传的镜像其它人是可以访问的，处于保密和需要，有些镜像不适合上传

到DockerHub。

2. 快捷性。DockerHub的上传和下载速度虽然已经不错了，但是有的时候还是不能满足我们的需求。

因此，在内网搭建一个私有的Docker镜像服务器，同时满足我们私密性和快捷性的要求，是非常有必要的。



## 准备工作

准备一台云服务器，并且安装Docker。



## 开始搭建

我们可以使用registry搭建一个非常简单的镜像仓库，但是有的时候我们需要对镜像仓库进行管理，所以在这里我选用了`Harbor`来搭建并且管理私有镜像仓库。

安装`Harbor`也有两种方式，一种是直接安装，另外一种使用`Helm`安装到k8s集群，下面分别使用两种方式进行安装。



### 直接安装Harbor

首先在参考内容[1]中下载harbor的最新release版本，下载速度可能不是很快，需要耐心等待。



### 使用Helm安装Harbor

使用Helm安装Harbor首先需要又一个Kubernetes集群，如果没有需要先行搭建，或者使用上面方式安装。

有了集群之后还需要安装Helm，可参考：微服务->Kunerneres->Helm相关->Helm安装。

参考内容[2]有具体文档。

首先，添加Helm仓库：

```shell
helm repo add harbor https://helm.goharbor.io
```

中间具体的配置可以看[2]的文档。

```shell
helm install my-harbor harbor/harbor
```









## 参考内容

1. Harbor Github：https://github.com/goharbor/harbor
2. Harbor Helm：https://github.com/goharbor/harbor-helm