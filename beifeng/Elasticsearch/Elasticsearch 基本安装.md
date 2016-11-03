# Elasticsearch 基本安装

标签（空格分隔）： Elasticsearch

[TOC]

---

##集群健康值说明:
```
红色,代表集群不可用
黄色,代表集群可用不可复制
绿色,代表集群正常
```
##1、下载
https://www.elastic.co/downloads/elasticsearch
##2、安装
###2.1解压 elasticsearch-*.tar.gz 文件
> tar -xvf elasticsearch-*.tar.gz

![014.png-12.8kB][1]

###2.2 安装marvel插件
> bin/plugin install license
bin/plugin install marvel-agent

Note
每个版本的安装命令都不同，具体参看最新官网文档，
https://www.elastic.co/downloads/marvel

###2.3安装 head
> bin/plugin install mobz/elasticsearch-head



##3、启动
> bin/elasticsearch
bin/elasticsearch -d #后台启动

##4、基本配置
***冒号后面必需预留一个空格***
```
cluster.name: hks
node.name: hks_data
node.master: true
node.data: true
#分片索引个数
index.number_of_shards : 5
#副本个数
index.number_of_replicas : 1
#索引文件路径,多个用逗号隔开
path.data : /hksdata/java/elasticsearch/data
#临时文件存储
path.work : /hksdata/java/elasticsearch/tmp
#日志文件路径
path.logs : /hksdata/java/elasticsearch/logs
#设置绑定ip,多个ip用逗号隔开，默认为本地
network.host : 10.163.101.230,120.27.43.49
http.port : 9200
transport.tcp.port : 9300 
transport.tcp.compress : true
#设置传输最大数据为100M
http.max_content_length : 10mb
# 启用对外http
http.enabled : true

#es集群配置，数据自动复制
discovery.zen.minimum_master_nodes: 2
discovery.zen.ping.timeout: 10s
discovery.zen.ping.multicast.enabled: true
discovery.zen.ping.unicast.hosts: ["10.163.101.230" , "10.45.19.14"]

#配置IK分词器
index.analysis.analyzer.default.type: ik
```
##5、IK分词器
a、下载ik分词器已发布的版本 https://github.com/medcl/elasticsearch-analysis-ik/releases
b、安装maven
c、编译ik分词器
  1、解压该zip到一个目录
  进入改目录，输入命令
```
 mvn clean install -Dmaven.test.skip=true 
```
  编译成功后会在target/releases目录下生成一个zip的文件，将这个文件复制到 es_home/plugins/analysis-ik 目录下（因为网络原因，一次可能编译不成功，可以尝试多次编译）
    ![02.png-23.3kB][3]
##6、设置某个字段不分词
```
curl -XPUT http://10.163.101.230:9200/hksesdata/ -d' { "mappings": { "part":{ "properties": { "relation_code": { "type": "string", "index": "not_analyzed" }, "materialname": { "type": "string" }, "relation_brandid": { "type": "string","index": "not_analyzed" }, "relation_picno": { "type": "string","index": "not_analyzed" }, "relation_postion": { "type": "string","index": "not_analyzed" }, "relation_oecode": { "type": "string","index": "not_analyzed" }, "relation_cartypecode": { "type": "string","index": "not_analyzed" }, "brandcode": { "type": "string", "index": "not_analyzed" }, "originscode": { "type": "string", "index": "not_analyzed" }, "ecpunit": { "type": "string", "index": "not_analyzed" }, "relation_groupid": { "type": "string","index": "not_analyzed" } }
```


