---
layout: post
title: IOS NSOperation
category: 技术
tag: [IOS, NSOperation]
description:  
---

### 简单使用
		// 1.创建一个队列（非主队列）
		NSOperationQueue *queue = [[NSOperationQueue alloc] init];
		
		// 2.添加操作到队列中（自动异步执行任务，并发）
		NSBlockOperation *operation1 = [NSBlockOperation blockOperationWithBlock:^{
			NSLog(@"下载图片1---%@", [NSThread currentThread]);
		}];
		NSBlockOperation *operation2 = [NSBlockOperation blockOperationWithBlock:^{
			NSLog(@"下载图片2---%@", [NSThread currentThread]);
		}];
		
		[queue addOperation:operation1];
		[queue addOperation:operation2];
		[queue addOperationWithBlock:^{
			NSLog(@"下载图片3---%@", [NSThread currentThread]);
		}];
		
### 设置最大并发数

			// 1.创建一个队列（非主队列）
			NSOperationQueue *queue = [[NSOperationQueue alloc] init];
			
			// 2.设置最大并发(最多同时并发执行3个任务)
			queue.maxConcurrentOperationCount = 3;

### 设置依赖

		// 设置依赖
		[operationB addDependency:operationA];
		[operationC addDependency:operationB];
		
### 回到主线程

		// 1.异步下载图片
        NSURL *url = [NSURL URLWithString:@"http://d.hiphotos.baidu.com/image/pic/item/37d3d539b6003af3290eaf5d362ac65c1038b652.jpg"];
        NSData *data = [NSData dataWithContentsOfURL:url];
        UIImage *image = [UIImage imageWithData:data];
        
        // 2.回到主线程，显示图片
        [self performSelectorOnMainThread:<#(SEL)#> withObject:<#(id)#> waitUntilDone:<#(BOOL)#>];
        dispatch_async(dispatch_get_main_queue(), ^{
            
        });
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
            self.imageView.image = image;
        }];
		
### 自定义operation


		- (void)main//operation添加到队列后，会执行main方法
		{
			@autoreleasepool {//需要自己写自动释放池
				if (self.isCancelled) return; //响应用户取消操作
				
				NSURL *url = [NSURL URLWithString:self.imageUrl];
				NSData *data = [NSData dataWithContentsOfURL:url]; // 下载
				UIImage *image = [UIImage imageWithData:data]; // NSData -> UIImage
				
				if (self.isCancelled) return; //响应用户取消操作
				
				// 回到主线程
				[[NSOperationQueue mainQueue] addOperationWithBlock:^{
					if ([self.delegate respondsToSelector:@selector(downloadOperation:didFinishDownload:)]) {
						[self.delegate downloadOperation:self didFinishDownload:image];
					}
				}];
			}
		}