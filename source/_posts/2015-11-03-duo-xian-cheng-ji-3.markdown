---
layout: post
title: "多线程基础03 NSOperation"
date: 2014-07-07 12:56:07 +0800
comments: true
categories: 多线程
---
<!--more-->
> NSOperation和GCD很类似


### NSOperation的作用
* 配合使用NSOperation 和 NSOperationQueue也能实现多线程编程

### NSOperation和NSOperationQueue实现多线程的具体步骤
* 现将需要执行的操作封住到一个NSOperation对象中
* 然后将NSoperation对象添加到NSOperationQueue中
* 系统会自动将NSOperationQueue中的NSOperation取出来
* 然后将NSOperation中封装的操作放到一个线程中执行

### NSOperation是一个操作类它的的子类才有封装操作的能力

1. NSInvocationOperation
2. NSBlockOperation
3. 自定义继承NSOperation的子类

创建NSOperation 前两种种方式其实作用一样 只是一个Block 一个target调用 
	
	NSInvocationOperation *operation = [[NSInvocationOperation alloc] initWithTarget:self selector:@selector(操作的方法) object:nil];
	
	NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
        // 要执行的任务或操作
    }];

#### 队列的类型
1. 主队列
	* [NSOperationQueue mainQueue];
	* 添加到"主队列"中的操作,都会放到主线程中执行
2. 非主队列
	* [[NSOperationQueue alloc] init];
	* 添加到"非主队列"中的操作,都会放到子线程中执行
	
#### 队列添加任务
*	-(void)addOperation:(NSOperation *)op;
*	-(void)addOperationWithBlock:(void(^)(void))block;


##### 上代码

情景一 、封装一个NSOperation操作, 直接调用start方法,

分析: 主线程 -- 同步执行 -- 不开启子线程

    NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
        // 要执行的任务或操作
    }];
    [operation start];

情景二 、封装一个NSOperation操作,往里添加多个block任务, 

分析1: 第一个操作 : 主线程 -- 异步执行 -- 不开启子线程 

分析2: 后添加的操作 : 子线程 -- 异步执行 -- 开启子线程  

	NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
        // 要执行的任务或操作A
    }];
  
    [operation addExecutionBlock:^{
        // 要执行的任务或操作B
    }];
    
    [operation addExecutionBlock:^{
        // 要执行的任务或操作C
    }];
    
    [operation start];	 

操作A在主线程, 操作B、C在子线程


情景三 、创建一个NSOperationQueue的队列 封装一个(或者多个)NSOperation操作 

分析: 子线程 -- 异步执行 -- 开启子线程
    
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
    NSBlockOperation *operation = [NSBlockOperation blockOperationWithBlock:^{
        // 要执行的任务或操作
    }];
    
    [queue addOperation:operation];
    
操作是在子线程异步执行
    
    NSOperationQueue *queue = [[NSOperationQueue alloc] init];
    
    // 封装
    NSBlockOperation *operationA = [NSBlockOperation blockOperationWithBlock:^{
        // 要执行的任务或操作A
    }];
    
    NSBlockOperation *operationB = [NSBlockOperation blockOperationWithBlock:^{
        // 要执行的任务或操作B
    }];
  
    [queue addOperation:operationA];
    [queue addOperation:operationB];
    [queue addOperationWithBlock:^{
       // 要执行的操作或任务C
    }];

操作A、B、C是在子线程异步执行的


#### 常见的用法
1. 设置最大并发数
	* -(void)setMaxConcurrentOperationCount:(NSInteger)count;
	* (NSInteger)maxConcurrentOperationCount;
	
2. 队列其它操作
	* 取消所有操作 -(void)cancelAllOperations;
	* 暂停所有操作 [queue setSuspended:YES];
	* 恢复所有操作 [queue setSuspended:NO];

#### 操作之间的依赖
* NSOperation之间可以设置依赖来保证执行顺序
* [operationB addDependency:operationA];  --- ```等操作A执行完毕后才能执行操作B``` （不能相互依赖）

#### 线程之间的通讯
	[queue addOperationWithBlock:^{
       // 要执行耗时操作
        
        [[NSOperationQueue mainQueue] addOperationWithBlock:^{
            // 回到主线程执行的操作
        }];
    }];
    







