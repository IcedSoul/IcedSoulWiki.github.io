# Docker常用命令

## 基本命令
```shell
# 查看Docker版本
docker --version

# 查看容器日志
docker logs [container id]

# 实时查看容器日志
docker logs -f -t [container id]

```

## 容器与宿主机之间文件复制

```shell
# 把文件从容器中复制到宿主机中
docker cp [container id]:/mnt/test.txt /root/test.txt
```

