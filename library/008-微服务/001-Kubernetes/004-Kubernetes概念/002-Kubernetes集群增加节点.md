# Kubernetes集群增加节点

## Kubeadm方式添加节点方法

### 默认添加方式

在使用kubeadm init初始化整个集群之后，会生成一个join命令，包含了token和一串hash值，在node上面执行该命令即可成功加入集群。

### 后续增加节点

kebeadm init生成的命令在24小时后会过期，后面再想加入新的节点就需要重新生成token和hash值

先执行如下命令获取token：

```
kubeadm token create
```

在执行以下命令来获取ca证书sha256的hash值：

```shell
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

然后把获取到的token和hash值替换下面命令的对应位置，在想要加入集群的节点上面执行该命令：

```shell
kubeadm join --token [token] --discovery-token-ca-cert-hash sha256:[hash] [ip]:6443 --skip-preflight-checks
```



```
kubeadm join --token 3k7pon.vywlvwfnjqgfy011 --discovery-token-ca-cert-hash sha256:249add2dcdc786ce51db6bcf9a01446a34e950b108a9a074a50b4f7fe9bad15c 10.141.211.164:6443
```

