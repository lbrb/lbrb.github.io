---
layout: post
title: ElasticSearch ROR 实战
category: 技术
tag: [ElasticSearch, Ruby]
description: ElasticSearch ROR 实战
---

### 添加相应gem包

        # Elasticsearch
        gem "elasticsearch"
        gem "elasticsearch-model"
        gem "elasticsearch-rails"


### 在model中使用elasticsearch 实例

        class Article < ActiveRecord::Base
          include Elasticsearch::Model #增加ES相关的方法
          include Elasticsearch::Model::Callbacks # 当article变化时调用回调函数更改ES中内容
        end

        # 将目前的article导入到ES
        Article.import

        @articles = Article.search('foobar').records

### 使用IK中文分词

   编译elasticsearch-analysis-ik

        #clone 最新的ik项目并编译
        git clone https://github.com/medcl/elasticsearch-analysis-ik.git --depth=1
        cd elasticsearch-analysis-ik

        #如果没有安装maven，执行 sudo apt-get install maven
        maven package

   配置ES，三步就搞定了；

   - 拷贝jar,maven执行完毕后会在当前目录下生成target/releases目录，将其中的elasticsearch-analysis-ik-1.2.9.zip 拷贝到ES目录下的plugins/analysis-ik，没有这个目录并解压可以自己建。
   - 拷贝辞典将elasticsearch-analysis-ik目录中的config/ik 拷贝到ES的config目录。
   - 修改ES的配置文件，指定IK为分词工具，到ES目录下，打开config/elasticsearch.yml,在最后添加

         index.analysis.analyzer.ik.type : "ik"

