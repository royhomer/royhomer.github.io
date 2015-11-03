---
layout: post
title: "单例模式"
date: 2014-07-15 19:13:20 +0800
comments: true
categories: 单例模式相关
---
<!--more-->
> 数学与逻辑学中，singleton定义为“有且仅有一个元素的集合”。
单例模式最初的定义出现于《设计模式》（艾迪生维斯理, 1994）：“保证一个类仅有一个实例，并提供一个访问它的全局访问点。”


### 写在前面的话

一开始我并不了解单例模式的应用场景。只是知道有这么一种设计模式，怎么去实现它。但是随着使用频率的上升。您就能体会到单例模式的好处，简单的说----单例模式就是保证这个单例类的实例化的对象只有一份，不管怎么多次实例化对象，他得对象都是同一个。


### 一步步实现iOS中单例的写法

1 集成NSObject 实现一个单例类, 实现一个类方法。让这个方法返回的实例唯一。
	
	// 定义一个实例对象
	id _mySingleton;

	+ (instancetype)shareSingleton {
    	// 如果不为空直接返回
   	 	if (_mySingleton == nil) {
        _mySingleton = [[self alloc] init];
    	}
    	return _mySingleton;
	}
	
如下图所示 : 不论调用多少次内存地址都一样。保证单例类的对象的唯一

![](/images/2014/07/danlidayin1.png)

2 我们还得考虑多个线程 同时实例化的时候 所以需要加线程锁 且在最外面加一层判断

	// 定义一个实例对象
	id _mySingleton;

	+ (instancetype)shareSingleton {
    
    	if (_mySingleton == nil) {// 防止多线程重复加锁
       	 	@synchronized(self) { // 加线程锁
            	if (_mySingleton == nil) { // 防止重复实例化
               	 _mySingleton = [[self alloc] init];
            	}
        	}
   		}
    return _mySingleton;
 	}
3 为了保证 全局对象 _mySingleton的唯一性 加static修饰
	
	// static 修饰全局变量 全局变量的作用域只在当前类
	static id _mySingleton;

4 但是当使用copy 或 alloc 实例化对象的时候就会发生对象地址改变，实例化对象不唯一现象，解决办法就是重写allocWithZone 和 copyWithZone```注意这里必须要实现<NSCopying>协议才能打出来``` 一下就是完整版的单例类实现
	
	// 定义一个实例对象
	static id _mySingleton;
	+ (instancetype)shareSingleton {
    
    	if (_mySingleton == nil) {// 防止多线程重复加锁
        	@synchronized(self) { // 加线程锁
            	if (_mySingleton == nil) { // 防止重复实例化
                	_mySingleton = [[self alloc] init];
            	}
        	}
   		 }
    return _mySingleton;
	}

	// 重写allocWithZone 调用父类的方法
	+ (instancetype)allocWithZone:(struct _NSZone *)zone {
    	if (_mySingleton == nil) {
        	@synchronized(self) {
            	if (_mySingleton == nil) {
               	 	_mySingleton = [super allocWithZone:zone];
            	}
        	}
    	}
    	return _mySingleton;
	}
	// 重写copyWithZone
	- (id)copyWithZone:(NSZone *)zone {
    	return _mySingleton;
	}
	
### 懒汉式单例和饿汉式单例-- 推荐大家使用懒汉式
	
上面我们提到这种实现方式就是懒汉式 用到的时候加载

饿汉式实现方式主要思路是程序加载到内存时候进行创建

	// 定义一个实例对象
	static id _mySingleton;

	+ (void)load {
    	_mySingleton = [[self alloc] init];
	}
	
	+ (instancetype)shareSingleton {
    	return _mySingleton;
	}
	
	+ (instancetype)allocWithZone:(struct _NSZone *)zone {
    	if (_mySingleton == nil) {
        	_mySingleton = [super allocWithZone:zone];
    	}
    	return _mySingleton;
	}

	- (id)copyWithZone:(NSZone *)zone {
    	return _mySingleton;
	}
### MRC实现单例的时候防止对象被释放重写一下三个方法

	- (oneway void)release {
    // 什么也不写
	}

	- (id)retain {
    	return self;
	}	

	- (NSUInteger)retainCount {
    	return 1;
	}

### 用GCD得方式实现单例类

	// 定义一个实例对象
	static id _mySingleton;
	+ (instancetype)shareSingleton {

    	static dispatch_once_t onceToken;
    	dispatch_once(&onceToken, ^{
        	_mySingleton = [[self alloc] init];
   		});

    return _mySingleton;
	}

	// 重写allocWithZone 调用父类的方法
	+ (instancetype)allocWithZone:(struct _NSZone *)zone {

    	static dispatch_once_t onceToken;
    	dispatch_once(&onceToken, ^{
       		_mySingleton = [super allocWithZone:zone];
    	});
    
    	return _mySingleton;
	}

	// 重写copyWithZone
	- (id)copyWithZone:(NSZone *)zone {
    	return _mySingleton;
	}

### 用宏实现快捷单例类

	#define Single(name) \
	\
	static id _my##name;\
	+ (instancetype)share##name {\
	\
    	static dispatch_once_t onceToken;\
    	dispatch_once(&onceToken, ^{\
        	_my##name = [[self alloc] init];\
    	});\
	\
    	return _my##name;\
	}\
	\
	+ (instancetype)allocWithZone:(struct _NSZone *)zone {\
	\
    	static dispatch_once_t onceToken;\
    	dispatch_once(&onceToken, ^{\
       		_my##name = [super allocWithZone:zone];\
   		});\
    	\
    	return _my##name;\
	}\
	\
	- (id)copyWithZone:(NSZone *)zone {\
    	return _my##name;\
	}
 