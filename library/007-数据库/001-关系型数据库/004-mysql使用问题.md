# Mysql使用问题

## 问题描述

今天在使用mysql5.6的时候遇到了一个神奇的问题：

```shell
Caused by: java.sql.SQLSyntaxErrorException: Specified key was too long; max key length is 767 bytes
```

这个问题在我本地测试使用测试数据库的时候没有出现，但是在我使用docker-compose启动项目的时候出现了，经过查找资料和尝试之后，我确定了出现问题的原因：数据库引擎对于索引长度的限制。

根据资料1,2,当数据库的default_storage_engine="InnoDB"，不管建表语句使用的是InnoDB还是MYISAM，实际使用的引擎都是InnoDB。

设置字符集为UTF-8时，索引长度为255时可以建表成功，大于255则建表失败。

不设置字符集时，索引长度为191时可以建表成功。



## 解决方法

1. 缩短索引长度。
2. 修改InnoDB索引长度限制。



## 参考资料

1. https://www.cnblogs.com/huchong/p/8758568.html
2. https://cloud.tencent.com/developer/article/1005696