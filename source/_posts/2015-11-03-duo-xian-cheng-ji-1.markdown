---
layout: post
title: "多线程基础01 概念和NSThread"
date: 2014-07-04 12:21:39 +0800
comments: true
categories: 多线程 
---
<!-- more-->
   在软件开发中必不可少的会接触到一个词语--多线程；
   那么什么是多线呢，本文主要是对多线程的基础知识做简单讲解。


***
## 1 基本概念
#### 1.1 进程的概念

* 进程是指在系统正在运行的一个应用程序，每个进程是相对独立

	eg: 当我们启动QQ，那么就会开启一个进程

#### 1.2 线程的概念
* 每个进程中至少要有一条线程。（一个程序的所有任务都是在线程中执行的）
* 一个线程中任务的执行是```串行的```

#### 1.3 多线程的概念
* 一个进程中可以开多条线程，每条线程可以并行（同时）执行
	* 主线程 : 自动创建的，默认的 又叫UI线程
	* 子线程 : 又叫后台线程

###### 多线程原理
* CPU同一时间只能有一条线程在工作，其实是CPU快速的在多条线程之间调度（切换）；调度频率快，就形成了多线程同时执行的“假象”。所以移动端不建议开太多线程一般在5条以内

###### 多线程优缺点
* 优点：能适当提高执行效率、资源（CPU，内存）利用率
* 缺点：开启线程占用内存，开销大，性能降低

###### 多线程在iOS开发中的应用
* 默认有一条主线程（UI线程）有且只有一条。作用：负责显示和刷新界面，处理UI事件
* ```注意```：不要把耗时操作放在主线程中

## 2 iOS中多线程的实现方案

![](/images/2014/07/duoxianchengjichu.png)

## 3 pthread的例子
	- (void)viewDidLoad {
	    [super viewDidLoad];
	    
	    // 创建一个线程标识符
	    pthread_t myrestrict;
	    
	    // 1param 线程标识符变量的地址, 2param 写NULL 3param 执行的的函数 4param NULL
	    pthread_create(&myrestrict, NULL, run, NULL);
	}
	
	// 定义线程的函数
	void *run(void *data) {
	    
	    NSLog(@"%@", [NSThread currentThread]);
	    
	    return nil;
	}

## 4 NSThread的基本用法

####  创建方式1 创建后需启动

	    //创建线程
		NSThread *thread = [[NSThread alloc] initWithTarget:self selector:@selector(download) object:nil];
		// 启动线程
	 	[thread start];
	 
####  创建方式2  创建后自动启动
		// 创建后直接启动  传一个字符串
		[NSThread detachNewThreadSelector:@selector(download2:) toTarget:self withObject:@"param"];
	    
####  创建方式3  隐式启动
	    // 开启子线程
	    [self performSelectorInBackground:@selector(download) withObject:nil];
	    // 开启主线程
	    [self performSelector:@selector(download) withObject:nil];
	    // 开启传入的线程
	    [self performSelector:@selector(download) onThread:[NSThread currentThread] withObject:nil waitUntilDone:NO];
	
####  其它常用方法
	    // 获取当前线程
	    NSThread *current = [NSThread currentThread];
	    // 获取主线程
	    NSThread *main = [NSThread mainThread];
	    // 判断是否主线程-类方法
	    BOOL isMain = [NSThread isMainThread];
	    // 判断是否主线程-对象方法
	    BOOL isMain2 = [main isMainThread];
	    // 给线程起名字
	    current.name = @"下载线程";
	    // 线程睡眠状态5秒
	    [NSThread sleepForTimeInterval:5.0];
	    // 线程睡眠从现在开始3秒以后的时间
	    [NSThread sleepUntilDate:[NSDate dateWithTimeIntervalSinceNow:3]];
		// 线程退出 线程进入死亡状态
		[NSThread exit];
		
#### 线程之间的通信

	[self performSelectorOnMainThread:@selector(method) withObject:nil waitUntilDone:YES];
    
    [self performSelector:@selector(method) onThread:[NSThread mainThread] withObject:nil waitUntilDone:YES];
    
  * waitUntilDone: 是否等待执行完   performSelector:执行的方法  withObject: 传参数

## 5.线程安全
		
* 线程安全
	* 概念 : 多个线程夺取同一个资源，访问一个变量
	* 解决方案 : 加锁--线程锁, 互斥锁 
	* 互斥锁的格式 : ```@synchronized(self) { // 插入锁定代码 }```
	* ```注意``` : 一份代码只能用一个锁, 多个锁无效

* 线程锁优缺点
	* 优点 : 能够有效防止因多线程抢夺资源造成的数据安全问题
	* 缺点 : 消耗大量CPU资源

* 线程同步
	* 多条线程在同一条线上执行（按顺序执行）
	* 互斥锁就运用了线程同步

* 原子属性和非原子属性
	* atomic 会自动为setter方法加锁
	* nonatomic 不会为setter方法  推荐使用这个
	
## 6.多线程状态示意图
##### 在了解完线程安全后咱们再看看完整的线程状态示意图
![](/images/2014/07/多线程状态示意图.png)
	
    	

 
 
