# Jackson JSON

## Use Jackson to parse Json String[1]

If you have a JSON String and you want to parse it to List of Object, then you can write as follow:

```shell
ObjectMapper mapper = new ObjectMapper();
List<MyObject> objects = mapper.readValue(jsonString, new TypeReference<List<MyObject>>(){});
```

or:

```
List<MyObjects> objects = mapper.readValue(jsonString, mapper.getTypeFactory().constructCollectionType(List.class, MyObject.class));
```

## Timestamp[2]

If your class have fields of type Timestamp ,you will failed when you want to parse json string to object. Then you can add annotations to fields of type Timestamp. Just like:

```java
@JsonFormat(pattern="yyyy-MM-dd HH:mm:ss.SSS")
private Timestamp time;
```

## 注意点

使用jacson把字符串解析对象有以下特点：

1. 对象中的属性数可以少于Json字符串中的属性数。
2. 对象中的小写字母名称可以直接被转化，如果名字含有大写字母应该加上`@JsonProperty("NAME")`注解，否则该字段无法成功转化。

## Reference

1.  https://stackoverflow.com/questions/6349421/how-to-use-jackson-to-deserialise-an-array-of-objects 
2.  https://stackoverflow.com/questions/43373270/jackson-deserialize-json-with-timestamp-field 