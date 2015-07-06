---
layout: post
title: IOS MapKit
category: 技术
tag: [IOS, MapKit]
description:  
---

### MapKit 基本使用

MKMapView 地图控件

	// 1.设置地图显示的类型
	    /*
	     typedef enum : NSUInteger {
	     MKMapTypeStandard , 标准(默认)
	     MKMapTypeSatellite ,卫星
	     MKMapTypeHybrid 混合(标准 + 卫星)
	     } MKMapType;
	     */
	//    self.customMapView.mapType = MKMapTypeSatellite;
	    /*
	    CoreLocation框架定位
	    CLLocationManager mgr;
	    [mgr startUpdatingLocation];
	     */
    
	    // 注意:在iOS8中, 如果想要追踪用户的位置, 必须自己主动请求隐私权限
	    if ([[UIDevice currentDevice].systemVersion doubleValue] >= 8.0) {
	        // 主动请求权限
	        self.mgr = [[CLLocationManager alloc] init];
        
	        [self.mgr requestAlwaysAuthorization];
	    }
    
	    // 设置不允许地图旋转
	    self.customMapView.rotateEnabled = NO;
    
	    // 成为mapVIew的代理
	    self.customMapView.delegate = self;
    
	    // 如果想利用MapKit获取用户的位置, 可以追踪
	    /*
	     typedef NS_ENUM(NSInteger, MKUserTrackingMode) {
	     MKUserTrackingModeNone = 0, 不追踪/不准确的
	     MKUserTrackingModeFollow, 追踪
	     MKUserTrackingModeFollowWithHeading, 追踪并且获取用的方向
	     }
	     */
	    self.customMapView.userTrackingMode =  MKUserTrackingModeFollow;
		
		#pragma MKMapViewDelegate
		/**
		 *  每次更新到用户的位置就会调用(调用不频繁, 只有位置改变才会调用)
		 *
		 *  @param mapView      促发事件的控件
		 *  @param userLocation 大头针模型
		 */
		- (void)mapView:(MKMapView *)mapView didUpdateUserLocation:(MKUserLocation *)userLocation
		{
		    /*
		     地图上蓝色的点就称之为大头针
		     大头针可以拥有标题/子标题/位置信息
		     大头针上显示什么内容由大头针模型确定(MKUserLocation)
		     */
		    // 设置大头针显示的内容
		//    userLocation.title = @"黑马";
		//    userLocation.subtitle = @"牛逼";
    
		    // 利用反地理编码获取位置之后设置标题
		    [self.geocoder reverseGeocodeLocation:userLocation.location completionHandler:^(NSArray *placemarks, NSError *error) {
		        CLPlacemark *placemark = [placemarks firstObject];
		        NSLog(@"获取地理位置成功 name = %@ locality = %@", placemark.name, placemark.locality);
		        userLocation.title = placemark.name;
		        userLocation.subtitle = placemark.locality;
		    }];
    
    
    
		    // 移动地图到当前用户所在位置
		    // 获取用户当前所在位置的经纬度, 并且设置为地图的中心点
		//    [self.customMapView setCenterCoordinate:userLocation.location.coordinate animated:YES];
    
		    // 设置地图显示的区域
		    // 获取用户的位置
		    CLLocationCoordinate2D center = userLocation.location.coordinate;
		    // 指定经纬度的跨度
		    MKCoordinateSpan span = MKCoordinateSpanMake(0.009310,0.007812);
		    // 将用户当前的位置作为显示区域的中心点, 并且指定需要显示的跨度范围
		    MKCoordinateRegion region = MKCoordinateRegionMake(center, span);
    
		    // 设置显示区域
		    [self.customMapView setRegion:region animated:YES];
		}


