# Spring 整合 Elasticsearch


<!--more-->

## 引入依赖

spring-boot-starter-data-elasticsearch

https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-elasticsearch

```xml
<!-- Spring整合Elasticsearch -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

</br>

## 配置Elasticsearch

cluster-name、cluster-nodes

```properties
# 配置 Elasticsearch
spring.data.elasticsearch.cluster-name=community
spring.data.elasticsearch.cluster-nodes=localhost:9300
```

</br>

## Spring Data Elasticsearch

### 解决Elasticsearch和Redis的冲突

因为Elasticsearch和Redis底层都是基于Netty的，导致它们在使用Netty的时候会出现冲突

**查看NettyRuntime类**

```java
synchronized void setAvailableProcessors(final int availableProcessors) {
    ObjectUtil.checkPositive(availableProcessors, "availableProcessors");
    if (this.availableProcessors != 0) {
        final String message = String.format(
                Locale.ROOT,
                "availableProcessors is already set to [%d], rejecting [%d]",
                this.availableProcessors,
                availableProcessors);
        throw new IllegalStateException(message);
    }
    this.availableProcessors = availableProcessors;
}
```

**查看Netty4Utils类**

```java
public static void setAvailableProcessors(final int availableProcessors) {
    // we set this to false in tests to avoid tests that randomly set processors from stepping on each other
    final boolean set = Booleans.parseBoolean(System.getProperty("es.set.netty.runtime.available.processors", "true"));
    if (!set) {
        return;
    }

    /*
     * This can be invoked twice, once from Netty4Transport and another time from Netty4HttpServerTransport; however,
     * Netty4Runtime#availableProcessors forbids settings the number of processors twice so we prevent double invocation here.
     */
    if (isAvailableProcessorsSet.compareAndSet(false, true)) {
        NettyRuntime.setAvailableProcessors(availableProcessors);
    } else if (availableProcessors != NettyRuntime.availableProcessors()) {
        /*
         * We have previously set the available processors yet either we are trying to set it to a different value now or there is a bug
         * in Netty and our previous value did not take, bail.
         */
        final String message = String.format(
                Locale.ROOT,
                "available processors value [%d] did not match current value [%d]",
                availableProcessors,
                NettyRuntime.availableProcessors());
        throw new IllegalStateException(message);
    }
}
```

可以看到Netty4Utils中调用了NettyRuntime.setAvailableProcessors，其中的参数availableProcessors只要不等于0就会报错，但是Redis调用Netty后该参数已经不为0。为了使程序不执行到这一段，我们看到之前有一段类似“开关”一样的代码

```java
if (!set) {
	return;
}
```

只要在这里程序返回了，之后的代码就不会执行，所以我们可以将set的值设置为false，但是此操作必须要在服务启动之初就进行，基于此我们可以在Application中进行设置。

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import javax.annotation.PostConstruct;

@SpringBootApplication
public class CommunityApplication {

   // 该注解用于管理Bean的生命周期，主要是管理Bean的初始化方法，由该注解所修饰的方法会在构造器调用完以后被执行
   @PostConstruct
   public void init(){
      // 解决Netty启动冲突的问题
      // 设置为“false”，底层在用的时候会进行转型
      // see Netty4Utils.setAvailableProcessors
      System.setProperty("es.set.netty.runtime.available.processors","false");
   }


   public static void main(String[] args) {
      SpringApplication.run(CommunityApplication.class, args);
   }

}
```

### 访问Elasticsearch

**实体类**

这里不象Mybatis要写xml配置文件，直接在实体类上写注解就行。

Spring整合ES提供的这项技术，其底层在访问ES服务器的时候，它自动的将实体数据和ES服务器中的索引进行映射。所以我们需要在实体类上加上注解，并且在属性上也要加上必要的注解对应ES中的字段。

```java
// indexName：索引名，type：旧版的ES就直接写_doc好了，shards：分片数量，replicas：副本数量
@Document(indexName = "discusspost", type = "_doc", shards = 6, replicas = 3)
public class DiscussPost {

    @Id
    private int id;

    @Field(type = FieldType.Integer)
    private int userId;

    // 搜索的主要数据，analyzer存储时的解析器，searchAnalyzer搜索时的解析器，这里使用中文分词插件ik作为解析器
    @Field(type = FieldType.Text, analyzer = "ik_max_mord", searchAnalyzer = "ik_smart")
    private String title;

    @Field(type = FieldType.Text, analyzer = "ik_max_mord", searchAnalyzer = "ik_smart")
    private String content;

    @Field(type = FieldType.Integer)
    private int type;

    @Field(type = FieldType.Integer)
    private int status;

    @Field(type = FieldType.Date)
    private Date createTime;

    @Field(type = FieldType.Integer)
    private int commentCount;

    @Field(type = FieldType.Double)
    private Double score;
}
```

**Repository接口**

```java
@Repository
// 需要继承ElasticsearchRepository，加上泛型，声明该接口要处理的实体类是谁，以及ID是什么类型
public interface DiscussPostRepository extends ElasticsearchRepository<DiscussPost, Integer> {
}
```

**存入数据的测试方法**

```java
@SpringBootTest
//使用正式环境里面的配置类
@ContextConfiguration(classes = CommunityApplication.class)
public class ElasticsearchTests {

    @Autowired
    private DiscussPostMapper discussPostMapper;

    @Autowired
    private DiscussPostRepository discussPostRepository;

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;

	// 插入一条数据
    @Test
    public void testInsert(){
        discussPostRepository.save(discussPostMapper.selectDiscussPostById(289));
    }
    
    // 插入多条数据
    @Test
    public void testInsertList() {
        discussPostRepository.saveAll(discussPostMapper.selectDiscussPosts(155, 0, 100));
    }

}
```

