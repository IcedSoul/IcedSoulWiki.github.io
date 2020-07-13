# MySQL安装

虽然安装MySQL是一件非常简单的事，但是每次换了新的电脑或者新的服务器基本都要重新装，所以留下记录以便于下次安装时节约时间还是有必要的。

安装MySQL一共有两种方式，一种是在本机直接安装，另外一种是使用docker安装，一般电脑和服务器都会安装docker，所以这里就使用docker来安装MySQL。

## Docker

首先可以参考docker安装mysql

```shell
docker run --name mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.6
```

这个命令可以直接运行一个MySQL实例，但是定义的参数比较少，当实例挂掉的时候数据也会丢失，那么可以增加一些参数如定义映射文件夹之类的。但是因为参数太多时命令过长，所以我们通常使用docker-compose来创建相对完整的mysql实例。

## Docker Compose

```yaml
version: '3'
services:
  mysql:
    image: mysql:5.6
    ports:
    - 3307:3306
    command: [
      --character-set-server=utf8mb4,
      --collation-server=utf8mb4_unicode_ci
    ]
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: shopping
    volumes:
    - ./data:/var/lib/mysql
```

## Kubernetes

有时我们会想在k8s集群上创建MySQL，因为mysql需要用到存储资源，所以我们需要在集群上面创建pv和pvc，提供存储资源。然后部署deployment和service，创建实例并且对外提供服务。

pv&pvc:

mysql-pv.yaml

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv-volume
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 20Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
```

deployment&service:

mysql-deployment.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
  selector:
    app: mysql
  type: LoadBalancer
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
          # Use secret in real usage
        - name: MYSQL_ROOT_PASSWORD
          value: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```

创建了两个文件之后，可以直接使用kubectl工具部署：

```shell
kubectl create -f ./
```

然后可以看到对应的`PV,PVC,Pod,Service`都创建成功，但是中间可能会遇到一些问题，需要注意以下点：

1. 需要创建StorageClass，能够自动分配存储资源，如果集群没有配置需要先创建。否则`PV`和`PVC`会一直`Pending`.
2. 需要对外暴露端口，否则外部无法访问MySQL，对外提供接口有很多种方式。我这里采用LoadBalance的方式。



然后使用生成的LoadBalancer的IP和端口访问MySQL即可。



# 参考内容

1. Docker Hub: https://hub.docker.com/_/mysql
2. Run a single stateful application：https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/
3. 从外部集群访问Pod：https://zhaohuabing.com/2017/11/28/access-application-from-outside/