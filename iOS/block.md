---
title: block解析
categories: iOS
---

## 简介

Blocks是C语言的扩充功能。可以用一句话来表示Blocks的扩充功能：带有自动变量（局部变量）的匿名函数。



```c
int multiplier = 7;
int (^myBlock)(int) = ^(int num) {
	return num * multiplier;
};
```

![](https://raw.githubusercontent.com/gaonian/HexoDocument/master/iOS/block_img/block1.png)



### 声明

Block变量保存对block的引用，使用类似于声明函数指针的语法来声明他们，除了使用 ^ 代替 * ，block类型可以与c类型系统的其余部分完全互相操作。以下是所有有效的block变量声明：

```c
void（^ blockReturningVoidWithVoidArgument）（void）;
int（^ blockReturningIntWithIntAndCharArguments）（int，char）;
void（^ arrayOfTenBlocksReturningVoidWithIntArgument [10]）（int）;
```

block还可以支持可变参数(......)，不带参数的block必须在参数列表中指定为void

当在多个位置使用相同的block时，通常可以使用typedef为block创建类型

```c
typedef float (^MyBlockType)(float, float);
 
MyBlockType myFirstBlock = // ... ;
MyBlockType mySecondBlock = // ... ;
```



### 创建

```c
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

```c
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

```c
char *myCharacters[3] = { "TomJohn", "George", "Charles Condomine" };
 
qsort_b(myCharacters, 3, sizeof(char *), ^(const void *l, const void *r) {
    char *left = *(char **)l;
    char *right = *(char **)r;
    return strncmp(left, right, 1);
});
// Block implementation ends at "}"
 
// myCharacters is now { "Charles Condomine", "George", "TomJohn" }
```

```c
#include <dispatch/dispatch.h>
size_t count = 10;
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
 
dispatch_apply(count, queue, ^(size_t i) {
    printf("%u\n", i);
});
```

```c
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

```c
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

```c
int x = 123;
 
void (^printXAndY)(int) = ^(int y) {
 
    printf("%d %d\n", x, y);
};
 
printXAndY(456); // prints: 123 456
```

在尝试修改x的值时，会出现error

```c
int x = 123;
 
void (^printXAndY)(int) = ^(int y) {
 
    x = x + y; // error
    printf("%d %d\n", x, y);
};
```

若要允许在block中更改变量，请使用 `__block` 修饰符

#### __block修饰类型

通过 `__block` 修饰符可以使外部导入的变量可变的，即读写。__block可以同register，auto，static一样修饰局部变量

```c
__block int x = 123; //  x lives in block storage
 
void (^printXAndY)(int) = ^(int y) {
 
    x = x + y;
    printf("%d %d\n", x, y);
};
printXAndY(456); // prints: 579 456
// x is now 579
```

```c
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

```c
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



### Block Copy/Release

当block从栈上拷贝到堆上时，会调用`_Block_copy` 函数拷贝block，同时也会block内部所有引用的对象和 使用`__block`修饰的变量 调用`_Block_object_assign`进行拷贝



#### _Block_copy

在源码中`runtime.cpp` 中可以找到此函数的实现

```c
// Copy, or bump refcount, of a block.  If really copying, call the copy helper if present.
void *_Block_copy(const void *arg) {
    struct Block_layout *aBlock;

    if (!arg) return NULL;
    
    // The following would be better done as a switch statement
    aBlock = (struct Block_layout *)arg;
    if (aBlock->flags & BLOCK_NEEDS_FREE) {
        // latches on high
        latching_incr_int(&aBlock->flags);
        return aBlock;
    }
    else if (aBlock->flags & BLOCK_IS_GLOBAL) {
        return aBlock;
    }
    else {
        // Its a stack block.  Make a copy.
        struct Block_layout *result =
            (struct Block_layout *)malloc(aBlock->descriptor->size);
        if (!result) return NULL;
        memmove(result, aBlock, aBlock->descriptor->size); // bitcopy first
#if __has_feature(ptrauth_calls)
        // Resign the invoke pointer as it uses address authentication.
        result->invoke = aBlock->invoke;
#endif
        // reset refcount
        result->flags &= ~(BLOCK_REFCOUNT_MASK|BLOCK_DEALLOCATING);    // XXX not needed
        result->flags |= BLOCK_NEEDS_FREE | 2;  // logical refcount 1
        _Block_call_copy_helper(result, aBlock);
        // Set isa last so memory analysis tools see a fully-initialized object.
        result->isa = _NSConcreteMallocBlock;
        return result;
    }
}
```

1. 首先将传进来的block转换为一个aBlock，然后根据block的flags做相应的处理

2. 第二步，如果flags包含BLOCK_NEEDS_FREE，则证明该block已经存在引用计数，代表就是在堆上，只用对应的增加引用计数即可。

   此处调用了`latching_incr_int` 函数，内部对原始flags做了一个加2的操作

3. 第三步，判断如果flags包含BLOCK_IS_GLOBAL，则证明block为全局block，全局block一直存在内存中，不需要做对应内存管理，所以此步骤直接返回该block

4. 第四步，else 则说明此block是存在于栈上的，需要进行拷贝，下面分析拷贝的步骤：

   1. 首先在堆上malloc分配一段空间，大小是`aBlock->descriptor->size` ，对应原有block的大小

   2. 然后`memmove`复制原有block到堆上，复制大小为原有block大小，完全复制

   3. 根据 `__has_feature(ptrauth_calls)` 编译器特性，执行了一句代码 `result->invoke = aBlock->invoke;`，此处的猜想是如果使用了地址空间随机化，则重新指定invoke指针的地址

   4. 接下来有两句设置flags的代码

      ```c
      // reset refcount
              result->flags &= ~(BLOCK_REFCOUNT_MASK|BLOCK_DEALLOCATING);    // XXX not needed
              result->flags |= BLOCK_NEEDS_FREE | 2;  // logical refcount 1
      ```

      第一句=右边的代码计算结果为0，一个数&=0，则会清空其他位置的数据，也就是注释所说的重置引用计数，因为引用计数存在低位，其他位被清空之后，引用计数也就对应被清空

      在清空之后，第二句代码则重新按位或上了` BLOCK_NEEDS_FREE | 2`，设置引用计数。 在这里我的理解是 BLOCK_NEEDS_FREE 标识有引用计数，需要释放，|2在低二位设置一个标识，标识引用计数的数量

   5. 接下来 `_Block_call_copy_helper` 会根据原block中是否有需要内存管理的对象，来进行对应拷贝到堆上

   6. 最后一步设置isa为`_NSConcreteMallocBlock` 标明block的类型

大体上block的copy过程分析完了，其中一些细节由于知识有限不是很清楚，有待进一步学习分析



#### _Block_release

有copy就会对应有release，接下来就来分析一下release的代码

```c
// API entry point to release a copied Block
void _Block_release(const void *arg) {
    struct Block_layout *aBlock = (struct Block_layout *)arg;
    if (!aBlock) return;
    if (aBlock->flags & BLOCK_IS_GLOBAL) return;
    if (! (aBlock->flags & BLOCK_NEEDS_FREE)) return;

    if (latching_decr_int_should_deallocate(&aBlock->flags)) {
        _Block_call_dispose_helper(aBlock);
        _Block_destructInstance(aBlock);
        free(aBlock);
    }
}
```

1. 第一步是先转换block，判空处理

2. 接下来看是否是`BLOCK_IS_GLOBAL` 全局区，是的话返回不做处理

3. 第三步看`BLOCK_NEEDS_FREE` 状态，此状态标明是在堆区有引用计数，如果不是此状态，也直接返回

4. 处理完一些其他情况之后，就该正式做release操作了，首先调用`latching_decr_int_should_deallocate` 判断是否需要dealloc

   ```c
   // return should_deallocate?
   static bool latching_decr_int_should_deallocate(volatile int32_t *where) {
       while (1) {
           int32_t old_value = *where;
           if ((old_value & BLOCK_REFCOUNT_MASK) == BLOCK_REFCOUNT_MASK) {
               return false; // latched high
           }
           if ((old_value & BLOCK_REFCOUNT_MASK) == 0) {
               return false;   // underflow, latch low
           }
           int32_t new_value = old_value - 2;
           bool result = false;
           if ((old_value & (BLOCK_REFCOUNT_MASK|BLOCK_DEALLOCATING)) == 2) {
               new_value = old_value - 1;
               result = true;
           }
           if (OSAtomicCompareAndSwapInt(old_value, new_value, where)) {
               return result;
           }
       }
   }
   ```

   我们在copy函数增加引用计数的时候会对flags+2，所以在这里首先会对flags做-2的操作

   然后如果old_value == 2 ，直接对old_value-1，减1之后flags就变成了`BLOCK_DEALLOCATING` ，这个值表示block需要释放

   最后交换flags的值，返回result，告诉外界是否需要释放

5. 由于上一步判断需要释放的时候已经对flags进行了减引用计数的操作，如果不需要释放，则此时release函数已经执行完毕了。 

6. 如果需要释放，调用`_Block_call_dispose_helper` 函数对block内部对象进行相应内存管理释放。接着调用`_Block_destructInstance` 函数，此函数没有找到对应的源码，故根据字面意思猜测是对block实例对象进行释放。最后free释放空间



#### _Block_object_assign

当block拷贝到堆上时，可以引用四种需要帮助的不同类型的东西

1. 基于c++的堆栈对象
2. 引用Objective-C对象
3. 其他的block
4. `__block ` 修改的变量类型

在Block_copy和Block_release调用情况下，编译器会合成 copy 和 dispose 助手函数。

在第一种基于c++堆栈对象的情况下，copy助手函数会调用其构造函数，dispose助手函数会调用析构函数

其余三种情况下，copy助手函数会调用 `_Block_object_assign`， dispose助手函数会调用 `_Block_object_dispose` 去做相应的处理

`__Block_object_assign` 和 `__Block_object_dispose` 的 flags参数类型如下：

- `BLOCK_FIELD_IS_OBJECT (3)`, Objective-C对象

- `BLOCK_FIELD_IS_BLOCK (7)` ，其他的block对象

- `BLOCK_FIELD_IS_BYREF (8)`， `__block` 修饰的变量

  如果`__block`修饰的变量设置为`__weak`，则flags也是 `BLOCK_FIELD_IS_WEAK (16)`

所以，Block copy/dispose 应该只生成 3、7、8、24 这四种flags



当`__block`修饰符修饰了一个c++对象或者OC对象或者其他的block对象时，编译器也会生成 copy/dispose 助手函数，也会做相应的内存管理操作。

在为oc对象或者其他block调用相同的助手函数时，同时会在flags中提供额外的位信息标识，`BLOCK_BYREF_CALLER (128)`

 

所以 `__block` copy/dispose 助手将在为 对象 或 其他block 生成3、7的同时，会根据不同的情况增加16或者128。下面是可能出现的情况：

```
__block id                   128+3       (0x83)
__block (^Block)             128+7       (0x87)
__weak __block id            128+3+16    (0x93)
__weak __block (^Block)      128+7+16    (0x97)
```

如果是修饰的对象或者block，则对应增加128，如果同时修饰了weak属性，则再对应增加16



当Blocks或者Block_byrefs包含了对象时，copy方法如下

```c
void _Block_object_assign(void *destArg, const void *object, const int flags) {
    const void **dest = (const void **)destArg;
    switch (os_assumes(flags & BLOCK_ALL_COPY_DISPOSE_FLAGS)) {
      case BLOCK_FIELD_IS_OBJECT:
        /*******
        id object = ...;
        [^{ object; } copy];
        ********/

        _Block_retain_object(object);
        *dest = object;
        break;

      case BLOCK_FIELD_IS_BLOCK:
        /*******
        void (^object)(void) = ...;
        [^{ object; } copy];
        ********/

        *dest = _Block_copy(object);
        break;
    
      case BLOCK_FIELD_IS_BYREF | BLOCK_FIELD_IS_WEAK:
      case BLOCK_FIELD_IS_BYREF:
        /*******
         // copy the onstack __block container to the heap
         // Note this __weak is old GC-weak/MRC-unretained.
         // ARC-style __weak is handled by the copy helper directly.
         __block ... x;
         __weak __block ... x;
         [^{ x; } copy];
         ********/

        *dest = _Block_byref_copy(object);
        break;
        
      case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_OBJECT:
      case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_BLOCK:
        /*******
         // copy the actual field held in the __block container
         // Note this is MRC unretained __block only. 
         // ARC retained __block is handled by the copy helper directly.
         __block id object;
         __block void (^object)(void);
         [^{ object; } copy];
         ********/

        *dest = object;
        break;

      case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_OBJECT | BLOCK_FIELD_IS_WEAK:
      case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_BLOCK  | BLOCK_FIELD_IS_WEAK:
        /*******
         // copy the actual field held in the __block container
         // Note this __weak is old GC-weak/MRC-unretained.
         // ARC-style __weak is handled by the copy helper directly.
         __weak __block id object;
         __weak __block void (^object)(void);
         [^{ object; } copy];
         ********/

        *dest = object;
        break;

      default:
        break;
    }
}
```

首先来分析接收的三个参数

```c
static void _Block_call_copy_helper(void *result, struct Block_layout *aBlock)
{
    struct Block_descriptor_2 *desc = _Block_descriptor_2(aBlock);
    if (!desc) return;

    (*desc->copy)(result, aBlock); // do fixup
}
```

从Block_copy中找到上面的函数，从此函数可以看出copy助手函数传入的两个函数一个为拷贝之后堆上的block，一个为原始的block

```c
static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {
    _Block_object_assign((void*)&dst->a, (void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);
    _Block_object_assign((void*)&dst->weakPerson, (void*)src->weakPerson, 3/*BLOCK_FIELD_IS_OBJECT*/);
    _Block_object_assign((void*)&dst->s, (void*)src->s, 3/*BLOCK_FIELD_IS_OBJECT*/);
    _Block_object_assign((void*)&dst->s1, (void*)src->s1, 3/*BLOCK_FIELD_IS_OBJECT*/);}
```

从编译后的cpp中可以找到`void (*copy)(x, x)`实际调用的是`__main_block_copy_0`，  在这里面看到了 `_Block_object_assign`

这样综合分析之后就可以明白三个参数传递的都是什么

1. destArg为copy之后在堆上的block对象

2. object指向copy之前的block对象

3. flags对应如下，区分引用的对象类型

   ```c
   // Values for _Block_object_assign() and _Block_object_dispose() parameters
   enum {
       // see function implementation for a more complete description of these fields and combinations
       BLOCK_FIELD_IS_OBJECT   =  3,  // id, NSObject, __attribute__((NSObject)), block, ...
       BLOCK_FIELD_IS_BLOCK    =  7,  // a block variable
       BLOCK_FIELD_IS_BYREF    =  8,  // the on stack structure holding the __block variable
       BLOCK_FIELD_IS_WEAK     = 16,  // declared __weak, only used in byref copy helpers
       BLOCK_BYREF_CALLER      = 128, // called from __block (byref) copy/dispose support routines.
   };
   
   enum {
       BLOCK_ALL_COPY_DISPOSE_FLAGS = 
           BLOCK_FIELD_IS_OBJECT | BLOCK_FIELD_IS_BLOCK | BLOCK_FIELD_IS_BYREF |
           BLOCK_FIELD_IS_WEAK | BLOCK_BYREF_CALLER
   };
   ```



接着正式来看到底做了什么

1. 如果是`BLOCK_FIELD_IS_OBJECT`，即OC对象。对object执行retain操作，赋值为destArg
2. 如果是`BLOCK_FIELD_IS_BLOCK`，即block对象。则执行Block_copy，进行对应的拷贝操作，赋值给destArg
3. 如果是`BLOCK_FIELD_IS_BYREF | BLOCK_FIELD_IS_WEAK` 或者 `BLOCK_FIELD_IS_BYREF`，代表`__block` 或者`__block __weak` 修饰的变量。则执行`_Block_byref_copy` 函数，此函数在后面分析
4. 如果是`BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_OBJECT` 或者 `BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_BLOCK`，代表的是`__block`修饰的oc对象或者block，直接赋值给destArg
5. 如果是`BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_OBJECT | BLOCK_FIELD_IS_WEAK` 或者 `BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_BLOCK | BLOCK_FIELD_IS_WEAK`，代表的是 `__block __weak` 修饰的oc对象或者block，也是直接赋值



#### _Block_object_dispose

dispose对应的是assign，当block销毁时，同时dispose block内部引用的对象

``` c
// When Blocks or Block_byrefs hold objects their destroy helper routines call this entry point
// to help dispose of the contents
void _Block_object_dispose(const void *object, const int flags) {
    switch (os_assumes(flags & BLOCK_ALL_COPY_DISPOSE_FLAGS)) {
      case BLOCK_FIELD_IS_BYREF | BLOCK_FIELD_IS_WEAK:
      case BLOCK_FIELD_IS_BYREF:
        // get rid of the __block data structure held in a Block
        _Block_byref_release(object);
        break;
      case BLOCK_FIELD_IS_BLOCK:
        _Block_release(object);
        break;
      case BLOCK_FIELD_IS_OBJECT:
        _Block_release_object(object);
        break;
      case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_OBJECT:
      case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_BLOCK:
      case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_OBJECT | BLOCK_FIELD_IS_WEAK:
      case BLOCK_BYREF_CALLER | BLOCK_FIELD_IS_BLOCK  | BLOCK_FIELD_IS_WEAK:
        break;
      default:
        break;
    }
}
```

此函数和assign操作是一一对应的，接下来具体步骤分析

1. 如果是`BLOCK_FIELD_IS_BYREF | BLOCK_FIELD_IS_WEAK` 或者 `BLOCK_FIELD_IS_BYREF`，代表`__block` 或者 `__block __weak` 修饰的变量，则执行`_Block_byref_release` 函数，此函数也会在后面分析，此处跳过
2. 如果是`BLOCK_FIELD_IS_BLOCK` ，代表引用的block对象，执行Block_release函数进行释放
3. 如果是`BLOCK_FIELD_IS_OBJECT`，执行relase对象的操作，和assign中retain是对应关系
4. 最后如果是`__block` 或者 `__block __weak` 修饰的oc对象或者block对象，则什么也不做，因为在assign中对应的情况也没有做任何处理



#### _Block_byref_copy





#### _Block_byref_release







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
