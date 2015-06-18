---
layout: post
title: IOS 文件上传下载
category: 技术
tag: [IOS, Http]
description:  
---

### NSURLConnection

#### 下载文件

##### 单线程下载

		//开始下载
		- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
		{
			// 1.URL
			NSURL *url = [NSURL URLWithString:@"http://localhost:8080/MJServer/resources/music.zip"];
			
			// 2.请求
			NSURLRequest *request = [NSURLRequest requestWithURL:url];
			
			// 3.下载(创建完conn对象后，会自动发起一个异步请求)
			[NSURLConnection connectionWithRequest:request delegate:self];
		}
		//代理
		/**
		*  1.接收到服务器的响应就会调用
		*
		*  @param response   响应
		*/
		- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
		{
			// 文件路径
			NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
			NSString *filepath = [caches stringByAppendingPathComponent:@"videos.zip"];
			
			// 创建一个空的文件 到 沙盒中
			NSFileManager *mgr = [NSFileManager defaultManager];
			[mgr createFileAtPath:filepath contents:nil attributes:nil];
			
			// 创建一个用来写数据的文件句柄
			self.writeHandle = [NSFileHandle fileHandleForWritingAtPath:filepath];
			
			// 获得文件的总大小
			self.totalLength = response.expectedContentLength;
		}
		
		/**
		*  2.当接收到服务器返回的实体数据时调用（具体内容，这个方法可能会被调用多次）
		*
		*  @param data       这次返回的数据
		*/
		- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
		{
			// 移动到文件的最后面
			[self.writeHandle seekToEndOfFile];
			
			// 将数据写入沙盒
			[self.writeHandle writeData:data];
			
			// 累计文件的长度
			self.currentLength += data.length;
			
			NSLog(@"下载进度：%f", (double)self.currentLength/ self.totalLength);
		}
		
		/**
		*  3.加载完毕后调用（服务器的数据已经完全返回后）
		*/
		- (void)connectionDidFinishLoading:(NSURLConnection *)connection
		{
			self.currentLength = 0;
			self.totalLength = 0;
			
			// 关闭文件
			[self.writeHandle closeFile];
			self.writeHandle = nil;
		}
		// 断点续传
		- (IBAction)download:(UIButton *)sender {
			// 状态取反
			sender.selected = !sender.isSelected;
			
			// 断点下载
			
			if (sender.selected) { // 继续（开始）下载
				// 1.URL
				NSURL *url = [NSURL URLWithString:@"http://localhost:8080/MJServer/resources/videos.zip"];
				
				// 2.请求
				NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
				
				// 设置请求头
				NSString *range = [NSString stringWithFormat:@"bytes=%lld-", self.currentLength];
				[request setValue:range forHTTPHeaderField:@"Range"];
				
				// 3.下载(创建完conn对象后，会自动发起一个异步请求)
				self.conn = [NSURLConnection connectionWithRequest:request delegate:self];
			} else { // 暂停
				[self.conn cancel];
				self.conn = nil;
			}
		}
		
##### 多线程下载

	首先创建一个总大小的文件，然后每个线程下载不同的部分
	
### NSURLSession

// 任务：任何请求都是一个任务
// NSURLSessionDataTask : 普通的GET\POST请求
// NSURLSessionDownloadTask : 文件下载
// NSURLSessionUploadTask : 文件上传

#### 普通请求

		// 1.得到session对象
		NSURLSession *session = [NSURLSession sharedSession];
		
		// 2.创建一个task，任务
		//    NSURL *url = [NSURL URLWithString:@"http://localhost:8080/MJServer/video"];
		//    NSURLSessionDataTask *task = [session dataTaskWithURL:url completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
		//        NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableLeaves error:nil];
		//        NSLog(@"----%@", dict);
		//    }];
		
		NSURL *url = [NSURL URLWithString:@"http://192.168.15.172:8080/MJServer/login"];
		
		// 创建一个请求
		NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
		request.HTTPMethod = @"POST";
		// 设置请求体
		request.HTTPBody = [@"username=123&pwd=123" dataUsingEncoding:NSUTF8StringEncoding];
		
		NSURLSessionDataTask *task = [session dataTaskWithRequest:request completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
			NSDictionary *dict = [NSJSONSerialization JSONObjectWithData:data options:NSJSONReadingMutableLeaves error:nil];
			NSLog(@"----%@", dict);
		}];
		
		// 3.开始任务
		[task resume];

