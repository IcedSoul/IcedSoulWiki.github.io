# Spring AOP踩坑记录

## aop代理controller导致报错

今天使用kieker监控某项目时遇到一个问题，aop代理使controller方法重复注入导致报错。

但是之前的代理明明都不会出现这种问题的，这里这个问题很奇怪，先尝试着解决一下吧。