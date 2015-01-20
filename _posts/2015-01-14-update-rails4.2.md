---
layout: post
title: 升级Rials4.2
category: 技术
tag: Ruby
description:
---

- 网站的配置文件建议放在config/initializers，这个文件夹里的rb文件会自动加载，不需要config.autoload_paths自己手动加载。

- 静态文件更改

        #css文件改为.css.erb,并修改其中图片地址为asset方法生成
        url(/images/dotted.png) #rails2
        url(<%= asset_path 'dotted.png'%>) #rails4

        #如果需要编译assets中所有js, css
        # config/application.rb
        config.assets.precompile << Proc.new do |path|
          if path =~ /\.(css|js)\z/
            full_path = Rails.application.assets.resolve(path).to_path
            app_assets_path = Rails.root.join('app', 'assets').to_path
            if full_path.starts_with? app_assets_path
              puts "including asset: " + full_path
              true
            else
              puts "excluding asset: " + full_path
              false
            end
          else
            false
          end
        end

        #关闭实时编译
        #config/environments/production.rb
        config.assets.compile = false

        #本地无法取到public/assets下静态文件
        #config/environments/production.rb
        config.serve_static_files = true

- 全局变量

        #环境变量
        RAILS_ENV #rails2
        Rails.env #rails4

- Scope

        #scope
        named_scope #rails2
        scope       #rails4

        #查询条件
        scope :aaa, :conditions => {:name => "asf"} #rails2
        scope :aaa, -> { where(name: "asf") } #rails4
        scope :created_before, ->(time) { where("created_at < ?", time)} #rails4,带参数scope

        #Merging of scopes
        class User < ActiveRecord::Base
          scope :active, -> { where state: 'active' }
          scope :inactive, -> { where state: 'inactive' }
        end

        User.active.inactive
        # SELECT "users".* FROM "users" WHERE "users"."state" = 'active' AND "users"."state" = 'inactive'

- validation 写法

     更改太多，具体参考[validation](http://guides.rubyonrails.org/active_record_validations.html)

- erb

        #rails4.2默认情况下会转移html,禁用使用raw(html_safe,这两个时一样的)，建议使用sanitize,这个可以自定义白名单，更灵活，也比前面两个安全

        # Marks a string as trusted safe. It will be inserted into HTML with no
        # additional escaping performed. It is your responsibilty to ensure that the
        # string contains no malicious content. This method is equivalent to the
        # `raw` helper in views. It is recommended that you use `sanitize` instead of
        # this method. It should never be called on user input.

        <%= "<" %> # => '&lt;' rails 默认会将html标签转义
        <%=raw "<" %> # => '<'     关闭rails安全机制，不使用转义
        <%= "<".html_safe %> # => '<' 同上
        <%=sanitize "<".html_safe %> # => '<' 推荐使用这个。


        #form也要加=
        <% form_for .. %> rails2
        <%= form_for .. %> rails4

- spring gem

    可以提高速度说的，但会碰到rails console打不开的情况，可以关闭spring,在运行就ok了

        bin/spring stop #关闭
        bin/spring status #查看状态

- each 方法

        '123'.each { |e| p e} #rails2 可以执行
        '123'.each { |e| p e} #rails4 不可以
        '123'.split.each { |e| p e} #rails4 解决办法

- includes方法

        Company.includes(:user).references(:user).where('users.id=1') # 用到join表中的查询时需要将表references


        #查询方案也会不同
        Company.includes(:user).where(id: 1)
        #sql: select companies.* from companies where companies.id =1
              select users.* from users where id in (1)

        Company.includes(:user).where('users.id =1').references(:user)
        #sql: select companies.*, users.* from companies left out join users on companies.user_id = users.id where users.id = 1

- Can't mass-assign protected attributes for

        #这个是rails4更严格造成的
        #解决办法
        #第一种方法（比较简单）：
        #添加protected_attributes gem
        #在model中写清更改的字段：attr_accessible :key, :owner
        #第二种方法（rails4方案）：
        #permit(:name, :etc, :etc)