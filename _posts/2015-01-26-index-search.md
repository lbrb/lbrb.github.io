---
layout: post
title: Index Search
category: 技术
tag: [Search]
description: Index Search
---

### 简单的索引搜索实现

       @index = {}

       # tokenize('what is your name.') => ["what", "is", "your", "name"]

       def tokenize(text)
         text.gsub(/&/, 'and').               # Synonym substitution
              gsub(/[^\w\s]/,'').             # Remove illegal expressions
              gsub(/\b[a|an|is]\b/i, '').     # Remove stopwords
              split(/\s+/).map do |token|     # Split the text into words
                token.downcase                # Downcase each token
              end
       end

       # docs = {
       #    0 => 'what is your name?',
       #    1 => 'my name is liangbin.',
       #    2 => 'hi, liangbin.'
       # }
       # create_index(docs) => {"what"=>[0], "is"=>[0, 1], "your"=>[0], "name"=>[0, 1], "my"=>[1], "liangbin"=>[1, 2], "hi"=>[2]}

       def create_index(docs)
          docs.each do |id, text|
            tokenize(text).map do |word|
              if @index.has_key? word
                @index[word] << id
              else
                @index[word] = [id]
              end
            end
          end
       end

       #search('liangbin is') => [1]

       def search(text)
         tokenize(text).map do |word|
           @index[word] || []
         end.reduce(:&)
       end

       #使用方法
       docs = {
           0 => 'what is your name?',
           1 => 'my name is liangbin.',
           2 => 'hi, liangbin.'
       }
       create_index(docs)                     #创建索引
       search('liangbin is')                     #搜索
       #output
       #[1]

[参考](http://shopperplus.github.io/blog/2014/12/28/how-search-and-index-works.html)