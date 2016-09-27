title: 原来isKindOfClass并非那么可靠
date: 2016-01-21 11:15:29
tags: [IT]
categories: objc
---

从正式开发iOS的那天起，我便了解**-isKindOfClass:**可以用来确定一个对象是否是一个类的成员，也可以用来确定一个对象是否是派生自该类的类的成员。可实际上它并不值得被信任。

下面以**NSURLSessionDataTask**这个类来阐述为什么**isKindOfClass**并不可靠。

我们可以打印出**NSURLSessionDataTask**的继承树，代码如下：

```
NSURLSessionDataTask *task = [[NSURLSession sharedSession] dataTaskWithURL:[NSURL URLWithString:@"http://blog.leonluo.com"]];
NSLog(@"%@", [task class]);
NSLog(@"%@", [task superclass]);
NSLog(@"%@", [[task superclass] superclass]);
NSLog(@"%@", [[[task superclass] superclass] superclass]);
```

在iOS8以上的结果是：


	__NSCFLocalDataTask
	__NSCFLocalSessionTask
	NSURLSessionTask
	NSObject


在iOS7上的结果是：



	__NSCFLocalDataTask
	__NSCFLocalSessionTask
	__NSCFURLSessionTask
	NSObject


    
    
如果我们去问，我们建立的这些**data task**到底是不是**NSURLSessionDataTask**，调用
**[task isKindOfClass:[NSURLSessionDataTask class]]**，得到的结果还是YES，可事实上它来自**__NSCFLocalDataTask**。           

其实**-isKindOfClass:**是可以override掉的，所以，即使**__NSCFLocalDataTask**根本就不是**NSURLSessionDataTask**，但我们可以把**__NSCFLocalDataTask**的**-isKindOfClass:**写成这样：




	- (BOOL)isKindOfClass:(Class)aClass
	{
    	if (aClass == NSClassFromString(@"NSURLSessionDataTask")) {
        return YES;
    	}
    	if (aClass == NSClassFromString(@"NSURLSessionTask")) {
        return YES;
    	}
    	return [super isKindOfClass:aClass];
	}






>* 无论是iOS8还是iOS7我们建立的data task，都不是直接产生**NSURLSessionDataTask**，而是产生**__NSCFLocalDataTask**这样的**private class**。
>* **-isKindOfClass:**并不那么可靠。


