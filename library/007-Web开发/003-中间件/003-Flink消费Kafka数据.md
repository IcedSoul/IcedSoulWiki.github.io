# Flink消费Kafka数据

## 前提

先按照前面内容使用docker-compose快速安装kafka和flink，在kafka的安装记录中写了使用spring boot向kafka推送消息并且消费消息的相关内容，这一篇记录的是使用flink消费kafka的方法。

## 新建Job

首先，我们需要根据官方文档[1]的说明来新建flink的job，用以消费kafka的数据。

### 新建Maven项目

首先需要新建Maven项目，并且添加相关依赖。

有两点需要注意：

1. flink-connector-kafka的版本请根据自己的kafka和flink的版本做调整，具体参考官方文档。且官方文档没有说需要自己额外加`flink-streaming-java`的依赖，但是事实上如果不加就会缺少jar包，需要注意。
2. Maven项目打包默认是不会把依赖包打进去的，但是因为我们的jar包是需要运行的，所以在build里面需要指定主类和打包方式，这样打包时会额外打一个包含所有依赖的jar包。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.icedsoul</groupId>
    <artifactId>kafka-consumer</artifactId>
    <version>1.0-SNAPSHOT</version>

    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka_2.12</artifactId>
            <version>1.9.1</version>
        </dependency>
        <!--如果不加会报错-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_2.12</artifactId>
            <version>1.9.1</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.10.1</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.10</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.0.0</version>
                <configuration>
                    <archive>
                        <!--指定入口类-->
                        <manifest>
                            <mainClass>cn.icedsoul.kafka.consumer.KafkaConnector</mainClass>
                        </manifest>
                    </archive>
                    <!--指定打包方式-->
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### 实现Consumer

然后需要新建主类，连接kafka并且实现Consumer：

```java
    public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        log.info("Job start!");
        env.enableCheckpointing(5000);
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

        Properties props = new Properties();
        props.setProperty("zookeeper.connect", "zookeeper:2181");
        props.setProperty("bootstrap.servers", "kafka:9092");
        props.setProperty("group.id", "flink");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("auto.offset.reset", "latest");

        //topic name需要和producer保持一致。
        FlinkKafkaConsumer<String> consumer = new FlinkKafkaConsumer<String>("spans", new SimpleStringSchema(), props);

        DataStream<String> stream = env.addSource(consumer);
        //将获得的字符串解析为Object，此处需要替换为自己的Object，或者不做处理。JsonUtil是我自定义的一个工具类，用于将字符串解析为Object，可以替换为fastjson等相关方法。
        SingleOutputStreamOperator<Span> spans = stream.map(string -> JsonUtil.toEntity(string, Span.class));
		
        //自定义sink，对数据进行处理，如果使用别的处理方法如定义windows可以修改此处。
        spans.addSink(new TraceCache());
		//特别注意，需要加上这句，否则job会执行失败。
        env.execute("Kafka consumer");
    }
```

其中需要特别注意的一点是需要加上最后一句的execute，如果不加job就会报错，官方文档和不少博客都没有提到这一点，我在这里卡了很久。

### 自定义sink

自定义sink其实不麻烦，继承`RichSinkFunction`的几个方法即可：

```java
public class SpanSink extends RichSinkFunction<Span> {
    
    // 开始处理时回调方法
    @Override
    public void open(Configuration parameters) throws Exception {
        super.open(parameters);
    }

    // 结束处理时回调方法
    @Override
    public void close() throws Exception {
        super.close();
    }

    // 处理每一个消息的回调消息
    @Override
    public void invoke(Span span, Context context) throws Exception {

    }
}

```

### 上传jar包

注意，上述代码不完整，需要继续根据自己的需求修改或者完善。

代码完成后，执行`mvn clean package -DskipTests`打包，然后访问`http://localhost:8081`即可找到`Submit new job`的选项，在其中选择**包含依赖的**jar包然后上传即可。



## 参考

1. Apache Kafka Connector：https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/connectors/kafka.html