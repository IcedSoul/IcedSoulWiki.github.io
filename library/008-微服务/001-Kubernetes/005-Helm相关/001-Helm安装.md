# Helm安装

## 介绍

Helm是用于管理k8s软件的一个工具，它类似于mvnrepository和dockerhub，但是和它们又有所不同，在使用k8s集群是，使用helm是非常有必要的，这里简单介绍一下如何安装helm。

## 安装Helm3.0

从参考内容[2]可以中可以获取到最新版Helm的安装文件，下载操作系统对应的文件即可完成安装。

如：

```shell
wget https://get.helm.sh/helm-v3.2.1-linux-amd64.tar.gz
tar -zxvf helm-v3.2.1-linux-amd64.tar.gz 
mv ./linux-amd64/helm /usr/local/bin/
helm version
```



## 参考内容

1. Helm官网：https://github.com/helm/helm
2. Helm Github：https://github.com/helm/helm