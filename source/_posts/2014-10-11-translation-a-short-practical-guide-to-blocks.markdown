---
layout: post
title: "翻译 - A Short Practical Guide to Blocks"
date: 2014-10-11 20:45:46 +0800
comments: true
categories: 
- 原创翻译
- Obj-C
---

### 写在前面

这是我的原创翻译的第一篇。

近期在阅读苹果官方文档的时候，突然萌生了想要翻译它的想法。一来是觉得可以加强阅读质量、加深印象，二来也可以练习一下英语理解能力，第三也是觉得可以为那些不愿意阅读英文文档的朋友尽一点微薄之力。反正现在没有工作，也是有些时间的（笑）。所以没有经过太多考虑，就开始做了起来。

对于翻译的原则，我是以严格尊重原著的表达方式为主，再根据中文的语言习惯和方便理解为目的进行微量的语法调整。出于对科学的谨慎和尊重（其实也是对自己的英语能力的担忧哈哈，毕竟只是六级水准），对于语法结构比较复杂的长句，我都会反复斟酌和理解，力求没有偏译。对于一些特定的专有名词，我也尽可能查阅相关资料寻求中文环境中最普遍的对应词汇。

希望我的这点工作能对一些人产生帮助。
如果对某些翻译存在异议，或者您也想加入我一起进行苹果文档的翻译工作，随时欢迎留言 ：D

<!-- more -->
---

A Short Practical Guide to Blocks
==
## “块”的简要实践指南

