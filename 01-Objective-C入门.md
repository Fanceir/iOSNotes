# Objective-C 入门

首先，Objective-C 是用于 iOS 开发的语言，和 C 语言的区别主要有`#import`，`@interface`，`@end`，`@property`，`@synthesize`，`@implementation`等关键字。Objective-C 是一种面向对象的编程语言，它是在 C 语言的基础上添加了 Smalltalk 式的消息传递机制。

开始第一次看到 OC 神奇的样式还是很让人奇怪的，它的调用不像其他语言那样直接，当第一次看到`[object method]`时，心里就想，这是什么鬼？为什么要用中括号？世界上怎么有这么奇怪的语言。

下面简述 OC 的一些基本语法

## NSLog 函数

NSLog 函数使用来打印东西的可以理解为 JS 中的 console.log

```objective-c
NSLog(@"Hello World");
```

## 变量

基本是兼容 C 语言的，说几个特有的 NSString 和 NSNumber

```objective-c
NSString *str = @"Hello World";
NSNumber *num = [NSNumber numberWithInt:10];
```

## 面向对象

面向对象部分的内容还是比较多的，这其实是一个重点。

```objective-c
// 定义一个类 .h 文件
@interface Person : NSObject
@property (nonatomic, strong) NSString *name;
@property (nonatomic, assign) int age;
- (void)printInfo;
// 以下两个方法定义可以省略，因为使用@property后会自动生成
// - (int) age;
// - (NSString *) name;
+ (instancetype)personWithName:(NSString *)name age:(int)age;
@end
```

```objective-c
// 实现类 .m 文件
#import "Person.h"
@implementation Person
- (void)printInfo {
    NSLog(@"Name: %@, Age: %d", self.name, self.age);
}
// 注意：使用@property后，编译器会自动生成这些getter方法
// 只有在需要自定义实现时才需要重写
- (int)age {
    return _age;
}
- (NSString *)name {
    return _name;
}
+ (instancetype)personWithName:(NSString *)name age:(int)age {
    Person *person = [[Person alloc] init];
    person.name = name;
    person.age = age;
    return person;
}
@end
```

- `@implementation`表示实现类，`@interface`表示声明类。
- `@property`表示属性，`@synthesize`表示合成属性。在现代 OC 中，`@synthesize`通常不需要显式写出，编译器会自动为`@property`生成对应的实例变量和存取方法。
- `@end`表示结束
- `@protocol`表示协议，类似于接口。

### 类中的方法

下面来具体解析一下类中的方法

```objective-c
- (void) eatsomeFood:(NSString *)food  drinkSome:(NSString *)drink {
    NSLog(@"%@ is eating %@ and drinking %@", self.name, food, drink);
}
//调用
[self eatsomeFood:@"apple" drinkSome:@"water"];
```

这里的 `-` 是指方法的类型标识符号，其中需要注意的是`+`和`-`的区别，`+`表示类方法，`-`表示实例方法。
一般在定义类的时候，都是用`-`来定义实例方法，`+`来定义类方法。
然后需要使用括号包裹返回值类型，这里的`food`和`drink`是参数名，表示传入的参数。

方法的调用是通过`[object method]`来调用的，object 是实例对象，method 是方法名。这种调用方式实际上是 Objective-C 特有的"消息传递"(message passing)机制，而不仅仅是简单的方法调用。这种消息传递机制允许 Objective-C 实现运行时的动态绑定，是其区别于其他语言的重要特性。

### self 语法

当我们需要在实现的过程中调用当前类中的方法时就可以使用`self`，它表示当前对象的实例。

```objective-c
- (void)jump{
    NSLog(@"%@ is jumping", self.name);
}
- (void)run{
    NSLog(@"%@ is running", self.name);
    [self jump]; // 调用当前对象的 jump 方法
}
```

这里需要注意的是，`self` 是一个指针，指向当前对象的实例，所以可以直接调用当前对象的方法。

## OC 中的单例模式

有的时候一个类在一个系统中只能创建一个实例，这个时候就需要使用单例模式来创建一个类的实例。单例模式的实现方式有很多种，下面是其中一种比较常见的实现方式。

```objective-c
+ (instancetype)sharedInstance {
    // 1. 创建一个静态变量，用来保存单例对象
    static MySingleton *sharedInstance = nil;
    // 2. 使用 dispatch_once 确保只执行一次
    static dispatch_once_t onceToken;
    // 3. 创建单例对象
    dispatch_once(&onceToken, ^{
        sharedInstance = [[self alloc] init];
    });
    return sharedInstance;
}
```

这里的 `dispatch_once` 是 GCD (Grand Central Dispatch) 的一部分，它是 iOS 和 macOS 中用于并行计算的 API。使用 `dispatch_once` 可以确保代码块只执行一次，这是一种线程安全的实现方式，能够避免多线程环境下可能出现的问题。

## 内存管理

在 Objective-C 中，内存管理主要有两种方式：手动引用计数（MRC）和自动引用计数（ARC）。现代 iOS 开发主要使用 ARC。

在 ARC 下，属性声明时的内存管理修饰符非常重要：

- `strong`: 强引用，默认的引用类型，会增加对象的引用计数
- `weak`: 弱引用，不会增加引用计数，当对象被释放时，指针会自动设为 nil
- `assign`: 用于基本数据类型（int, float 等），不适用于对象类型（会造成野指针）
- `copy`: 会创建被引用对象的副本

```objective-c
@property (nonatomic, strong) NSString *name; // 强引用
@property (nonatomic, weak) UIView *weakView; // 弱引用
@property (nonatomic, assign) int age; // 用于基本数据类型
@property (nonatomic, copy) NSString *copiedName; // 创建副本
```

其中 `nonatomic` 表示非原子性，不保证线程安全但性能更好，常用于 UI 元素；`atomic` 是默认值，提供基本的线程安全。
