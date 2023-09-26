## 注意事项

1.  ***kibana***\*\* 的版本需要与 es 的版本对应，否则不能使用\*\*
2.  **es 的版本与 spring boot 版本有对应关系，如果不对应，会出现一些问题。**

    \*\*查看版本对应： \*\*[Spring Data Elasticsearch - Reference Documentation](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#preface.versions)
3.  **修改了实体类的名称、包所在位置等信息，会导致es增删查改报错，需要清除之前的所有数据及缓存。**
4.  以下内容基于 ==spring boot 2.3.7.RELEASE ==版本

# 安装服务（Windows环境）

### 安装与启动

1.  下载网址：[Download Elasticsearch | Elastic](https://www.elastic.co/cn/downloads/elasticsearch)
2.  下载后为一个zip文件，解压缩到目录中
3.  解压后为 elasticsearch-X.X.X  目录，进入/bin目录下，点击 ***elasticsearch.bat*** 启动服务
4.  如果启动出现以下报错，则原因为：在GeoIpDownloader下载更新的环节存在bug，可能导致索引分片状态判断不准确，从而抛出错误

        exception during geoip databases updateorg.elasticsearch.ElasticsearchException: not all primary shards of [.geoip_databases] index are active at org.elasticsearch.ingest.geoip@8.4.3/org.elasticsearch.ingest.geoip.GeoIpDownloader.updateDatabases(GeoIpDownloader.java:134)
        ...
        could not delete old chunks for geoip database [GeoLite2-ASN.mmdb]org.elasticsearch.action.search.SearchPhaseExecutionException: all shards failed at org.elasticsearch.server@8.4.3/org.elasticsearch.action.search.AbstractSearchAsyncAction.onPhaseFailure(AbstractSearchAsyncAction.java:728)
        GeoIpDo
5.  解决办法：修改/config/elasticsearch.yml 中的配置信息，在尾部添加以下代码，作用为：关闭geoip库的更新

        ingest.geoip.downloader.enabled: false
6.  启动后，输入 [localhost:9200](http://localhost:9200/) 进入es。启动成功后，出现以下内容

    ```json
    {
        "name": "LAPTOP-LTQV032Q",
        "cluster_name": "elasticsearch",
        "cluster_uuid": "RaCLRqFDROC5HbzBDaE01w",
        "version": {
            "number": "8.5.3",
            "build_flavor": "default",
            "build_type": "zip",
            "build_hash": "4ed5ee9afac63de92ec98f404ccbed7d3ba9584e",
            "build_date": "2022-12-05T18:22:22.226119656Z",
            "build_snapshot": false,
            "lucene_version": "9.4.2",
            "minimum_wire_compatibility_version": "7.17.0",
            "minimum_index_compatibility_version": "7.0.0"
        },
        "tagline": "You Know, for Search"
    }
    ```

### 账号密码

1.  默认的账号密码

    user: elastic&#x20;

    password: changeme（或者123456）
2.  修改密码

    ```bash
    查看es中所有的用户
    在浏览器地址输入 http://localhost:9200/_security/user

    重置  kibana_system 的密码
    bin/elasticsearch-reset-password --batch --user kibana_system  
     
    重置后，会输出重置的密码。可以通过重置后的密码，将密码置为指定密码
    curl -XPUT -u elastic（用户）:"otpKRXJZqe9Dzs5iXTjO"（密码） 'http://localhost:9200/_security/user/elastic（对应用户）/_password' -H "Content-Type: application/json" -d '{  "password": "1234qwer."（需要修改的密码）}'
    ```

### 可视化工具--head插件安装

1.  安装node,安装后，使用 node -v 查看node版本
2.  安装grunt

    使用命令安装：npm install -g grunt-cli 安装grunt

    安装结束后，使用命令grunt -version查看是否安装成功
3.  安装head插件

    1.  \*\*下载head插件 \*\*下载地址：<https://github.com/mobz/elasticsearch-head>
    2.  **解压zip包**
    3.  在dos窗口进入到head路径下，使用命令npm install安装pathomjs
    4.  使用命令npm start启用服务
    5.  浏览器中访问，使用地址：localhost:9100访问.默认端口9100

### 使用kibana查看es（推荐）

1.  从官网下载kibana
2.  解压缩
3.  修改kibana的yml配置，在尾部添加相关设置项代码

    ```bash
    修改为中文
    i18n.locale: "zh-CN"
    设置连接的es的用户名密码

    ```

&#x9;&#x9;

# ES分词器&#x20;

### 常见分词器

standard：ES默认的分词器，会将词汇单元进行小写形式，并且去除一些停用词和标点符号等等。支持中文，采用的方法为单字切分。

IK分词器：目前ES开源社区对于中文分词支持最好的第三方的插件。

### 使用IK进行分词

1.  下载ik。GitHub地址：

    <https://github.com/medcl/elasticsearch-analysis-ik>&#x20;

    下载与es版本一致的ik版本，若找不到对应版本，则下载最接近的版本。
2.  解压缩到 elasticsearch 目录下的  plugins 目录。

    例如解压缩后，目录为：**D:\workspace\elasticsearch-8.5.3\plugins\ik** 。ik文件夹中为插件文件
3.  修改 解压缩后的 ik目录中的 **plugin-descriptor.properties** 配置文件中的配置。

    在配置文件中，有配置的es版本信息，将配置文件中的版本信息与自己安装的es版本进行对应。
4.  重启es
5.  分词效果。ik默认提供两种分词器。

    1.  **ik\_smart**

        ```json
        // 在kibana 的控制台中 输入以下命令

        POST _analyze
        {
          "analyzer": "ik_max_word", 
          "text": "我是中国人"
        }

        // 输出结果为
        {
          "tokens": [
            {
              "token": "我",
              "start_offset": 0,
              "end_offset": 1,
              "type": "CN_CHAR",
              "position": 0
            },
            {
              "token": "是",
              "start_offset": 1,
              "end_offset": 2,
              "type": "CN_CHAR",
              "position": 1
            },
            {
              "token": "中国人",
              "start_offset": 2,
              "end_offset": 5,
              "type": "CN_WORD",
              "position": 2
            }
          ]
        }
        ```
    2.  **ik\_max\_word**

        ```json
        // 在kibana 的控制台中 输入以下命令

        POST _analyze
        {
          "analyzer": "ik_max_word", 
          "text": "我是中国人"
        }

        // 输出结果为
        {
          "tokens": [
            {
              "token": "我",
              "start_offset": 0,
              "end_offset": 1,
              "type": "CN_CHAR",
              "position": 0
            },
            {
              "token": "是",
              "start_offset": 1,
              "end_offset": 2,
              "type": "CN_CHAR",
              "position": 1
            },
            {
              "token": "中国人",
              "start_offset": 2,
              "end_offset": 5,
              "type": "CN_WORD",
              "position": 2
            },
            {
              "token": "中国",
              "start_offset": 2,
              "end_offset": 4,
              "type": "CN_WORD",
              "position": 3
            },
            {
              "token": "国人",
              "start_offset": 3,
              "end_offset": 5,
              "type": "CN_WORD",
              "position": 4
            }
          ]
        }
        ```

# 集成Spring Boot

## 依赖引入

在pom中引入以下依赖，其中version的版本需要与项目中spring boot 的版本一致

           <!--ES start-->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
                <version>2.3.7.RELEASE</version>
            </dependency>
            <!--ES end-->

## 配置修改

在 application.yml 中，添加以下依赖。elasticsearch 无任何前缀

```
server:
  port: 9099


# es配置项
elasticsearch:
  schema: http
  # es的地址
  address: 127.0.0.1:9200
  # 建立连接的超时时间
  connectTimeout: 5000
  # 数据交换的超时时间
  socketTimeout: 5000
  # 请求的超时时间
  connectionRequestTimeout: 5000
  # 最大连接数
  maxConnectNum: 100
  # 每条路由的最大连接数
  maxConnectPerRoute: 100
  # es的用户名及密码
  UserName: elastic
  Password: 123456


```

## 示例类

### dto

```java
package code.spring.dto;

import lombok.Data;
import nonapi.io.github.classgraph.json.Id;
import org.springframework.data.elasticsearch.annotations.DateFormat;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

import java.util.Date;

/**
 *
 * @Document注解 用于指定该实体类映射到Elasticsearch中的哪个索引
 * @Id注解 用于指定该实体类的主键字段
 * @Field注解 用于指定该字段在Elasticsearch中的映射方式。
 *
 * @author liuxin
 */
@Data
@Document(indexName = "product", shards = 1)
public class Product {
    @Id
    private Long id;

    @Field(type = FieldType.Keyword)
    private String name;

    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String description;

    @Field(type = FieldType.Double)
    private Double price;

    @Field(type = FieldType.Keyword)
    private String category;

    @Field(type = FieldType.Date, format = DateFormat.date_optional_time)
    private Date createTime;
}

```

### repository

```java
package code.spring.repository;

import code.spring.dto.Product;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @author liuxin
 */
@Service
public interface ProductRepository extends ElasticsearchRepository<Product, Long> {


    /**
     * 通过名称或者描述搜索
     *
     * @param name
     * @param description
     * @return
     */
    List<Product> findByNameOrDescription(String name, String description);

}

```

### service

```java
package code.spring.service;

import code.spring.dto.Product;
import code.spring.repository.ProductRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

/**
 * Elasticsearch
 *
 * @Author 刘新
 * @Date 2023/8/25 15:10
 */
@Service
public class EsService {

    @Autowired
    private ProductRepository productRepository;


    /**
     * 通过名称或者描述查询
     *
     * @param name        名称
     * @param description 描述
     * @return
     */
    public List<Product> findByNameOrDescription(String name, String description) {
        List<Product> list = productRepository.findByNameOrDescription(name, description);
        return list;
    }


    /**
     * 保存方法
     * 使用try catch 避免版本不对应时的报错
     *
     * @param product
     * @return
     */
    public String save(Product product) {
        try {
            productRepository.save(product);
        } catch (Exception e) {
            // 校验。如果返回值存在 200、ok 则保存成功；如果返回值存在201、created 则创建成功。两种情况均视为保存成功
            List<String> code = Arrays.asList("201", "200");
            List<String> str = Arrays.asList("created", "ok");
            String message = e.getMessage().toLowerCase();
            boolean codeCheck = code.stream().anyMatch(message::contains);
            boolean strCheck = str.stream().anyMatch(message::contains);
            if (codeCheck && strCheck) {
                return "保存成功";
            } else {
                return "保存失败";
            }
        }
        return "保存成功";
    }

    /**
     * 根据id查询
     *
     * @param id
     * @return
     */
    public Optional<Product> findById(Long id) {
        return productRepository.findById(id);
    }

    /**
     * 根据id删除
     *
     * @param id
     */
    public void deleteById(Long id) {
        productRepository.deleteById(id);
    }
}

```

### controller

```java
package code.spring.controller;

import code.spring.dto.Product;
import code.spring.service.EsService;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.Date;
import java.util.List;
import java.util.Optional;

/**
 * Elasticsearch
 * <a href="https://note.youdao.com/s/4aqVIQp1">...</a>
 *
 * @Author 刘新
 * @Date 2023/8/24 14:33
 */
@RestController
@RequestMapping("/es")
public class EsController {


    @Autowired
    private EsService esService;

    @ApiOperation(value = "查询", notes = "查询")
    @GetMapping("/search")
    public List<Product> search(@RequestParam String keyword) {
        return esService.findByNameOrDescription(keyword, keyword);
    }


    @ApiOperation(value = "保存", notes = "保存")
    @PostMapping("/save")
    public String saveArticle(@RequestBody Product product) {
        product.setCreateTime(new Date());
        String result = esService.save(product);
        return result;
    }

    @DeleteMapping("/delete")
    @ApiOperation(value = "删除商品")
    public String delete(@RequestParam("id") Long id) {
        esService.deleteById(id);
        return "删除成功";
    }

    @PutMapping("/update")
    @ApiOperation(value = "修改商品")
    public String update(@RequestBody Product product) {
        Optional<Product> productOptional = esService.findById(product.getId());
        if (productOptional.isPresent()) {
            String result = esService.save(product);
            return result;
        }
        return "修改失败";
    }
}

```

#

# 高级查询

## 范围查询

```java
package code.spring.service;

import code.spring.dto.Product;
import code.spring.repository.ProductRepository;
import org.elasticsearch.index.query.QueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.elasticsearch.core.ElasticsearchRestTemplate;
import org.springframework.data.elasticsearch.core.SearchHit;
import org.springframework.data.elasticsearch.core.SearchHits;
import org.springframework.data.elasticsearch.core.query.NativeSearchQuery;
import org.springframework.data.elasticsearch.core.query.NativeSearchQueryBuilder;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

/**
 * Elasticsearch
 *
 * @Author 刘新
 * @Date 2023/8/25 15:10
 */
@Service
public class EsService {

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private ElasticsearchRestTemplate elasticsearchTemplate;

    /**
     * 根据商品价格查询
     *
     * @param minPrice 价格最小值
     * @param maxPrice 价格最大值
     * @return
     */
    public List<Product> findPrice(double minPrice, double maxPrice) {
		// "price" 为要查询的字段
        QueryBuilder queryBuilder = QueryBuilders.rangeQuery("price")
                .from(minPrice)
                .to(maxPrice);
        NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withQuery(queryBuilder)
                .build();
        SearchHits<Product> products = elasticsearchTemplate.search(searchQuery, Product.class);
        List<Product> resList = products.stream().map(SearchHit::getContent).collect(Collectors.toList());
        return resList;
    }
}

```

## 模糊查询

```java
// ---- 省略 import、方法名、自动注入、controller层 等代码 ---- //
    /**
     * 根据名称模糊查询
     *
     * @param name 商品名称
     */
    public List<Product> findLike(String name) {
        QueryBuilder queryBuilder = QueryBuilders.matchQuery("name",name);
        NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withQuery(queryBuilder)
                .build();
        SearchHits<Product> products = elasticsearchTemplate.search(searchQuery, Product.class);
        List<Product> list = products.stream().map(SearchHit::getContent).collect(Collectors.toList());
        return list;
    }
```

## 分页查询

```java
// ---- 省略 import、方法名、自动注入、controller层 等代码 ---- //
	 /**
     * 分页查询
     *
     * @param pageNum 当前页 默认从第0页开始
     * @param pageSize 每页条数
     */
    public List<Product> findPage(int pageNum, int pageSize) {
        PageRequest pageRequest = PageRequest.of(pageNum, pageSize);

        NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withPageable(pageRequest)
                .build();
        SearchHits<Product> products = elasticsearchTemplate.search(searchQuery, Product.class);
        List<Product> list = products.stream().map(SearchHit::getContent).collect(Collectors.toList());
        return list;
    }
```

## 排序

```java
// ---- 省略 import、方法名、自动注入、controller层 等代码 ---- //
  	/**
     * 排序查询
     *
     * @param sort 排序类型 asc正序 desc倒序
     */
    public List<Product> findSort(String sort) {
        SortOrder sortOrder = SortOrder.DESC;
        if ("asc".equals(sort)) {
            sortOrder = SortOrder.ASC;
        }
        FieldSortBuilder fieldSort = SortBuilders.fieldSort("price").order(sortOrder);
        NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withSort(fieldSort)
                .build();
        SearchHits<Product> products = elasticsearchTemplate.search(searchQuery, Product.class);
        List<Product> list = products.stream().map(SearchHit::getContent).collect(Collectors.toList());
        return list;
    }
```

## 分词查询

```java
// ---- 省略 import、方法名、自动注入、controller层 等代码 ---- //

	/**
     * 分词查询
     *
     * @param keyword
     * @return
     */
    public List<Product> findSplitWords(String keyword) {
        BoolQueryBuilder query = QueryBuilders.boolQuery().
                must(QueryBuilders.matchQuery("description", keyword).analyzer("ik_max_word"));
        NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
                .withQuery(query)
                .build();
        SearchHits<Product> products = elasticsearchTemplate.search(searchQuery, Product.class);
        // 找到最高分，向下取整
        float maxScore = products.getMaxScore();
        double floor = Math.floor(maxScore);
        // 筛选出大于取整的数据
        List<SearchHit<Product>> searchHitList = products.getSearchHits().stream().filter(x -> x.getScore() > floor).collect(Collectors.toList());
        List<Product> list = searchHitList.stream().map(SearchHit::getContent).collect(Collectors.toList());
        return list;
    }
```

# kibana命令

## 查询

```bash
// 查询数据，默认查询前10条
GET _search

// 查询指定数量数据
GET _search?size=1000

// 查询数据的条数
GET _search?size=0

```

## 新增

```bash
// 指定id进行插入或修改。如果id不存在则新增，存在则修改
POST /product/_doc/10
{
  "name": "充电宝",
  "description": "充电宝",
  "price": 30.5,
  "category": "充电宝",
  "createTime": "2023-09-04T07:58:36.577Z"
}
```

## 删除

```bash
// 根据id删除
DELETE /product/_doc/10

// 批量删除


// 根据范围进行删除


// 删除 product中，创建时间大于 2023-09-04T03:09:11.989Z 的数据
POST product/_delete_by_query
{
  "query": {
    "range": {
      "createTime": {
        "gt":"2023-09-04T03:09:11.989Z"
      }
    }
  }
}
```

