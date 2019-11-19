---
title : iOS hook
---

hook，又叫钩子，原理简单来说就是在正常调用之前，先获取该函数的控制权，可以进行数据加工等等操作，处理完成之后再调用原有函数。

常用于hook系统函数处理我们的逻辑，hook第三方不能修改的函数，进行三方注入等等

iOS上的hook简单的分为三种实现

- Method Swizzle

- fishhook

- Cydia Substrate

  

## Method Swizzle

method swizzle由于是基于runtime的，所以只能用于hook OC方法，利用runtime的特性，在运行时交换方法实现。





## fishhook

fishhook是facebook开源的用于hook系统C语言函数的轻量级库





## Cydia Substrate

Cydia Substrate是cydia作者写的hook库，可以实现对OC和C的hook，





