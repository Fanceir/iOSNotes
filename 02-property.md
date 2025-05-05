# Property 和访问控制符号

## 访问控制符

有四个类型分别是`@private` `@protected` `@public` `@package`
其中`@private`表示私有属性，`@protected`表示保护属性，`@public`表示公共属性，`@package`表示包属性。
一般情况下我们使用的是`@private`和`@protected`，这两个属性的区别在于访问权限的不同，前者只能在类内部访问，后者可以在子类中访问。

### public

public 可以在任何地方被访问
如果一个类的属性是 public 的，那么这个属性可以在任何地方被访问，包括类的外部和子类中。

```objective-c
// Person.h
#import <Foundation/Foundation.h>
@interface Person : NSObject{
    @public
    NSString *name;
    int age;
}
@end

// Person.m

#import "Person.h"
@implementation Person
@end

// main.m
#import <Foundation/Foundation.h>
#import "Person.h"
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Person *person = [[Person alloc] init];
        person->name = @"John Doe";
        person->age = 30;
        NSLog(@"Name: %@, Age: %d", person->name, person->age);
    }
    return 0;
}
```

上面就是一个使用`public`的例子，`name`和`age`都是`public`的属性，所以可以在类的外部访问。

### private

private 只能在类的内部被访问，也不能够在子类中访问。

```objective-c
// Person.h
#import <Foundation/Foundation.h>
@interface Person : NSObject{
    @private
    NSString *name;
    int age;
}
- (void)setName:(NSString *)name;
- (void)setAge:(int)age;
- (NSString *)getName;
- (int)getAge;
@end
// Person.m
#import "Person.h"
@implementation Person
- (void)setName:(NSString *)name {
    self->name = name;
}
- (void)setAge:(int)age {
    self->age = age;
}
- (NSString *)getName {
    return self->name;
}
- (int)getAge {
    return self->age;
}
@end
// main.m
#import <Foundation/Foundation.h>
#import "Person.h"
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Person *person = [[Person alloc] init];
        [person setName:@"John Doe"];
        [person setAge:30];
        NSLog(@"Name: %@, Age: %d", [person getName], [person getAge]);
    }
    return 0;
}
```

上面就是一个使用`private`的例子，`name`和`age`都是`private`的属性，所以不能在类的外部访问，只能通过方法来访问。

### protected

默认使用的访问控制符是`@protected`，也就是保护属性。
protected 属性可以在类的内部和子类中被访问，但是不能在类的外部访问。

```objective-c
// Person.h
#import <Foundation/Foundation.h>
@interface Person : NSObject{
    @protected
    NSString *name;
    int age;
}
- (void)setName:(NSString *)name;
- (void)setAge:(int)age;
- (NSString *)getName;
- (int)getAge;
@end
// Person.m
#import "Person.h"
@implementation Person
- (void)setName:(NSString *)name {
    self->name = name;
}
- (void)setAge:(int)age {
    self->age = age;
}
- (NSString *)getName {
    return self->name;
}
- (int)getAge {
    return self->age;
}
@end
// Student.h
#import <Foundation/Foundation.h>
#import "Person.h"
@interface Student : Person
- (void)printInfo;
@end
// Student.m
#import "Student.h"
@implementation Student
- (void)printInfo {
    NSLog(@"Name: %@, Age: %d", self->name, self->age);
}
@end
// main.m
#import <Foundation/Foundation.h>
#import "Person.h"
#import "Student.h"
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        Student *student = [[Student alloc] init];
        [student setName:@"John Doe"];
        [student setAge:30];
        [student printInfo];
    }
    return 0;
}
```

上面就是一个使用`protected`的例子，`name`和`age`都是`protected`的属性，所以可以在类的内部和子类中访问，但是不能在类的外部访问。

### package

package 只能在类的内部和同一个包中的类中被访问，不能够在类的外部访问。这是一个介于`private`和`protected`之间的访问控制符。

## setter 和 getter

注意到上面的例子中，`name`和`age`都是直接访问的，这种方式是不推荐的，应该使用`setter`和`getter`方法来访问属性。
`setter`方法用于设置属性的值，`getter`方法用于获取属性的值。

这和之前 js 的`get`和`set`是一样的，下面是一个使用`setter`和`getter`方法的例子：

```objective-c
- (void) setNumber:(int)number {
    _number = number;
}
- (int) getNumber {
    return _number;
}
```

这个很好的反映了面向对象中的封装性，使用`setter`和`getter`方法来访问属性，可以在方法中添加一些逻辑，比如判断属性的值是否合法等。
比如当输入的值不合法时，抛出异常：

```objective-c
- (void) setNumber:(int)number {
    if (number < 0) {
        @throw [NSException exceptionWithName:@"InvalidNumberException" reason:@"Number must be non-negative" userInfo:nil];
    }
    _number = number;
}
```

## `@property` 的语法

```objective-c
@property (可选属性) type name;
```

`@property` 是一个关键字，用于定义属性，后面跟着属性的类型和名称。

可选属性有`nonatomic` `readonly` `copy` `strong` `weak` `assign` `readwrite`等。

- 多线程相关`nonatomic`：表示属性不是线程安全的，默认是`atomic`，表示属性是线程安全的。`nonatomic`在多线程环境下多个线程同时访问这个属性的时候是不会进行加锁操作的，可能导致数据不一致。`atomic`在多线程环境下多个线程同时访问这个属性的时候是会进行加锁操作的，可能导致性能下降。
- 读取权限`readonly`：表示属性是只读的，不能被修改，默认是`readwrite`，表示属性是可读可写的，编译器回自动生成`setter`和`getter`方法，允许对属性进行读写操作。
- 强弱引用：`strong`表示强引用，`weak`表示弱引用，`assign`表示赋值引用。强引用会增加对象的引用计数，弱引用不会增加对象的引用计数，赋值引用不会增加对象的引用计数，也不会设置为 nil。
- `assign`：表示属性是一个基本数据类型，比如`int` `float` `double`等，使用`assign`表示赋值引用。
- `copy`：表示属性是一个对象类型，比如`NSString` `NSArray` `NSDictionary`等，使用`copy`表示复制引用。使用`copy`可以避免字符串被修改，导致数据不一致的问题。

## `@synthesize` 的语法

如果使用 `@property` 定义了属性，编译器会自动生成`setter`和`getter`方法，但是如果需要自定义这些方法，可以使用 `@synthesize` 来指定属性的名称和类型。

```objective-c
//Person.h
#import <Foundation/Foundation.h>

@interface Person : NSObject
@property (nonatomic, strong) NSString *name;
@property (nonatomic, assign) int age;
@end

//Person.m
#import "Person.h"

@implementation Person
@synthesize name = _name;
@synthesize age = _age;
// 上面这两行代码表示将属性 name 和 age 的实例变量命名为 _name 和 _age
@end

// main.m
#import <Foundation/Foundation.h>
#import "Person.h"
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        //创建对象
        Person *person = [[Person alloc] init];
        //设置属性值
        //使用 setter 方法设置属性值
        [person setName:@"John Doe"];
        [person setAge:30];
        //获取属性值
        NSLog(@"Name: %@, Age: %d", [person name], [person age]);
    }
    return 0;
}

```
