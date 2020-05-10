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



#### __block可修饰类型

`__block` 可修饰的类型可以分为三种，每一种类型的处理方式也不一样，具体处理会在下面`_Block_byref_copy` 部分讲解

- 修饰变量 `__block int a`
- 修饰OC对象 `__block id p`
- 修饰block `__block (^block)`



#### __forwarding

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

2. 第二步，如果flags包含 `BLOCK_NEEDS_FREE`，free代表是在堆上，只用对应的增加引用计数即可。

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

      第一句右边的代码`BLOCK_REFCOUNT_MASK|BLOCK_DEALLOCATING` 结果为0xFFFF，`~(0xFFFF)`结果为`0x0000`，一个数`&=0x0000`，则会清空低16位的数据，也就是此处注释上所说的重置引用计数，因为引用计数就存在于低16上（低1位代表`BLOCK_DEALLOCATING`）

      在清空之后，第二句代码则重新按位或上了` BLOCK_NEEDS_FREE | 2`，设置引用计数。 free代表是在堆上，`| 2` 增加引用计数，每次增加引用计数都是递增2 。

   5. 接下来 `_Block_call_copy_helper` 会根据原block中是否有需要内存管理的对象，来进行对应拷贝到堆上

   6. 最后一步设置isa为`_NSConcreteMallocBlock` 标明block的类型



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

3. 第三步看`BLOCK_NEEDS_FREE` 状态，此状态标明是在堆区，如果不是此状态，也直接返回

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

   - 首先会判断边界情况，最大`BLOCK_REFCOUNT_MASK` 或者等于0，都返回false，不处理

   - 接着往下看，我们在copy函数增加引用计数的时候会对flags+2，所以在这里首先会对flags做-2的操作

   - 然后如果此时old_value正好等于2 ，说明只有一个引用计数，需要释放了。直接对old_value-1，减1之后flags就变成了`BLOCK_DEALLOCATING` ，这个值表示block需要释放
   - 最后交换flags的值，返回result，告诉外界是否需要释放
   - 总结就是如果有多个引用计数，则减一个，如果只有一个，则释放

   

5. 由于上一步判断需要释放的时候已经对flags进行了减引用计数的操作，如果不需要释放，则此时release函数已经执行完毕了。 

6. 如果需要释放，调用`_Block_call_dispose_helper` 函数对block内部对象进行相应内存管理释放。接着调用`_Block_destructInstance` 函数，此函数没有找到对应的源码，故根据字面意思猜测是对block实例对象进行释放。最后free释放空间



#### _Block_object_assign

当block拷贝到堆上时，可以引用四种不同的类型

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

之前有介绍过byref对象，知道是`__block` 修饰的变量或者对象等就会生成一个byref对象。现在再来认识一下byref对象的布局

``` c
// Values for Block_byref->flags to describe __block variables
enum {
    // Byref refcount must use the same bits as Block_layout's refcount.
    // BLOCK_DEALLOCATING =      (0x0001),  // runtime
    // BLOCK_REFCOUNT_MASK =     (0xfffe),  // runtime

    BLOCK_BYREF_LAYOUT_MASK =       (0xf << 28), // compiler
    BLOCK_BYREF_LAYOUT_EXTENDED =   (  1 << 28), // compiler
    BLOCK_BYREF_LAYOUT_NON_OBJECT = (  2 << 28), // compiler
    BLOCK_BYREF_LAYOUT_STRONG =     (  3 << 28), // compiler
    BLOCK_BYREF_LAYOUT_WEAK =       (  4 << 28), // compiler
    BLOCK_BYREF_LAYOUT_UNRETAINED = (  5 << 28), // compiler

    BLOCK_BYREF_IS_GC =             (  1 << 27), // runtime

    BLOCK_BYREF_HAS_COPY_DISPOSE =  (  1 << 25), // compiler
    BLOCK_BYREF_NEEDS_FREE =        (  1 << 24), // runtime
};

struct Block_byref {
    void *isa;
    struct Block_byref *forwarding;
    volatile int32_t flags; // contains ref count
    uint32_t size;
};

struct Block_byref_2 {
    // requires BLOCK_BYREF_HAS_COPY_DISPOSE
    BlockByrefKeepFunction byref_keep;
    BlockByrefDestroyFunction byref_destroy;
};

struct Block_byref_3 {
    // requires BLOCK_BYREF_LAYOUT_EXTENDED
    const char *layout;
};
```

