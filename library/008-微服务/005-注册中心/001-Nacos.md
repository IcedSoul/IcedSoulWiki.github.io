# Nacos

啊，真的好难用。



捣鼓了几天，终于把Spring Cloud Alibaba + Dubbo + Nacos这一套给跑了起来。

但是今天测试完成之后发现一个非常奇怪的情况：

一旦我把注册中心的地址改成非localhost，就会报错，出现的问题事在localhost请求不到接口：

```
failed to request http://localhost:8848/nacos/v1/ns/instance?
```

真的有毒，地址已经全都改成非localhost的了，而且注册中心也显示了该服务，但是就是报了这个问题，查看一下源码吧。



好吧，还没看源码找到了原因：

三个配置项，缺一不可：

```yaml
dubbo.registry.address=nacos://47.93.45.54:8848
spring.cloud.nacos.discovery.server-addr=47.93.45.54:8848
spring.cloud.nacos.config.server-addr=47.93.45.54:8848
```

