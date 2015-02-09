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

### rails4.2&ruby2.1圣经笔记

- development环境每次修改后可以在浏览器查看到结果的原因是development使用load来加载，每次都会加载，production环境使用require加载，只会加载一次，所以production必须重启才能看到变化。
- Gemfile 中版本号“～> x.y.z”的意思是，只升级z.例如：～>2.1.2,会升级到2.1.3，但不会升级到2.2.1。这样做可以减少bug但又不会产生兼容性问题（通常版號的命名有其慣例：x major 版號升級表示有 API 發生不向後的相容性變動，y minor 版號升級表示有功能新增，z tiny 版號升級表示 bugs 修正。）。bundle install 只会做必要的更新到Gemfile.lock,速度比较快，一般我们使用bundle install 就可以了。bundle update 会更新所有的gem包并重新生成Gemfile.lock。

      #可以列举出可以升级的gem
      bundle outdated

      #将用到的gem包打包到 vendor/cache 目录，再执行bundle install 时就不会到网上下载了
      bundle package

      #绑定正确的环境
      bundle exec

- Migration 加入 :bulk => true 优化migration执行效率，会将多个操作合并成一句sql执行。

- html_safe

  将一个html_safe 与 字符串 结合时，新的字符串是html_safe状态，但加入的字符串还是会做html转译

        "".html_safe + "<" # => "&lt;"
        "".html_safe + "<".html_safe # => "<" 如果是 html_safe 的內容則不會作逸出碼的處理

- ActiveRecord Touch 屬性

被當作快取Key的ActiveRecord物件的最後更新時間updated_at，在一對一或一對多的關係中，預設並不會根據底下的物件而自動更新。例如以下的例子中，如果有新的attendee進來，並不會自動更新該event的最後更新時間，會導致這整個快取不會被更新到。
      
      <% cache event do %>
        <%= event.name %>
        <%= event.attendees.last.try(:name) %>
      <% end %>
      
解決的辦法是使用Touch屬性：
    
    class Attendee < ActiveRecord::Base
      belongs_to :event, :touch => true
      # ...
    end