---
title : 通过汇编认识if-else
---

## if-else
```
int i = 4;
if (i == 0) {
    NSLog(@"i = 0");
} else if (i == 1) {
    NSLog(@"i = 1");
} else if (i == 2) {
    NSLog(@"i = 2");
} else if (i == 3) {
    NSLog(@"i = 3");
} else if (i == 4) {
    NSLog(@"i = 4");
} else {
    NSLog(@"other");
}

// $0*4 : 4
// -0x14(%rbp) 变量a
// int a = 4
0x100000e6b <+27>:  movl   $0x4, -0x14(%rbp)

// comp 0 -0x14(%rbp) (4)
// if (i == 0)
0x100000e72 <+34>:  cmpl   $0x0, -0x14(%rbp)
0x100000e76 <+38>:  movq   %rax, -0x20(%rbp)
0x100000e7a <+42>:  jne    0x100000e96               ; <+70> at main.m:16:22

0x100000e80 <+48>:  leaq   0x1a1(%rip), %rax         ; @"i = 0"
0x100000e87 <+55>:  movq   %rax, %rdi
0x100000e8a <+58>:  movb   $0x0, %al
0x100000e8c <+60>:  callq  0x100000f4c               ; symbol stub for: NSLog
0x100000e91 <+65>:  jmp    0x100000f3b               ; <+235> at main.m:27:5

// if (i == 1)
// comp 1 -0x14(%rbp) (4)
0x100000e96 <+70>:  cmpl   $0x1, -0x14(%rbp)
// 如果不等，跳转到 0x100000eb6 下一个条件语句比较
0x100000e9a <+74>:  jne    0x100000eb6               ; <+102> at main.m:18:22
// 如果相等，call NSLog 打印 @“i = 1”
0x100000ea0 <+80>:  leaq   0x1a1(%rip), %rax         ; @"i = 1"
0x100000ea7 <+87>:  movq   %rax, %rdi
0x100000eaa <+90>:  movb   $0x0, %al
0x100000eac <+92>:  callq  0x100000f4c               ; symbol stub for: NSLog
// 跳到函数尾部
0x100000eb1 <+97>:  jmp    0x100000f36               ; <+230> at main.m

// if (i == 2)
0x100000eb6 <+102>: cmpl   $0x2, -0x14(%rbp)
0x100000eba <+106>: jne    0x100000ed6               ; <+134> at main.m:20:22
0x100000ec0 <+112>: leaq   0x1a1(%rip), %rax         ; @"i = 2"
0x100000ec7 <+119>: movq   %rax, %rdi
0x100000eca <+122>: movb   $0x0, %al
0x100000ecc <+124>: callq  0x100000f4c               ; symbol stub for: NSLog
0x100000ed1 <+129>: jmp    0x100000f31               ; <+225> at main.m

// if (i == 3)
0x100000ed6 <+134>: cmpl   $0x3, -0x14(%rbp)
0x100000eda <+138>: jne    0x100000ef6               ; <+166> at main.m:22:22
0x100000ee0 <+144>: leaq   0x1a1(%rip), %rax         ; @"i = 3"
0x100000ee7 <+151>: movq   %rax, %rdi
0x100000eea <+154>: movb   $0x0, %al
0x100000eec <+156>: callq  0x100000f4c               ; symbol stub for: NSLog
0x100000ef1 <+161>: jmp    0x100000f2c               ; <+220> at main.m

// if (i == 4)
0x100000ef6 <+166>: cmpl   $0x4, -0x14(%rbp)
0x100000efa <+170>: jne    0x100000f16               ; <+198> at main.m
0x100000f00 <+176>: leaq   0x1a1(%rip), %rax         ; @"i = 4"
0x100000f07 <+183>: movq   %rax, %rdi
0x100000f0a <+186>: movb   $0x0, %al
0x100000f0c <+188>: callq  0x100000f4c               ; symbol stub for: NSLog
0x100000f11 <+193>: jmp    0x100000f27               ; <+215> at main.m

// else { other }
// 如果所有条件语句都不相等，执行elses
0x100000f16 <+198>: leaq   0x1ab(%rip), %rax         ; @"other"
0x100000f1d <+205>: movq   %rax, %rdi
0x100000f20 <+208>: movb   $0x0, %al
0x100000f22 <+210>: callq  0x100000f4c               ; symbol stub for: NSLog
0x100000f27 <+215>: jmp    0x100000f2c               ; <+220> at main.m

0x100000f2c <+220>: jmp    0x100000f31               ; <+225> at main.m
0x100000f31 <+225>: jmp    0x100000f36               ; <+230> at main.m
0x100000f36 <+230>: jmp    0x100000f3b               ; <+235> at main.m:27:5

// 相当于函数最后的大括号 }
0x100000f3b <+235>: movq   -0x20(%rbp), %rdi


0x100000f4b <+251>: retq


```

