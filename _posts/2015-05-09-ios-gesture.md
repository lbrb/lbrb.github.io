---
layout: post
title: Gesture 手势
category: 技术
tag: [IOS, Gesture]
description:  
---

### 实例代码

	- (void)viewDidLoad
	{
		[super viewDidLoad];
		// Do any additional setup after loading the view, typically from a nib.
		
		// pan
		UIPanGestureRecognizer *pan = [[UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(pan:)];
	
		
		[_imagView addGestureRecognizer:pan];
	}
	
	- (void)pan:(UIPanGestureRecognizer *)pan
	{
		// 获取手指移动的位置
		CGPoint trans = [pan translationInView:_imagView];
		
		_imagView.transform = CGAffineTransformTranslate(_imagView.transform, trans.x, trans.y);
		
		// 复位
		[pan setTranslation:CGPointZero inView:_imagView];
		NSLog(@"%@",NSStringFromCGPoint(trans));
		
	}
	
	#warning pinch
	- (void)addPinch
	{
		UIPinchGestureRecognizer *pinch = [[UIPinchGestureRecognizer alloc] initWithTarget:self action:@selector(pinch:)];
		// 设置代理的原因：想要同时支持多个手势
		pinch.delegate = self;
		[_imagView addGestureRecognizer:pinch];
		
	}
	
	- (void)pinch:(UIPinchGestureRecognizer *)pinch
	{
		_imagView.transform = CGAffineTransformScale(_imagView.transform, pinch.scale, pinch.scale);
		
		// 复位
		pinch.scale = 1;
	}
	
	// Simultaneous:同时
	// 默认是不支持多个手势
	// 当你使用一个手势的时候就会调用
	- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldRecognizeSimultaneouslyWithGestureRecognizer:(UIGestureRecognizer *)otherGestureRecognizer
	{
		return YES;
	}
	
	
	
	#warning rotation
	- (void)addRotation
	{
		// rotation
		UIRotationGestureRecognizer *rotation = [[UIRotationGestureRecognizer alloc] initWithTarget:self action:@selector(rotation:)];
		rotation.delegate = self;
		[_imagView addGestureRecognizer:rotation];
	}
	
	
	
	- (void)rotation:(UIRotationGestureRecognizer *)rotation
	{
		NSLog(@"%f",rotation.rotation);
	//    _imagView.transform = CGAffineTransformMakeRotation(rotation.rotation);
		_imagView.transform = CGAffineTransformRotate(_imagView.transform, rotation.rotation);
		
		// 复位
		rotation.rotation = 0;
	}
	
	#warning Swipe
	- (void)addSwipe
	{
		// Swipe
		// 一个手势只能识别一个方向
		UISwipeGestureRecognizer *swipe = [[UISwipeGestureRecognizer alloc] initWithTarget:self action:@selector(swipe:)];
		swipe.direction = UISwipeGestureRecognizerDirectionRight;
		[_imagView addGestureRecognizer:swipe];
	}
	
	- (void)swipe:(UISwipeGestureRecognizer *)swipe
	{
		NSLog(@"swipe");
	}
	
	#warning longPress
	- (void)addLongPress
	{
		// longPress
		UILongPressGestureRecognizer *longPress = [[UILongPressGestureRecognizer alloc] initWithTarget:self action:@selector(longPress:)];
		
		[_imagView addGestureRecognizer:longPress];
	}
	
	
	- (void)longPress:(UILongPressGestureRecognizer *)longPress
	{
		// 根据状态执行事件
		if (longPress.state == UIGestureRecognizerStateBegan) {
			
			NSLog(@"longPress");
		}
	}
	
	
	#warning tap
	- (void)addTap
	{
		// tap
		UITapGestureRecognizer *tap = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tap:)];
		// 点按多少次才能触发手势
		//    tap.numberOfTapsRequired = 2;
		//
		//    // 必须多少个手指触摸才能触发手势
		//    tap.numberOfTouchesRequired = 2;
		
		tap.delegate = self;
		
		[_imagView addGestureRecognizer:tap];
	}
	
	// 这个触摸点能否触发手势
	//- (BOOL)gestureRecognizer:(UIGestureRecognizer *)gestureRecognizer shouldReceiveTouch:(UITouch *)touch
	//{
	//    CGPoint currentPoint = [touch locationInView:_imagView];
	//    
	//    if (currentPoint.x < _imagView.bounds.size.width * 0.5) {
	//        return NO;
	//    }else{
	//        
	//        return YES;
	//    }
	//}
	
	- (void)tap:(UITapGestureRecognizer *)tap
	{
		NSLog(@"tap");
	}
	
	- (void)didReceiveMemoryWarning
	{
		[super didReceiveMemoryWarning];
		// Dispose of any resources that can be recreated.
	}
