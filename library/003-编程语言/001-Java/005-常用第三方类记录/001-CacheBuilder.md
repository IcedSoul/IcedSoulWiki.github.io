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



CacheBuilder并不会主动触发过期时间，只有在再次访问这个键值对的时候才会处理，有的时候我们不会再次去访问某个key，那么它即便超过了过期时间也会一直滞留在内存，时间久了像这种键值对越来越多就会导致各种问题。

但是CacheBuilder提供了cleanup()方法，用法可参考[1]中说明：

> ### When Does Cleanup Happen?
>
> Caches built with `CacheBuilder` do *not* perform cleanup and evict values "automatically," or instantly after a value expires, or anything of the sort. Instead, it performs small amounts of maintenance during write operations, or during occasional read operations if writes are rare.
>
> The reason for this is as follows: if we wanted to perform `Cache` maintenance continuously, we would need to create a thread, and its operations would be competing with user operations for shared locks. Additionally, some environments restrict the creation of threads, which would make `CacheBuilder` unusable in that environment.
>
> Instead, we put the choice in your hands. If your cache is high-throughput, then you don't have to worry about performing cache maintenance to clean up expired entries and the like. If your cache does writes only rarely and you don't want cleanup to block cache reads, you may wish to create your own maintenance thread that calls [`Cache.cleanUp()`](http://google.github.io/guava/releases/11.0.1/api/docs/com/google/common/cache/Cache.html#cleanUp--) at regular intervals.
>
> If you want to schedule regular cache maintenance for a cache which only rarely has writes, just schedule the maintenance using [`ScheduledExecutorService`](http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html).

大意就是因为CacheBuilder不会主动触发回调，所以我们可以使用cleanup()来手动检查，它会检查所有的key，如果过期就会触发回调方法。我们可以根据需求在处理流式数据过程中调用，也可以加一个定时任务定期调用。

## 参考文档

1. guava github wiki：https://github.com/google/guava/wiki/CachesExplained