分为了三部分组成

**Block_byref** 

- isa，默认为NULL
- forwarding 栈上的指向堆上的对象，堆上的指向自身
- flags 包含一些状态，例如引用计数，堆栈信息等
- size 整个byref大小 包含了下面两部分

**Block_byref_2**

- 如果`__block` 修饰的是对象类型，则此部分就会存在，用于内存管理。实际调用的`_Block_object_assign` 和 `_Block_object_dispose` 

**Block_byref_3**

- 引用的所有原来对象都在layout里面，layout采用了一种压缩布局方式，这种压缩布局会在第三大部分block内部引用对象里详细讲解，这里只需要了解引用的对象信息在这里就行



了解byref的布局方式之后，接下来看copy函数更有利于理解

```c
static struct Block_byref *_Block_byref_copy(const void *arg) {
    struct Block_byref *src = (struct Block_byref *)arg;

    if ((src->forwarding->flags & BLOCK_REFCOUNT_MASK) == 0) {
        // src points to stack
        struct Block_byref *copy = (struct Block_byref *)malloc(src->size);
        copy->isa = NULL;
        // byref value 4 is logical refcount of 2: one for caller, one for stack
        copy->flags = src->flags | BLOCK_BYREF_NEEDS_FREE | 4;
        copy->forwarding = copy; // patch heap copy to point to itself
        src->forwarding = copy;  // patch stack to point to heap copy
        copy->size = src->size;

        if (src->flags & BLOCK_BYREF_HAS_COPY_DISPOSE) {
            // Trust copy helper to copy everything of interest
            // If more than one field shows up in a byref block this is wrong XXX
            struct Block_byref_2 *src2 = (struct Block_byref_2 *)(src+1);
            struct Block_byref_2 *copy2 = (struct Block_byref_2 *)(copy+1);
            copy2->byref_keep = src2->byref_keep;
            copy2->byref_destroy = src2->byref_destroy;

            if (src->flags & BLOCK_BYREF_LAYOUT_EXTENDED) {
                struct Block_byref_3 *src3 = (struct Block_byref_3 *)(src2+1);
                struct Block_byref_3 *copy3 = (struct Block_byref_3*)(copy2+1);
                copy3->layout = src3->layout;
            }

            (*src2->byref_keep)(copy, src);
        }
        else {
            // Bitwise copy.
            // This copy includes Block_byref_3, if any.
            memmove(copy+1, src+1, src->size - sizeof(*src));
        }
    }
    // already copied to heap
    else if ((src->forwarding->flags & BLOCK_BYREF_NEEDS_FREE) == BLOCK_BYREF_NEEDS_FREE) {
        latching_incr_int(&src->forwarding->flags);
    }
    
    return src->forwarding;
}
```

此段代码执行了两个条件判断

