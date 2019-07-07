title: iOS 逆向

# 前言

## 什么是越狱？

越狱是指通过分析iOS系统的代码，找出iOS系统的漏洞，绕过系统的安全权限检查，最终获取系统root权限的过程。

如果没有越狱，所有的操作就只能局限于沙盒，系统的目录层级我们也不清楚，所有的一切学习都是停留在表面中，无法得知背后的道理。而越狱之后我们就可以访问设备的整个文件系统，更改一些设置，编写一些工具注入指定app等等。设备越狱可以帮助我们更加深入的了解iOS系统的结构，分析系统的行为，为日后的学习和使用打下坚实的基础！

## 准备工作

开始之前，我们需要做一些准备工作:

- 一台完美越狱的手机 (安装了`cydia`)
- phone上在cydia安装 `adv-cmds` ，用于查看当前进程 (`ps -A`)
- mac 上安装 `iFunBox` ，用于查看手机上文件目录

## 目录结构 

![文件目录](./jailbreak_image/jailbreak_1.png)

- `Applications`  存放所有系统的App和来自Cydia的App，不包括从App Store下载的App
- `Library`  系统资源，用户设置。系统日志、系统自带铃声等。重要目录为`Library/MobileSubstrate`，里面存放的是所有基于Cydia Substrate的插件，之后自己开发的插件也是存放在此目录下
- `User`  用户目录，实际指向的是`var/mobile` 
  - `/Users/Media` 存放相册等
  - `/Users/Library`  存放短信，邮件等

- `bin`  存放用户级二进制文件，例如：mv，ls等
- `etc`  存放系统脚本，hosts配置，SSH配置文件等，实际指向`private/etc`
- `sbin`  存放系统级二进制文件，例如：reboot，mount等
- `usr`  用户工具和程序、`/usr/include` 中存放标准`c`头文件。`/usr/lib` 中存放库文件
- `var` 一些经常改动的文件，包括`keychains`、临时文件、从App Store下载的应用
- …...

## 逆向思路

- 界面分析
  - Cycript 内存分析调试
  - Reveal UI层级分析
  - ...
- 代码分析
  - class-dump导出头文件
  - Hopper 反汇编分析
  - ...
- 动态调试
  - LLDB断点调试分析
  - ...
- 编写代码
  - tweak编写
  - CaptainHook
  - ...

- 重签名app，打包安装

#  SSH

iOS和MacOS都是基于Darwin（苹果的一个基于Unix的开源系统内核），所以iOS中同样支持终端操作。SSH是一种网络协议，用于计算机之间的加密登录。

## OpenSSH

- openSSH是SSH协议的免费开源实现，我们通过OpenSSH登录iPhone

- 在iPhone Cydia上搜索安装OpenSSH，安装成功之后可以通过 OpenSSH Access How-To 查看使用说明

- 确保mac和iPhone在同一wifi下，从mac登录iPhone，账户名`root@ip地址`，密码默认为`alpine` 

  ```shell
  ssh root@192.168.124.15
  ```

  ```
  ➜  ~ ssh root@192.168.124.15
  iPhone:~ root# 
  ```

- 登录成功之后，修改登录密码

  ```
  iPhone:~ root# passwd
  ```



## usbmuxd

在wifi不稳定的情况下，通过ssh连接的在输入命令时可能会出现卡顿的情况（ssh走的是tcp协议，mac是通过网络连接的方式连接到wifi的）。所以为了加快传输速度，还可以通过USB

  连接的方式连接到iPhone。

mac上有一个服务程序`usbmuxd`，可以将mac的数据通过USB传输到iPhone

- 下载工具包 https://cgit.sukimashita.com/usbmuxd.git/snapshot/usbmuxd-1.0.8.tar.gz

  ```
  python python-client/tcprelay.py -t 22:10010
  ```

  将iPhone上的22端口映射到mac本地的10010端口

  

- 新开终端窗口，访问本地的10010端口，同样连接到iPhone上

  ```
  ➜  ~ ssh root@localhost -p 10010
  iPhone:~ root# 
  ```

  

# Cycript

Cycript 是一个允许开发者使用Objective-C++ 和 JavaScript 组合语法查看及修改运行时App内存信息的工具。官网 http://www.cycript.org/ 提供了一些经典使用方法

![cycript](./jailbreak_image/jailbreak_2.png)

- 在iPhone Cydia中搜索Cycript并安装

  ![cycript](./jailbreak_image/jailbreak_4.png)