##7、maven项目，依赖包
```
<dependency>
			<groupId>org.elasticsearch</groupId>
			<artifactId>elasticsearch</artifactId>
			<version>2.3.2</version>
		</dependency>
		<dependency>
			<groupId>com.google.guava</groupId>
			<artifactId>guava</artifactId>
			<version>19.0</version>
		</dependency>
```
##8、java查询api
```
package org.es;

import java.net.InetAddress;
import java.util.Iterator;
import java.util.Map;
import org.elasticsearch.action.search.SearchRequestBuilder;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.action.search.SearchType;
import org.elasticsearch.client.transport.TransportClient;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.settings.Settings.Builder;
import org.elasticsearch.common.transport.InetSocketTransportAddress;
import org.elasticsearch.index.query.BoolQueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.elasticsearch.search.aggregations.Aggregation;
import org.elasticsearch.search.aggregations.AggregationBuilders;
import org.elasticsearch.search.aggregations.bucket.terms.StringTerms;
import org.elasticsearch.search.aggregations.bucket.terms.Terms;
import org.elasticsearch.search.aggregations.bucket.terms.Terms.Bucket;
import org.elasticsearch.search.aggregations.bucket.terms.TermsBuilder;
import org.elasticsearch.search.sort.SortBuilders;

public class ElasticSearchUtils {
	
	protected TransportClient client ; 
	
	protected String index ; 
	
	protected String type ; 
	
	protected String idKey ;
	/**
	 * 创建TransportClient
	 * @param clusterName 集群名称
	 * @param host 地址
	 * @param port es端口 
	 * */
	TransportClient getTransportClient(String clusterName , String host , Integer port) throws  Exception {
		Builder builder = Settings.builder().put("cluster.name",  clusterName ) ;
		TransportClient client = TransportClient
				.builder()
				.settings(builder)
				.build()
				.addTransportAddress(
						new InetSocketTransportAddress(InetAddress
								.getByName( host ), port));
		return client ; 
	}
	/**
	 * 创建TransportClient
	 * @param clusterName 集群名称
	 * @param host 地址
	 * @param port es端口 , 末日�?300
	 * @param index 索引
	 * @param type 类型
	 * @param idKey 主名
	 * */
	public static ElasticSearchUtils getElasticSearch(String clusterName
			 , String host , Integer port , String index , String type , String idKey){
		try {
			ElasticSearchUtils search = new ElasticSearchUtils();
			search.client = search.getTransportClient(clusterName, host, port) ;
			search.idKey = idKey ;
			search.index = index ; 
			search.type = type ; 
			return search ; 
		} catch(Exception e ) {
			throw new RuntimeException( e.getMessage() , e )  ;
		}
	}
	
	public static void main(String[] args) {
		ElasticSearchUtils searchUtils = ElasticSearchUtils.getElasticSearch("hks"
				, "hksdata01", 9300, "hksesdata", "part", "id") ;  
		/*创建搜索引擎*/
		SearchRequestBuilder requestBuilder = searchUtils.client.prepareSearch( searchUtils.index )
				.setTypes( searchUtils.type ) .setSearchType(SearchType.DEFAULT
						).setFrom(0)
				.setSize( 20 ) ; 
		/*指定排序字段,可以指定多个*/
		requestBuilder.addSort( SortBuilders.fieldSort("p1001_customertype2") );
		requestBuilder.addSort( SortBuilders.fieldSort("p1001_customertype2") );
		//requestBuilder.setQuery(QueryBuilders.matchPhraseQuery( "keyword" , "FDB" ));
		/*使用bool查询*/
		BoolQueryBuilder boolQueryBuilder = new BoolQueryBuilder();
		/*设置查询*/
		requestBuilder.setQuery(
				//QueryBuilders.termsQuery("keyword", "FDB" , "鲁达" )
				boolQueryBuilder
				.must(QueryBuilders.matchQuery("keyword", "奥迪 Q5"))
		);
		SearchResponse actionGet = requestBuilder.setExplain(true).execute().actionGet();
		SearchHits searchHits =  actionGet.getHits() ;
		SearchHit[] hits = searchHits.getHits();
		System.out.println( "totalHits: " + searchHits.getTotalHits() ); 
		for (SearchHit hit : hits) {
			Map<String, Object> source = hit.getSource() ; 
			if(null != source){
				System.out.println( source.get("materialcode") + "-->" + source.get("materialname"));
			}
		}
		/*---------------------分组 搜索------------------------*/
		SearchRequestBuilder searchRequestBuilder = searchUtils.client.prepareSearch( searchUtils.index )
		.setTypes( searchUtils.type ) .setSearchType(SearchType.DEFAULT
				).setFrom(0)
		.setSize( 100 ) ; 
		
		searchRequestBuilder.setQuery(
				//QueryBuilders.termsQuery("keyword", "FDB" , "鲁达" )
				boolQueryBuilder
				.must(QueryBuilders.matchQuery("keyword", "奥迪"))
		);
		TermsBuilder  partidTermsBuilder = AggregationBuilders.terms("postion").field("relation_postion") ;
		partidTermsBuilder.size( 100 );
		searchRequestBuilder.addAggregation( partidTermsBuilder );
		SearchResponse sr = searchRequestBuilder.execute().actionGet();
		Map<String, Aggregation> aggMap = sr.getAggregations().asMap() ; 
		StringTerms postionTerms = (StringTerms) aggMap.get("postion") ; 
		Iterator<Terms.Bucket> iterator = postionTerms.getBuckets().iterator();
		while(iterator.hasNext()){
			Bucket next = iterator.next();
			System.out.println( next.getKey() + "--->" + next.getDocCount());
		}
	}
}

```



  [1]: http://static.zybuluo.com/Great-Chinese/130dknbwxaldacou2h5z6b50/014.png
  [2]: http://static.zybuluo.com/Great-Chinese/z04ap0koqiaovevn3bwzfr7t/00.png
  [3]: http://static.zybuluo.com/Great-Chinese/9x39tisypkm6owo2elxdzyom/02.png