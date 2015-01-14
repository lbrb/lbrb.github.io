---
layout: post
title: ElasticSearch 介绍
category: 技术
tag: ElasticSearch
description: ElasticSearch 介绍
---

### ElasticSearch介绍 [(Detail)](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/getting-started.html)

- ElasticSearch 是开源搜索平台领域的一个新成员。 ElasticSearch（简称 ES） 是一个基于 Lucene 构建的开源，分布式，RESTful 搜索引擎。 设计用于云计算中，能够达到搜索实时、稳定、可靠和快速，并且安装使用方便。 支持通过 HTTP 请求，使用 JSON 进行数据索引。
- Elasticsearch is a highly scalable open-source full-text search and analytics engine. It allows you to store, search, and analyze big volumes of data quickly and in near real time. It is generally used as the underlying engine/technology that powers applications that have **complex search features and requirements**.

### 基本概念 [(Detail)](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/_basic_concepts.html)
- cluster(集群)
- node(节点)
- index（索引）
- type（类型）
- document（文档）
- shard（片）
- replicas（副本）

### 安装 [(Detail)](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/_installation.html)
- 依赖java

        sudo apt-get install oracle-java8-installer

- 下载ElasticSearch,无须安装解压即可,解压后设置好PATH路径就可以方便的使用了

        curl -L -O https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.2.tar.gz
        tar -xvf elasticsearch-1.4.2.tar.gz
        elasticsearch #elasticsearch --cluster.name my_cluster_name --node.name my_node_name

### 查看ElasticSearch状态 [(Detail)](http://www.elasticsearch.org/guide/en/elasticsearch/guide/current/_cat_api.html)
- 使用_cat API判断启动的ElasticSearch是否可用

        curl 'localhost:9200/_cat/health?v'

        epoch      timestamp cluster       status node.total node.data shards pri relo init unassign
        1421156321 21:38:41  elasticsearch green           1         1      0   0    0    0        0

- 查看集群中的所有节点

        curl 'localhost:9200/_cat/nodes?v'

        host     ip        heap.percent ram.percent load node.role master name
        liangbin 127.0.1.1            4          70 0.11 d         *      Blink

- 查看集群中的所有索引

        curl 'localhost:9200/_cat/indices?v'

        health status index pri rep docs.count docs.deleted store.size pri.store.size

### 开始使用 [(Detail)](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/_create_an_index.html)

- 建立一个索引

        curl -XPUT 'localhost:9200/customer?pretty'

        {
          "acknowledged" : true
        }

        curl 'localhost:9200/_cat/indices?v'

        health status index    pri rep docs.count docs.deleted store.size pri.store.size
        yellow open   customer   5   1          0            0       575b           575b

- Create, GET, DELETE, UPDATE文档

    - Create

        - 指定ID

                curl -XPUT 'localhost:9200/customer/wuliushang/1?pretty' -d '
                {
                "name": "ElasticSearch"
                }'

                {
                  "_index" : "customer",
                  "_type" : "wuliushang",
                  "_id" : "1",
                  "_version" : 2,
                  "created" : false
                }

        - 不指定ID

                curl -XPOST 'localhost:9200/customer/wuliushang?pretty' -d '
                > {
                > "name": "zhangsan"
                > }'

                {
                  "_index" : "customer",
                  "_type" : "wuliushang",
                  "_id" : "AUrj06I8YZdXMvFoCTGg",
                  "_version" : 1,
                  "created" : true
                }


    - GET

            curl -XGET 'localhost:9200/customer/wuliushang/1?pretty'

            {
              "_index" : "customer",
              "_type" : "wuliushang",
              "_id" : "1",
              "_version" : 1,
              "found" : true,
              "_source":
            {
            "name": "liangbin"
            }
            }

    - UPDATE

            curl -XPUT 'localhost:9200/customer/wuliushang/1?pretty' -d'
            {
            "name": "liangbin"
            }'

            {
              "_index" : "customer",
              "_type" : "wuliushang",
              "_id" : "1",
              "_version" : 1,
              "created" : true
            }

    - DELETE

            curl -XDELETE 'localhost:9200/customer/wuliushang/2?pretty'
            {
              "found" : true,
              "_index" : "customer",
              "_type" : "wuliushang",
              "_id" : "2",
              "_version" : 2
            }




