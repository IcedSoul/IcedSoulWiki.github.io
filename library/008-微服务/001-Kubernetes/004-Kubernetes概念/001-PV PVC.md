# PV & PVC
## 定义
PV（Persistent Volume）是外部存储系统的一块儿存储空间，代表着“网络存储”资源。
- PV只能是网络存储，不属于任何Node，但可以在每个Node访问
- PV不是定义在Pod上的，而是独立于Pod之外定义。
- PV目前只有几种类型： GCE Persistent Disks、NFS、RBD、iSCSCI、AWS ElasticBlockStore、GlusterFS等。

PVC(Persistent Volume Claim)是对PV的申请，PVC通常由普通用户创建和维护。用户在PVC中指明存储空间的大小和访问模式，Kubernetes会查找并提供满足条件的PV。

此处使用NFS作为PV。

## 搭建NFS服务器
简单起见，我准备在mater节点搭建NFS服务器。
```
# 安装nfs 和rpcbind
yum -y install nfs-utils rpcbind

# 检查是否成功安装
rpcinfo -p localhost

# 为nfs服务添加用户，创建共享目录，设置用户访问权限
useradd -u nfs
mkdir -p /nfs-share
chmod a+w /nfs-share

# 配置共享目录
echo "/nfs-share *(rw,async,no_root_squash)" >> /etc/exports

# 使配置生效
exportfs -r

# 启动rpcbind服务
systemctl start rpcbind

# 启动nfs服务器
systemctl start nfs-server

# 设置开机启动
systemctl enable rpcbind
systemctl enable nfs-server

# 检查是否正常启动
showmount -e localhost
```

## 创建PV
接下来新建一个pv：
```
vi nfs-pv.yaml

apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    # 设置容量
    storage: 5Gi
  volumeMode: Filesystem
  # 访问模式此处是允许多Node挂载及读写，有其它选项
  accessModes:
    - ReadWriteMany
  # 指定PV回收策略，recycle表示清空pv数据
  persistentVolumeReclaimPolicy: Recycle
  # 注意，此处可以不设置，那么PVC也可以不设置，但是此处设置PVC必须也设置，否则无法访问资源
  storageClassName: nfs
  mountOptions:
    - hard
    - nfsvers=4.1
  # 存储目录以及NFS服务器地址
  nfs:
    # 需要在共享文件夹新建对应的文件夹
    path: /nfs/share/mysql/data
    server: 10.141.212.138
```

## 设置pvc
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  creationTimestamp: null
  labels:
    io.kompose.service: group-message-mysql-claim0
  name: group-message-mysql-claim0
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 100Mi
  storageClassName: nfs
status: {}

```

## 创建动态存储

使用helm安装：

```shell
helm install stable/nfs-client-provisioner --set nfs.server=10.141.211.176 --set nfs.path=/nfs-share
```

不使用helm安装方式请参考[4]。

然后设置该StorageClass为默认的：

```shell
kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```



## 参考资料

1. 《Kubernetes权威指南》 第二版 龚正，吴治辉等 电子工业出版社
2. 《每天五分钟 玩转Kubernetes》 CloudMan 清华大学出版社
3. Kubernetes-基于NFS的存储：https://www.kubernetes.org.cn/4022.html
4. Kubernetes 集群使用 NFS 网络文件存储:https://blog.csdn.net/aixiaoyang168/article/details/83988253
