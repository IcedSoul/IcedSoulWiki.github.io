# Kafka

## What is Kafka

>  Kafka® is used for building real-time data pipelines and streaming apps. It is horizontally scalable, fault-tolerant, wicked fast, and runs in production in thousands of companies. [1]

It can be used as a Message Queue.

## Install with Docker-compose

You can install Kafka with docker-compose quickly:

```yml
version: "2.1"
services:
  zookeeper:
    image: wurstmeister/zookeeper
    ports:
      - "2181:2181"
    networks:
      - data-handle

  kafka:
    image: wurstmeister/kafka
    depends_on: [ zookeeper ]
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: sdf-3718715.lvs02.dev.ebayc3.com
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://sdf-3718715.lvs02.dev.ebayc3.com:9092
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_CREATE_TOPICS: "spans:1:1:compact"

    networks:
      - data-handle

  kafka-manager:
    image: sheepkiller/kafka-manager
    links:
      - zookeeper
      - kafka
    ports:
      - "9001:9000"
    environment:
      ZK_HOSTS: zookeeper:2181
      APPLICATION_SECRET: letmein
      KM_ARGS: -Djava.net.preferIPv4Stack=true
    networks:
      - data-handle

networks:
  data-handle:
    driver: bridge
```

Just execute `docker-compose up -d` then Kafka with run on your machine.

 ## Work with Spring-boot

Kafka in running now, but how we use it?

The development framework I'm most familiar with is Spring boot, so I will try to use Kafka on Spring Boot.

First, add Kafka library in `pom.xml` file:

```xml
  		<dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
```

Then,configure Kafka through `application.yml`file:

```yml
spring:
  kafka:
    bootstrap-servers: [host]:9092
    producer:
      batch-size: 16
      retries: 0
      buffer-memory: 33554432
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      auto-offset-reset: latest
      enable-auto-commit: true
      auto-commit-interval: 100
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      group-id: test-consumer-group
```

After that we create a producer and a consumer:

```java
@Service
public class Producer{

    @Autowired
    private KafkaTemplate kafkaTemplate;

    @Override
    public void send(String message) {
        kafkaTemplate.send("test", message);
    }
}
```

```java
@Service
@Slf4j
public class Consumer {

    @Autowired
    KafkaTemplate kafkaTemplate;

    @KafkaListener(topics = {"test"}, groupId = "test-consumer-group")
    private void listen(ConsumerRecord<?, ?> record){
        Optional<?> kafkaMessage = Optional.ofNullable(record.value());
        if(kafkaMessage.isPresent()){
            Object message = kafkaMessage.get();
            log.info((String) message);
        }
    }
}
```

Then we call `send`in controller:

```java
producer.send(message);
```

Note that if kafka does not have a listening topic, you will encounter the following error when starting the project:

```verilog
Topic(s) [spans] is/are not present and missingTopicsFatal is true
```

This problem can be solved by simply creating the missing topic in kafka.



## Visualization

Download and install `Kafka Tool 2.0`to connect and check state of kafka.

## Reference

1. Kafka Official Site:https://kafka.apache.org/
2. How to Work with Apache Kafka in Your Spring Boot Application:https://www.confluent.io/blog/apache-kafka-spring-boot-application/