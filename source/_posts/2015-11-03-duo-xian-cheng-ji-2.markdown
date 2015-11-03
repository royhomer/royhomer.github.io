---
layout: post
title: "多线程基础02 GCD"
date: 2014-07-05 12:53:07 +0800
comments: true
categories: 多线程 
---
  <!--more-->   
   了解完多线基础概念和NSTread的基本使用后，本文介绍GCD的主要用方法
   
   GCD全称 : Grand Central Dispatch  -- ```伟大的中枢调度器``` 
  
   ***
   
   > CGD的优势: 多核并行运算提出解决方案，自动管理线程生命周期，自动利用更多的CPU内核
   

### 1 两大核心概念
* 任务 : 执行什么操作
* 队列 : 用来存放任务

* 一般使用步骤: 定制任务，将任务添加到队列，GCD会自动将队列中得任务取出，放到对应的线程中执行;任务取出顺序，遵循FIFO，先进先出，后进后出，


### 2 用来执行任务的两个函数
同步执行

	dispatch_sync(dispatch_queue_t queue, ^(void)block)
		
异步执行

	dispatch_async(dispatch_queue_t queue, ^(void)block)
	* 参数 queue 队列
	* 参数 block 任务的代码块

* 两者的区别
	* 同步 : 不开启新线程
	* 异步 : 会开启新线程
	
### 3 队列的类型

#### GCD的队列分为两大类型

并发队列 (Concurrent Dispatch Queue) : ```多个任务并发（同时）执行``` (自动开启多个线程同时执行),只有在异步函数下有效
   
	dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)

  参数1 : 队列优先级	 
  参数2 : 默认 0	
	
	
串行队列 (Serial Dispatch Queue) : 一个任务执行完成再接着执行下一个
   
	dispatch_queue_t queue = dispatch_queue_create("C语言的字符串-队列标识", NULL);
			
### 4 四种情况分析

##### 1. 异步执行 全局并发队列 添加5个任务 
* 分析 ```常用这种方式```
	* 会不会创建线程 : 会创建5条子线程
	* 任务的执行方式 : 并发执行
	
##### 2. 异步执行 串行队列 添加5个任务 
* 分析 ```偶尔常用这种方式```
	* 会不会创建线程 : 会创建1条子线程（不论添加多少个任务都只开启一条）
	* 任务的执行方式 : 顺序执行
	
##### 3. 同步执行 全局并发队列 添加5个任务 
* 分析 ```不常用```
	* 会不会创建线程 : 不会开启新的子线程
	* 任务的执行方式 : 顺序执行，（因为只有一条线程所以只能顺序执行）
	
##### 4. 同步执行 串行队列 添加5个任务 
* 分析 ```不常用```
	* 会不会创建线程 : 不会开启新的子线程
	* 任务的执行方式 : 顺序执行

##### 5. 特殊情况 -- 异步执行 串行主队列 添加5个任务
	// 获取主队列  这个也是一个串行队列
   	dispatch_queue_t queue = dispatch_get_main_queue();
  	
* 分析 
	* 会不会创建线程 : 不会开启新的线程 因为主队列对应的线程就是主线程,主线程只能有一个
	* 任务的执行方式 : 顺序执行

### 5 GCD线程之间的通信

	dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
       
        // 耗时操作
        
        dispatch_async(dispatch_get_main_queue(), ^{
            
            // 主线程刷新UI
            
        });
    });
		 
### 6 延迟执行

1 下面这种方式会卡住主线程

	[NSThread sleepForTimeInterval:2];
	
2 下面这种方式不会卡住主线程,但是不能改变延迟后代码执行的线程

	[self performSelector:@selector(download:) withObject:@"下载中" afterDelay:2];```

3 这种GCD的方式，比较好，不卡住主线程，也能随意定制延迟后代码执行的线程
	
	dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(“时间” * NSEC_PER_SEC)), dispatch_get_main_queue()(这个队列可以改成全局队列), ^{
        // 延迟后要执行的代码
        });
    
### 7 一次执行

一次性执行的代码主要用来创建单例模式

	static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        // 只执行一次的代码
    });
    
### 8 队列组

队列组的功能主要是，当多个任务需要同时执行，且所有任务都执行完毕后，还要触发执行别的任务的时候用到。

 	//获取队列组
 	dispatch_group_t group = dispatch_group_create();
 	
 	//队列组添加异步执行任务
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
 	dispatch_group_async(group, queue, ^{
       // 任务1代码 
    });
 	.
 	添加多个任务
 	.
 	dispatch_group_async(group, queue, ^{
       // 任务n代码 
    });
 	.
 	dispatch_group_notify(group, queue, ^{
        // 所有任务执行完后唤醒需要执行的代码        
    });
 	
 	
    