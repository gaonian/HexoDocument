---
title : iOS hook
---

hook，又叫钩子，原理简单来说就是在正常调用之前，先获取该函数的控制权，可以进行数据加工等等操作，处理完成之后再调用原有函数。

常用于hook系统函数处理我们的逻辑，hook第三方不能修改的函数，进行三方注入等等

本文从四个方面去分别讲解hook的工具以及原理

- Method Swizzle

- fishhook

- libffi

- Cydia Substrate

  

## Method Swizzle

method swizzle由于是基于runtime的，所以只能用于hook OC方法，利用runtime的特性，在运行时交换方法实现。

来看一段简单的实现

```objective-c
+ (void)load {
    Method oldMethod = class_getInstanceMethod(self, @selector(test));
    Method newMethod = class_getInstanceMethod(self, @selector(newText));
    method_exchangeImplementations(oldMethod, newMethod);
}

- (void)viewDidLoad {
    [super viewDidLoad];

    [self test];
}

- (void)test {
    NSLog(@"---原来test---");
}

- (void)newText {
    NSLog(@"---新test---");
}
```

```
2019-12-11 13:20:14.812644+0800 testMethodSwizzing[31017:1740918] ---新test---
```

执行结果为新test，可以看出来已经实现了方法的交换。

接下来看一下这几个方法的内部具体实现，[objc源码在这里](https://opensource.apple.com/tarballs/objc4/)

- class_getInstanceMethod(Class cls, SEL sel)

```objective-c
Method class_getInstanceMethod(Class cls, SEL sel)
{
    if (!cls  ||  !sel) return nil;

    // This deliberately avoids +initialize because it historically did so.

    // This implementation is a bit weird because it's the only place that 
    // wants a Method instead of an IMP.

#warning fixme build and search caches
        
    // Search method lists, try method resolver, etc.
    lookUpImpOrNil(cls, sel, nil, 
                   NO/*initialize*/, NO/*cache*/, YES/*resolver*/);

#warning fixme build and search caches

    return _class_getMethod(cls, sel);
}
```

`lookUpImpOrNil`内部调用了`lookUpImpOrForward`，在lookUpImpOrForward内部进行方法查找和消息转发。

在`_class_getMethod`中返回具体的Method结构体。`_class_getMethod`调用了`getMethod_nolock`, `getMethod_nolock`中又调用了`getMethodNoSuper_nolock`

```objective-c
static method_t *
getMethodNoSuper_nolock(Class cls, SEL sel)
{
    runtimeLock.assertLocked();

    assert(cls->isRealized());
    // fixme nil cls? 
    // fixme nil sel?

    // 遍历methodlist
    for (auto mlists = cls->data()->methods.beginLists(), 
              end = cls->data()->methods.endLists(); 
         mlists != end;
         ++mlists)
    {
        // 根据方法的name查找，如果找到则返回method_t
        method_t *m = search_method_list(*mlists, sel);
        if (m) return m;
    }

    return nil;
}
```

看一下method_t对应的结构体

```objective-c
struct method_t {
    SEL name;  // 方法名
    const char *types;  // 方法签名
    MethodListIMP imp;  // imp就是具体的方法实现
};
```

最终就是通过交换method结构体内的imp来实现方法交换的。



- void method_exchangeImplementations(Method m1, Method m2)

```objective-c
void method_exchangeImplementations(Method m1, Method m2)
{
    if (!m1  ||  !m2) return;

    mutex_locker_t lock(runtimeLock);

    // 交换imp
    IMP m1_imp = m1->imp;
    m1->imp = m2->imp;
    m2->imp = m1_imp;


    // RR/AWZ updates are slow because class is unknown
    // Cache updates are slow because class is unknown
    // fixme build list of classes whose Methods are known externally?
    flushCaches(nil);

    updateCustomRR_AWZ(nil, m1);
    updateCustomRR_AWZ(nil, m2);
}
```

可以清楚的看到就是通过交换两个method内的imp来达到交换的目的



## fishhook

fishhook是facebook开源的用于hook系统C语言函数的轻量级库





## libffi

libffi 中 ffi 的全称是 Foreign Function Interface（外部函数接口），提供最底层的接口，在不确定参数个数和类型的情况下，根据相应规则，完成所需数据的准备，生成相应汇编指令的代码来完成函数调用。





## Cydia Substrate

Cydia Substrate是cydia作者写的hook库，可以实现对OC和C的hook，







