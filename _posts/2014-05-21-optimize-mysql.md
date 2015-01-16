---
layout: post
title: MySQL 优化
category: 技术
tags: MySQL
keywords: MySQL
description: 
---

### 整体查看mysql状态(Innodb存储引擎)

     show status

   - Com_select  执行select操作的次数，一次查询只累加1；
   - Com_insert 执行insert操作的次数，对于批量插入的insert操作，只累加一次；
   - Com_update 执行update操作的次数；
   - Com_delete　执行delete操作的次数。
   - Innodb_rows_read　select查询返回的行数；
   - Innodb_rows_inserted执行Insert操作插入的行数；
   - Innodb_rows_updated 执行update操作更新的行数；
   - Innodb_rows_deleted　执行delete操作删除的行数。

###单条语句查看执行情况

     explain mysql_str

   总的来说，where，order等语句中用到的字段尽量建索引（unique索引最好），参考[Mysql Explain 详解[强烈推荐]](http://www.cnitblog.com/aliyiyi08/archive/2008/09/09/48878.html)