**修改数据的测试方法**

```java
@Test
public void testUpdate() {
    // 其实就是修改数据之后，进行覆盖插入，不演示了
}
```

**删除数据的测试方法**

```java
// 删除数据
@Test
public void testDelete() {
    // 删除单条数据
    discussPostRepository.deleteById(289);
    // 删除所有数据
    discussPostRepository.deleteAll();
}
```

**查询数据 ByRepository**

```java
// 查询数据 ByRepository
@Test
public void testSearchByRepository() {
    SearchQuery searchQuery = new NativeSearchQueryBuilder()
            // 搜索关键字和范围
            .withQuery(QueryBuilders.multiMatchQuery("Client例子", "title", "content"))
            // 排序方式
            .withSort(SortBuilders.fieldSort("type").order(SortOrder.DESC))
            .withSort(SortBuilders.fieldSort("score").order(SortOrder.DESC))
            .withSort(SortBuilders.fieldSort("createTime").order(SortOrder.DESC))
            // 分页
            .withPageable(PageRequest.of(0, 10))
            // 高亮部分，添加标签
            .withHighlightFields(
                    new HighlightBuilder.Field("title").preTags("<em>").postTags("</em>"),
                    new HighlightBuilder.Field("content").preTags("<em>").postTags("</em>")
            ).build();

    // elasticremplate.queryForPage（searchQuery，class，SearchResultMapper）
    // 底层获取得到了高亮显示的值，但是进行进一步的处理。

    Page<DiscussPost> page = discussPostRepository.search(searchQuery);

    System.out.println(page.getTotalElements());
    System.out.println(page.getTotalPages());
    System.out.println(page.getNumber());
    System.out.println(page.getSize());

    for (DiscussPost post : page) {
        System.out.println(post);
    }
}
```

**查询数据 ByTemplate**

```java
// 查询数据 ByTemplate
@Test
public void testSearchByTemplate() {
    SearchQuery searchQuery = new NativeSearchQueryBuilder()
            // 搜索关键字和范围
            .withQuery(QueryBuilders.multiMatchQuery("Client例子", "title", "content"))
            // 排序方式
            .withSort(SortBuilders.fieldSort("type").order(SortOrder.DESC))
            .withSort(SortBuilders.fieldSort("score").order(SortOrder.DESC))
            .withSort(SortBuilders.fieldSort("createTime").order(SortOrder.DESC))
            // 分页
            .withPageable(PageRequest.of(0, 10))
            // 高亮部分，添加标签
            .withHighlightFields(
                    new HighlightBuilder.Field("title").preTags("<em>").postTags("</em>"),
                    new HighlightBuilder.Field("content").preTags("<em>").postTags("</em>")
            ).build();

    Page<DiscussPost> page = elasticsearchTemplate.queryForPage(searchQuery, DiscussPost.class, new SearchResultMapper() {
        @Override
        public <T> AggregatedPage<T> mapResults(SearchResponse searchResponse, Class<T> aClass, Pageable pageable) {
            SearchHits hits = searchResponse.getHits();
            if (hits.getTotalHits() <= 0) {
                return null;
            }

            List<DiscussPost> list = new ArrayList<>();
            for (SearchHit hit : hits) {
                DiscussPost post = new DiscussPost();
                String id = hit.getSourceAsMap().get("id").toString();
                post.setId(Integer.valueOf(id));

                String userId = hit.getSourceAsMap().get("userId").toString();
                post.setUserId(Integer.valueOf(userId));

                String title = hit.getSourceAsMap().get("title").toString();
                post.setTitle(title);

                String content = hit.getSourceAsMap().get("content").toString();
                post.setContent(content);

                String status = hit.getSourceAsMap().get("status").toString();
                post.setStatus(Integer.valueOf(status));

                String createTime = hit.getSourceAsMap().get("createTime").toString();
                post.setCreateTime(new Date(Long.valueOf(createTime)));

                String commentCount = hit.getSourceAsMap().get("commentCount").toString();
                post.setCommentCount(Integer.valueOf(commentCount));

                // 处理高亮显示的结果
                HighlightField titleField = hit.getHighlightFields().get("title");
                if (titleField != null) {
                    // 用带有标签的覆盖
                    post.setTitle(titleField.getFragments()[0].toString());
                }

                HighlightField contentField = hit.getHighlightFields().get("content");
                if (contentField != null) {
                    post.setContent(contentField.getFragments()[0].toString());
                }

                list.add(post);
            }

            return new AggregatedPageImpl(list, pageable,
                    hits.getTotalHits(), searchResponse.getAggregations(), searchResponse.getScrollId(), hits.getMaxScore());
        }

        @Override
        public <T> T mapSearchHit(SearchHit searchHit, Class<T> aClass) {
            return null;
        }
    });

    System.out.println(page.getTotalElements());
    System.out.println(page.getTotalPages());
    System.out.println(page.getNumber());
    System.out.println(page.getSize());

    for (DiscussPost post : page) {
        System.out.println(post);
    }
}
```



</br>

<center> ·End </center>
<center> 转载请注明出处: <a href="https://halklein.github.io/">https://halklein.github.io/</a> </center>
