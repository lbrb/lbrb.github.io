---
layout: post
title: Rails 常用资源
category: 资源
tags: Ruby
keywords: Ruby
description: 
---

### active_record优化

        #加到initializers文件夹内
        class << ActiveRecord::Base
          def concerned_with(*concerns)
            concerns.each do |concern|
              require_dependency "#{name.underscore}/#{concern}"
            end
          end
        end

### N+1 query

        #problem
        clients = Client.limit(10)

        clients.each do |client|
          puts client.address.postcode
        end

        #solution
        clients = Client.includes(:address).limit(10)

        clients.each do |client|
          puts client.address.postcode
        end

### 根据文件内容生成sha256,解决文件重复上传问题

        require 'digest'
        Digest::SHA256.file(file_path).hexdigest