- 基本使用

  ```shell
  iPhone:~ root# cycript
  cy# var a = 3
  3
  cy# var b = 4
  4
  cy# a + b
  7
  cy# 
  
  iPhone:~ root# cycript -p SpringBoard
  cy# UIApp
  #"<SpringBoard: 0x15481c200>"
  cy# NSHomeDirectory()
  @"/var/mobile"
  
  ...
  ```

- 通过Cycript分析应用

  ```shell
  iPhone:~ root# cycript -p WeChat
  cy# [NSBundle mainBundle].bundleIdentifier
  @"com.tencent.xin"
  cy# [NSBundle mainBundle].bundlePath
  @"/var/mobile/Containers/Bundle/Application/C2F6EDC4-2B37-4A1E-B974-D9DE607CF2A8/WeChat.app"
  cy# NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)[0]
  @"/var/mobile/Containers/Data/Application/36F8BF2B-D67D-4072-8B0F-FCCD21FBA31F/Documents"
  cy# UIApp.keyWindow.rootViewController
  #"<MMUINavigationController: 0x1400a9800>"
  cy# 
  ```

- Cycript高级应用，自定义脚本

  Cycript本身支持加载自己的脚本，具体可参考 https://github.com/CoderMJLee/mjcript

  ```shell
  iPhone:~ root# cycript -p WeChat
  cy# @import mjcript
  {}
  cy# MJAppId
  @"com.tencent.xin"
  cy# MJAppPath
  @"/var/mobile/Containers/Bundle/Application/C2F6EDC4-2B37-4A1E-B974-D9DE607CF2A8/WeChat.app"
  cy# MJDocPath 
  @"/var/mobile/Containers/Data/Application/36F8BF2B-D67D-4072-8B0F-FCCD21FBA31F/Documents"
  cy# MJFrontVc()
  #"<WCAccountMainLoginViewController: 0x13faaf600>"
  ```

  

# 脱壳

应用上传至App Store后，苹果会对其进行加密，当应用运行时才会动态解密，在这样的情况下是无法使用一些工具对app进行分析破解的。所以，我们要先对加密过的app进行解密，也就是脱壳操作。iOS中主要脱壳工具有两种 `Clutch` `dumpdecrypted`

## 如何查看是否脱壳

- MachOView

  https://github.com/gdbinit/MachOView/ 

  在MachOView中查看`Load Commands` -> `LC_ENCRYPTION_INFO` -> `Crypt ID` ，0代表未加密，1代表加密

  ![MachOView](./jailbreak_image/jailbreak_7.png)

- 通过otool命令查看

  ```shell
  ➜  Desktop otool -l iQiYiPhoneVideo.decrypted | grep crypt
  iQiYiPhoneVideo.decrypted:
       cryptoff 16384
      cryptsize 97370112
        cryptid 0
  ➜  Desktop 
  ```

## Clutch

https://github.com/KJCracks/Clutch/releases ，下载最新release版本，建议去掉版本号，将`Clutch` 文件拷贝到iPhone的`/usr/bin` 目录下，赋予执行权限并使用

- `Clutch -i` 列出设备上安装的app和bundle id

```shell
iPhone:~ root# Clutch -i
Installed apps:
1:   爱奇艺-破冰行动全网独播 <com.qiyi.iphone>
2:   喜马拉雅FM「听书社区」电台有声小说相声评书 <com.gemd.iting>
```

- `Clutch -d 1` `Clutch -d com.qiyi.iphone` 输入序号或者bundleId进行脱壳操作，成功之后会生成一个新的ipa文件

```shell
iPhone:~ root# Clutch -d 1
Zipping iQiYiPhoneVideo.app
...
...
DONE: /private/var/mobile/Documents/Dumped/com.qiyi.iphone-iOS9.0-(Clutch-2.0.4).ipa
Finished dumping com.qiyi.iphone in 65.5 seconds

iPhone:~ root# 
```

## dumpdecrypted

- https://github.com/stefanesser/dumpdecrypted ，下载源代码，然后切换到目录下，执行`make` 进行编译，编译完成会生成一个`dumpdecrypted.dylib` 动态库文件

![dumpdecrypted](./jailbreak_image/jailbreak_5.png)

- 将dylib拷贝到iPhone上`/var/root` 目录下，连接iPhone，终端进入dylib所在目录，使用环境变量`DYLD_INSERT_LIBRARIES` 将dylib注入到需要脱壳的可执行文件

  `DYLD_INSERT_LIBRARIES=dumpdecrypted.dylib 可执行文件路径`

  `xx.decrypted`就是脱壳后的ipa文件

![dumpdecrypted](./jailbreak_image/jailbreak_6.png)

# class-dump

