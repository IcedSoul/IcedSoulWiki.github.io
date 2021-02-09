# Spring AOP运行
## 前言
因为一个需求，我需要简单修改spring-aop组件的源码然后重新打包并使用。当然，这样做并不是好也不推荐，但是我并不知道怎么通过第三方配置的方式来自定义Spring AOP默认的代理策略的选择，目前只能采用这种方法。如果以后找到了别的更好的方法，再回头修改这里。

## 下载源码
spring-framework源码在Github可以下载：
https://github.com/spring-projects/spring-framework

## 编译打包spring aop模块
源码包含多个版本，我想要使用的版本是4.1.9，因此我切换到4.1.x的分支。
