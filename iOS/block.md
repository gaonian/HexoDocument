---
title: block解析
categories: iOS
---

## 简介

Blocks是C语言的扩充功能。可以用一句话来表示Blocks的扩充功能：带有自动变量（局部变量）的匿名函数。



```
int multiplier = 7;
int (^myBlock)(int) = ^(int num) {
	return num * multiplier;
};
```

![](https://raw.githubusercontent.com/gaonian/HexoDocument/master/iOS/block_img/block1.png)



### 声明

Block变量保存对block的引用，使用类似于声明函数指针的语法来声明他们，除了使用 ^ 代替 * ，block类型可以与c类型系统的其余部分完全互相操作。以下是所有有效的block变量声明：

```
void（^ blockReturningVoidWithVoidArgument）（void）;
int（^ blockReturningIntWithIntAndCharArguments）（int，char）;
void（^ arrayOfTenBlocksReturningVoidWithIntArgument [10]）（int）;
```

block还可以支持可变参数(......)，不带参数的block必须在参数列表中指定为void

当在多个位置使用相同的block时，通常可以使用typedef为block创建类型

```
typedef float (^MyBlockType)(float, float);
 
MyBlockType myFirstBlock = // ... ;
MyBlockType mySecondBlock = // ... ;
```



### 创建

```
float (^oneFrom)(float);
 
oneFrom = ^(float aFloat) {
    float result = aFloat - 1.0;
    return result;
};
```

block是由  `^ 返回值类型 参数列表 表达式` 几部分组成的，返回值类型和参数列表和c语言函数一致，表达式中含有return语句时，其类型必须与返回值类型相同

完整形式的block语言与一般的c语言函数定义相比，仅有两点不同：

- 没有函数名
- 带有 **^**



### 使用blocks

#### 调用block

如果声明了一个block为变量，则可以像函数一样使用

```
int (^oneFrom)(int) = ^(int anInt) {
    return anInt - 1;
};
 
printf("1 from 10 is %d", oneFrom(10));
// Prints "1 from 10 is 9"
 
 
float (^distanceTraveled)(float, float, float) =
                         ^(float startingSpeed, float acceleration, float time) {
 
    float distance = (startingSpeed * time) + (0.5 * acceleration * time * time);
    return distance;
};
 
float howFar = distanceTraveled(0.0, 9.8, 1.0);
// howFar = 4.9
```

但是，经常会将block作为参数传递给函数或方法。在这些情况下，通常会创建一个block "inline"

#### 使用block作为函数、方法参数

可以像传递任何其他参数一样，将block作为函数参数传递。然而，在许多情况下，不需要声明block；相反，只需在需要block作为参数的地方内联实现它们

```
char *myCharacters[3] = { "TomJohn", "George", "Charles Condomine" };
 
qsort_b(myCharacters, 3, sizeof(char *), ^(const void *l, const void *r) {
    char *left = *(char **)l;
    char *right = *(char **)r;
    return strncmp(left, right, 1);
});
// Block implementation ends at "}"
 
// myCharacters is now { "Charles Condomine", "George", "TomJohn" }
```

```
#include <dispatch/dispatch.h>
size_t count = 10;
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
 
dispatch_apply(count, queue, ^(size_t i) {
    printf("%u\n", i);
});
```

```
NSArray *array = @[@"A", @"B", @"C", @"A", @"B", @"Z", @"G", @"are", @"Q"];
NSSet *filterSet = [NSSet setWithObjects: @"A", @"Z", @"Q", nil];
 
BOOL (^test)(id obj, NSUInteger idx, BOOL *stop);
 
test = ^(id obj, NSUInteger idx, BOOL *stop) {
 
    if (idx < 5) {
        if ([filterSet containsObject: obj]) {
            return YES;
        }
    }
    return NO;
};
 
NSIndexSet *indexes = [array indexesOfObjectsPassingTest:test];
 
NSLog(@"indexes: %@", indexes);
 
/*
Output:
indexes: <NSIndexSet: 0x10236f0>[number of indexes: 2 (in 2 ranges), indexes: (0 3)]
*/
```

#### Copying Blocks

通常，不需要复制（或保留）block。仅当您希望block在声明它的作用域被销毁后使用时，才需要制作一个副本。复制会将block移动到堆中。

可以使用C函数复制和释放block：

```
Block_copy();
Block_release();
```

为了避免内存泄漏，必须对应使用这两个函数



### Blocks和变量

#### 变量类型

在block对象的代码体中，变量可以用五种不同的方式处理。

可以引用三种标准类型的变量，就像引用函数一样：

- 全局变量，包括静态局部变量

- 全局函数（技术上是不可变的）

- 封闭作用域中的局部变量和参数

Blocks也支持其他两种类型的变量:

1. `__block` 修饰的变量，它们在block（和封闭作用域）中是可变的，并且如果有任何引用block复制到堆中，它们将被保留。
2. const imports

最后，在方法实现中，block可以引用Objective-C实例变量.

以下规则适用于块中使用的变量：

- 全局变量，静态全局变量，静态变量，自动变量
- 传递给block的参数（就像传递给函数的参数一样）
- 用`__block`修饰符修饰的局部变量



下面例子展示了使用非静态的自动变量

```
int x = 123;
 
void (^printXAndY)(int) = ^(int y) {
 
    printf("%d %d\n", x, y);
};
 
printXAndY(456); // prints: 123 456
```

在尝试修改x的值时，会出现error

```
int x = 123;
 
void (^printXAndY)(int) = ^(int y) {
 
    x = x + y; // error
    printf("%d %d\n", x, y);
};
```

若要允许在block中更改变量，请使用 `__block` 修饰符

#### __block修饰类型

通过 `__block` 修饰符可以使外部导入的变量可变的，即读写。__block可以同register，auto，static一样修饰局部变量

```
__block int x = 123; //  x lives in block storage
 
void (^printXAndY)(int) = ^(int y) {
 
    x = x + y;
    printf("%d %d\n", x, y);
};
printXAndY(456); // prints: 579 456
// x is now 579
```

```
extern NSInteger CounterGlobal;
static NSInteger CounterStatic;
 
{
    NSInteger localCounter = 42;
    __block char localCharacter;
 
    void (^aBlock)(void) = ^(void) {
        ++CounterGlobal;
        ++CounterStatic;
        CounterGlobal = localCounter; // localCounter fixed at block creation
        localCharacter = 'a'; // sets localCharacter in enclosing scope
    };
 
    ++localCounter; // unseen by the block
    localCharacter = 'b';
 
    aBlock(); // execute the block
    // localCharacter now 'a'
}
```



#### Object 和 Block变量

Blocks支持OC、c++对象、和另外的block作为变量

拷贝block时，它会创建对block中使用的对象变量的强引用

- 如果通过引用访问实例变量，则对self进行强引用
- 如果通过值访问实例变量，则会对该变量进行强引用

```
dispatch_async(queue, ^{
    // instanceVariable is used by reference, a strong reference is made to self
    doSomethingWithObject(instanceVariable);
});
 
 
id localVariable = instanceVariable;
dispatch_async(queue, ^{
    /*
      localVariable is used by value, a strong reference is made to localVariable
      (and not to self).
    */
    doSomethingWithObject(localVariable);
});
```



## block本质

上节描述了block的使用和一些使用规则。看到了如果在block内修改局部变量会直接报错，接下来就从这个角度来分析block的本质







## block内部引用对象





## hook block





## 参考资料

[官方文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502-CH1-SW1)

[官方开源库](https://opensource.apple.com/source/libclosure/)















![](https://raw.githubusercontent.com/gaonian/HexoDocument/master/iOS/block_img/fork@2x.png)
