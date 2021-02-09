# Flink

## 简介

> Apache Flink is a framework and distributed processing engine for stateful computations over *unbounded and bounded* data streams. Flink has been designed to run in *all common cluster environments*, perform computations at *in-memory speed* and at *any scale*.[1]

简而言之，是处理实时数据的一个工具，今天简单尝试一下。

## 安装

目前先选择一种非常简单的安装方式[2]：

新建一个yml文件`docker-compose-flink.yml`：

```yml
version: "2.1"
services:

  jobmanager:
    image: ${FLINK_DOCKER_IMAGE_NAME:-flink}
    expose:
      - "6123"
    ports:
      - "8081:8081"
    command: jobmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
    networks:
      - data-handle

  taskmanager:
    image: ${FLINK_DOCKER_IMAGE_NAME:-flink}
    expose:
      - "6121"
      - "6122"
    depends_on:
      - jobmanager
    command: taskmanager
    links:
      - "jobmanager:jobmanager"
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager
    networks:
      - data-handle

networks:
  data-handle:
    driver: bridge

```

然后直接执行：

```shell
docker-compose -f docker-compose-flink.yml up -d
```

然后访问`http://localhost:8081`即可看到flink的界面。



## 参考资料

1. flink官网: https://flink.apache.org/
2. flink dockerhub page: https://hub.docker.com/_/flink