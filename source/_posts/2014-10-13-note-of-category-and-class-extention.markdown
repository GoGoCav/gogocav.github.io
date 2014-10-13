---
layout: post
title: "笔记 - Category and Class Extention"
date: 2014-10-13 00:57:30 +0800
comments: true
categories: obj-c, note
---

Objective-C 中如果想对类进行自定义和功能扩展，有2种常用技术：

- category 类别
- class extention 类扩展

两者有相似之处，也有区别。


### 1. category 类别

Category 可以给任何类（包括不是由你自己实现的类，比如框架类）增加方法或函数。该类和子类的所有实例都可以使用它们。在运行时，这些方法和函数与类原生的方法和函数没有任何差别。
<!-- more -->
#### 语法形式：

*ClassName+XYZCategoryName.h* (命名形式仅供参考，下同)

```objc
@interface ClassName (XYZCategoryName)

# Category 不支持定义新的属性或实例变量

- (SomeClass *)xyz_someAdditionInstanceMethod;
+ (SomeClass *)xyz_someAdditionClassMethod; # Category 支持类方法

@end
```
</br>

*ClassName+XYZCategoryName.m*

```objc
#import "ClassName+XYZCategoryName.h"

@implementation ClassName (XYZCategoryName)
- (SomeClass *)xyz_someAdditionInstanceMethod {
    # implement code here
}

+ (SomeClass *)xyz_someAdditionClassMethod {
    # implement code here
}
@end
```

#### 应用场景：
 1. 为类扩充功能。
 2. 如果一个类的功能复杂，可以将功能分类、拆分到不同的 categories 中，分别在独立的文件中声明和实现。

#### 注意事项：
 1. Category 不支持声明新的类属性或实例变量。（类扩展技术 Class Extensions 支持）。
 2. Category 中新定义的方法或函数不能重名，建议对 category 名字和方法都增加前缀。

</br>
 
### 2. Class Extension 类扩展

Class Extension 顾名思义是对类进行扩展，可以为有实现代码的类再声明一些新的属性、实例参数、方法或函数。

类扩展又被称为匿名类别（anonymous category），因为类扩展的声明就如同声明一个名字为空的类别。也因此，类扩展新声明的方法和函数的实现，写在类本身的@implementation 代码段中。

类扩展的 @interface 声明部分通常放到类实现所在的文件中，以达到类扩展私有化的目的。这也是类扩展最常用的场景。如果实在需要被外部的类访问类扩展的方法，则可以将类扩展的接口声明放到单独的 .h 头文件中，被需要它的代码引用。

#### 语法形式：

```objc
@interface ClassName () { # 类扩展也被称为匿名类别（anonymous categories）
    id _someExtendInstanceVariable; # 类扩展支持添加实例变量
}

@property SomeClass *someExtendProperty; # 类扩展支持添加属性

- (SomeObject *)someExtendMethod;

@end

...

@implementation ClassName

...

- (SomeObject *)someExtendMethod {
    # implement code here
}
...

@end
```

#### 应用场景：
 1. 通过类扩展隐藏一些私有信息。
 例如想实现某个类属性对外只读、对内可写入的效果，则可以在类扩展中以 readwrite 重定义这个属性，为其添加写入权限，但将类扩展的接口声明放到类实现文件中，达到私有化的效果。因此做到了这个属性对外只读、对内可读写。
 2. 如果需要对某些特定的外部的类开通对类扩展私有方法或函数的访问，则可以将类扩展的接口声明放到独立的头文件中，供需要的代码引用。

#### 注意事项：
 1. 类扩展可以声明新的属性和实例变量。这一点和类别不同。
 2. 类扩展只能用在拥有实现源代码的类上面，像框架类这种没有实现源码的类是不能扩展的。类扩展声明的新属性和实例变量需要和类的方法实现同时编译，这样编译器才能合成新属性的存取方法和实例变量。
 3. 类扩展的实现部分直接放到类本身的 @implementation 区块中。
 4. 如果外部对象访问类扩展的私有属性或方法，编译器会报警。但其实也有办法通过动态运行时的特征（例如利用 NSObjet 的 performSelector:... 方法）实现外部对象对私有的类扩展方法的调用。