### MapKit 大头针

	#pragma MKMapViewDelegate
	/**
	 *  每次添加大头针就会调用(地图上有几个大头针就调用几次)
	 *
	 *  @param mapView    地图
	 *  @param annotation 大头针模型
	 *
	 *  @return 大头针的view
	 */
	- (MKAnnotationView *)mapView:(MKMapView *)mapView viewForAnnotation:(id<MKAnnotation>)annotation
	{
	//    NSLog(@"%s", __func__);
	    NSLog(@"annotation === %@", annotation);
	    // 对用户当前的位置的大头针特殊处理
	    if ([annotation isKindOfClass:[HMAnnotation class]] == NO) {
	        return nil;
	    }
    
	    // 注意: 如果返回nil, 系统会按照自己默认的方式显示
	//    return nil;
    
	    /*
	    static NSString *identifier = @"anno";
	    // 1.从缓存池中取
	    // 注意: 默认情况下MKAnnotationView是无法显示的, 如果想自定义大头针可以使用MKAnnotationView的子类MKPinAnnotationView
    
	    // 注意: 如果是自定义的大头针, 默认情况点击大头针之后是不会显示标题的, 需要我们自己手动设置显示
	//    MKPinAnnotationView *annoView = (MKPinAnnotationView *)[mapView dequeueReusableAnnotationViewWithIdentifier:identifier];
    
    
	    MKAnnotationView *annoView = [mapView dequeueReusableAnnotationViewWithIdentifier:identifier];
	    // 2.如果缓存池中没有, 创建一个新的
	    if (annoView == nil) {
	//        annoView = [[MKPinAnnotationView alloc] initWithAnnotation:nil reuseIdentifier:identifier];
	        annoView = [[MKAnnotationView alloc] initWithAnnotation:nil reuseIdentifier:identifier];
        
	        // 设置大头针的颜色
	//        annoView.pinColor = MKPinAnnotationColorPurple;
        
	        // 设置大头针从天而降
	//        annoView.animatesDrop = YES;
        
	        // 设置大头针标题是否显示
	        annoView.canShowCallout = YES;
        
	        // 设置大头针标题显示的偏移位
	//        annoView.calloutOffset = CGPointMake(-50, 0);
        
	        // 设置大头针左边的辅助视图
	        annoView.leftCalloutAccessoryView = [[UISwitch alloc] init];
	        // 设置大头针右边的辅助视图
	        annoView.rightCalloutAccessoryView = [UIButton buttonWithType:UIButtonTypeContactAdd];
      
	    }
    
	    // 设置大头针的图片
	    // 注意: 如果你是使用的MKPinAnnotationView创建的自定义大头针, 那么设置图片无效, 因为系统内部会做一些操作, 覆盖掉我们自己的设置
	//    annoView.image = [UIImage imageNamed:@"category_4"];
	    HMAnnotation *anno = (HMAnnotation *)annotation;
	    annoView.image = [UIImage imageNamed:anno.icon];
    
	    // 3.给大头针View设置数据
	    annoView.annotation = annotation;
    
	    // 4.返回大头针View
	    return annoView;
	     */
    
	    // 1.创建大头针
	    HMAnnotationView *annoView = [HMAnnotationView annotationViewWithMap:mapView];
	    // 2.设置模型
	    annoView.annotation = annotation;
    
	    // 3.返回大头针
	    return annoView;
	}

### MapKit 导航

#### iOS自带导航

	- (void)startNavigation
	{
	    // 1.获取用户输入的起点和终点
	    NSString *startStr = self.startField.text;
	    NSString *endStr = self.endField.text;
	    if (startStr == nil || startStr.length == 0 ||
	        endStr == nil || endStr.length == 0) {
	        NSLog(@"请输入起点或者终点");
	        return;
	    }
    
	    // 2.利用GEO对象进行地理编码获取到地标对象(CLPlacemark )
	    // 2.1获取开始位置的地标
	    [self.geocoder geocodeAddressString:startStr completionHandler:^(NSArray *placemarks, NSError *error) {
	        if (placemarks.count == 0) return;
        
	        // 开始位置的地标
	        CLPlacemark *startCLPlacemark = [placemarks firstObject];
        
        
	        // 3. 获取结束位置的地标
	        [self.geocoder geocodeAddressString:endStr completionHandler:^(NSArray *placemarks, NSError *error) {
            
	            if (placemarks.count == 0) return;
            
	            // 结束位置的地标
	            CLPlacemark *endCLPlacemark = [placemarks firstObject];
            
	            // 开始导航
	            [self startNavigationWithstartCLPlacemark:startCLPlacemark endCLPlacemark:endCLPlacemark];
	        }];
        
	    }];
	}
	/**
	 *  开始导航
	 *
	 *  @param startCLPlacemark 起点的地标
	 *  @param endCLPlacemark   终点的地标
	 */
	- (void)startNavigationWithstartCLPlacemark:(CLPlacemark *)startCLPlacemark endCLPlacemark:(CLPlacemark *)endCLPlacemark
	{
	    // 0.创建起点和终点
	    // 0.1创建起点
	    MKPlacemark *startPlacemark = [[MKPlacemark alloc] initWithPlacemark:startCLPlacemark];
	    MKMapItem *startItem = [[MKMapItem alloc] initWithPlacemark:startPlacemark];;
    
	    // 0.2创建终点
	    MKPlacemark *endPlacemark = [[MKPlacemark alloc] initWithPlacemark:endCLPlacemark];
	    MKMapItem *endItem = [[MKMapItem alloc] initWithPlacemark:endPlacemark];
    
	    // 1. 设置起点和终点数组
	    NSArray *items = @[startItem, endItem];
    
    
	    // 2.设置启动附加参数
	    NSMutableDictionary *md = [NSMutableDictionary dictionary];
	    // 导航模式(驾车/走路)
	    md[MKLaunchOptionsDirectionsModeKey] = MKLaunchOptionsDirectionsModeDriving;
	    // 地图显示模式
	//    md[MKLaunchOptionsMapTypeKey] = @(MKMapTypeHybrid);
    
    
	    // 只要调用MKMapItem的open方法, 就可以打开系统自带的地图APP进行导航
	    // Items: 告诉系统地图APP要从哪到哪
	    // launchOptions: 启动系统自带地图APP的附加参数(导航的模式/是否需要先交通状况/地图的模式/..)
	    [MKMapItem openMapsWithItems:items launchOptions:md];
	}