xcode模拟器采用的是AT&T汇编
通过以上汇编代码可以看出if-else会对每一条条件语句逐步进行判断
所以在使用的时候尽量把命中率高的条件放到前面，少了很多不必要的执行，提升了效率。

## switch

1. case较少 （1、2 为例）
2. case连续，正常 （1、2、3、4、5、为例）
3. case不连续 （1、3、5、7 为例）
4. case差别较大 （1、2、5、100 为例）
5. case差别很大 （1、2、5、10000 为例）

### case较少
```
int i = 4;
switch (i) {
    case 0:
        NSLog(@"i = 0");
        break;
    case 1:
        NSLog(@"i = 1");
        break;
    default:
        NSLog(@"other");
        break;
}

// int a = 4
0x100000ebb <+27>:  movl   $0x4, -0x14(%rbp)
// %edi = -0x14(%rbp) = $0x4 = 4
// %edi = 4
0x100000ec2 <+34>:  movl   -0x14(%rbp), %edi
// 测试%edi是否为空
0x100000ec5 <+37>:  testl  %edi, %edi
// -0x20(%rbp) = %rax
0x100000ec7 <+39>:  movq   %rax, -0x20(%rbp)
// -0x24(%rbp) = %edi = 4
// -0x24(%rbp) = 4
0x100000ecb <+43>:  movl   %edi, -0x24(%rbp)
// test %edi = 0 则跳转 0x100000eed
0x100000ece <+46>:  je     0x100000eed               ; <+77> at main.m
// test %edi != 0 则跳转 0x100000ed9 下一行
0x100000ed4 <+52>:  jmp    0x100000ed9               ; <+57> at main.m:14:9
// %eax = -0x24(%rbp) = 4
// %eax = 4
0x100000ed9 <+57>:  movl   -0x24(%rbp), %eax
// %eax = %eax - $0x1 = 4 - 1 = 3
// %eax = 3
0x100000edc <+60>:  subl   $0x1, %eax
// -0x28(%rbp) = %eax = 3
// -0x28(%rbp) = 3
0x100000edf <+63>:  movl   %eax, -0x28(%rbp)
// if (-0x28(%rbp) == 0)  跳转 0x100000f03（case 1： i = 1）
0x100000ee2 <+66>:  je     0x100000f03               ; <+99> at main.m
// if (-0x28(%rbp) != 0)  跳转 0x100000f19（default: other）
0x100000ee8 <+72>:  jmp    0x100000f19               ; <+121> at main.m

// case 0:
0x100000eed <+77>:  leaq   0x134(%rip), %rax         ; @"i = 0"
0x100000ef4 <+84>:  movq   %rax, %rdi
0x100000ef7 <+87>:  movb   $0x0, %al
0x100000ef9 <+89>:  callq  0x100000f4c               ; symbol stub for: NSLog

0x100000efe <+94>:  jmp    0x100000f2a               ; <+138> at main.m

0x100000f03 <+99>:  leaq   0x13e(%rip), %rax         ; @"i = 1"
0x100000f0a <+106>: movq   %rax, %rdi
0x100000f0d <+109>: movb   $0x0, %al
0x100000f0f <+111>: callq  0x100000f4c               ; symbol stub for: NSLog

0x100000f14 <+116>: jmp    0x100000f2a               ; <+138> at main.m

0x100000f19 <+121>: leaq   0x148(%rip), %rax         ; @"other"
0x100000f20 <+128>: movq   %rax, %rdi
0x100000f23 <+131>: movb   $0x0, %al
0x100000f25 <+133>: callq  0x100000f4c               ; symbol stub for: NSLog

```
通过以上的汇编代码分析，可以看出在case比较少的时候，实现方式和if-else是一样的，都是逐个比较。 所以可以说在条件比较小的时候，使用if-else和switch效率是一样的。

