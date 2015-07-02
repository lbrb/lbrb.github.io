---
layout: post
title: IOS CoreLocation
category: 技术
tag: [IOS, CoreLocation]
description:  
---

### CoreLocation基本使用

	- (void)viewDidLoad {
	    [super viewDidLoad];
	    // Do any additional setup after loading the view, typically from a nib.
    
	    // 1.创建CoreLocation管理者
	//    CLLocationManager *mgr = [[CLLocationManager alloc] init];
    
	    // 2.成为CoreLocation管理者的代理监听获取到的位置
	    self.mgr.delegate = self;
    
	    // 设置多久获取一次
	//    self.mgr.distanceFilter = 500;
    
	    // 设置获取位置的精确度
	    /*
	      kCLLocationAccuracyBestForNavigation 最佳导航
	      kCLLocationAccuracyBest;  最精准
	      kCLLocationAccuracyNearestTenMeters;  10米
	      kCLLocationAccuracyHundredMeters;  百米
	      kCLLocationAccuracyKilometer;  千米
	      kCLLocationAccuracyThreeKilometers;  3千米
	     */
	//    self.mgr.desiredAccuracy = kCLLocationAccuracyNearestTenMeters;
    
    
	    /*
	     注意: iOS7只要开始定位, 系统就会自动要求用户对你的应用程序授权. 但是从iOS8开始, 想要定位必须先"自己""主动"要求用户授权
	      在iOS8中不仅仅要主动请求授权, 而且必须再info.plist文件中配置一项属性才能弹出授权窗口
	     NSLocationWhenInUseDescription，允许在前台获取GPS的描述
	     NSLocationAlwaysUsageDescription，允许在后台获取GPS的描述
	    */
    
	    // 判断是否是iOS8
	    if([[UIDevice currentDevice].systemVersion doubleValue] >= 8.0)
	    {
	        NSLog(@"是iOS8");
	        // 主动要求用户对我们的程序授权, 授权状态改变就会通知代理
	        //
	        [self.mgr requestAlwaysAuthorization]; // 请求前台和后台定位权限
	//        [self.mgr requestWhenInUseAuthorization];// 请求前台定位权限
	    }else
	    {
	        NSLog(@"是iOS7");
	        // 3.开始监听(开始获取位置)
	        [self.mgr startUpdatingLocation];
	    }
    
	}
	/**
	 *  授权状态发生改变时调用
	 *
	 *  @param manager 触发事件的对象
	 *  @param status  当前授权的状态
	 */
	- (void)locationManager:(CLLocationManager *)manager didChangeAuthorizationStatus:(CLAuthorizationStatus)status
	{
	    /*
	     用户从未选择过权限
	     kCLAuthorizationStatusNotDetermined
	     无法使用定位服务，该状态用户无法改变
	     kCLAuthorizationStatusRestricted
	     用户拒绝该应用使用定位服务，或是定位服务总开关处于关闭状态
	     kCLAuthorizationStatusDenied
	     已经授权（废弃）
	     kCLAuthorizationStatusAuthorized
	     用户允许该程序无论何时都可以使用地理信息
	     kCLAuthorizationStatusAuthorizedAlways
	     用户同意程序在可见时使用地理位置
	     kCLAuthorizationStatusAuthorizedWhenInUse
	     */
    
	    if (status == kCLAuthorizationStatusNotDetermined) {
	        NSLog(@"等待用户授权");
	    }else if (status == kCLAuthorizationStatusAuthorizedAlways ||
	              status == kCLAuthorizationStatusAuthorizedWhenInUse)
        
	    {
	        NSLog(@"授权成功");
	        // 开始定位
	        [self.mgr startUpdatingLocation];
        
	    }else
	    {
	        NSLog(@"授权失败");
	    }
	}
	#pragma mark - CLLocationManagerDelegate
	//- (void)locationManager:(CLLocationManager *)manager didUpdateToLocation:(CLLocation *)newLocation fromLocation:(CLLocation *)oldLocation
	/**
	 *  获取到位置信息之后就会调用(调用频率非常高)
	 *
	 *  @param manager   触发事件的对象
	 *  @param locations 获取到的位置
	 */
	- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations
	{
	    NSLog(@"%s", __func__);
	    // 如果只需要获取一次, 可以获取到位置之后就停止
	//    [self.mgr stopUpdatingLocation];
    
	}

### CoreLocation 计算距离速度等

	//    CLLocation; timestamp 当前获取到为止信息的时间
	    /*
	     获取走了多远（这一次的位置 减去上一次的位置）
	     获取走这段路花了多少时间 （这一次的时间 减去上一次的时间）
	     获取速度 （走了多远 ／ 花了多少时间）
	     获取总共走的路程 （把每次获取到走了多远累加起来）
	     获取平均速度 （用总路程 ／ 总时间）
	     */
	    // 获取当前的位置
	    CLLocation *newLocation = [locations lastObject];
    
	    if (self.previousLocation != nil) {
	        // 计算两次的距离(单位时米)
	        CLLocationDistance distance = [newLocation distanceFromLocation:self.previousLocation];
	        // 计算两次之间的时间（单位只秒）
	        NSTimeInterval dTime = [newLocation.timestamp timeIntervalSinceDate:self.previousLocation.timestamp];
	        // 计算速度（米／秒）
	        CGFloat speed = distance / dTime;
        
        
	        // 累加时间
	        self.sumTime += dTime;
        
	        // 累加距离
	        self.sumDistance += distance;
        
	        //  计算平均速度
	        CGFloat avgSpeed = self.sumDistance / self.sumTime;
        
	        NSLog(@"距离%f 时间%f 速度%f 平均速度%f 总路程 %f 总时间 %f", distance, dTime, speed, avgSpeed, self.sumDistance, self.sumTime);
	    }
    
	    // 纪录上一次的位置
	    self.previousLocation = newLocation;