*原文地址：[iOS Developer Library](https://developer.apple.com/library/ios/featuredarticles/Short_Practical_Guide_Blocks/index.html#//apple_ref/doc/uid/TP40009758-CH1-SW1)*

*译者：[Canvas Hsu](http://gogocav.github.io)*

*日期：2014-10-11*


Blocks are a powerful C-language feature that is part of Cocoa application development. They are similar to “closures” and “lambdas” you may find in scripting and programming languages such as Ruby, Python, and Lisp. Although the syntax and storage details of blocks might at first glance seem cryptic, you’ll find that it’s actually quite easy to incorporate blocks into your projects’ code.

> “块”作为 Cocoa 应用开发的一部分，是一个非常强大的 C 语言特性。它很像你在 Ruby、Python、Lisp 等脚本语言和编程语言中碰到的 “闭包”和“匿名函数”。虽然块的语法和存储细节第一眼看上去很晦涩，最终你将发现在你的项目编码中使用块实际上是非常简单的。

The following discussion gives a high-level survey of the features of blocks and illustrates the typical ways in which they are used. Refer to [Blocks Programming Topics](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502) for the definitive description of blocks.

> 接下来的讨论将从高层次观察一些块的特性，并描绘一些块的典型用法。关于块的明确说明，可以参阅[块编程主题](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502)文档。

### Contents:

- Why Use Blocks?
- Blocks in the System Framework APIs
- Blocks and Concurrency

#### Why Use Blocks?
#### 为什么用块？

Blocks are objects that encapsulate a unit of work—or, in less abstract terms, a segment of code—that can be executed at any time. They are essentially portable and anonymous functions that one can pass in as arguments of methods and functions or that can be returned from methods and functions. Blocks themselves have a typed argument list and may have inferred or declared returned type. You may also assign a block to a variable and then call it just as you would a function.
>块是一个封装了一组任务的对象，稍微具体一点说，是一段代码，它能随时被执行。块本质上是具有可移植性、并可以作为方法或函数的入参或者返回值的匿名函数。块自身有一个指定类型的参数列表，也可以有可推断或声明类型的返回值。你还可以将块分配给一个变量，然后像调用函数一样调用块。

The caret symbol (^) is used as a syntactic marker for blocks. For example, the following code declares a block variable taking two integers and returning an integer value. It provides the parameter list after the second caret and the implementing code within the braces, and assigns these to the Multiply variable:

```objc
int (^Multiply)(int, int) = ^(int num1, int num2) {
    return num1 * num2;
};
int result = Multiply(7, 4); // Result is 28.
```

As method or function arguments, blocks are a type of callback and could be considered a form of delegation limited to the method or function. By passing in a block, calling code can customize the behavior of a method or function. When invoked, the method or function performs some work and, at the appropriate moments, calls back to the invoking code—via the block—to request additional information or to obtain application-specific behavior from it.

> 当块作为方法或函数的参数时，是一种回调的形式，可以被理解为方法或函数的一种委托形式。通过传入一个块，调用代码可以定制方法或函数的行为表现。方法或函数被调用后完成特定的工作，然后在合适的时机通过块来回调调用方法或函数的代码，通过这种方式请求一些额外的信息或者申请特定的行为许可。

An advantage of blocks as function and method parameters is that they enable the caller to provide the callback code at the point of invocation. Because this code does not have to be implemented in a separate method or function, your implementation code can be simpler and easier to understand. Take notifications of the [NSNotification](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSNotification_Class/index.html#//apple_ref/occ/cl/NSNotification) variety as an example. In the “traditional” approach, an object adds itself as an observer of a notification and then implements a separate method (identified by a selector in the addObserver:.. method) to handle the notification:

> 将块作为函数和方法的参数的一个好处是：块让调用者提供了函数和方法在被援引时的回调代码。因为这部分代码不需要在单独的方法或函数中实现，所以你的实现代码会更简单易懂。下面以 [NSNotification](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSNotification_Class/index.html#//apple_ref/occ/cl/NSNotification) 的通知变量为例。“传统”的实现方法是，一个对象将自身添加为一个通知的观察者，然后单独定义一个方法（通过 addObserver:.. 方法中的 selector 指定）来处理通知：

```objc
- (void)viewDidLoad {
   [super viewDidLoad];
    [[NSNotificationCenter defaultCenter] addObserver:self
        selector:@selector(keyboardWillShow:)
        name:UIKeyboardWillShowNotification object:nil];
}
 
- (void)keyboardWillShow:(NSNotification *)notification {
    // Notification-handling code goes here.
}
```

With the [addObserverForName:object:queue:usingBlock:](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSNotificationCenter_Class/index.html#//apple_ref/occ/instm/NSNotificationCenter/addObserverForName:object:queue:usingBlock:) method you can consolidate the notification-handling code with the method invocation:

> 通过使用[addObserverForName:object:queue:usingBlock:](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSNotificationCenter_Class/index.html#//apple_ref/occ/instm/NSNotificationCenter/addObserverForName:object:queue:usingBlock:)这个方法，你可以将通知处理操作以块的形式整合到方法的调用上。

```objc
- (void)viewDidLoad {
    [super viewDidLoad];
    [[NSNotificationCenter defaultCenter] addObserverForName:UIKeyboardWillShowNotification
        object:nil queue:[NSOperationQueue mainQueue]
        usingBlock:^(NSNotification *notif) {
            // Notification-handling code goes here. 
    }];
}
```

An even more valuable advantage of blocks over other forms of callback is that a block shares data in the local lexical scope. If you implement a method and in that method define a block, the block has access to the local variables and parameters of the method (including stack variables) as well as to functions and global variables, including instance variables. This access is read-only by default, but if you declare a variable with the __block modifier, you can change its value within the block. Even after the method or function enclosing a block has returned and its local scope is destroyed, the local variables persist as part of the block object as long as there is a reference to the block.

> 相较于其他形式的回调，块有一个优势，那就是一个块可以在本地词汇范围内共享数据。如果你实现一个方法，然后在这个方法中定义一个块，这个块就可以访问这个方法里的本地变量和参数（包括栈里的变量），就和它能访问函数、全局变量，包括市里变量一样。这种访问默认是只读的，但是你可以在声明一个变量时使用 __block 修饰词，这样的变量就可以被块修改赋值。甚至当这个包含块的方法或函数已经经返回了、本地范围已经销毁了，只要这个块还被某个对象引用着，这些本地变量依然作为块对象的一部分继续存在。

#### Blocks in the System Framework APIs
#### 块在系统框架 API 中的应用

One obvious motivation for using blocks is that an increasing number of the methods and functions of the system frameworks take blocks as parameters. One can discern a half-dozen or so use cases for blocks in framework methods:

> 在系统框架中，块被越来越多的作为框架方法和函数的参数来使用。在以下这些框架方法中，可以看到非常多的块的使用案例：

- Completion handlers
- Notification handlers
- Error handlers
- Enumeration
- View animation and transitions
- Sorting

The sections that follow describe each of these cases. But before we get to that, here is a quick overview on interpreting block declarations in framework methods. Consider the following method of the [NSSet](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSSet_Class/index.html#//apple_ref/occ/cl/NSSet) class:

> 接下来的章节会逐一描述上述这些案例。在此之前，我们先快速的了解一下在框架方法中，块是怎样声明的。以 [NSSet](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSSet_Class/index.html#//apple_ref/occ/cl/NSSet) 类的一个方法为例：

```objc
- (NSSet *)objectsPassingTest:(BOOL (^)(id obj, BOOL *stop))predicate
```

The block declaration indicates that the method passes into the block (for each enumerated item) a dynamically typed object and a by-reference Boolean value; the block returns a Boolean value. (What these parameters and return value are actually for are covered in [Enumeration](https://developer.apple.com/library/ios/featuredarticles/Short_Practical_Guide_Blocks/index.html#//apple_ref/doc/uid/TP40009758-CH1-SW1).) When specifying your block, begin with a caret (^) and the parenthesized argument list; follow this with the block code itself, enclosed by braces.

> 从块的声明可以看到，这个方法对每一个枚举项都向块传入两个参数：一个动态类型的对象和一个布尔类型的指针；块再返回一个布尔类型的值。（关于这些参数和返回值的具体含义请参考 [Enumeration](https://developer.apple.com/library/ios/featuredarticles/Short_Practical_Guide_Blocks/index.html#//apple_ref/doc/uid/TP40009758-CH1-SW1) 章节。）块的声明是用一个脱字号(^)开头，再用圆括号包住参数列表；之后用花括号包住块体的代码。

```objc
[mySet objectsPassingTest:^(id obj, BOOL *stop) {
    // Code goes here: Return YES if obj passes the test and NO if obj does not pass the test.
}];
```

#### Completion and Error Handlers
#### 完成和错误处理程序

Completion handlers are callbacks that allow a client to perform some action when a framework method or function completes its task. Often the client uses a completion handler to free state or update the user interface. Several framework methods let you implement completion handlers as blocks (instead of, say, delegation methods or notification handlers).

> 完成处理程序(comletion handlers)是回调程序，客户端可以通过它在一个框架方法或函数执行完毕以后再进行一些处理。通常客户端用完成处理函数来释放空间或更新用户界面。有一些框架方法允许你使用块来作为完成处理程序（代替通常被作为完成处理程序的代理方法或者通知程序）。

The [UIView](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/cl/UIView) class has several class methods for animations and view transitions that have completion-handler block parameters. ( [View Animation and Transitions](https://developer.apple.com/library/ios/featuredarticles/Short_Practical_Guide_Blocks/index.html#//apple_ref/doc/uid/TP40009758-CH1-SW31) gives an overview of these methods.) The example in Listing 1-1 shows an implementation of the [animateWithDuration:animations:completion:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/clm/UIView/animateWithDuration:animations:completion:) method. The completion handler in this example resets the animated view back to its original location and transparency (alpha) value a few seconds after the animation concludes.

> [UIView](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/cl/UIView) 类有一些操作动画和视图过渡的类方法可以使用块作为完成处理程序。（在 [视图动画和过渡](https://developer.apple.com/library/ios/featuredarticles/Short_Practical_Guide_Blocks/index.html#//apple_ref/doc/uid/TP40009758-CH1-SW31) 章节有对这些方法的概述。）Listing 1-1 的例子展示了 [animateWithDuration:animations:completion:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/clm/UIView/animateWithDuration:animations:completion:) 方法是如何实现的。在这个例子中，完成处理程序在视图动画完成以后，重置该视图的初始位置和透明度。

**Listing 1-1** A completion-handler block

```objc
- (IBAction)animateView:(id)sender {
    CGRect cacheFrame = self.imageView.frame;
    [UIView animateWithDuration:1.5 animations:^{
        CGRect newFrame = self.imageView.frame;
        newFrame.origin.y = newFrame.origin.y + 150.0;
        self.imageView.frame = newFrame;
        self.imageView.alpha = 0.2;
    }
                    completion:^ (BOOL finished) {
                        if (finished) {
                            // Revert image view to original.
                            self.imageView.frame = cacheFrame;
                            self.imageView.alpha = 1.0;
                         }
    }];
}
```

Some framework methods have error handlers, which are block parameters similar to completion handlers. The method invokes them (and passes in an [NSError](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSError_Class/index.html#//apple_ref/occ/cl/NSError) object) when it cannot complete its task because of some error condition. You typically implement an error handler to inform the user about the error.

> 类似的，有一些框架方法也支持用块作为错误处理程序（error handlers）。这些方法在程序因为某些错误无法完成时调用块（并传入一个 [NSError](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSError_Class/index.html#//apple_ref/occ/cl/NSError) 类型的对象）。通常错误处理程序用来将错误通知给用户。

#### Notification Handlers
#### 通知处理程序

The NSNotificationCenter method [addObserverForName:object:queue:usingBlock:](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSNotificationCenter_Class/index.html#//apple_ref/occ/instm/NSNotificationCenter/addObserverForName:object:queue:usingBlock:) lets you implement the handler for a notification at the point you set up the observation. Listing 1-2 illustrates how you might call this method, defining a block handler for the notification. As with notification-handler methods, an [NSNotification](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSNotification_Class/index.html#//apple_ref/occ/cl/NSNotification) object is passed in. The method also takes an [NSOperationQueue](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/NSOperationQueue_class/index.html#//apple_ref/occ/cl/NSOperationQueue) instance, so your application can specify an execution context on which to run the block handler.

> NSNotificationCenter 类的 [addObserverForName:object:queue:usingBlock:](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSNotificationCenter_Class/index.html#//apple_ref/occ/instm/NSNotificationCenter/addObserverForName:object:queue:usingBlock:) 方法允许你对某个通知设定一个处理程序，该程序在你为这个通知设置了观察者时执行。Listing 1-2 的例子展示了如何在调用这个方法时用块作为通知处理程序。通知处理方法会传入一个 [NSNotification](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSNotification_Class/index.html#//apple_ref/occ/cl/NSNotification) 对象到块里。同时该方法还有一个[NSOperationQueue](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/NSOperationQueue_class/index.html#//apple_ref/occ/cl/NSOperationQueue) 类型的实例参数，该实例让应用知道在怎样的特定环境下执行这个块。

**Listing 1-2** Adding an object as an observer and
handling a notification using a block

```objc
- (void)applicationDidFinishLaunching:(NSNotification *)aNotification {
    opQ = [[NSOperationQueue alloc] init];
    [[NSNotificationCenter defaultCenter] addObserverForName:@"CustomOperationCompleted"
            object:nil queue:opQ
        usingBlock:^(NSNotification *notif) {
        NSNumber *theNum = [notif.userInfo objectForKey:@"NumberOfItemsProcessed"];
        NSLog(@"Number of items processed: %i", [theNum intValue]);
    }];
}
```

#### Enumeration
#### 枚举

The collection classes of the Foundation framework—[NSArray](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSArray_Class/index.html#//apple_ref/occ/cl/NSArray), [NSDictionary](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSDictionary_Class/index.html#//apple_ref/occ/cl/NSDictionary), [NSSet](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSSet_Class/index.html#//apple_ref/occ/cl/NSSet), and [NSIndexSet](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSIndexSet_Class/index.html#//apple_ref/occ/cl/NSIndexSet)—declare methods that perform the enumeration of a particular type of collection and that specify blocks for clients to supply code to handle or test each enumerated item. In other words, the methods perform the equivalent of the fast-enumeration construct:

> Foundation 框架中的集合类型（[NSArray](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSArray_Class/index.html#//apple_ref/occ/cl/NSArray)， [NSDictionary](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSDictionary_Class/index.html#//apple_ref/occ/cl/NSDictionary)， [NSSet](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSSet_Class/index.html#//apple_ref/occ/cl/NSSet)，和 [NSIndexSet](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSIndexSet_Class/index.html#//apple_ref/occ/cl/NSIndexSet)）声明了一些针对特定集合的枚举方法，这些方法支持客户端利用块代码来操作或测试每一个枚举项。换句话说，这些方法实现了和下面这种快速枚举结构相同的功能：

```objc
 for (id item in collection) {
    // Code to operate on each item in turn.
}
```

There are two general forms of the enumeration methods that take blocks. The first are methods whose names begin with enumerate and do not return a value. The block for these methods performs some work on each enumerated item. The block parameter of the second type of method is preceded by passingTest; this kind of method returns an integer or an NSIndexSet object. The block for these methods performs a test on each enumerated item and returns YEStrue if the item passes the test. The integer or index set identifies the object or objects in the original collection that passed the test.

> 枚举方法使用块的形式主要有两种。第一种是以 enumerate 开头命名、并且没有返回值的方法。块在这些方法中对每一个枚举项都执行一些操作。第二种形式是块以名为 passingTest 的参数形式传入方法，这一类方法返回一个整型数值或者一个 NSIndexSet 对象。在这类方法中，块对每一个枚举项都执行一套测试，然后对测试通过的枚举项返回 YEStrue 。方法的整型数值或 NSIndexSet 对象返回值标识出原来的集合中通过了测试的对象或对象集。

The code in Listing 1-3 calls an NSArray method of each of these types. The block of the first method (a “passing test” method) returns YEStrue for every string in an array that has a certain prefix. The code subsequently creates a temporary array using the index set returned from the method. The block of the second method trims away the prefix from each string in the temporary array and adds it to a new array.

> 例子 Listing 1-3 中的代码调用了一个 NSArray 的方法，案例中对上述两种应用形式都有使用。在第一个方法中（一个”通过测试”的方法），块对数组中每一个以特定前缀命名的字符串对象返回 YEStrue。接下来的代码通过从第一个方法中得到的索引组创建了一个临时数组。第二个方法中的块枚举临时数组中的每一个字符串，去掉它们的前缀再放入一个新数组中。

**Listing 1-3** Processing enumerated arrays using two blocks

```objc
NSString *area = @"Europe";
NSArray *timeZoneNames = [NSTimeZone knownTimeZoneNames];
NSMutableArray *areaArray = [NSMutableArray arrayWithCapacity:1];
NSIndexSet *areaIndexes = [timeZoneNames indexesOfObjectsWithOptions:NSEnumerationConcurrent
                                passingTest:^(id obj, NSUInteger idx, BOOL *stop) {
    NSString  *tmpStr = (NSString *)obj;
    return [tmpStr hasPrefix:area];
}];
 
NSArray *tmpArray = [timeZoneNames objectsAtIndexes:areaIndexes];
[tmpArray enumerateObjectsWithOptions:NSEnumerationConcurrent|NSEnumerationReverse
                           usingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
                               [areaArray addObject:[obj substringFromIndex:[area length]+1]];
}];
NSLog(@"Cities in %@ time zone:%@", area, areaArray);
```

The stop parameter in each of these enumeration methods (which is not used in the example) allows the block to pass YEStrue by reference back to the method to tell it to quit enumerating. You do this when you just want to find the first item in the collection that matches some criteria.

> 每一个上面列举的这种枚举方法中都有一个名为 stop 的参数（在上个例子中没有被使用到），这个参数允许块通过引用回传 YEStrue 给方法，让方法退出枚举。当你只需要找出第一个符合特定条件的项时可以这么做。 

Even though it does not represent a collection, the [NSString](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/index.html#//apple_ref/occ/cl/NSString) class also has two methods with block parameters whose names begin with enumerate: [enumerateSubstringsInRange:options:usingBlock:](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/index.html#//apple_ref/occ/instm/NSString/enumerateSubstringsInRange:options:usingBlock:) and [enumerateLinesUsingBlock:](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/index.html#//apple_ref/occ/instm/NSString/enumerateLinesUsingBlock:). The first method enumerates a string by a text unit of a specified granularity (line, paragraph, word, sentence, and so on); the second method enumerates it by line only. Listing 1-4 illustrates how you might use the first method.

> 虽然 [NSString](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/index.html#//apple_ref/occ/cl/NSString) 不是表现集合的类，但它也有两个以 enumerate 开头命名的包含块参数的方法：
[enumerateSubstringsInRange:options:usingBlock:](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/index.html#//apple_ref/occ/instm/NSString/enumerateSubstringsInRange:options:usingBlock:) 和 [enumerateLinesUsingBlock:](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSString_Class/index.html#//apple_ref/occ/instm/NSString/enumerateLinesUsingBlock:) 。第一个方法通过指定粒度（例如行、段落、单词、句子等等）将一个字符串分成一个个文本单元进行枚举；第二个方法则以行为单元来枚举字符串。Listing 1-4 的案例描绘了第一个方法的应用场景。

**Listing 1-4** Using a block to find matching substrings in a string

```objc
NSString *musician = @"Beatles";
NSString *musicDates = [NSString stringWithContentsOfFile:
    @"/usr/share/calendar/calendar.music"
    encoding:NSASCIIStringEncoding error:NULL];
[musicDates enumerateSubstringsInRange:NSMakeRange(0, [musicDates length]-1)
    options:NSStringEnumerationByLines
    usingBlock:^(NSString *substring, NSRange substringRange, NSRange enclosingRange, BOOL *stop) {
           NSRange found = [substring rangeOfString:musician];
           if (found.location != NSNotFound) {
                NSLog(@"%@", substring);
           }
      }];
```

#### View Animation and Transitions
#### 视图动画和过渡

The [UIView](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/cl/UIView) class in iOS 4.0 introduced several class methods for animation and view transitions that take blocks. The block parameters are of two kinds (not all methods take both kinds):
- Blocks that change the view properties to be animated
- Completion handlers

> iOS 4.0 的 [UIView](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/cl/UIView) 类中新增了几个关于动画和视图过渡的类方法，在这些类方法中使用到了块。这些块参数分为两种类型（不是所有的方法都同时使用到了这两种类型）：
 - 通过块改变视图的属性来形成动画
 - 将块做为完成处理程序

Listing 1-5 shows an invocation of [animateWithDuration:animations:completion:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/clm/UIView/animateWithDuration:animations:completion:), a method that has both kinds of block parameters. In this example, the animation makes the view disappear (by specifying zero alpha) and the completion handler removes it from its superview.

> Listing 1-5 展示了一个调用 [animateWithDuration:animations:completion:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/clm/UIView/animateWithDuration:animations:completion:) 方法的例子，这个方法中同时包含了上述两种块参数的应用类型。在这个例子中，动画让视图消失（通过调整透明度为 0 ）然后完成处理程序将视图从父视图中移除。

**Listing 1-5** Simple animation of a view using blocks

```objc
[UIView animateWithDuration:0.2 animations:^{
        view.alpha = 0.0;
    } completion:^(BOOL finished){
        [view removeFromSuperview];
    }];
```

Other UIView class methods perform transitions between two views, including flips and curls. The example invocation of [transitionWithView:duration:options:animations:completion:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/clm/UIView/transitionWithView:duration:options:animations:completion:) in Listing 1-6 animates the replacement of a subview as a flip-left transition. (It does not implement a completion handler.)

> 其他 UIView 类的方法实现两个视图之间的过渡，比如翻转和弯曲。案例 Listing 1-6 调用 [transitionWithView:duration:options:animations:completion:](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/clm/UIView/transitionWithView:duration:options:animations:completion:) 方法实现了一个子视图向左翻转消失的动画效果。（这个方法没有实现完成处理程序）

**Listing 1-6** Implementing a flip transition between two views

```objc
[UIView transitionWithView:containerView duration:0.2
                   options:UIViewAnimationOptionTransitionFlipFromLeft
                animations:^{
                    [fromView removeFromSuperview];
                    [containerView addSubview:toView]
                }
                completion:NULL];
```

#### Sorting
#### 排序

The Foundation framework declares the [NSComparator](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Miscellaneous/Foundation_DataTypes/index.html#//apple_ref/c/tdef/NSComparator) type for comparing two items:

> 在 Foundation 框架中有一个 [NSComparator](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Miscellaneous/Foundation_DataTypes/index.html#//apple_ref/c/tdef/NSComparator) 类型用于比较两个项目：

```objc
typedef NSComparisonResult (^NSComparator)(id obj1, id obj2);
```

NSComparator is a block type that takes two objects and returns an [NSComparisonResult](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Miscellaneous/Foundation_Constants/index.html#//apple_ref/c/tdef/NSComparisonResult) value. It is a parameter in methods of [NSSortDescriptor](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSSortDescriptor_Class/index.html#//apple_ref/occ/cl/NSSortDescriptor), [NSArray](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSArray_Class/index.html#//apple_ref/occ/cl/NSArray), and [NSDictionary](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSDictionary_Class/index.html#//apple_ref/occ/cl/NSDictionary) and is used by instances of those classes for sorting. Listing 1-7 gives an example of its use.

> NSComparator 是一个块的类型，它获得两个对象并返回一个[NSComparisonResult](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Miscellaneous/Foundation_Constants/index.html#//apple_ref/c/tdef/NSComparisonResult) 类的值。它是
[NSSortDescriptor](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSSortDescriptor_Class/index.html#//apple_ref/occ/cl/NSSortDescriptor)， [NSArray](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSArray_Class/index.html#//apple_ref/occ/cl/NSArray)，和 [NSDictionary](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSDictionary_Class/index.html#//apple_ref/occ/cl/NSDictionary) 这些类的方法的参数，被这些类的实例用于排序。Listing 1-7 展示了这种应用。

**Listing 1-7** Sorting an array using an NSComparator block

```objc
NSArray *stringsArray = [NSArray arrayWithObjects:
                                 @"string 1",
                                 @"String 21",
                                 @"string 12",
                                 @"String 11",
                                 @"String 02", nil];
static NSStringCompareOptions comparisonOptions = NSCaseInsensitiveSearch | NSNumericSearch |
        NSWidthInsensitiveSearch | NSForcedOrderingSearch;
NSLocale *currentLocale = [NSLocale currentLocale];
NSComparator finderSort = ^(id string1, id string2) {
    NSRange string1Range = NSMakeRange(0, [string1 length]);
    return [string1 compare:string2 options:comparisonOptions range:string1Range locale:currentLocale];
};
NSLog(@"finderSort: %@", [stringsArray sortedArrayUsingComparator:finderSort]);
```

This example is taken from [Blocks Programming Topics](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502).

> 这个例子来自 [块编程主题](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502) 文档。

#### Blocks and Concurrency
#### 块和并发

Blocks are portable and anonymous objects encapsulating a unit of work that can be performed asynchronously at a later time. Because of this essential fact, blocks are a central feature of both Grand Central Dispatch (GCD) and the  [NSOperationQueue](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/NSOperationQueue_class/index.html#//apple_ref/occ/cl/NSOperationQueue) class, the two recommended technologies for concurrent processing.

> 块具有可移植性，并且使用匿名对象来封装一段任务代码，可以被异步地调用。因此，块是中央调度（Grand Central Dispatch (GCD)）和   [NSOperationQueue](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/NSOperationQueue_class/index.html#//apple_ref/occ/cl/NSOperationQueue) 类这两个推荐的并发处理技术的主要特色。

- The two central functions of GCD, [dispatch_sync(3) OS X Developer Tools Manual Page](https://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/func/dispatch_sync) (for synchronous dispatching) or [dispatch_async(3) OS X Developer Tools Manual Page](https://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/func/dispatch_async) (for asynchronous dispatching) take as their second arguments a block.
- An NSOperationQueue is an object that schedules tasks to be performed concurrently or in an order defined by dependency relationships. The tasks are represented by [NSOperation](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/NSOperation_class/index.html#//apple_ref/occ/cl/NSOperation) objects, which frequently use blocks to implement their tasks.

> - 中央调度（GCD）的两个核心函数[dispatch_sync(3) OS X Developer Tools Manual Page](https://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/func/dispatch_sync) （用于同步调度）和 [dispatch_async(3) OS X Developer Tools Manual Page](https://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/func/dispatch_async) （用于异步调度）都采用块作为它们的第二个参数。

> - 通过 NSOperationQueue 对象，调度任务可以被同时执行，或由依赖关系所限定的顺序依次执行。

For more about GCD, NSOperationQueue, and NSOperation, see [Concurrency Programming Guide](https://developer.apple.com/library/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091).

> 更多关于 GCD、NSOperationQueue 和 NSOperation 的信息，可以参阅 [Concurrency Programming Guide](https://developer.apple.com/library/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008091) 文档。

-END-