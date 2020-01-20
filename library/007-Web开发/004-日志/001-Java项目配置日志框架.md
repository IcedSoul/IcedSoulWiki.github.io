# Java项目配置日志框架

## 背景

因为项目需要新建了一个Maven项目，使用Java语言，但是当我像以前一样引入`lombok`，直接使用日志注解`@Slf4j`时却出现了一些问题，因为spring项目会自动配置`logback`,但是如果是自己新建的纯Java项目，没有配置日志框架，那么需要自行配置。

## 添加日志框架

首先，在`pom.xml`添加如下依赖：

```xml
		<dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.11.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-slf4j-impl</artifactId>
            <version>2.11.1</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.10</version>
            <scope>provided</scope>
        </dependency>
```

然后再resource目录下新建`log4j2.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="OFF">
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="[%-level]%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %logger{36} - %msg%n"/>
        </Console>
        <RollingFile name="RollingFileInfo" fileName="/log/flink.log" filePattern="/log/%d{yyyy-MM-dd}/guitool_%d{yyyy-MM-dd}_%i.log">
            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout pattern="[%d{yyyy-MM-dd HH:mm:ss:SSS}] [%p] - %l - %m%n"/>
            <Policies>
                <TimeBasedTriggeringPolicy interval="1"/>
                <SizeBasedTriggeringPolicy size="20MB"/>
            </Policies>
            <DefaultRolloverStrategy max="20"/>
        </RollingFile>
    </Appenders>

    <Loggers>
        <!-- 包名称 -->
        <Logger name="cn.icedsoul.kafka.consumer" level="debug" additivity="true"/>
        <Root level="info">
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>
```

最后，因为导入了`Lombok`，那么直接在类名上面使用`@Slf4j`注解，即可使用日志。

## 参考

1. https://www.cnblogs.com/zeng1994/p/fe096e4423deef6b57fdf735b9df5e7f.html