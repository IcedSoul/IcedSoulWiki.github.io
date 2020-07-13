# StorageClass

## 定义

StorageClass是存储资源的抽象定义，可以由系统自动完成PV的创建和绑定，实现动态的资源供应。

根据官方实例和参考用例，最终决定以GlusterFS提供动态存储来创建StorageClass。

## 安装GlusterFS

首先，在各个节点安装GlusterFS:

```shell
yum -y install glusterfs glusterfs-fuse
```

然后下载GlusterFS的官方安装工具：

```shell
git clone https://github.com/gluster/gluster-kubernetes.git
```

按照上面的说明文档来做。

因为我已经有了k8s集群，所以不需要使用vagrant来进行安装，可以跳过这一步，直接进入第二步。

进入deploy/文件夹，可以看到有一个topology.json.sample的文件，是提供的一个模板，我们需要将其中的一些参数设置为自己集群的IP：

```shell
root@master deploy]# ls
gk-deploy  gk-deploy-minikube  heketi.json.template  kube-templates  ocp-templates  topology.json.sample  topology-minikube.json
[root@master deploy]# vi topology.json.sample
```

可以看到其中内容如下，需要改的地方只有hostname和IP地址，有几个节点就添加几项，多了删少了复制：

```json
{
  "clusters": [
    {
      "nodes": [
        {
          "node": {
            "hostnames": {
              "manage": [
                //此处为hostname
                "master"
              ],
              "storage": [
                //此处为IP地址
                "10.141.211.176"
              ]
            },
            "zone": 1
          },
          "devices": [
            "/dev/vdb",
            "/dev/vdc",
            "/dev/vdd"
          ]
        },
        {
          "node": {
            "hostnames": {
              "manage": [
                "node1"
              ],
              "storage": [
                "10.141.211.164"
              ]
            },
            "zone": 1
          },
          "devices": [
            "/dev/vdb",
            "/dev/vdc",
            "/dev/vdd"
          ]
        },
        {
          "node": {
            "hostnames": {
              "manage": [
                "node2"
              ],
              "storage": [
                "10.141.212.143"
              ]
            },
            "zone": 1
          },
          "devices": [
            "/dev/vdb",
            "/dev/vdc",
            "/dev/vdd"
          ]
        },
        {
          "node": {
            "hostnames": {
              "manage": [
                "node3"
              ],
              "storage": [
                "10.141.212.144"
              ]
            },
            "zone": 1
          },
          "devices": [
            "/dev/vdb",
            "/dev/vdc",
            "/dev/vdd"
          ]
        }
      ]
    }
  ]
}
```

然后执行：

```shell
[root@master deploy]# mv topology.json.sample topology.json
```

继续按照文档说明，执行：

```
./gk-deploy -g --admin-key admin --user-key admin
```







nfs安装：

```shell
helm install stable/nfs-client-provisioner --set nfs.server=10.141.211.176 --set nfs.path=/nfs-share
```

一行搞定

或者参考[4]



##　参考资料

1. Kubernetes StorageClass: https://kubernetes.io/docs/concepts/storage/storage-classes/
2. Kubernetes 权威指南 第四版 龚正 等 著
3. Github-gluster-kubernetes:https://github.com/gluster/gluster-kubernetes
4. :Kubernetes 集群使用 NFS 网络文件存储:https://blog.csdn.net/aixiaoyang168/article/details/83988253
5. https://www.kubernetes.org.cn/4078.html