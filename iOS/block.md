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



#### Block存储位置

block具体有三种类型

- `__NSGlobalBlock__`  全局区

  没有访问外部变量 或者 只访问全局变量、静态变量、全局静态变量，这种情况下存放在全局区。生命周期从创建到应用程序结束。

- `__NSStackBlock__ ` 栈区

  用到外部局部变量，没有强指针引用的block都是放在栈区。在arc下，默认会将block从栈区拷贝到堆区

- `__NSMallocBlock__` 堆区

  访问了处于堆区的变量，有强指针引用等就会放在堆区。在arc下，栈区的也默认都会拷贝到堆区

所以在arc下，默认只有两种形式存在，全局区和堆区。使用strong或者copy修饰都没有问题



## block本质

上节描述了block的使用和一些使用规则。看到了如果在block内修改局部变量会直接报错，接下来就从这个角度来分析block的本质

```c
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        
        int a = 10;
        
        void (^block)(void) = ^{
            printf("%d \n", a);
        };
        
        a = 20;
        
        printf("%d \n", a);
        
        block();
        
    }
    return 0;
}

result:
20
10
```

这里有两种方法去窥探一下block的本质

1. 执行clang命令把代码转换为cpp，可以基本上看到苹果内部是怎么实现的

```
xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc main.m
```

2. 通过[官方开源库](https://opensource.apple.com/source/libclosure/) 查看 `Block_private.h` 头文件可以看到block的定义。这里看到的代码和第一种方式转换出来的基本保持一致



我们看通过第一种转换出来的cpp文件，有三万多行代码，直接拉到最底部就可以找到main函数

```c
int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 

        int a = 10;

        void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, a));

        a = 20;

        printf("%d \n", a);

        ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);

    }
    return 0;
}
```

```
void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, a));
```

这一行是我们写的block的定义和创建，实际调用了`__main_block_impl_0` 函数

```c
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
}

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  int a;
  
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _a, int flags=0) : a(_a) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

![block_layout](https://raw.githubusercontent.com/gaonian/HexoDocument/master/iOS/block_img/block2.png)

从代码上也可以看出block的布局，上图引用网上一张著名的block_layout图片。

- 第一个就是isa，oc中所有对象都有isa，从这点可以判断block实际也是一个oc对象

- flags对象block的一些状态

  ```c
  // Values for Block_layout->flags to describe block objects
  enum {
      BLOCK_DEALLOCATING =      (0x0001),  // runtime
      BLOCK_REFCOUNT_MASK =     (0xfffe),  // runtime
      BLOCK_NEEDS_FREE =        (1 << 24), // runtime
      BLOCK_HAS_COPY_DISPOSE =  (1 << 25), // compiler
      BLOCK_HAS_CTOR =          (1 << 26), // compiler: helpers have C++ code
      BLOCK_IS_GC =             (1 << 27), // runtime
      BLOCK_IS_GLOBAL =         (1 << 28), // compiler
      BLOCK_USE_STRET =         (1 << 29), // compiler: undefined if !BLOCK_HAS_SIGNATURE
      BLOCK_HAS_SIGNATURE  =    (1 << 30), // compiler
      BLOCK_HAS_EXTENDED_LAYOUT=(1 << 31)  // compiler
  };
  ```

- reserved 目前从资料上来看是保留未用的

- invoke 指向函数指针，调用invoke实际上就是实现的block内部代码

- descriptor 对应的是 `Block_descriptor_* ` 结构体

  - reserved
  - size： 整个block内存布局的大小
  - copy、dispose： 对应外部对象类型捕获的内存管理

- variables: block捕获的外部对象或者基本类型数据都会在这里展示，例如int a、Person *p对象

  ```c
  struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    
    int a;
    Person *p;
  };
  ```



上面简单分析了block的内存布局，接下来回到调用时候的传递参数，传了三个参数，分别对应构造函数的形参

- `__main_block_func_0` 赋值给了impl.FuncPtr，也就是调用的函数指针

  ```c
  static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    int a = __cself->a; // bound by copy
  
              printf("%d \n", a);
          }
  ```

  也就是说调用block的时候会调用__main_block_func_0这个函数，内部实现的代码就是我们之前写的block的代码

- `__main_block_desc_0_DATA`  主要是对应的一些描述信息。对应Desc，也就是 `__main_block_desc_0` 

  ```c
  static struct __main_block_desc_0 {
    size_t reserved;
    size_t Block_size;
  } __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
  ```

  reserved字段保留为0，block_size是整个 __main_block_impl_0 的大小

- 第三个字段 a 就是我们定义的局部变量，在`__main_block_impl_0`内部同时也有一个int a，这里直接把外面的局部变量值直接赋值给了内部的int a。

- 如果有多个引用，比如还引用了一个person对象，这里就会把person对象也传递进去

  ```c
          void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, a, p, 570425344));
  ```

- 最后一个参数是flags，对应的就是block_layout -> flags，具体falgs类型上部分有提到



至此，block的定义和实现部分转换后的代码基本分析完毕了。接着往下看调用的实现

```c
        ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);
