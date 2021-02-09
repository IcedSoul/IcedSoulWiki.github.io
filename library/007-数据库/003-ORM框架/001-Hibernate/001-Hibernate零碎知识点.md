# Hibernate零碎知识点

## HIbernate SQL路径探究

Hibernate可以使用HQL语句，原生SQL语句，自带save，update方法以及最新的根据方法名生成SQL等方法来对数据库进行增删改查，因为工作需要，今天主要研究一下使用HQL语句、save，update语句的关键路径在哪里，怎么把SQL语句从代码中给找出来。

### HQL: org.hibernate.engine.query.spi.HQLQueryPlan

其中的构造方法中的sqlStringList会包含所有HQL转化成的SQL语句，当然底层还有更多实现，虽然看下去了但是最后还是发现这里的是最完整的，只要是HQL语句都可以从这里找出转化之后的SQL。

### Native SQL: org.hibernate.internal.AbstractSharedSessionContract

原生SQL查询因为自带了SQL，其实在哪一步拦截都无所谓，这里我选择了比较深入的一个类，目前还没有尝试过使用@Query注解是否会走到这里来，如果没有调用这里还需要再往底层找。这个类中的createNativeQuery() 方法直接拿参数就行。

### save(),update(),saveOrUpdate,delete(): org.hibernate.persister.entity.AbstractEntityPersister

找这个其实挺麻烦的，里面七绕八绕的，最后才发现其实这三个方法最终都要调用这个类中的方法来获取最终的SQL，方法有多个，得一一拦截，而且不得不吐槽一句这个类竟然有5600多行。。。规范呢。



最后一种JPA推荐的，根据方法名来生成SQL的场景我并没有研究 = =

不过，貌似可以直接拦截jdbc执行的SQL是不是就没有这么多麻烦了？好像我之前有找到JDBC执行SQL的位置来着？。。。好像很有道理啊，先不管了，这个暂时就放在这里吧，就当做学习了。