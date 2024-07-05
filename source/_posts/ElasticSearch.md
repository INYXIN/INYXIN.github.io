---
title: ElasticSearch
date: 2024-06-13 10:53:54
tags: [ default ]
categories: [ default ]

---

## 安装

### docker 安装



```sh
docker run -d \
        --name es \
        -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
        -e "discovery.type=single-node" \
        -v es-data:/usr/share/elasticsearch/data \
        -v es-plugins:/usr/share/elasticsearch/plugins \
        -v es-logs:/usr/share/elasticsearch/logs \
        --privileged \ --network es-net \
        -p 9200:9200 \
        -p 9300:9300 \
elasticsearch:7.12.1
```

### docker-compose.yml 

```yaml
services:
  es:
    container_name: es
    image: elasticsearch:7.12.1
    environment:
      - "discovery.type=single-node" # 非集群模块
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - "es-data:/usr/share/elasticsearch/data"
      - "es-plugins:/usr/share/elasticsearch/plugins"
      - "es-logs:/usr/share/elasticsearch/logs"
    networks:
      - 1panel-network
networks:
  1panel-network:
    external: true
volumes:
  es-logs:
  es-data:
  es-plugins:
```





### ELK compose

```yml
networks:
  1panel-network:
    external: true
volumes:
  es-logs:
    name: es-log
  es-data:
    name: es-data
  es-plugins:
    name: es-plugins
services:
  elasticsearch:
    container_name: elk_elasticsearch
    image: elasticsearch:7.12.1
    environment:
      - "discovery.type=single-node" # 非集群模块
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - "es-data:/usr/share/elasticsearch/data"
      - "es-plugins:/usr/share/elasticsearch/plugins"
      - "es-logs:/usr/share/elasticsearch/logs"
    networks:
      - 1panel-network
  kibana:
    image: kibana:7.12.1
    container_name: elk_kibana
    restart: always
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    environment:
      - ELASTICSEARCH_URL=http://elasticsearch:9200 #设置访问elasticsearch的地址
    ports:
      - 5601:5601
    networks:
      - 1panel-network
  logstash:
      image: logstash:7.7.0
      container_name: elk_logstash
      restart: always
      volumes:
        - /opt/1panel/docker/compose/elasticsearch/logstash-springboot.conf:/usr/share/logstash/pipeline/logstash.conf #挂载logstash的配置文件
      depends_on:
        - elasticsearch #kibana在elasticsearch启动之后再启动
      links:
        - elasticsearch:es #可以用es这个域名访问elasticsearch服务
      ports:
        - 4560:4560
      networks:
        - 1panel-network
```

### 部署地址:

```
ES : http://172.19.143.22:9200/
Kibanna http://172.19.143.22:5601/app/dev_tools#/console
```

[ES](http://172.19.143.22:9200/)

[KibanaDev Tools - Elastic ](http://172.19.143.22:5601/app/dev_tools#/console)



## IK分词器插件安装

### 下载

[IK分词器插件Tags · infinilabs/analysis-ik (github.com)](https://github.com/infinilabs/analysis-ik/tags?after=v7.12.1)

```
将对于版本的插件 如 7.12.1 版本的ik zip解压 放在 es 的plugins目录下
```

### 分词测试

```json
POST /_analyze
{
	"analyzer":"ik_smart",
	"text":"这个到秦始皇就是非好像是被那种高层吧高层想消灭他，然后就用那个他的软肋，就因为它已经变成掌控了掌控他有个软肋是他婆娘。复生者，对，但是掌控可以影响啊在宾馆里面是吧？嗯，对，但是。会影响他的那个秦始皇然后反正之后就相当于是自爆我靠晕了自己，把自己给灭了。用腿狼。不用推导与他人。自己是秦始皇，然后他本来是是天外的那种修行者然后到地球来寻找突破之法然后地球灵气不够，他传不回去了，然后反正秦始皇那些秦始皇就是它几千年后他被人挖了起来，走了。"
}

```

## 索引库

### Mapping

- type 
  - 字符串 : `text`(可分词)   `keyword`(精确值 , 不可分词, 如IP, 地址, 品牌)
  -  数值 : `long integer short byte  double float number`
  - 布尔 : `boolean`
  - 日期 : `date`
  - 对象 : `object`
- index : 是否创建索引 , 默认值 `true`
- analyzer : 使用哪种分词器
- properties : 该字段的子字段

### 索引库 的CRUD

- 创建索引库

  ```json
  PUT /索引库
  {
      "mappings":{
          "field1":{
              "type":"text",
              "analyzer":"ik_smart"
          }
      }
  }
  ```

  ```json
  PUT /inyxin
  {
      "mappings": {
          "properties": {
              "name": {
                  "type": "keyword"
              },
              "age": {
                  "type": "integer"
              },
              "address": {
                  "properties": {
                      "city": {
                          "type": "text"
                      },
                      "details": {
                          "type": "text"
                      }
                  }
              },
              "description": {
                  "type": "text",
                  "analyzer": "ik_smart"
              }
          }
      }
  }
  
  ```

- 查询索引库

  ```
  GET /索引库
  ```

- 修改索引库 `只支持新增字段 ` 

  如果字段已经存在 , 则会报错

  ```
  PUT /索引库/_mapping
  ```

  ```json
  PUT /inyxin/_mapping
  {
      "properties":{
          "username":{
              "type":"keyword",
              "index":"true"
          }
      }
  }
  ```

- 删除索引库

  ```
  DELETE /索引库
  ```

## 文档的CRUD

- 新增

  ```
  PUT /索引库/_doc/文档ID
  {
  文档数据
  }
  ```

  ```json
  PUT /inyxin/_doc/1
  {
      "name":"inyxin",
      "username":"inyxin",
      "age":"20",
      "description":"确实是描述",
      "address":{
          "city":"四川·达州",
          "details":"四川文理学院·南坝校区"
      }
  }
  ```

  添加成功后响应

  ```json
  {
    "_index" : "inyxin",  索引库名
    "_type" : "_doc",		文档名
    "_id" : "1",			文档ID
    "_version" : 1,
    "result" : "created",
    "_shards" : {
      "total" : 2,
      "successful" : 1,
      "failed" : 0
    },
    "_seq_no" : 0,
    "_primary_term" : 1
  }
  
  ```

- 读取

  ```
  GET /索引库/_doc/文档ID
  ```

  ```
  GET /inyxin/_doc/1
  ```

  查询结果

  `_source` : 原始文档

  ```json
  {
    "_index" : "inyxin",
    "_type" : "_doc",
    "_id" : "1",
    "_version" : 1,
    "_seq_no" : 0,
    "_primary_term" : 1,
    "found" : true,
    "_source" : {
      "name" : "inyxin",
      "username" : "inyxin",
      "age" : "20",
      "description" : "确实是描述",
      "address" : {
        "city" : "四川·达州",
        "details" : "四川文理学院·南坝校区"
      }
    }
  }
  ```

  

- 修改

  - ` 全量修改 (覆盖修改)` 

    若文档ID已存在 , 则会先删除旧的文档, 在进行新增 ,

    否则 就是添加一个新的文档

    ```
    PUT /索引库/_doc/文档ID  同新增
    {}
    ```

  - `增量修改 (局部修改)`

    ```json
    UPDATE  /索引库/_update/文档ID
    {
        "doc":{
            "name":"YINIXN",
            "age":21
        }
    }
    ```

    

  

- 删除

  ```
  DELETE /索引库/_doc/1
  ```

  

