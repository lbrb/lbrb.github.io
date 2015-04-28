---
layout: post
title: IOS SQLite3
category: 技术
tag: [IOS, SQLite3]
description:  
---


#Demo
[SQLite3DBSample](https://github.com/lbrb/SQLite3DBSample)(sqlite3原生方法实现)。

TM项目，目前的trunk版本使用的是FMDB，具体源码参考Database.h, Database.m文件。

#SQLite

###为什么选择SQLite
轻量，适合手机有限的资源。
功能不逊色于其他数据库。
可靠，速度快。
官方支持，无论是iOS，还是Mac OS,都支持sqlite。
强大的社区。
###SQLite3 命令行常用命令
	#进入sqlite3命令行模式
	sqlite3 filename
	#创建表
	create table peopleInfo(peopleInfoID integer primary key, firstname text, lastname text, age integer);
	#查看数据库中的所有表
	.tables
	#查看某张表的信息
	.schema TABLE_NAME
	#退出sqlite3
	.quit
	
###SQLite3库（iOS中使用的是libsqlit3.dylib)中常用方法简介
- sqlite3_open:打开或者创建一个数据库文件，
SQLITE_API int sqlite3_open(
  const char *filename,   /* Database filename (UTF-8) */
  sqlite3 **ppDb          /* OUT: SQLite db handle */
);

- sqlite3_prepare_v2:将字符串SQL转化为sqlite3可以执行的格式。
SQLITE_API int sqlite3_prepare_v2(
  sqlite3 *db,            /* Database handle */
  const char *zSql,       /* SQL statement, UTF-8 encoded */
  int nByte,              /* Maximum length of zSql in bytes. */
  sqlite3_stmt **ppStmt,  /* OUT: Statement handle */
  const char **pzTail     /* OUT: Pointer to unused portion of zSql */

- sqlite3_step:执行上面的方法生成的可执行的sql语句。
int  sqlite3_step(
sqlite3_stmt       /* 转化后的可执行的语句 */ 
);

- sqlite3_column_count：返回表中字段的个数。
int sqlite3_column_count(
sqlite3_stmt *pStmt  / *转化后的可执行的语句*/ 
);

- sqlite3_column_text：返回字段对应的值。
const unsigned char *sqlite3_column_text(
sqlite3_stmt*, / *转化后的可执行的语句*/ 
int iCol       / *字段的下标*/ 
);

- sqlite3_column_name：返回字段的名字。
const char *sqlite3_column_name(
sqlite3_stmt*, / *转化后的可执行的语句*/ 
 int N           / *字段对应的下标*/ 
);

- sqlite3_changes：影响的行数。
int sqlite3_changes(
sqlite3  / *数据库*/ 
);

- sqlite3_last_insert_rowid：返回最后一行的ID
sqlite3_int64 sqlite3_last_insert_rowid(
sqlite3＊ / *数据库*/ 
);

- sqlite3_errmsg：出错时的错误信息
const char *sqlite3_errmsg(
sqlite3＊   / *数据库*/  
);

- sqlite3_finalize：释放内存
int sqlite3_finalize(
sqlite3_stmt *pStmt   / *转化后的可执行的语句*/ 
);

- sqlite3_close: 关闭数据库链接
int sqlite3_close(
sqlite3＊  / *数据库*/   
);
###在IOS中使用SQLite3原生方法
- 选择自己的app项目，在General标签下，找到Linked FrameWorks and Libraries, 添加libsqlit3.dylib库。
- 具体的代码就不粘贴在这里了，即占空间，看着也让人头疼。可以直接查看我写的一个demoSQLite3DBSample，更容易理解。
- 注意点：
	- 只有document这个目录才有读写权限，所以我们要把数据库文件从我们的app目录复制到document目录下。
	- 数据库使用常识，使用完毕后记得释放内存和数据库连接（sqlite3_finalize，sqlite3_close）。
	- 每个步骤，基本都会有个判断成功与否的返回值，需要注意的是，这个返回值是int类型的，不是BOOL，更要注意的是成功时返回的是0，如果直接放在if中判断，”恭喜你“得到的结果刚好是相反的。
#IOS中SQLite3类库FMDB（github地址）

###FMDB简介
FMDB是对SQLite API进行封装的库。并且不需要自己手动控制内存管理。

使用起来也很方便，下面是一个简单的例子：

	NSString* docsdir = [NSSearchPathForDirectoriesInDomains( NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
	NSString* dbpath = [docsdir stringByAppendingPathComponent:@"user.sqlite"];
	FMDatabase* db = [FMDatabase databaseWithPath:dbpath];
	[db open];
	FMResultSet *rs = [db executeQuery:@"select * from people"];
	while ([rs next]) {
    NSLog(@"%@ %@",
        [rs stringForColumn:@"firstname"],
        [rs stringForColumn:@"lastname"]);
	}
	[db close];
我们使用的类主要有三个

- FMDatabase -   连接数据库并执行SQL。
- FMResultSet -   执行查询语句后，保存查询结果的地方。
- FMDatabaseQueue -  
###FMDB 使用说明
#####引入相关文件
	FMDatabase.h
	FMDatabase.m
	FMDatabaseAdditions.h
	FMDatabaseAdditions.m
	FMDatabasePool.h
	FMDatabasePool.m
	FMDatabaseQueue.h
	FMDatabaseQueue.m
	FMResultSet.h
	FMResultSet.m
#####建立数据库
建立数据库只需要提供数据库文件路径即可。如果你传入的参数是空串:@"",FMDB就会在临时文件目录下创建这个数据库；如果你传入的参数是NULL，则它会建立一个在内存中的数据库。

	FMDatabase *db = [FMDatabase databaseWithPath:@"/tmp/tmp.db"];
#####执行更新操作
这里需要搞清楚一个概念。除了Select操作之外，其他的都是更新操作。

	- (FMResultSet *)executeQuery:(NSString*)sql, ... ;
#####执行查询操作
注意：无论返回结果有几行，都要执行FMResultSet的next方法。

	FMResultSet *s = [db executeQuery:@"SELECT * FROM myTable"];
	while ([s next]) {
	    //retrieve values for each record
	}
	FMResultSet *s = [db executeQuery:@"SELECT COUNT(*) FROM myTable"];
	if ([s next]) {
	    int totalCount = [s intForColumnIndex:0];
	}
其他获取不同类型的数据的方法：

	intForColumn:
	longForColumn:
	longLongIntForColumn:
	boolForColumn:
	doubleForColumn:
	stringForColumn:
	dateForColumn:
	dataForColumn:
	dataNoCopyForColumn:
	UTF8StringForColumnIndex:
	objectForColumn:
通常情况下，我们并不需要清空FMResultSet。因为当我们关闭数据库连接时，FMResultSet会自动清空。

#####数据参数
通常情况下，你可以按照标准的SQL语句，用?表示执行语句的参数，如

	INSERT INTO myTable VALUES (?, ?, ?)
然后，可以我们可以调用executeUpdate方法来将?所指代的具体参数传入，通常是用变长参数来传递进去的，如下：

	NSString *sql = @"insert into User (name, password) values (?, ?)";
	[db executeUpdate:sql, user.name, user.password];
这里需要注意的是，参数必须是NSObject的子类，所以象int,double,bool这种基本类型，需要封装成对应的包装类才行，如下所示：

	// 错误，42不能作为参数
	[db executeUpdate:@"INSERT INTO myTable VALUES (?)", 42];
	// 正确，将42封装成 NSNumber 类
	[db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:42]];
###线程安全（TM目前好像还没用到）
如果我们的app需要多线程操作数据库，那么就需要使用FMDatabaseQueue来保证线程安全了。 切记不能在多个线程中共同一个FMDatabase对象并且在多个线程中同时使用，这个类本身不是线程安全的，这样使用会造成数据混乱等问题。

使用FMDatabaseQueue很简单，首先用一个数据库文件地址来初使化FMDatabaseQueue，然后就可以将一个闭包(block)传入inDatabase方法中。 在闭包中操作数据库，而不直接参与FMDatabase的管理。

	// 创建，最好放在一个单例的类中
	FMDatabaseQueue *queue = [FMDatabaseQueue databaseQueueWithPath:aPath];
	// 使用
	[queue inDatabase:^(FMDatabase *db) {
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:1]];
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:2]];
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:3]];
	    FMResultSet *rs = [db executeQuery:@"select * from foo"];
	    while ([rs next]) {
	        // …
	    }
	}];
	// 如果要支持事务
	[queue inTransaction:^(FMDatabase *db, BOOL *rollback) {
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:1]];
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:2]];
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:3]];
	    if (whoopsSomethingWrongHappened) {
	        *rollback = YES;
	        return;
	    }
	    // etc…
	    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", [NSNumber numberWithInt:4]];
	}];








