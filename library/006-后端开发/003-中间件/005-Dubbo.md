# Dubbo 问题记录

## 版本异常导致Dubbo启动失败

再使用Dubbo的时候发现一个神奇的问题。

### 问题背景

目前我有三个服务：front-service，cart-service和product-service，他们都使用Spring boot开发，并且选用dubbo作为通信协议，选用nacos作为服务注册、发现中心。

### 问题表现

当我开发完cart-service和product-service的几个接口以后，也稍微改动了front-service的一些代码，front-service本来可以正常启动并且正常调用剩下两个服务的，但是在我重新启动之后，发现front-service已经无法调用cart-service了。

经过测试，旧版未经改动的front-service可以成功调用新版的cart-service的接口，但是新版的front-service不能。

我仔细对比了新旧版本的front-service的不同，发现依赖完全相同，我只改动了一个类里面的一个方法，完全不应该导致这种结果的出现。

### 问题原因

经过仔细对比之后，我发现改动之后的front-service启动的时候虽然注册到了nacos，但是并没有启动dubbo，对应配置文件中的dubbo.registry.address也显示没有这项配置，然而实际代码中dubbo的依赖正常。

我初步判断原因是我在cart-service和product-service中使用了其它版本的dubbo，导致Intellij IDEA错误识别依赖，引起了这种问题的出现，但是我更新了front-servie的dubbo版本，与cart-service和product-serrvice中的保持一致，这个问题仍然存在。

经确认，导致问题的原因是与front-service同级的其它service使用了其它版本的dubbo，front-service的dubbo服务无法正常启动导致最终调用出现空指针异常。



### 解决方案

最终解决问题的方法是，让这个项目也使用与其它相同相同的dubbo版本和依赖引入方式，即都使用spring-cloud-starter-dubbo即可。



## Conuser先于Provider启动，报错

dubbo默认会在consumer初始化的时候生成reference的bean，如果provider没有启动，会产生错误。

只需要在配置中加上下面一项即可解决此问题：

```
dubbo.consumer.check=false
```



