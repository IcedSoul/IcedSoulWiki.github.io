# Spring boot Neo4j

## 背景

我经常使用Spring boot & neo4j用于处理和存储数据，这个文档会记录一些使用Spring data neo4j的一些技巧。

## 建立索引

neo4j中有时也需要建立联合索引，使用如下注解来建立联合索引。

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@NodeEntity
@CompositeIndex(value = {"a", "b"}, unique = true)
public class Api {
    @Id
    @GeneratedValue
    private Long id;

    private String a;
    private String b;
}
```



## 参考内容

1. https://docs.spring.io/spring-data/neo4j/docs/current/reference/html/#reference:indexing:creation