---
title: ElasticSearch使用心得
date: 2018-03-07 
tags: 
- ElasticSearch
- Java
author: Faushine
layout: post
---


[ElasticSearch](https://www.elastic.co/blog/found-java-clients-for-elasticsearch)是一个分布式、可扩展、实时的搜索与数据分析引擎。
- 一个分布式的实时文档存储，每个字段 可以被索引与搜索
- 一个分布式实时分析搜索引擎
- 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据

在Java应用中一般会采用ElasticSearch原生接口或者[Jest](https://github.com/searchbox-io/Jest/tree/master/jest)提供的API来跟ES进行数据交互。如果你要用到ES的所有功能的话原生接口当然是最好的选择，但是如果你只是需要实现一些基本的操作的话，Jest这种轻量级的客户端可能会更好一点，相对来说Jest也更好上手。详细的分析可以参照这篇[博客](https://www.elastic.co/blog/found-java-clients-for-elasticsearch)。唯一不方便的一点就是Jest说明文档太简洁，很多接口的使用要对着集成测试用例去看才知道什么意思，不过也只是少数了，下面把最近实践过的例子记录一下。

## 初始化客户端 
``` java
 JestClientFactory factory = new JestClientFactory();
        factory.setHttpClientConfig(new HttpClientConfig.Builder("https://xxxxxxx.amazonaws.com")
                .connTimeout(36000)
                .readTimeout(36000)
                .multiThreaded(true)
                .build());
 return factory.getObject();
```

## 批量操作
  
``` java
public JestResult bulk(JestClient client, List<? extends BulkableAction> actions) throws IOException {
	Bulk bulk = new Bulk.Builder()
                .defaultIndex(INDEX)
                .defaultType(TYPE)
                .addAction(actions)
                .build();
    return client.execute(bulk);
}
```

**批量添加**
 ```java
@Test
public void insertBulk() throws IOException {
	for (int i = 1; i <= 10; i++) {
		bulk(client(), createIndex(prepareData(i)));
	}
}

private List<Index> createIndex(Map<String, DataEntity> data) {
	List<Index> indices = new ArrayList<>();
	data.forEach((key, value) -> {
		indices.add(new Index.Builder(GsonUtil.DEFAULT_GSON.toJson(value)).id(key).build());
	});
	return indices;
}

private Map<String, DataEntity> prepareData(int t) {
	Map<String, DataEntity> map = new HashMap<>();
	for (int i = 50000*(t-1) ; i < 50000 * t; i++) {
		map.put(String.valueOf(i), new DataEntity("test",String.valueOf(i), "a", "b", "c", "d"));
	}
	return map;
}
```

**批量更新**  
```java
@Test
public void updateBulk() throws IOException {
	for (int i = 1; i <= 2; i++) {
		bulk(client(), createUpdate(prepareDoc(i)));
	}
}

private List<Update> createUpdate(Map<String, UpdateDoc> updateDocs) {
	List<Update> updates = new ArrayList<>();
	updateDocs.forEach((key, value) -> {
		updates.add(new Update.Builder(GsonUtil.DEFAULT_GSON.toJson(value)).index(INDEX).type(TYPE).id(key).build());
	});
	return updates;
}

private Map<String, UpdateDoc> prepareDoc(int t) {
	Map<String, UpdateDoc> updateDocs = new HashMap<>();
	for (int i = 50000*(t-1) ; i < 50000 * t; i++) {
		updateDocs.put(String.valueOf(i), new UpdateDoc(new Doc("b")));
	}
	return updateDocs;
}
```

## 删除  
由于ElasticSearch 6.X之后就不再支持多个doc_type，所以创建delete对象的时候要么指定Index删除整个Index的document，要么指定Index和id， 删除特定id的document。
```java
@Test
public void delete() throws IOException {
	client().execute(new Delete.Builder("")
			.index(INDEX)
			.id("")
			.build());
}
```

## 搜索  
Jest提供的查询方式有很多，主要是通过创建get和search对象来执行查询。
**指定id查询**  
```java
public JestResult get(JestClient client) throws IOException {
	Get get = new Get.Builder(INDEX, ID).build();
	JestResult result = client.execute(get);
	return result;
}
```

**利用scroll遍历数据** 

*初始化* 
 初始化时需要像普通 search 一样，指明 index 和 type (当然，search 是可以不指明 index 和 type 的)，然后，加上参数 scroll，表示暂存搜索结果的时间，其它就像一个普通的search请求一样。初始化返回一个 _scroll_id，_scroll_id 用来下次取数据用。
```java
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
searchSourceBuilder.query(QueryBuilders.matchAllQuery()).size(size);
Search search = new Search.Builder(searchSourceBuilder.toString())
				.addIndex(INDEX)
				.addType(TYPE)
				.setParameter(Parameters.SCROLL, "1m")
				.build();
JestResult result = client().execute(search);
String scrollId = result.getJsonObject().get("_scroll_id").getAsString();
```

*遍历* 
这里的 scroll_id 即上一次遍历取回的 _scroll_id 或者是初始化返回的 _scroll_id，同样的，需要带 scroll 参数。重复这一步骤，直到返回的数据为空，即遍历完成。注意，每次都要传参数 scroll，刷新搜索结果的缓存时间。另外，不需要指定 index 和 type。设置scroll的时候，需要使搜索结果缓存到下一次遍历完成，同时，考虑到空间有限也不能太长。
```java
for (int i = 1; i < (int) Math.ceil(total / size); i++) {
	SearchScroll scroll = new SearchScroll.Builder(scrollId, "1m").build();
	result = client().execute(scroll);
	hits = result.getJsonObject().getAsJsonObject("hits").getAsJsonArray("hits");
	scrollId = result.getJsonObject().getAsJsonPrimitive("_scroll_id").getAsString();
}
```

## 统计  
```java
@Test
public void getCount() throws IOException {
	Count count = new Count.Builder()
                .addIndex(INDEX)
                .addType(TYPE).build();
	client().execute(count).getCount();
}
```

## 备注 

**不支持多类型**

*概述*  
关于6.x之后不支持multiple type这个事情可以多说几句，官方的说法有这么几点：
（1） 把 index , doc_type 类比为 RDBMS 的 db , table 这个出发点本身就是不对的，而且实际使用过程中也很少用到 type 这个字段。
（2） 在传统db中，不同的表中可以有相同命名但是不同数据类型、不同含义的字段，但是到了ES中，同一个index下的字段名相同的值类型必须一致。比如你有一个索引名叫做blog，其中 type 为  `user`  的  `region`  字段类型为object，又想在type为  `articles`  的  `region`  创建字段类型为text，这是没办法实现的，ES是不允许这种做法。其根本原因在 Lucene 底层实现上，同一索引的不同type中的相同字段，存储结构都是相同的。

（所以可以不用指定`type`）

*解决方案*  
在文档中手动添加一个字段为 `type`，在查询的时候匹配即可。

**TTL**

  
不管用什么方法设置mapping都提示说不支持_ttl这个参数，心力交猝。


update > **TTL and Timestamp Meta-fieldsedit**
Elasticsearch 6.0 is seeing the removal of the TTL and Timestamp meta-fields from being allowed on document index and update requests. Because of this, we will be removing the ability to specify them in 6.0 and deprecating their usage in 5.x.



## 性能分析  

下文提及的数据量表示document的数量，document 示例如下：

```json
{  
	"_index":"volume",  
	"_type":"test",  
	"_id":"1",  
	"_version":20,  
	"found":true,  
	"_source":{  
		"Type":"test",  
		"Id":"1",  
		"A":"a",  
		"B":"b",  
		"C":"c",  
		"D":"d"  
	}  
}
 ```
 
 - 批量添加   
 
| | 单批次数据量 | 总数据量 | 耗时 |
|--|--|--|--|
| 1 | 5w | 10w | 23s |
| 2 | 5w | 100w | 261s |


 - 批量查询   
 
利用scroll search遍历全部数据
 
| | 单批次数据量 | 总数据量 | 耗时 |
|--|--|--|--|
| 1 | 1w | 10w | 42s |

 - 批量更新

| | 单批次数据量 | 总更新量 | 总数据量 | 耗时 |
|--|--|--|--|--|
| 1 | 1w | 2w | 10w | 16s |
| 2 | 5w | 25w | 100w | 133s |

 - 条件查询  
 
| | 单批次数据量 | 查询数据量 | 总数据量 | 耗时 |
|--|--|--|--|--|
| 1 | 1w | 5w | 10w | 15s |
| 2 | 1w | 25w | 100w | 85s |

 - 不同存储量下并发更新
 
| | 单批次数据量 | 总数据量 | 耗时 |
|--|--|--|--|
| 1 | 40 | 10w | 5s |
| 2 | 40 | 100w | 5s |
