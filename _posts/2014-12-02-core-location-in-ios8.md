---
layout: post
title: "Core Location 在 iOS 8 中的一些变化"
categories: iOS
tags: CoreLocation
---

iOS 8 在定位方面有了不小的变化，在项目中碰到了，这里稍微总结一下。

# 明确告知用户你将如何使用 TA 的位置信息

在 iOS 8 以前，APP 想要使用用户的位置信息通常是这样：

{% highlight objc %}
if ([CLLocationManager locationServicesEnabled]) {
    [manager startUpdatingLocation];
}
{% endhighlight %}
    
系统会以弹窗告知 APP 的位置请求，用户也只能选择允许或拒绝，不能做更细的划分。而在 iOS 8 中，系统新增了两个方法用以位置请求，分别是 `requestAlwaysAuthorization` 和 `requestWhenInUseAuthorization`。顾名思义，前者就是以前的完全权限，APP 在前后台都能进行位置获取，后者则是所谓的“When In Use”，即 APP 在前台运行时。

须注意的是，如果要在程序中使用这两个方法，还必须在 **Info.plist** 中添加相应的字段，分别是 `NSLocationAlwaysUsageDescription` 和 `NSLocationWhenInUseUsageDescription`，其作用是在系统弹出位置请求窗口的时候作为描述文本，使用户进一步了解 APP 为什么要使用 TA 的位置。效果如下图中的中文信息：


![Core Location](/assets/location_alert.png)


一旦用户做出选择，则会调用下面的委托方法：

{% highlight objc %}
- (void)locationManager:(CLLocationManager *)manager 
			didChangeAuthorizationStatus:(CLAuthorizationStatus)status
{% endhighlight %}
    
其中 `status` 的值可能为：
    
- kCLAuthorizationStatusNotDetermined：用户尚未做出选择
- kCLAuthorizationStatusRestricted：限制状态且无法改变，比如开启了家长控制之类
- kCLAuthorizationStatusDenied：用户不允许访问或者定位服务未开启
- kCLAuthorizationStatusAuthorizedAlways：用户允许完全的位置信息访问
- kCLAuthorizationStatusAuthorizedWhenInUse：前台可访问

根据不同取值进行相应处理即可，比如状态如果为 `NotDetermined` 则可再次调用 `requestAlwaysAuthorization`，为 `Denied` 则向用户做出提示。

# 室内定位和场所定位

iOS 8 中新增了 `CLFloor` 和 `CLVisit` 两个类，分别用来表示用户在室内的位置和到过的场所信息，结构比较简单，看看文档即可。因为我目前也没有用到这两个类，下面只大略地提一下。

`CLFloor` 只有一个 `level` 的整型属性，用来标示用户所处某建筑物内的楼层数。

而 `CLVisit` 又是什么呢？如果一个用户在某个地方呆了一段时间，则能用一个 `CLVisit` 来表示，其中包括这个地点的位置信息，用户来到和离开此处的时间，有一个专门的委托方法来处理 Visit 事件：

{% highlight objc %}
- (void)locationManager:(CLLocationManager *)manager
              didVisit:(CLVisit *)visit
{% endhighlight %}