```

直接就是调用 block -> FuncPtr，上面已经提到了，FuncPtr就是对应的block的内部实现代码，就是调用的 `__main_block_func_0` 



从上面的分析可以看出来一点为什么局部变量的值没有被修改，因为是把外部值直接传递给了我们内部自己定义的 int a，所以在外面的所有修改已经和内部没有关系了。



### __block修饰符

如果想要修改局部变量的值，官方给出的做法就是在变量定义前加上 `__block` 修饰符

```c
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        
        __block int a = 10;
        Person *p = [[Person alloc] init];
        p.age = 10;
        
        void (^block)(void) = ^{
            printf("%d %d \n", a, p.age);
        };
        
        a = 20;
        
        printf("%d \n", a);
        
        block();
    }
    return 0;
}
```

添加上`__block` 修饰符之后，运行结果是符合预期的。我们再转换为cpp文件看一看发生了什么变化

```c
int main(int argc, const char * argv[]) {
    /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 

        __attribute__((__blocks__(byref))) __Block_byref_a_0 a = {(void*)0,(__Block_byref_a_0 *)&a, 0, sizeof(__Block_byref_a_0), 10};
                            

        void (*block)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_a_0 *)&a, 570425344));

        (a.__forwarding->a) = 20;

        printf("%d \n", (a.__forwarding->a));

        ((void (*)(__block_impl *))((__block_impl *)block)->FuncPtr)((__block_impl *)block);

    }
    return 0;
}
```

对比之前发现了定义变量的地方变的复杂了很多，生成了一个`__Block_byref_a_0` 对象，可以看到实际上也是一个oc对象

```c
struct __Block_byref_a_0 {
  void *__isa;
__Block_byref_a_0 *__forwarding;
 int __flags;
 int __size;
 int a;
};
```

- 第二个参数 `__forwarding` 对应的是自身
- 第四个参数是size
- 最后一个参数是保存的局部变量的值

接着来看一下block的内部布局发生了什么变化

```c
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
  __Block_byref_a_0 *a; // by ref
  
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, Person *_p, __Block_byref_a_0 *_a, int flags=0) : p(_p), a(_a->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

之前的 int a，现在变成了 `__Block_byref_a_0 *a;`

```c
static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  __Block_byref_a_0 *a = __cself->a; // bound by ref
  Person *p = __cself->p; // bound by copy

            printf("%d %d \n", (a->__forwarding->a), ((int (*)(id, SEL))(void *)objc_msgSend)((id)p, sel_registerName("age")));
        }
```

具体实现里面也是通过调用__Block_byref_a_0对象内部的a也获取值

外部修改值得时候，也会对应的修改byref对象内部的值，所以在里面我们获取的就是最新值

总结来说，使用`__block` 修饰符的时候，会将基本数据类型包装成一个`__Block_byref_a_0` 对象类型，基本数据类型的值存储在对象内部的属性中。



**__forwarding**

在使用`__block`修饰符的时候会发现一个问题， `__Block_byref_a_0` 结构体内第二个参数 `__forwarding` 指向的就是自身，而且在修改或使用的时候都会通过 `__forwarding` 去间接调用

```c
(a.__forwarding->a) = 20;

printf("%d \n", (a->__forwarding->a));
```

这是为什么呢？

我们已经知道了block会存在全局区、栈区和堆区。

- 当block在栈上时候，forwarding指针就指向了自身。

- 当block被拷贝到堆上的时候，访问栈上的block，forwarding指针这时候指向了堆上的block结构体，访问堆上的block，forwarding指针指向了自身，也就是堆上的block

这样通过forwarding中转调用，无论栈或者堆每次都能访问到正确的`__block`变量





### Copy/Release

_Block_copy

_Block_release



_Block_object_assign

_Block_object_dispose



_Block_byref_copy

_Block_byref_release



### Block循环引用

`__weak`  建议使用此修饰符

`__unsafe_retained`

`__block` 必须调用block



## block内部引用对象





## hook block





## 参考资料

[官方文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502-CH1-SW1)

[官方开源库](https://opensource.apple.com/source/libclosure/)















![](https://raw.githubusercontent.com/gaonian/HexoDocument/master/iOS/block_img/fork@2x.png)
