## springboot2.3 中使用elasticsearch

####  maven 安装依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
            <version>2.3.1.RELEASE</version>
        </dependency>
```

#### 版本差异

> 1.Elasticsearch支持到Elasticsearch7.x
>
> 2.ElasticsearchTemplate弃用改为 ElasticsearchRestTemplate 
>
> 3.ElasticsearchTemplate的 `query()` 方法改为 ElasticsearchRestTemplate的  `search()`
>
> 4.配置项 Elasticsearch访问路径和集群名称的配置已经不建议使用 改为 spring.elasticsearch.rest.uris=http://localhost:9200



#### 文档实体

```java
@Data //lombok包
@EqualsAndHashCode(callSuper = false) //生成equals(Object other) 和 hashCode()
@Document(indexName = "Company", shards = 1, replicas = 0) // 新版本不在使用type 
//@Mapping(mappingPath = "company_mapping.json") 如果不想用注解直接生成maping 可使用自定义 文件在resources下
public class EsCompany implements Serializable {

    @Id
    private Long id;
     
     //添加 ik 分词 插件自行安装
    @Field(analyzer = "ik_max_word", searchAnalyzer = "ik_smart", type = FieldType.Text)
    private String title;
  
    //建议搜索
    @CompletionField(maxInputLength = 100, analyzer = "ik_max_word")
    private Completion suggest;
  
 }

```



####  使用Repository进行操作

> 更高级的复杂查询建议使用elasticsearchRestTemplate

1.创建 Repository

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import com.chen.stencil.search.domain.EsAllCompany;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

//这里使用上面创建的实体
public interface EsAllCompanyRepository extends ElasticsearchRepository<EsCompany, Long> {


    /**
     * 根据名称搜索排序根据 level 字段降序
     */
    Page<EsCompany> findByTitleOrderByLevelDesc(String title, Pageable page);
}

```

2.关键字举例子

| 关键字        | 例子                     | Elasticsearch Query String                                   |
| ------------- | ------------------------ | :----------------------------------------------------------- |
| And           | findByNameAndPrice       | { "query" : { "bool" : { "must" : [ { "query_string" : { "query" : "?", "fields" : [ "name" ] } }, { "query_string" : { "query" : "?", "fields" : [ "price" ] } } ] } }} |
| Or            | findByNameOrPrice        | { "query" : { "bool" : { "should" : [ { "query_string" : { "query" : "?", "fields" : [ "name" ] } }, { "query_string" : { "query" : "?", "fields" : [ "price" ] } } ] } }} |
| Is            | findByName               | { "query" : { "bool" : { "must" : [ { "query_string" : { "query" : "?", "fields" : [ "name" ] } } ] } }} |
| Not           | findByNameNot            | { "query" : { "bool" : { "must_not" : [ { "query_string" : { "query" : "?", "fields" : [ "name" ] } } ] } }} |
| Between       | findByPriceBetween       | `{ "query" : { "bool" : { "must" : [ {"range" : {"price" : {"from" : ?, "to" : ?, "include_lower" : true, "include_upper" : true } } } ] } }}` |
| LessThan      | findByPriceLessThan      | { "query" : { "bool" : { "must" : [ {"range" : {"price" : {"from" : null, "to" : ?, "include_lower" : true, "include_upper" : false } } } ] } }} |
| LessThanEqual | findByPriceLessThanEqual | { "query" : { "bool" : { "must" : [ {"range" : {"price" : {"from" : null, "to" : ?, "include_lower" : true, "include_upper" : true } } } ] } }} |



2. [更多查看此链接]: https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#elasticsearch.repositories

   





#### elasticsearchRestTemplate 复杂搜索



```java
    @Autowired
    private ElasticsearchRestTemplate elasticsearchRestTemplate;
    

    /**
     * 复杂搜索
     */
    public Page<EsCompany> search(String title, Integer pageNum, Integer pageSize) {
      
       //构建类
       Pageable pageable = PageRequest.of(pageNum, pageSize);
       
       //构建查询
       NativeSearchQueryBuilder builder = new NativeSearchQueryBuilder();
       //添加分页 注意 es分页是从0页开始
       builder.withPageable(pageable);
        
       // 查询全部
       builder.withQuery(QueryBuilders.matchAllQuery()); 
       // must查询
       builder.withQuery(QueryBuilders.boolQuery().must(QueryBuilders.matchQuery("title", title)));
      
       // shoud查询
       builder.withQuery(QueryBuilders.boolQuery().should(QueryBuilders.matchQuery("title",         title)).must(QueryBuilders.matchQuery("title", title)));
 
      
       //范围查询
       builder.withQuery(QueryBuilders.rangeQuery("level").gte(1));
      
      
       //查询需要的字段
       builder.withFields("title", "xx");
       //排序相关 
       builder.withSort(SortBuilders.fieldSort("level").order(SortOrder.DESC));
      
        //随机排序 即随机查数据
       Script script = new Script("Math.random()");
       builder.withSort(SortBuilders.scriptSort(script, ScriptSortBuilder.ScriptSortType.NUMBER));
      
       //生成查询语句
       NativeSearchQuery searchQuery = builder.build();
       
       //搜索 注意 SearchCompanyReturn 实体需要@Document注解否则会报错找不到index
       SearchHits<SearchCompanyReturn> c = elasticsearchRestTemplate.search(searchQuery,SearchCompanyReturn.class);
      
      
       //下面返回数据
        if (c.getTotalHits() <= 0) {
            return new PageImpl<>(new ArrayList<>(), pageable, 0);
        }
        //System.out.println(c.getTotalHits());
        List<SearchCompanyReturn> searchProductList = c.stream().map(SearchHit::getContent).collect(Collectors.toList());
        return new PageImpl(searchProductList, pageable, c.getTotalHits());

    }

    /*
     * 前缀建议搜索
     *
     * */
    public List<String> suggestSearch(String title) {

        if (StringUtils.isEmpty(title)) {

            return new ArrayList<>();
        }

        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        SuggestBuilder suggestBuilder = new SuggestBuilder();


        //构造搜索建议语句,搜索条件字段
        CompletionSuggestionBuilder completionSuggestionBuilder = SuggestBuilders.completionSuggestion("suggest");
        //搜索关键字
        completionSuggestionBuilder.text(title);
        //去除重复
        completionSuggestionBuilder.skipDuplicates(true);

        //匹配数量
        completionSuggestionBuilder.size(10);

        //创建自己的suggest
        suggestBuilder.addSuggestion("title_suggest", completionSuggestionBuilder);
        sourceBuilder.suggest(suggestBuilder);

       
        SearchRequest searchRequest = new SearchRequest("company");
        searchRequest.source(sourceBuilder);


        SearchResponse A = elasticsearchRestTemplate.suggest(suggestBuilder, IndexCoordinates.of("company"));

        //取出自定义的建议搜索内容
        CompletionSuggestion completionSuggestion = A.getSuggest().getSuggestion("title_suggest");
        List<CompletionSuggestion.Entry.Option> options = completionSuggestion.getEntries().get(0).getOptions();
        List<String> suggestList = new ArrayList<>();
        options.forEach(item -> {
            suggestList.add(item.getText().string());
        });

        return suggestList;


    }





```

