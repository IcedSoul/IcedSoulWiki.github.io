# CacheBuilder

CacheBuilder是一种类似于Map的数据结构，可以存储键值对，但是它和Map的不同之处在于它可以指定每个键值对的过期时间，并且触发对应的回调函数，在我们处理流式数据的时候非常实用。

## 引入依赖

它包含在google的guava包里面，Spring项目默认有，如果你的项目没有可以在`Pom.xml`文件中添加以下依赖：

```xml
		<dependency>
            <groupId>com.google.guava</groupId>
            <artifactId>guava</artifactId>
            <version>23.0</version>
        </dependency>
```



## 测试

因为这个对象需要驻留在内存才能够触发回调，所以我在Spring Boot的controller中进行了测试，主要代码如下：

```java
private Cache<String, List<Span>> cache = null;


@ApiOperation(value = "Test", notes = "test")
@PostMapping(value = "/test")
public void test(){
    if(this.cache == null){
        this.cache = CacheBuilder.newBuilder().initialCapacity(4000 * 300).maximumSize(4000 * 300)
                .expireAfterAccess(10, TimeUnit.SECONDS)
                .expireAfterWrite(10, TimeUnit.SECONDS)
                .removalListener(new RemovalListener<String, List<Span>>() {
                    @Override
                    public void onRemoval(RemovalNotification<String, List<Span>> notification) {
                        log.info("Remove :" + notification.getKey());
                    }
                })
                .build();
        log.info("Build cache" + cache.stats());
    }
    else {
        List<Span> spans = cache.getIfPresent("test");
        if(isNull(spans)){
            spans = new ArrayList<>();
            spans.add(new Span());
            cache.put("test", spans);
            log.info("Put key test and value" + spans.toString());
        }
        else {
        	spans = new ArrayList<>();
            Span span = new Span();
            span.setTime(new Timestamp(System.currentTimeMillis()));
            spans.add(span);
            log.info("Add value!");
        }
    }
}
```

然后使用swagger 调用此接口进行简单测试。

测试结合文档[1]，发现以下几点需要注意：

1. remove回调并不是CacheBuilder主动触发。也就是说某个键值对过期时间到了也不会自行触发回调方法，除非你再次访问这个键值对，它通过时间比较才会知道超时了然后去触发回调方法。
2. access和write过期时间处理时有所不同的：
   1. expireAfterAccess(10, TimeUnit.SECONDS)：自上次访问以来再超过10s那么移除并且回调onRemoval。访问键值对可以更新过期时间，**注意，只要访问就算。**
   2. expireAfterWrite(10, TimeUnit.SECONDS)：自此键值对创建以来超过10s那么移除并且回调onRemoval。更新值并不会更新过期时间，**注意，更新了值也没用，还是会过期。**



经过测试和阅读文档我们发现了CacheBuilder并不会主动触发过期时间，只有在再次访问这个键值对的时候才会处理，有的时候我们不会再次去访问某个key，那么它即便超过了过期时间也会一直滞留在内存，时间久了像这种键值对越来越多就会导致各种问题。

但是CacheBuilder提供了cleanup()方法，用法可参考[1],

大意就是因为CacheBuilder不会主动触发回调，所以我们可以使用cleanup()来手动检查，它会检查所有的key，如果过期就会触发回调方法。我们可以根据需求在处理流式数据过程中调用，也可以加一个定时任务定期调用。

## 参考文档

1. guava github wiki：https://github.com/google/guava/wiki/CachesExplained