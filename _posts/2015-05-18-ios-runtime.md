---
layout: post
title: ios 元编程
category: 技术
tag: [IOS, 元编程]
description:  
---

### 常用方法积累

	//获得方法列表
	Method *methodList = class_copyMethodList([self class], &methodCount);
	//获得方法名字符串
	NSString *methodName = [NSString stringWithCString:sel_getName(method_getName(methodList[i])) encoding:NSUTF8StringEncoding];
	
	//最后要记得自己释放资源
	free(methodList);