#### 下载文件

##### 简单下载

		// 1.得到session对象
		NSURLSession *session = [NSURLSession sharedSession];
		
		// 2.创建一个下载task
		NSURL *url = [NSURL URLWithString:@"http://localhost:8080/MJServer/resources/test.mp4"];
		NSURLSessionDownloadTask *task = [session downloadTaskWithURL:url completionHandler:^(NSURL *location, NSURLResponse *response, NSError *error) {
			// location : 临时文件的路径（下载好的文件）
			
			NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
			// response.suggestedFilename ： 建议使用的文件名，一般跟服务器端的文件名一致
			NSString *file = [caches stringByAppendingPathComponent:response.suggestedFilename];
			
			// 将临时文件剪切或者复制Caches文件夹
			NSFileManager *mgr = [NSFileManager defaultManager];
			
			// AtPath : 剪切前的文件路径
			// ToPath : 剪切后的文件路径
			[mgr moveItemAtPath:location.path toPath:file error:nil];
		}];
		
		// 3.开始任务
		[task resume];
	
##### 需要显示下载进度

		- (void)downloadTask2
		{
			NSURLSessionConfiguration *cfg = [NSURLSessionConfiguration defaultSessionConfiguration];
			
			// 1.得到session对象
			NSURLSession *session = [NSURLSession sessionWithConfiguration:cfg delegate:self delegateQueue:[NSOperationQueue mainQueue]];
			
			// 2.创建一个下载task
			NSURL *url = [NSURL URLWithString:@"http://localhost:8080/MJServer/resources/test.mp4"];
			NSURLSessionDownloadTask *task = [session downloadTaskWithURL:url];
			
			// 3.开始任务
			[task resume];
			
			// 如果给下载任务设置了completionHandler这个block，也实现了下载的代理方法，优先执行block
		}
		
		#pragma mark - NSURLSessionDownloadDelegate
		/**
		*  下载完毕后调用
		*
		*  @param location     临时文件的路径（下载好的文件）
		*/
		- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location
		{
			// location : 临时文件的路径（下载好的文件）
			
			NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
			// response.suggestedFilename ： 建议使用的文件名，一般跟服务器端的文件名一致
			
			NSString *file = [caches stringByAppendingPathComponent:downloadTask.response.suggestedFilename];
			
			// 将临时文件剪切或者复制Caches文件夹
			NSFileManager *mgr = [NSFileManager defaultManager];
			
			// AtPath : 剪切前的文件路径
			// ToPath : 剪切后的文件路径
			[mgr moveItemAtPath:location.path toPath:file error:nil];
		}
		
		/**
		*  恢复下载时调用
		*/
		- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didResumeAtOffset:(int64_t)fileOffset expectedTotalBytes:(int64_t)expectedTotalBytes
		{
		
		}
		
		/**
		*  每当下载完（写完）一部分时就会调用（可能会被调用多次）
		*
		*  @param bytesWritten              这次调用写了多少
		*  @param totalBytesWritten         累计写了多少长度到沙盒中了
		*  @param totalBytesExpectedToWrite 文件的总长度
		*/
		- (void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
		{
			double progress = (double)totalBytesWritten / totalBytesExpectedToWrite;
			NSLog(@"下载进度---%f", progress);
		}
		
##### 断点续传

		- (void)start
		{
			// 1.创建一个下载任务
			NSURL *url = [NSURL URLWithString:@"http://192.168.15.172:8080/MJServer/resources/videos/minion_01.mp4"];
			self.task = [self.session downloadTaskWithURL:url];
			
			// 2.开始任务
			[self.task resume];
		}
		
		/**
		*  恢复（继续）
		*/
		- (void)resume
		{
			// 传入上次暂停下载返回的数据，就可以恢复下载
			self.task = [self.session downloadTaskWithResumeData:self.resumeData];
			
			// 开始任务
			[self.task resume];
			
			// 清空
			self.resumeData = nil;
		}
		
		/**
		*  暂停
		*/
		- (void)pause
		{
			__weak typeof(self) vc = self;
			[self.task cancelByProducingResumeData:^(NSData *resumeData) {
				//  resumeData : 包含了继续下载的开始位置\下载的url
				vc.resumeData = resumeData;
				vc.task = nil;
			}];
		}