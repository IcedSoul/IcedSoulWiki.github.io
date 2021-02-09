# Spring 自定义注解

## 新建注解
注解的由来元注解的使用参考：[Java Annotation](?file=003-编程语言/001-Java/002-Java Annotation/001-Java Annotation.md "Java Annotation")

## 扫描解析注解
因为我的需求是制作第三方jar包，然后在主项目中添加注解，所以就需要有扫描解析注解的方法。目前找到了一下几种可行方法，接下来一一尝试。

### 使用aop
aop可以通过设置aspect，在指定类被执行时，判断这个类是否是拥有指定注解，然后获取对象对其进行修改。  
这个方法实现起来比较简单，能实现的功能也比较多，接下来需要确认的有以下几点：
1. 是否可以获取，修改静态变量。（应该可以）
2. 是否可以封装为第三方jar包使用。（等待验证）
3. 是否会增加调用时间。（等待验证）
4. 是否可以方便地指定扫描范围，使用是否方便。（等待实现）

### 使用reflect

### 注入bean
#### 方法一
BeanDefinitionRegistryPostProcessor  
ClassPathBeanDefinitionScanner
#### 方法二
@ConditionalOnBean
####
### 使用cglib