- 首先判断`(src->forwarding->flags & BLOCK_REFCOUNT_MASK) == 0` ，等于0代表没有引用计数，也说明了不在堆上，所以需要执行拷贝操作

  1. malloc分配空间，size为栈上byref的大小

  2. 设置isa等于NULL

  3. 设置flags `src->flags | BLOCK_BYREF_NEEDS_FREE | 4`，`BLOCK_BYREF_NEEDS_FREE` 代表位于堆上，`| 4` 代表引用计数为2，一份是堆上持有，一份是栈上持有。（之前我们之前讨论过，引用计数递增flags每次都会增加2，所以这里两份持有，就按位或4 `| 4` ）

  4. 设置堆上的forwarding指向堆上的byref对象（之前也讨论过在堆上时forwarding指针指向自身，在栈上时forwarding指向堆上，所以这里也没有疑问）

  5. 设置原来栈上的forwarding指针指向堆上

  6. 设置size为原有栈上的size

  7. 接下来就是设置内存管理和引用对象了。 如果需要内存管理

     1. 根据src+1取出`Block_byref_2` ，设置堆上的byref_keep和byref_destroy指向栈上对应的函数
     2. 如果有extended，则根据src2+1取出`Block_byref_3`，赋值layout
     3. 接下来 `(*src2->byref_keep)(copy, src);` 调用byref_keep函数进行拷贝

     ``` c
     static void __Block_byref_id_object_copy_131(void *dst, void *src) {
      _Block_object_assign((char*)dst + 40, *(void * *) ((char*)src + 40), 131);
     }
     ```

     根据clang编译出来的cpp文件，可以推测出实际调用的就是`_Block_object_assign` 函数，传入+40的目的是直接找到layout对象，也就是需要拷贝的对象。 （layout前面有`*isa`、`*forwarding`、`int flags`、`int size`、`*byref_keep`、`*byfre_destroy`，总共就是40字节）

  8. 如果不需要内存管理，说明捕获的是一个变量，没有`Block_byref_2`，`Block_byref` 中size之后就直接是layout了，所以src+1取出的就是layout。直接进行字节拷贝，大小是 `src->size - sizeof(*src)` 总大小减`Block_byref` 的大小，就是layout的大小

- 接着回到最外层，另外一条判断语句。`(src->forwarding->flags & BLOCK_BYREF_NEEDS_FREE) == BLOCK_BYREF_NEEDS_FREE` 表示byref对象已经在堆上了，只需要调用`latching_incr_int` 增加引用计数即可

- 最后返回 `src->forwarding` ，也就是堆上的byref对象

 

#### _Block_byref_release

``` c
static void _Block_byref_release(const void *arg) {
    struct Block_byref *byref = (struct Block_byref *)arg;

    // dereference the forwarding pointer since the compiler isn't doing this anymore (ever?)
    byref = byref->forwarding;
    
    if (byref->flags & BLOCK_BYREF_NEEDS_FREE) {
        int32_t refcount = byref->flags & BLOCK_REFCOUNT_MASK;
        os_assert(refcount);
        if (latching_decr_int_should_deallocate(&byref->flags)) {
            if (byref->flags & BLOCK_BYREF_HAS_COPY_DISPOSE) {
                struct Block_byref_2 *byref2 = (struct Block_byref_2 *)(byref+1);
                (*byref2->byref_destroy)(byref);
            }
            free(byref);
        }
    }
}
```

1. 首先通过forwarding取出真正的byref对象
2. 根据flags判断是否在堆上
3. 取出refcount，引用计数
4. 然后调用`latching_decr_int_should_deallocate` 判断此byref对象是否需要释放
5. 如果需要释放，再根据是否有copy、dispose，调用`byref_destroy` 释放byref对象内部的引用对象



### Block循环引用

block循环引用问题是开发中最常见的问题，也是最熟悉的。 

对象持有block，block又强引用了对象，这样就造成了循环引用。我们要做的就是把其中一条强引用关系给解除，就可以避免循环引用问题

- 使用 `__weak`  修饰符。

  `__weak` 不会产生强引用，指向的对象销毁时，会自动让指针置为nil。 

  ``` c
  __weak typeof(p) weakPerson = p;
  __weak Person *weakPerson = p;
  ```

- `__unsafe_retained` 修饰符

  `__unsafe_retained` 不会产生强引用，不安全，指向的对象销毁时，指针存储的地址值不变

  ``` c
  __unsafe_retained Person *weakPerson = p;
  __unsafe_unretained typeof(p) weakpp = p;
  ```

- 使用`__block` ，但是block实现之后必须要把引用对象置为nil，而且block必须要被调用

  ``` c
  __block id weakSelf - self;
  self.block = ^{
   	printf("%p", weakSelf);
    weakSelf = nil;
  };
  self.block();
  ```

arc下建议使用 `__weak` 解决方案



ps: cell的隐性循环引用

分析其原因在于cell实际是tableView的子视图,每个子视图都是会被其父视图的subviews(NSArray *)属性所强引用,即tableView ~> subviews ~> cell,而cell因为使用block作为回调强引用了block内部的对象,形成了这样的循环引用链条,即 controller ~> tableView ~> cell ~> block -> controller。所以构成了循环引用，解决的方法同样是block弱引用controller，就打破了闭环