### CoreLocation 获取方向

	- (void)viewDidLoad {
	    [super viewDidLoad];
    
	    // 1.添加指南针图片
    
	    UIImageView *iv = [[UIImageView alloc] initWithImage: [UIImage imageNamed:@"bg_compasspointer"]];
	    iv.center = CGPointMake(self.view.center.x, self.view.center.y);
	    [self.view addSubview:iv];
	    self.compasspointer = iv;
    
	    // 2.成为CoreLocation管理者的代理监听获取到的位置
	    self.mgr.delegate = self;
    
	    // 3.开始获取用户位置
	    // 注意:获取用户的方向信息是不需要用户授权的
	    [self.mgr startUpdatingHeading];
    
    
	}
	#pragma mark - CLLocationManagerDelegate
	// 当获取到用户方向时就会调用
	- (void)locationManager:(CLLocationManager *)manager didUpdateHeading:(CLHeading *)newHeading
	{
	//    NSLog(@"%s", __func__);
	    /*
	     magneticHeading 设备与磁北的相对角度
	     trueHeading 设置与真北的相对角度, 必须和定位一起使用, iOS需要设置的位置来计算真北
	     真北始终指向地理北极点
	     */
	//    NSLog(@"%f", newHeading.magneticHeading);
    
	    // 1.将获取到的角度转为弧度 = (角度 * π) / 180;
	    CGFloat angle = newHeading.magneticHeading * M_PI / 180;
	    // 2.旋转图片
	    /*
	     顺时针 正
	     逆时针 负数
	     */
	//    self.compasspointer.transform = CGAffineTransformIdentity;
	    self.compasspointer.transform = CGAffineTransformMakeRotation(-angle);
	}

### CoreLocation 区域监测

	- (void)viewDidLoad {
	    [super viewDidLoad];
	    // 2.成为CoreLocation管理者的代理监听获取到的位置
	    self.mgr.delegate = self;
    
	    // 注意:如果是iOS8, 想进行区域检测, 必须自己主动请求获取用户隐私的权限
	    if  ([[UIDevice currentDevice].systemVersion doubleValue] >= 8.0 )
	    {
	        [self.mgr requestAlwaysAuthorization];
	    }
    
	    // 3.开始检测用户所在的区域
	    // 3.1创建区域
	//    CLRegion 有两个子类是专门用于指定区域的
	//    一个可以指定蓝牙的范围/ 一个是可以指定圆形的范围
    
	    // 创建中心点
	    CLLocationCoordinate2D center = CLLocationCoordinate2DMake(40.058501, 116.304171);
    
	    // c创建圆形区域, 指定区域中心点的经纬度, 以及半径
	    CLCircularRegion *circular = [[CLCircularRegion alloc] initWithCenter:center radius:500 identifier:@"软件园"];
    
	    [self.mgr startMonitoringForRegion:circular];
    
	}
	#pragma mark - CLLocationManagerDelegate
	// 进入监听区域时调用
	- (void)locationManager:(CLLocationManager *)manager didEnterRegion:(CLRegion *)region
	{
	    NSLog(@"进入监听区域时调用");
	}
	// 离开监听区域时调用
	- (void)locationManager:(CLLocationManager *)manager didExitRegion:(CLRegion *)region
	{
	    NSLog(@"离开监听区域时调用");
	}

### CoreLocation 地理编码

    // 1.创建地理编码对象
    self.geocoder = [[CLGeocoder alloc] init];
    // 2.利用地理编码对象编码
    // 根据传入的地址获取该地址对应的经纬度信息
    [self.geocoder geocodeAddressString:addressStr completionHandler:^(NSArray *placemarks, NSError *error) {
    }];

### CoreLocation 反地理编码

    // 1.根据用户输入的经纬度创建CLLocation对象
    CLLocation *location = [[CLLocation alloc] initWithLatitude:[latitude doubleValue]  longitude:[longtitude doubleValue]];
    
    // 2.根据CLLocation对象获取对应的地标信息
    [self.geocoder reverseGeocodeLocation:location completionHandler:^(NSArray *placemarks, NSError *error) {
    }];

### CoreLocation 第三方框架

	INTULocationManager
    // 1.创建位置管理者
    INTULocationManager *mgr = [INTULocationManager sharedInstance];
    // 2.利用位置管理者获取位置
    [mgr requestLocationWithDesiredAccuracy:INTULocationAccuracyRoom  timeout:5 delayUntilAuthorized:YES block:^(CLLocation *currentLocation, INTULocationAccuracy achievedAccuracy, INTULocationStatus status) {
        if (status == INTULocationStatusSuccess) {
            NSLog(@"获取位置成功 %f %f", currentLocation.coordinate.latitude , currentLocation.coordinate.longitude);
        }else if(status ==  INTULocationStatusError)
        {
            NSLog(@"获取失败");
        }
    }];