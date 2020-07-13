# Docker安装

docker作为目前容器领域的绝对王者，已经成为我们本地和服务器必备的工具，以下记录了Docker在各种系统上的安装方式。

## Centos 安装Docker

在Centos上安装Docker及其方便，几行命令即可：

```shell
yum install -y yum-utils \
device-mapper-persistent-data \
lvm2
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install -y docker-ce-18.09.7 docker-ce-cli-18.09.7 containerd.io
```

运行以上命令即可安装`docker18.09.7`