## block内部引用对象

这一部分主要的目的是打印出block内部引用的所有外部对象，在平时开发中如果有难以排查的循环引用问题，可以帮助我们快速查找问题。一个实用的小工具。参考文章[一种查看Block中引用的所有外部对象的实现方法](https://juejin.im/post/5d7e3b8de51d4561ac7bcd5f) 

``` c
self.name = @"哈哈";

__weak typeof(self) weakSelf = self;

Person *p = [[Person alloc] init];
p.age = 88;

__block Student *s = [[Student alloc] init];
s.age = 10;

self.block = ^{
  NSLog(@"%@", weakSelf.name);
  NSLog(@"%d", p.age);
  NSLog(@"%@", self.name);
};

self.block();
```

`self.block` 内部引用了三个对象，弱引用了self，强引用person对象，强引用了self对象。然后我们使用lldb调用工具类打印一下结果

``` c
(lldb) po showBlockExtendedLayout(self.block)
  
2020-05-10 14:23:05.753129+0800 BlockExtends[21198:1007232] the refObj is:<Person: 0x6000025edc60>  type is:KB_BLOCK_LAYOUT_STRONG
  
2020-05-10 14:23:05.753389+0800 BlockExtends[21198:1007232] the refObj is:<ViewController: 0x7fcc807031f0>  type is:KB_BLOCK_LAYOUT_STRONG
 
2020-05-10 21:17:29.770055+0800 BlockExtends[2322:1354898] the refObj is:<Student: 0x600002ad0750>  type is KB_BLOCK_LAYOUT_BYREF
```

由于工具内部只打印强引用的对象，所以可以看到这里打印出了person对象和强引用的self对象，type is ： 3 代表强引用类型。这只是简单的示范，实际项目中使用可以通过不同情况自己封装。 



代码并不复杂，主要是实现的思路。如果flags包含了`BLOCK_HAS_EXTENDED_LAYOUT` ，说明引用了外部的对象，位置是在结构体的最下面部分。我们只需要找到这一部分就可以找到引用的对象。但是由于苹果对layout数据进行了编码，所以我们需要知道编码规则，才可以对应的解出来数据

``` c
// Extended layout encoding.

// Values for Block_descriptor_3->layout with BLOCK_HAS_EXTENDED_LAYOUT
// and for Block_byref_3->layout with BLOCK_BYREF_LAYOUT_EXTENDED

// If the layout field is less than 0x1000, then it is a compact encoding 
// of the form 0xXYZ: X strong pointers, then Y byref pointers, 
// then Z weak pointers.

// If the layout field is 0x1000 or greater, it points to a 
// string of layout bytes. Each byte is of the form 0xPN.
// Operator P is from the list below. Value N is a parameter for the operator.
// Byte 0x00 terminates the layout; remaining block data is non-pointer bytes.

enum {
    BLOCK_LAYOUT_ESCAPE = 0, // N=0 halt, rest is non-pointer. N!=0 reserved.
    BLOCK_LAYOUT_NON_OBJECT_BYTES = 1,    // N bytes non-objects
    BLOCK_LAYOUT_NON_OBJECT_WORDS = 2,    // N words non-objects
    BLOCK_LAYOUT_STRONG           = 3,    // N words strong pointers
    BLOCK_LAYOUT_BYREF            = 4,    // N words byref pointers
    BLOCK_LAYOUT_WEAK             = 5,    // N words weak pointers
    BLOCK_LAYOUT_UNRETAINED       = 6,    // N words unretained pointers
    BLOCK_LAYOUT_UNKNOWN_WORDS_7  = 7,    // N words, reserved
    BLOCK_LAYOUT_UNKNOWN_WORDS_8  = 8,    // N words, reserved
    BLOCK_LAYOUT_UNKNOWN_WORDS_9  = 9,    // N words, reserved
    BLOCK_LAYOUT_UNKNOWN_WORDS_A  = 0xA,  // N words, reserved
    BLOCK_LAYOUT_UNUSED_B         = 0xB,  // unspecified, reserved
    BLOCK_LAYOUT_UNUSED_C         = 0xC,  // unspecified, reserved
    BLOCK_LAYOUT_UNUSED_D         = 0xD,  // unspecified, reserved
    BLOCK_LAYOUT_UNUSED_E         = 0xE,  // unspecified, reserved
    BLOCK_LAYOUT_UNUSED_F         = 0xF,  // unspecified, reserved
};
```

我们可以在源码中找到layout的描述，在上面讲byref对象的时候也有一个扩展layout。这两个的存储实现方式是一致的。

layout采用了一种自己独特的编码形式

- 当layout字段小于`0x1000`，则是一个压缩的扩展布局描述，格式为`0xXYZ`，x代表强引用的对象指针，y代表byref对象的指针，z代表的是weak弱引用指针。也就是用12位存储数据 `0000 0000 0000` ，低4位代表weak弱引用(z)，中间4位代表byref对象的指针(y)，高4位代表强引用的对象(x)
- 如果layout字段大于等于`0x1000`，它指向一个以`0x00` 结尾的字符串指针，每个字节的格式为 `0xPN`，P就是枚举中描述的类型，N代表每种类型的数量。也就是8位 `0000 0000` ，低4位表示数量N，高4位表示类型P



我们在之前小节中已经了解了block的布局，刚刚也知道了layout的编码规则。相信再看代码的话已经很轻松了

``` c
/// kb_Block_layout Extended layout encoding.
typedef enum {
    KB_BLOCK_LAYOUT_STRONG = 3,
    KB_BLOCK_LAYOUT_BYREF = 4,
    KB_BLOCK_LAYOUT_WEAK = 5,
    KB_BLOCK_LAYOUT_UNRETAINED = 6
} KBBlockExtendFlags;

/// kb_Block_layout -> flags
typedef enum {
    KB_BLOCK_HAS_COPY_DISPOSE = (1 << 25),
    KB_BLOCK_HAS_EXTENDED_LAYOUT = (1 << 31)
} KBBlockLayoutFlags;

/// kb_Block_byref -> flags
typedef enum {
    KB_BLOCK_BYREF_LAYOUT_EXTENDED =   (  1 << 28),
    KB_BLOCK_BYREF_LAYOUT_NON_OBJECT = (  2 << 28),
    KB_BLOCK_BYREF_LAYOUT_STRONG =     (  3 << 28),
    KB_BLOCK_BYREF_LAYOUT_WEAK =       (  4 << 28),
    KB_BLOCK_BYREF_LAYOUT_HAS_COPY_DISPOSE = (1 << 25),
} KBBlockByrefFlags;

struct kb_Block_descriptor_1 {
    uintptr_t reserved;
    uintptr_t size;
};

struct kb_Block_descriptor_2 {
    // requires BLOCK_HAS_COPY_DISPOSE
    void *copy;
    void *dispose;
};

struct kb_Block_descriptor_3 {
    // requires BLOCK_HAS_SIGNATURE
    const char *signature;
    const char *layout;     // contents depend on BLOCK_HAS_EXTENDED_LAYOUT
};

struct kb_Block_layout {
    void *isa;
    volatile int32_t flags; // contains ref count
    int32_t reserved;
    void *invoke;
    struct Block_descriptor_1 *descriptor;
    // imported variables
};

struct kb_Block_byref {
    void *isa;
    struct kb_Block_byref *forwarding;
    volatile int32_t flags; // contains ref count
    uint32_t size;
};

struct kb_Block_byref_2 {
    // requires BLOCK_BYREF_HAS_COPY_DISPOSE
    void *byref_keep;
    void *byref_destroy;
};

struct kb_Block_byref_3 {
    // requires BLOCK_BYREF_LAYOUT_EXTENDED
    const char *layout;
};


void showBlockExtendedLayout(id block) {

    // 将block转化为自定义 kb_Block_layout 结构体指针
    struct kb_Block_layout *blockLayout = (__bridge struct kb_Block_layout*)(block);
    // 如果没有引用外部对象也就是没有扩展布局标志的话则直接返回。
    if (! (blockLayout->flags & KB_BLOCK_HAS_EXTENDED_LAYOUT)) return;
    
    // Block_descriptor_1都有，默认加上偏移
    uint8_t *desc = (uint8_t *)blockLayout->descriptor;
    desc += sizeof(struct kb_Block_descriptor_1);
    // 如果有BLOCK_HAS_COPY_DISPOSE，说明引用的有外部对象，需要内存管理，也要加上对应偏移
    if (blockLayout->flags & KB_BLOCK_HAS_COPY_DISPOSE) {
        desc += sizeof(struct kb_Block_descriptor_2);
    }
    
    // 最终转化为Block_descriptor_3中的结构指针。并且当布局值为0时表明没有引用外部对象。
    struct kb_Block_descriptor_3 *desc3 = (struct kb_Block_descriptor_3 *)desc;
    if (desc3->layout == 0)
        return;
    
    
    // 经测试，如果一种类型的外部引用超过0xF，则layout字段就会大于0x1000，如果同种类型外部引用小于0xF，则layout字段就会小于0x1000,进行压缩处理
    const char *extlayoutstr = desc3->layout;

    // 处理压缩布局描述的情况。最终也是包装成 0xPN
    if (extlayoutstr < (const char*)0x1000) {
        // 当扩展布局的值小于0x1000时则是压缩的布局描述，这里分别取出xyz部分的内容进行重新编码。 0xXYZ
        // 取出的xyz则代表每种类型的引用数量
        char compactEncoding[4] = {0};
        unsigned short xyz = (unsigned short)(extlayoutstr);
        unsigned char x = (xyz >> 8) & 0xF; // 取前四位 代表x
        unsigned char y = (xyz >> 4) & 0xF; // 取中间四位 代表y
        unsigned char z = (xyz >> 0) & 0xF; // 取末尾四位 代表z
        
        int idx = 0;
        if (x != 0) {
            // 0x30 | x
            compactEncoding[idx++] = (KB_BLOCK_LAYOUT_STRONG<<4) | x;
        }
        
        if (y != 0) {
            // 0x40 | y
            compactEncoding[idx++] = (KB_BLOCK_LAYOUT_BYREF<<4) | y;
        }
        
        if (z != 0) {
            // 0x50 | z
            compactEncoding[idx++] = (KB_BLOCK_LAYOUT_WEAK<<4) | z;
        }
        compactEncoding[idx++] = 0;
        extlayoutstr = compactEncoding;
    }
    
    unsigned char *blockmemoryAddr = (__bridge void*)block;
    // 得到外部引用对象的开始偏移位置
    int refObjOffset = sizeof(struct kb_Block_layout);
    
    for (int i = 0; i < strlen(extlayoutstr); i++) {
        
        // 取出字节中所表示的类型和数量。
        unsigned char PN = extlayoutstr[i];
        // P 取高4位 描述引用的类型。
        int P = (PN >> 4) & 0xF;
        // N 取低4位 代表数量
        int N = (PN & 0xF);
       
				// 这里只对类型为3，4，5，6四种类型进行处理。
        if (P >= KB_BLOCK_LAYOUT_STRONG && P <= KB_BLOCK_LAYOUT_UNRETAINED) {
            
            for (int j = 0; j < N; j++) {
                // 强引用类型
                if (P == KB_BLOCK_LAYOUT_STRONG) {
                    // 根据偏移得到引用外部对象的地址。并转化为OC对象。
                    void *refObjAddr = *(void**)(blockmemoryAddr + refObjOffset);
                    id refObj =  (__bridge id) refObjAddr;
                    
                    NSLog(@"the refObj is:%@  type is KB_BLOCK_LAYOUT_STRONG", refObj);
                    
                }
                // 如果使用__block修饰了oc对象或者block 没有加__weak，也会存在强引用的情况，首先要拿到byref对象，再去byref对象内部找到具体的对象
                else if (P == KB_BLOCK_LAYOUT_BYREF) {
                    struct kb_Block_byref *byrefObj = *(struct kb_Block_byref **)(blockmemoryAddr + refObjOffset);
                    int32_t flags = byrefObj->flags;
                    if ((flags & KB_BLOCK_BYREF_LAYOUT_HAS_COPY_DISPOSE) && (flags & KB_BLOCK_BYREF_LAYOUT_EXTENDED)) {
                        unsigned char *byrefAddr = (unsigned char *)byrefObj;
                        void *desc = *(void **)(byrefAddr + sizeof(struct kb_Block_byref) + sizeof(struct kb_Block_byref_2));
                        id obj = (__bridge id)desc;
                        
                        NSLog(@"the refObj is:%@  type is KB_BLOCK_LAYOUT_BYREF", obj);
                    }
                }
                
                // 因为布局中保存的是对象的指针，所以偏移要加上一个指针的大小继续获取下一个偏移。
                refObjOffset += sizeof(void*);
            }
        }
    }
}
```

1. 首先转换block，如果flags不包含 `BLOCK_HAS_EXTENDED_LAYOUT` ，则直接return，说明没有引用外部对象

2. 接着找到descriptor的位置，首先加上 `Block_descriptor_1` 的大小，因为descriptor1里的reserved和size是一直存在的

3. 接着根据 `BLOCK_HAS_COPY_DISPOSE` 来判断是否有`Block_descriptor_2`，这一部分是用于外部对象内存管理的。如果有，则加上对应的大小

4. 再接着就到了`Block_descriptor_3`这一部分，这是我们的主角。首先判断一下layout是否有值，有值继续往下处理

5. 如果layout 小于 `0x1000`，则代表是压缩布局，`0xXYZ`

   - 分别取出xyz，取出的值代表的就是对应的数量

     ``` c
     unsigned short xyz = (unsigned short)(extlayoutstr);
     unsigned char x = (xyz >> 8) & 0xF; // 取前四位 代表x
     unsigned char y = (xyz >> 4) & 0xF; // 取中间四位 代表y
     unsigned char z = (xyz >> 0) & 0xF; // 取末尾四位 代表z
     ```

     举个例子讲一下这里的位运算，假如xyz值为 `1010 0010 1100` ，取高四位。首先右移8位之后变为了 `0000 0000 1010` ，高四位的值到了低四位，前面补0，然后按位与上`0xF`，也就是`0000 0000 1111`，所以就取出了低四位的值 `1010`。其他一样的道理

   - 这里分别取出来值以后，把它包装成`0xPN`的形式，方便后面统一处理。如果是x，x表示强引用`BLOCK_LAYOUT_STRONG` , 因为类型是P表示，所以要对`BLOCK_LAYOUT_STRONG`类型左移4位，低4位就是x的值。 用16进制表示就是`0x30 | x`，若x为F，就是`0x3F`，用二进制表示就是 `0011 1111`。 以此类推，把xyz都包装成`0xPN`形式

6. 在上一步如果小于0x1000的话，也已经包装成了0xPN的形式。大于等于0x1000的话，本来就是0xPN的形式。所以下面就开始根据这种格式找出我们所需要的对象了

7. 根据`extlayoutStr`的length进行for循环，取出对应字节中的数据，这时候取出来的就是0XPN的形式，进行位运算取出p和n，然后判断p的类型，分别对`BLOCK_LAYOUT_STRONG` 和 `BLOCK_LAYOUT_BYREF` 这两种类型进行处理

   - 如果`P == BLOCK_LAYOUT_STRONG`，根据偏移找到对象，转换为oc对象打印
   - 如果`P == BLOCK_LAYOUT_BYREF` 根据偏移找到的对象为byref对象，byref对象需要根据flags判断内部引用的是对象还是基本数据类型，如果是对象类型，则根据byref内部偏移找到具体对象打印。如果为基本数据类型，则不做处理
   - 每次循环结束，偏移offset增加一个指针的大小，用于获取下一个对象





## hook block





## 参考资料

[官方文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502-CH1-SW1)

[官方开源库](https://opensource.apple.com/source/libclosure/)

[A look inside blocks](https://www.galloway.me.uk/2013/05/a-look-inside-blocks-episode-3-block-copy/)

[一种查看Block中引用的所有外部对象](https://juejin.im/post/5d7e3b8de51d4561ac7bcd5f)











![](https://raw.githubusercontent.com/gaonian/HexoDocument/master/iOS/block_img/fork@2x.png)