#### MapKit 绘制路线

	- (IBAction)drawLine
	{
	    // 1.获取用户输入的起点和终点
	    NSString *startStr = @"北京";
	    NSString *endStr = @"云南";
	    if (startStr == nil || startStr.length == 0 ||
	        endStr == nil || endStr.length == 0) {
	        NSLog(@"请输入起点或者终点");
	        return;
	    }
    
	    // 2.利用GEO对象进行地理编码获取到地标对象(CLPlacemark )
	    // 2.1获取开始位置的地标
	    [self.geocoder geocodeAddressString:startStr completionHandler:^(NSArray *placemarks, NSError *error) {
	        if (placemarks.count == 0) return;
        
	        // 开始位置的地标
	        CLPlacemark *startCLPlacemark = [placemarks firstObject];
        
	        // 添加起点的大头针
	        HMAnnotation *startAnno = [[HMAnnotation alloc ] init];
	        startAnno.title = startCLPlacemark.locality;
	        startAnno.subtitle = startCLPlacemark.name;
	        startAnno.coordinate = startCLPlacemark.location.coordinate;
	        [self.mapVIew addAnnotation:startAnno];
        
	        // 3. 获取结束位置的地标
	        [self.geocoder geocodeAddressString:endStr completionHandler:^(NSArray *placemarks, NSError *error) {
            
	            if (placemarks.count == 0) return;
            
	            // 结束位置的地标
	            CLPlacemark *endCLPlacemark = [placemarks firstObject];
            
	            // 添加终点的大头针
	            HMAnnotation *endAnno = [[HMAnnotation alloc ] init];
	            endAnno.title = endCLPlacemark.locality;
	            endAnno.subtitle = endCLPlacemark.name;
	            endAnno.coordinate = endCLPlacemark.location.coordinate;
	            [self.mapVIew addAnnotation:endAnno];
            
	            // 绘制路线
	            [self startDirectionsWithstartCLPlacemark:startCLPlacemark endCLPlacemark:endCLPlacemark];
	        }];
        
	    }];
	}
	/**
	 *  发送请求获取路线相信信息
	 *
	 *  @param startCLPlacemark 起点的地标
	 *  @param endCLPlacemark   终点的地标
	 */
	- (void)startDirectionsWithstartCLPlacemark:(CLPlacemark *)startCLPlacemark endCLPlacemark:(CLPlacemark *)endCLPlacemark
	{
	    /*
	     MKDirectionsRequest:说清楚:从哪里 --> 到哪里
	     MKDirectionsResponse:从哪里 --> 到哪里 :的具体路线信息
	     */
    
	    // -1.创建起点和终点对象
	    // -1.1创建起点对象
	    MKPlacemark *startMKPlacemark = [[MKPlacemark alloc] initWithPlacemark:startCLPlacemark];
	    MKMapItem *startItem = [[MKMapItem alloc] initWithPlacemark:startMKPlacemark];
    
	    // -1.2创建终点对象
	    MKPlacemark *endMKPlacemark = [[MKPlacemark alloc] initWithPlacemark:endCLPlacemark];
	    MKMapItem *endItem = [[MKMapItem alloc] initWithPlacemark:endMKPlacemark];
    
    
	    // 0.创建request对象
	    MKDirectionsRequest *request = [[MKDirectionsRequest alloc] init];
	    // 0.1设置起点
	    request.source = startItem;
	    // 0.2设置终点
	    request.destination = endItem;
    
    
    
	    // 1.发送请求到苹果的服务器获取导航路线信息
	    // 接收一个MKDirectionsRequest请求对象, 我们需要在该对象中说清楚:
	    // 从哪里 --> 到哪里
	    MKDirections *directions = [[MKDirections alloc] initWithRequest:request];
	    // 2.计算路线信息, 计算完成之后会调用blcok
	    // 在block中会传入一个响应者对象(response), 这个响应者对象中就存放着路线信息
	    [directions calculateDirectionsWithCompletionHandler:^(MKDirectionsResponse *response, NSError *error) {
        
        
	        // 打印获取到的路线信息
	        // 2.1获取所有的路线
	        NSArray *routes = response.routes;
	        for (MKRoute *route in routes) {
	            NSLog(@"%f千米 %f小时", route.distance / 1000, route.expectedTravelTime/ 3600);
            
	            //  3.绘制路线(本质: 往地图上添加遮盖)
	            // 传递当前路线的几何遮盖给地图, 地图就会根据遮盖自动绘制路线
	            // 当系统开始绘制路线时会调用代理方法询问当前路线的宽度/颜色等信息
	            [self.mapVIew addOverlay:route.polyline];
            
	            NSArray *steps = route.steps;
	            for (MKRouteStep *step in steps) {
	//                NSLog(@"%@ %f", step.instructions, step.distance);
	            }
            
	        }
	    }];
	}
	#pragma mark - MKMapViewDelegate
	// 过时
	//- (MKOverlayView *)mapView:(MKMapView *)mapView viewForOverlay:(id<MKOverlay>)overlay
	// 绘制路线时就会调用(添加遮盖时就会调用)
	- (MKOverlayRenderer *)mapView:(MKMapView *)mapView rendererForOverlay:(id<MKOverlay>)overlay
	{
	//    MKOverlayRenderer *renderer = [[MKOverlayRenderer alloc] init];
	    // 创建一条路径遮盖
	    NSLog(@"%s", __func__);
	#warning 注意, 创建线条时候,一定要制定几何路线
	    MKPolylineRenderer *line = [[MKPolylineRenderer alloc] initWithPolyline:overlay];
	    line.lineWidth = 5; // 路线的宽度
	    line.strokeColor = [UIColor redColor];// 路线的颜色
    
	    // 返回路线
	    return line;
	}

