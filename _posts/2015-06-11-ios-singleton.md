---
layout: post
title: IOS Singleton
category: 技术
tag: [IOS, Singleton]
description:  
---

### 懒汉式

		static id _instance;
		
		/**
		*  alloc方法内部会调用这个方法
		*/
		+ (id)allocWithZone:(struct _NSZone *)zone
		{
			if (_instance == nil) { // 防止频繁加锁
				@synchronized(self) {
					if (_instance == nil) { // 防止创建多次
						_instance = [super allocWithZone:zone];
					}
				}
			}
			return _instance;
		}
		
		+ (instancetype)sharedMusicTool
		{
			if (_instance == nil) { // 防止频繁加锁
				@synchronized(self) {
					if (_instance == nil) { // 防止创建多次
						_instance = [[self alloc] init];
					}
				}
			}
			return _instance;
		}
		
		- (id)copyWithZone:(NSZone *)zone
		{
			return _instance;
		}

### 饿汉式

		static id _instance;
		
		/**
		*  当类加载到OC运行时环境中（内存），就会调用一次（一个类只会加载1次）
		*/
		+ (void)load
		{
			_instance = [[self alloc] init];
		}
		
		+ (id)allocWithZone:(struct _NSZone *)zone
		{
			if (_instance == nil) { // 防止创建多次
				_instance = [super allocWithZone:zone];
			}
			return _instance;
		}
		
		+ (instancetype)sharedSoundTool
		{
			return _instance;
		}
		
		- (id)copyWithZone:(NSZone *)zone
		{
			return _instance;
		}
		
		///**
		// *  当第一次使用这个类的时候才会调用
		// */
		//+ (void)initialize
		//{
		//    NSLog(@"HMSoundTool---initialize");
		//}

### GCD

		// 用来保存唯一的单例对象
		static id _instace;
		
		+ (id)allocWithZone:(struct _NSZone *)zone
		{
			static dispatch_once_t onceToken;
			dispatch_once(&onceToken, ^{
				_instace = [super allocWithZone:zone];
			});
			return _instace;
		}
		
		+ (instancetype)sharedDataTool
		{
			static dispatch_once_t onceToken;
			dispatch_once(&onceToken, ^{
				_instace = [[self alloc] init];
			});
			return _instace;
		}
		
		- (id)copyWithZone:(NSZone *)zone
		{
			return _instace;
		}
