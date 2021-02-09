# 在Spring 启动时执行某方法

## @PostConstruct
根据查到的方法，有种比较简单的方法，类添加@Component后方法前添加@PostContruct注解即可。此方法会在完成bean初始化后立即执行。

```java

import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

@Component
public class StartUp {

    @PostConstruct
    public void text(){
        System.out.println("我跑了！！！");
    }

}

```
运行后的结果为：

![执行结果](http://img.icedsoul/img/blog/startUpWithSpring/postconstruct.png)

在项目内部能够生效，但是写在其它项目中，然后在依赖里面引入时就不生效了。

## 实现 InitializingBean
实现InitializingBean，然后在默认方法中执行即可。
```java
import org.springframework.beans.factory.InitializingBean;
import org.springframework.stereotype.Component;

@Component
public class StartUp2 implements InitializingBean{


    public void afterPropertiesSet() throws Exception {
        System.out.println("我跑了2！-------------------");
    }
}

```

## 参考内容
1. Running Setup Data on Startup in Spring： https://www.baeldung.com/running-setup-logic-on-startup-in-spring