class-dump是一个用于从可执行文件中获取类，方法和属性信息的工具，下载地址（https://github.com/nygard/class-dump），下载后把class-dump二进制文件拷贝到`/usr/local/bin` 目录下，可供全局访问。class-dump通过解析mach-o文件，来得到类的信息，具体实现过程可以进一步阅读源码。

```
class-dump -H 可执行文件 -o 生成头文件存放目录
```

```shell
➜  Desktop class-dump -H iQiYiPhoneVideo.decrypted -o iqiyiHeaders
2019-07-07 18:21:53.488 class-dump[10899:565880] Warning: Parsing instance variable type failed, marker
2019-07-07 18:21:56.056 class-dump[10899:565880] Warning: Parsing instance variable type failed, _pendingCount
2019-07-07 18:21:59.021 class-dump[10899:565880] Warning: Parsing instance variable type failed, _cameraTextureUploaded
2019-07-07 18:21:59.024 class-dump[10899:565880] Warning: Parsing instance variable type failed, _needUpdateEffect
2019-07-07 18:21:59.253 class-dump[10899:565880] Warning: Parsing method types failed, SetNofityTarget:
2019-07-07 18:21:59.254 class-dump[10899:565880] Warning: Parsing method types failed, wrapper
2019-07-07 18:21:59.255 class-dump[10899:565880] Warning: Parsing instance variable type failed, wrapper
2019-07-07 18:22:21.618 class-dump[10899:565880] Warning: Parsing method types failed, SetNofityTarget:
```

![headers](./jailbreak_image/jailbreak_8.png)

# Reveal

Reveal是iOS上用于查看程序界面结构和调试界面的工具，和xcode中的UI调试有点像。官网地址（https://revealapp.com/）。Reveal可以在开发过程中动态调试修改程序的样式，也可以注入第三方查看应用的界面结构。

- mac安装Reveal

- 在iPhone Cydia中搜索`Reveal Loader` 安装，安装成功之后在系统设置直接会有 Reveal 选项，点击进入可以选择要调试的app

  ![Reveal](./jailbreak_image/jailbreak_9.png)

  ![Reveal](./jailbreak_image/jailbreak_10.png)

- 找到mac的Reveal中的RevealServer文件，覆盖iPhone的`/Library/RHRevealLoader/RevealServer` 文件，重启iPhone

  ![Reveal](./jailbreak_image/jailbreak_11.png)

![Reveal](./jailbreak_image/jailbreak_12.png)



安装成功之后在iPhone上打开进程，可以直接在Reveal中选择调试

![Reveal](./jailbreak_image/jailbreak_13.png)



# Hopper

Hopper 是一款反汇编工具，提供Mac和Linux版本。Hopper可以显示被分析文件的反汇编代码、流程图及伪代码，也可以直接修改汇编指令，生成新的可执行文件

![Hopper](./jailbreak_image/jailbreak_14.png)

# 动态调试

动态调试与静态分析是相辅相成的。静态分析只能分析静态的函数内部执行，动态调试可以获取程序在运行时的参数传递，执行流程及寄存器内存等信息。

## LLDB

LLDB是xcode自带的调试工具，当使用xcode调试app时，xcode会自动将debugserver安装到iPhone`/Developer/usr/bin/debugserver` 目录上，iPhone启动服务，等待xcode进行连接远程调试。

### debugserver权限问题

debugserver只能调试自己开发的app，调试第三方app是没有权限的

如果希望debugserver调试其他app，则需要对debugserver赋予一定的权限，并重新签名

- get-task-allow
- task_for_pid-allow

### 重签debugserver

- 拷贝debugserver到mac上，导出debugserver原来的签名权限

  ```shell
  ➜  Desktop ldid -e debugserver > debugserver.entitlements
  ```

- 给debugserver.entitlements添加get-task-allow、task_for_pid-allow权限

- 通过ldid命令重新签名

  ```shell
  ➜  Desktop ldid -Sdebugserver.entitlements debugserver
  ```

- 将重新签名的debugserver放到iPhone `/usr/bin` 目录下

###debugserver使用

- iPhone，debugserver在iPhone上要附加到某一个正在运行的进程中

  `debugserver *:端口号 -a 进程`

  端口号：使用iPhone某个端口启动debugserver服务（除保留端口外）

  进程：app的进程信息，进程id或者进程名称，建议进程名称

  ```
  iPhone:~ root# debugserver *:10011 -a iQiYiPhoneVideo
  ```

- mac上启动lldb服务，连接debugserver服务

  ```shell
  ➜  Desktop lldb
  (lldb) process connect connect://localhost:10011
  ```

  连接成功之后被附加的app会处在断点状态，可按`c` 让程序继续运行

### 常用LLDB指令



# restore_symbols



# theos



# 重签名



# MonkeyDev