### case连续，正常
```
int i = 4;
switch (i) {
    case 1:
        NSLog(@"i = 1");
        break;
    case 2:
        NSLog(@"i = 2");
        break;
    case 3:
        NSLog(@"i = 3");
        break;
    case 4:
        NSLog(@"i = 4");
        break;
    case 5:
        NSLog(@"i = 5");
        break;


// int a = 4
// $0x4 常量4
// -0x14(%rbp) a
0x100000e5b <+27>:  movl   $0x4, -0x14(%rbp)
// %edi = -0x14(%rbp) = a = 4
// %edi = 4
0x100000e62 <+34>:  movl   -0x14(%rbp), %edi
// decl 自减
// %edi = %edi - 1  = 4 - 1 = 3
// %edi = 3
0x100000e65 <+37>:  decl   %edi
// %esi = %edi = 3
// %esi = 3
0x100000e67 <+39>:  movl   %edi, %esi
0x100000e69 <+41>:  subl   $0x4, %edi
//
0x100000e6c <+44>:  movq   %rax, -0x20(%rbp)
0x100000e70 <+48>:  movq   %rsi, -0x28(%rbp)
0x100000e74 <+52>:  movl   %edi, -0x2c(%rbp)

// ja 大于则转移到目标指令执行。
0x100000e77 <+55>:  ja     0x100000eff               ; <+191> at main.m
0x100000e7d <+61>:  leaq   0xb0(%rip), %rax          ; main + 244

0x100000e84 <+68>:  movq   -0x28(%rbp), %rcx
0x100000e88 <+72>:  movslq (%rax,%rcx,4), %rdx
0x100000e8c <+76>:  addq   %rax, %rdx
0x100000e8f <+79>:  jmpq   *%rdx

0x100000e91 <+81>:  leaq   0x190(%rip), %rax         ; @"i = 1"
0x100000e98 <+88>:  movq   %rax, %rdi
0x100000e9b <+91>:  movb   $0x0, %al
0x100000e9d <+93>:  callq  0x100000f48               ; symbol stub for: NSLog

0x100000ea2 <+98>:  jmp    0x100000f10               ; <+208> at main.m

0x100000ea7 <+103>: leaq   0x19a(%rip), %rax         ; @"i = 2"
0x100000eae <+110>: movq   %rax, %rdi
0x100000eb1 <+113>: movb   $0x0, %al
0x100000eb3 <+115>: callq  0x100000f48                ; symbol stub for: NSLog

0x100000eb8 <+120>: jmp    0x100000f10               ; <+208> at main.m

0x100000ebd <+125>: leaq   0x1a4(%rip), %rax         ; @"i = 3"
0x100000ec4 <+132>: movq   %rax, %rdi
0x100000ec7 <+135>: movb   $0x0, %al
0x100000ec9 <+137>: callq  0x100000f48               ; symbol stub for: NSLog

0x100000ece <+142>: jmp    0x100000f10               ; <+208> at main.m

0x100000ed3 <+147>: leaq   0x1ae(%rip), %rax         ; @"i = 4"
0x100000eda <+154>: movq   %rax, %rdi
0x100000edd <+157>: movb   $0x0, %al
0x100000edf <+159>: callq  0x100000f48               ; symbol stub for: NSLog

0x100000ee4 <+164>: jmp    0x100000f10               ; <+208> at main.m

0x100000ee9 <+169>: leaq   0x1b8(%rip), %rax         ; @"i = 5"
0x100000ef0 <+176>: movq   %rax, %rdi
0x100000ef3 <+179>: movb   $0x0, %al
0x100000ef5 <+181>: callq  0x100000f48               ; symbol stub for: NSLog

0x100000efa <+186>: jmp    0x100000f10               ; <+208> at main.m

0x100000eff <+191>: leaq   0x1c2(%rip), %rax         ; @"other"
0x100000f06 <+198>: movq   %rax, %rdi
0x100000f09 <+201>: movb   $0x0, %al
0x100000f0b <+203>: callq  0x100000f48               ; symbol stub for: NSLog

```
### case不连续
```
int i = 4;
switch (i) {
    case 2:
        NSLog(@"i = 2");
        break;
    case 4:
        NSLog(@"i = 4");
        break;
    case 6:
        NSLog(@"i = 6");
        break;
    case 8:
        NSLog(@"i = 8");
        break;
    default:
        NSLog(@"other");
        break;
}

0x100000e6b <+27>:  movl   $0x4, -0x14(%rbp)
0x100000e72 <+34>:  movl   -0x14(%rbp), %edi
0x100000e75 <+37>:  addl   $-0x2, %edi
0x100000e78 <+40>:  movl   %edi, %esi
0x100000e7a <+42>:  subl   $0x6, %edi
0x100000e7d <+45>:  movq   %rax, -0x20(%rbp)
0x100000e81 <+49>:  movq   %rsi, -0x28(%rbp)
0x100000e85 <+53>:  movl   %edi, -0x2c(%rbp)
0x100000e88 <+56>:  ja     0x100000efa               ; <+170> at main.m
0x100000e8e <+62>:  leaq   0x9b(%rip), %rax          ; main + 224
0x100000e95 <+69>:  movq   -0x28(%rbp), %rcx
0x100000e99 <+73>:  movslq (%rax,%rcx,4), %rdx
0x100000e9d <+77>:  addq   %rax, %rdx
0x100000ea0 <+80>:  jmpq   *%rdx
0x100000ea2 <+82>:  leaq   0x17f(%rip), %rax         ; @"i = 2"
0x100000ea9 <+89>:  movq   %rax, %rdi
0x100000eac <+92>:  movb   $0x0, %al
0x100000eae <+94>:  callq  0x100000f4c               ; symbol stub for: NSLog
0x100000eb3 <+99>:  jmp    0x100000f0b               ; <+187> at main.m
0x100000eb8 <+104>: leaq   0x189(%rip), %rax         ; @"i = 4"
0x100000ebf <+111>: movq   %rax, %rdi
0x100000ec2 <+114>: movb   $0x0, %al
0x100000ec4 <+116>: callq  0x100000f4c               ; symbol stub for: NSLog
0x100000ec9 <+121>: jmp    0x100000f0b               ; <+187> at main.m
0x100000ece <+126>: leaq   0x193(%rip), %rax         ; @"i = 6"
0x100000ed5 <+133>: movq   %rax, %rdi
0x100000ed8 <+136>: movb   $0x0, %al
0x100000eda <+138>: callq  0x100000f4c               ; symbol stub for: NSLog
0x100000edf <+143>: jmp    0x100000f0b               ; <+187> at main.m
0x100000ee4 <+148>: leaq   0x19d(%rip), %rax         ; @"i = 8"
0x100000eeb <+155>: movq   %rax, %rdi
0x100000eee <+158>: movb   $0x0, %al
0x100000ef0 <+160>: callq  0x100000f4c               ; symbol stub for: NSLog
0x100000ef5 <+165>: jmp    0x100000f0b               ; <+187> at main.m
0x100000efa <+170>: leaq   0x1a7(%rip), %rax         ; @"other"
0x100000f01 <+177>: movq   %rax, %rdi
0x100000f04 <+180>: movb   $0x0, %al
0x100000f06 <+182>: callq  0x100000f4c               ; symbol stub for: NSLog

```
### case差别较大
### case差别很大
```
int i = 4;
switch (i) {
    case 1:
        NSLog(@"i = 1");
        break;
    case 2:
        NSLog(@"i = 2");
        break;
    case 3:
        NSLog(@"i = 3");
        break;
    case 10000:
        NSLog(@"i = 10000");
        break;
    default:
        NSLog(@"other");
        break;
}

// int a = 4
0x100000e5b <+27>:  movl   $0x4, -0x14(%rbp)
// %edi = -0x14(%rbp) = 4
0x100000e62 <+34>:  movl   -0x14(%rbp), %edi
// %ecx = %edi = 4
0x100000e65 <+37>:  movl   %edi, %ecx
// %ecx = %ecx - $0x1 = 4 - 1 = 3
// %ecx = 3
0x100000e67 <+39>:  subl   $0x1, %ecx

// -0x20(%rbp) = 0
0x100000e6a <+42>:  movq   %rax, -0x20(%rbp)
// -0x24(%rbp) = %edi = 4
// -0x24(%rbp) = 4
0x100000e6e <+46>:  movl   %edi, -0x24(%rbp)
// -0x28(%rbp) = %ecx = 3
// -0x28(%rbp) = 3
0x100000e71 <+49>:  movl   %ecx, -0x28(%rbp)

// if (-0x28(%rbp) == 0) 跳转 0x100000ebd （case：1   i = 1）
0x100000e74 <+52>:  je     0x100000ebd               ; <+125> at main.m
// if (-0x28(%rbp) != 0) 跳转 0x100000e7f
0x100000e7a <+58>:  jmp    0x100000e7f               ; <+63> at main.m:14:9

// %eax = -0x24(%rbp) = 4
0x100000e7f <+63>:  movl   -0x24(%rbp), %eax
// %eax = %eax - $0x2 = 4 - 2 = 2
0x100000e82 <+66>:  subl   $0x2, %eax
// -0x2c(%rbp) = %eax = 2
0x100000e85 <+69>:  movl   %eax, -0x2c(%rbp)
// if (-0x2c(%rbp) == 0) 跳转 0x100000ed3 （case：2   i = 2）
0x100000e88 <+72>:  je     0x100000ed3               ; <+147> at main.m
// if (-0x2c(%rbp) != 0) 跳转 0x100000e93
0x100000e8e <+78>:  jmp    0x100000e93               ; <+83> at main.m:14:9

// %eax = -0x24(%rbp) = 4
0x100000e93 <+83>:  movl   -0x24(%rbp), %eax
// %eax = %eax - $0x3 = 4 - 3 = 1
0x100000e96 <+86>:  subl   $0x3, %eax
// -0x30(%rbp) = %eax = 1
0x100000e99 <+89>:  movl   %eax, -0x30(%rbp)
// if (-0x30(%rbp) == 0) 跳转 0x100000ee9 （case：3   i = 3）
0x100000e9c <+92>:  je     0x100000ee9               ; <+169> at main.m
// if (-0x30(%rbp) != 0) 跳转 0x100000ea7 （
0x100000ea2 <+98>:  jmp    0x100000ea7               ; <+103> at main.m:14:9

0x100000ea7 <+103>: movl   -0x24(%rbp), %eax
0x100000eaa <+106>: subl   $0x2710, %eax             ; imm = 0x2710
0x100000eaf <+111>: movl   %eax, -0x34(%rbp)
0x100000eb2 <+114>: je     0x100000eff               ; <+191> at main.m
0x100000eb8 <+120>: jmp    0x100000f15               ; <+213> at main.m



0x100000ebd <+125>: leaq   0x164(%rip), %rax         ; @"i = 1"
0x100000ec4 <+132>: movq   %rax, %rdi
0x100000ec7 <+135>: movb   $0x0, %al
0x100000ec9 <+137>: callq  0x100000f48               ; symbol stub for: NSLog

0x100000ece <+142>: jmp    0x100000f26               ; <+230> at main.m

0x100000ed3 <+147>: leaq   0x16e(%rip), %rax         ; @"i = 2"
0x100000eda <+154>: movq   %rax, %rdi
0x100000edd <+157>: movb   $0x0, %al
0x100000edf <+159>: callq  0x100000f48               ; symbol stub for: NSLog

0x100000ee4 <+164>: jmp    0x100000f26               ; <+230> at main.m

0x100000ee9 <+169>: leaq   0x178(%rip), %rax         ; @"i = 3"
0x100000ef0 <+176>: movq   %rax, %rdi
0x100000ef3 <+179>: movb   $0x0, %al
0x100000ef5 <+181>: callq  0x100000f48               ; symbol stub for: NSLog

0x100000efa <+186>: jmp    0x100000f26               ; <+230> at main.m

0x100000eff <+191>: leaq   0x182(%rip), %rax         ; @"i = 10000"
0x100000f06 <+198>: movq   %rax, %rdi
0x100000f09 <+201>: movb   $0x0, %al
0x100000f0b <+203>: callq  0x100000f48               ; symbol stub for: NSLog

0x100000f10 <+208>: jmp    0x100000f26               ; <+230> at main.m

0x100000f15 <+213>: leaq   0x18c(%rip), %rax         ; @"other"
0x100000f1c <+220>: movq   %rax, %rdi
0x100000f1f <+223>: movb   $0x0, %al
0x100000f21 <+225>: callq  0x100000f48               ; symbol stub for: NSLog
```