### 百度地图

	百度地图API 最新版本是2.4.1，需要关注，不支持64位
	注:静态库中采用ObjectC++实现，因此需要您保证您工程中至少有一个.mm后缀的源文件
	1> 没有64位架构的支持
	libbaidumapapi.a, missing required architecture x86_64
	.a文件缺少64位的架构
	解决办法：将Architectures修改位：$(ARCHS_STANDARD_32_BIT)
	2> 如果在导入第三方框架时，发现提示"std::"提示错误，说明框架使用了C++
	解决办法，随便把项目中的一个文件，扩展名.mm
	.m		c语言&OC混编
	.mm	c++语言&OC混编
	.c		纯C语言
	.cpp	纯C++
	3> 百度地图api的特点，代理方法，通常以onXXX，表示发生了什么事件时。。。
	4> 关于error的数字
		0	表示正确
		其他数字，表示错误代码
	5> 自2.0.0起，BMKMapView新增viewWillAppear、viewWillDisappear方法来控制BMKMapView的生命周期，并且在一个时刻只能有一个BMKMapView接受回调消息
		因此在使用BMKMapView的viewController中需要在viewWillAppear、viewWillDisappear方法中调用BMKMapView的对应的方法，并处理delegate，代码如下
	6> POI检索：周边检索、区域检索和城市内检索
		苹果原生地图框架不支持的功能