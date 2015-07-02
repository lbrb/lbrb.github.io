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
	