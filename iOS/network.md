为了更好的促进互联网网路的研究和发展，国际标准化组织ISO在1985年制定了网络互连模型

OSI参考模型（Open System Interconnect Reference Model），具有7层结构

| 7    | 应用层（Application）   |
| ---- | ----------------------- |
| 6    | 表示层（Presentation）  |
| 5    | 会话层（Session）       |
| 4    | 运输层（Transport）     |
| 3    | 网络层（Network）       |
| 2    | 数据链路层（Data Link） |
| 1    | 物理层（Physical）      |



TCP/IP协议

| 4    | 应用层（Application）        |
| ---- | ---------------------------- |
| 3    | 运输层（Transport）          |
| 2    | 网际层（Internet）           |
| 1    | 网络接口层（Network Access） |



学习研究

| 5    | 应用层（Application）   |
| ---- | ----------------------- |
| 4    | 运输层（Transport）     |
| 3    | 网络层（Network）       |
| 2    | 数据链路层（Data Link） |
| 1    | 物理层（Physical）      |



网络分层

| 5    | 应用层（Application）   | FTP、HTTP、SMTP、DNS、DHCP | 报文，用户数据 |
| ---- | ----------------------- | -------------------------- | -------------- |
| 4    | 运输层（Transport）     | TCP  UDP                   | 段（Segments） |
| 3    | 网络层（Network）       | IP  ARP  ICMP              | 包（Packets）  |
| 2    | 数据链路层（Data Link） | CSMA/CD  PPP               | 帧（Frames）   |
| 1    | 物理层（Physical）      |                            | 比特流（Bits） |



# 1. 物理层（Physical）

物理层定义了接口标准、线缆标准、传输速率、传输方式等



### 模拟信号（Analog Signal）

连续的信号，适合长距离传输

抗干扰能力差，受到干扰时波形变形很难纠正



### 数字信号（Digital Signal）

离散的信号，不合适长距离传输

抗干扰能力强，受到干扰时波形失真可以修复



### 数据通信模型

<img src="./network_img/net_21.png" style="zoom:70%;" />

  

### 信道（Channel）

信道：信息传输的通道，一条传输介质上（比如网线）上可以有多条信道



- 单工通信

  信号只能往一个方向传输，任何时候都不能改变信号的传输方向

  比如无线电广播、有线电视广播

  

- 半双工通信

  信号可以双向传输，但必须是交替进行，同一时间只能往一个方向传输

  比如对讲机



- 全双工通信

  信号可以同时双向传输

  比如手机（打电话，听说同时进行）



# 2. 数据链路层（Data Link）

链路：从1个节点到相邻节点的一段物理线路（有线或无线），中间没有其他交换节点

![img](./network_img/net_16.png)

数据链路：在一条链路上传输数据时，需要有对应的通信协议来控制数据的传输

不同类型的数据链路，所用的通信协议可能是不同的

- 广播信道：CSMA/CD协议（比如同轴电缆，集线器等组成的网络）

- 点对点信道：PPP协议（比如2个路由器之间的信道）



数据链路层的3个基本问题

- 封装成帧
- 透明传输
- 差错检验



### 封装成帧

- 帧（Frame）的数据部分

  就是网络层传递下来的数据包（IP数据包，Packet）

- 最大传输单元MTU（Maximum Transfer Unit）

  每一种数据链路层协议都规定了所能够传送的帧的数据长度上限，以太网的MTU为1500个字节

![](./network_img/net_17.png)



### 透明传输

![img](./network_img/net_18.png)

使用SOH（Start Of Header）：作为帧开始符

使用EOT（End Of Transmission）：作为帧结束符



数据部分一旦出现了SOH、EOT，就需要进行转义

<img src="./network_img/net_19.png" alt="img" style="zoom:75%;" />



### 差错检验

FCS：根据 数据部分+首部 计算得出的

![img](./network_img/net_20.png)



### CSMA/CD协议

CSMA/CD（Carrier Sense Multiple Access with Collision Detectio）

载波侦听多路访问、冲突检测



使用了CSMA/CD的网络可以成为是以太网（Ethernet），它传输的是以太网帧

以太网帧的格式有：Ethernet V2标准、IEEE的802.3标准

使用最多的是Ethernet V2标准



为了能够检测正在发生的帧是否产生了冲突，以太网的帧至少要64字节

用交换机组建的网络，已经支持全双工通信，不需要再使用CSMA/CD，但它传输的帧依然是以太网帧

所以，用交换机组建的网络，依然可以叫做以太网



### 以太网（Ethernet）

Ethernet V2帧的格式

<img src="./network_img/net_22.png" style="zoom:75%;" />

- 首部： 目标MAC地址 + 源MAC地址 + 网络类型

- 以太网帧：首部 + 数据 + FCS

- 数据长度至少是：64 - 6 - 6 - 2 - 4 = 46字节

- 当数据部分的长度小于46字节时，数据链路层会在数据的后面加入一些字节填充，接收端会将添加的字节去掉

<img src="./network_img/net_23.png" style="zoom:90%;" />

长度总结：

- 以太网帧的数据长度： 46 - 1500 字节
- 以太网帧的长度：64 - 1518字节（目标MAC + 源MAC + 网络类型 + 数据 + FCS）

网卡接收到一个帧，首先会进行差错校验，如果校验通过则接收，否则丢弃



### PPP（Point to Point Protocol）

<img src="./network_img/net_24.png" style="zoom:75%;" />

- Address字段：图中的值是0xFF，形同虚设，点到点信道不需要源MAC，目标MAC地址
- Control字段：图中的值是0x03，目前没什么作用
- Protocol字段：内部用到的协议类型
- 帧开始符、帧结束符：0x7E

<img src="./network_img/net_25.png" style="zoom:85%;" />

字节填充：

将 0x7E 替换成 0x7D5E

将 0x7D 替换成 0x7D5D



# 3. 网络层（Network）

网络层数据包（IP数据包，Packet）由首部、数据2部分组成

![](./network_img/net_14.png)

- 数据部分

  很多时候是由传输层传递下来的数据段（Segment）

  

## 首部数据格式

- 版本（Version）

  占4位，标识IP首部的版本号

  0b0100：IPV4

  0b0110：IPV6

- 首部长度（IHL：Internet Header Length）

  占4位，表明IP首部的大小，二进制乘以4才是最终长度

  0b0101：20（最小值）

  0b1111：60（最大值）

- 区分服务（TOS：Type Of Service）

  占8位，可以用于提高网络的服务质量（QoS，Quality Of Service）

- 总长度（Total Length）

  占16位

  首部 + 数据的长度之和，最大值65535 （2^16）

  <img src="./network_img/net_15.png" style="zoom:90%;" />

  由于帧的数据不能超过1500字节，所以过大的IP数据包，需要分成片（fragments）传输给数据链路层

  每一片都有自己的网络层首部（IP首部）

- 标识（ID：Identification）

  占16位

  数据包得ID，当数据包过大进行分片时，同一个数据包的所有片的标识都是一样的，有一个计数器专门管理数据包的ID，每发出一个数据包，ID就加1

- 标志（Flags）

  占3位

  - 第一位（Reserved Bit）：保留

  - 第二位（Don`t Fragment）

    指示是否可以分片

    1代表不允许分片，0代表允许分片

  - 第三位（More Fragment）

    包被分片的情况下，表示是否为最后一个包

    1代表不是最后一片，0代表是最后一片

- 片偏移（FO：Fragment Offset）

  占13位

  用来标识被分片的每一个分段相对于原始数据的位置。第一个分片对应的值为0.由于FO占13位，因此最多可以表示8192（2^13）个相对位置。单位为8字节，因此最大可表示原始数据8*8192=65536字节的位置

- 生存时间（TTL：Time To Live）

  占8位

  每个路由器在转发之前会将TTL减1，一旦发现TTL减为0，路由器会返回错误报告

  观察使用ping命令后的TTL，能够推测出对方的操作系统，中间经过了多少个路由器

  | 操作系统 | 版本                      | 默认TTL |
  | -------- | ------------------------- | ------- |
  | Windows  | Server 2003、XP、7、10    | 128     |
  | Linux    | 2.0.x kernel、Red Hat 9   | 64      |
  | Linux    | 2.2.14 kernel、2.4 kernel | 255     |
  | Mac OS   |                           | 60      |
  | Mac OS X |                           | 64      |

- 协议（Protocol）

  占8位

  表明所封装的数据是使用了什么协议

  | 协议         | ICMP | IGMP | IP   | TCP  | EGP  | IGP  | UDP  | IPV6 | ESP  | OSPF |
  | ------------ | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
  | 值（十进制） | 1    | 2    | 4    | 6    | 8    | 9    | 17   | 41   | 50   | 89   |

- 首部校验和（Header Checksum）

  占16位

  该字段值校验数据报的首部，不校验数据部分。它主要确保IP数据报不被破坏

- 源IP地址（Source Address）

  占32位，表示发送端的IP地址

- 目标IP地址（Destination Address）

  占32位，表示接收端IP地址

- 可选项（Options）

  长度可变，最大40字节。通常只在进行试验或诊断时使用。该字段包含如下几点信息

  - 安全级别
  - 源路径
  - 路径记录
  - 时间戳

- 填充（Padding）

  也称作填补物。在有可选项的情况下，首部长度可能不是32位的整数倍。为此，通过向字段填充0，调整为32位的整数倍



# 4. 传输层（Transport）

传输层有两个协议

- TCP（Transmission Control Protocol），传输控制协议
- UDP（User Datagram Protocol），用户数据报协议

|                  | TCP                        | UDP                                  |
| ---------------- | -------------------------- | ------------------------------------ |
| **连接性**       | 面向连接                   | 无连接                               |
| **可靠性**       | 可靠传输，不丢包           | 不可靠传输，尽最大努力交付，可能丢包 |
| **首都占用空间** | 大                         | 小                                   |
| **传输速率**     | 慢                         | 快                                   |
| **资源消耗**     | 大                         | 小                                   |
| **应用场景**     | 浏览器、文件传输、邮件发送 | 音视频通话、直播                     |
| **应用层协议**   | HTTP/HTTPS、FTP、SMTP、DNS | DNS                                  |



## TCP

### 数据格式

<img src="./network_img/net_3.png" style="zoom:60%;" />



- 源端口  (Source Port)

  占16位，2字节

  发送方的端口号

  

- 目的端口  (Destination Port)

  占16位，2字节

  接收方的端口号

  

- 序号  (Sequence Number)

  占4字节

  首先，在传输过程的每一个字节都会有一个编号

  在建立连接后，序号代表：这一次传给对方的TCP数据部分的第一个字节的编号

  

- 确认号  (Acknowledgment Number)

  占4字节

  在建立连接后，确认号代表：期望对方下一次传过来的TCP数据部分的第一个字节的编号



- 数据偏移  (Data Offset)

  占1字节，4位，取值范围是 0x0101 ~ 0x1111

  表示TCP所传输的数据部分应该从TCP包得哪个位置开始计算，也可以看做是TCP的首部长度。结果乘以4表示为最终首部长度

  首部长度是 20~60字节

  

- 保留  (Reserved)

  占6位，目前全为0

  有些资料中，TCP首部的保留字段占3位，标志字段占9位

  

- 标志位  (Flags)

  占6位

  - **URG** （Urgent）

    当URG=1时，紧急指针字段才有效。表明当前报文段中有紧急数据，应优先尽快传送

  - **ACK** （Acknowledgment）

    当ACK=1时，确认号字段才有效。TCP规定，当连接建立后，ACK必须为1

  - **PSH** （Push）

    告诉对方收到该报文段后是否立即把数据推送给上层。如果值为1，表示应当立即把数据提交给上层，而不是缓存起来

  - **RST** （Reset）

    当RST=1时，表明连接中出现严重差错，必须释放连接，然后再重新建立连接

  - **SYN**（Synchronization）

    在建立连接时使用，用来同步序号。

    当SYN=1、ACK=0时，表示这是一个建立连接的报文段

    当SYN=1、ACK=1时，表示对方同意建立连接

    SYN=1时，说明这是一个请求建立连接或同意建立连接的报文，只有在前两次握手中SYN才为1

  - **FIN**（Finish）

    标记数据是否发送完毕。当FIN=1时，表明数据已经发送完毕，要求释放连接

    

- 窗口  (Window Size)

  占2字节 16位

  表示从Ack Number开始还可以接受多少字节的数据量，也表示当前接收端的接收窗口还有多少剩余空间。

  该字段可以用于TCP的流量控制

  

- 检验和  (Header and Data Checksum)

  占2字节 16位

  用于确认传输的数据是否有损坏。发送端基于数据内容校验生成一个数值，接收端根据接收的数据校验生成一个值。两个值必须相同，才能证明数据是有效的。如果两个值不同，则丢掉这个数据包
  
  跟UDP一样，TCP检验和的计算内容：**伪首部 + 首部 + 数据**



- 紧急指针  (Urgent Pointer)

  占2字节

  当前面的URG控制位为1时才有意义。它指出本数据段中为紧急数据的字节数，占16位。当所有紧急数据处理完毕后，TCP就会告诉应用程序恢复到正常操作。即使当前的窗口大小为0，也是可以发送紧急数据的，因为紧急数据无须缓存

  

- 选项 （Options）

  最多40字节
  
  长度不定，但长度必须是32bits的整数倍



### 可靠传输

#### 停止等待ARQ协议

ARQ（Automatic Repeat-request），自动重传请求

<img src="./network_img/net_8.png" style="zoom:67%;" />

<img src="./network_img/net_9.png" style="zoom:67%;" />



若重传N次依旧失败，则会根据系统的设置，重传5次还未成功就会发送reset报文（RST）断开TCP连接

<img src="./network_img/net_10.png" style="zoom:80%;" />



#### 连续ARQ协议 + 滑动窗口协议

<img src="./network_img/net_11.png" style="zoom:60%;" />

#### SACK（选择性确认）

在TCP通信过程中，如果发送序列中间某个数据包丢失（比如1，2，3，4，5中的 3 丢失了）

TCP会通过重传最后确认的分组后续的分组（最后确认的是2，会重传3，4，5）

这样原先已经正确传输的分组也可能重复发送（比如4，5），降低了TCP性能



为了改善上述情况，发展出了SACK（Selective Acknowledgment，选择性确认）技术

告诉发送方哪些数据丢失，哪些数据已经提前收到

使TCP只重新发送丢失的包（比如3），不用发送后续所有的分组（比如4，5）



![](./network_img/net_12.jpg)

SACK信息会放在TCP首部的选项部分

- Kind：占1字节，值为5代表这是SACK选项
- Length：占1字节，表明SACK选项一共占用多少字节
- Left Edge：占4字节，左边界
- Right Edge：占4字节，右边界



![](./network_img/net_13.png)

一对边界信息需要占用8字节，由于TCP首部的选项部分最多40字节，

所以SACK选项最多携带4组边界信息，

SACK选项的最大占用字节数 = 4 * 8 + 2 = 34



### 流量控制

如果接收方的缓存区满了，发送方还在疯狂着发送数据，接收方只能把收到的数据包丢掉，大量的丢包会极大的浪费网络资源



什么是流量控制？

让发送方的发送速率不要太快，让接收方来得及接收处理

- 通过确认报文中窗口字段来控制发送方的发送速率
- 发送方的发送窗口大小不能超过接收方给出窗口大小
- 当发送方收到接收窗口的大小为0时，发送方就会停止发送数据

当发送方收到0窗口通知时，这是发送方停止发送报文，并且同时开启一个定时器，隔一段时间就发个测试报文去询问接收方最新的窗口大小，如果接收的窗口大小还是为0，则发送方再次刷新启动定时器



### 拥塞控制

- 防止过多的数据注入到网络中
- 避免网络中的路由器或链路过载

拥塞控制是一个全局性的过程，涉及到所有的主机，路由器以及与降低网络传输性能有关的所有因素，是大家共同努力的结果

相比而言，流量控制是点对点通信的控制

常见缩写：

- MSS (Maximum Segment Size)：每个段最大的数据部分大小，在建立连接时确定

- cwnd（congestion window）：拥塞窗口

- rwnd（receive window）：接收窗口

- swnd（send window）：发送窗口   swnd = min(cwnd, rwnd)

  当rwnd < cwnd时，是接收方的接收能力限制发送窗口的最大值

  当cwnd < rwnd时，则是网络的拥塞限制发送窗口的最大值



#### 慢开始（slow start）

<img src="./network_img/net_4.png" style="zoom:56%;" />

cwnd的初始值比较小，然后随着数据包被接收方确认（收到一个ACK），则cwnd就成倍增长（指数级）。



#### 拥塞避免（congestion avoidance）

<img src="./network_img/net_5.png" style="zoom:75%;" />

- ssthresh（slow start threshold）：慢开始阈值，cwnd达到阈值后，以线性方式增加

- 拥塞避免（加法增大）：拥塞窗口缓慢增大，以防止网络过早出现阻塞

- 乘法减小：只要网络出现拥塞，把ssthresh减为拥塞峰值的一半，同时执行慢开始算法（cwnd又恢复到初始值）

- 当网络出现频繁拥塞时，ssthresh值就下降的很快



#### 快速重传（fast retransmit）

<img src="./network_img/net_6.png" style="zoom:75%;" />

- **接收方**

  每收到一个失序的分组后就立即发出重复确认，使发送方及时知道有分组未到达，而不要等到自己发送数据时才进行确认

- **发送方**

  只要连续收到三个重复确认（总共4个相同的确认），就应当立即重传对方尚未收到的报文段，而不必继续等待重传计时器到期后再重传



#### 快速恢复（fast recovery）

当发送方连续收到三个重复确认，说明网络出现拥塞，就执行乘法减小算法，把ssthresh减为拥塞峰值的一半

与慢开始不同之处是现在不执行慢开始算法，即cwnd现在不恢复到初始值，而是把cwnd值设置为新的ssthresh值（减小后的值）

然后开始执行拥塞避免算法（加法增大），使拥塞窗口缓慢的线性增大



<img src="./network_img/net_7.png" style="zoom:75%;" />





### 连接管理

#### 建立连接 （三次握手）

TCP是面向连接的协议，所以每次发出的请求都需要对方进行确认。TCP客户端与TCP服务器在通信之前需要完成三次握手才能建立连接

<img src="./network_img/net_29.png" style="zoom:90%;" />

- SYN和ACK为标志位，seq表示请求序列号，ack表示确认序列号。x表示发送端初始化的随机序号，y表示接收端初始化的随机序号
- CLOSED: 客户端处于关闭状态
- LISTEN：服务端处于监听状态，等待客户端连接
- SYN-SENT：表示客户端已经发送 SYN 报文，等待服务端的第二次握手
- SYN-RCVD：表示服务端接收到了 SYN 报文，当收到客户端的 ACK 报文后，它会进入 ESTABLISHED 已连接状态
- ESTABLISHED：表示连接已经建立



##### 第一次握手

第一次握手建立连接时，客户端向服务器发送 SYN报文（SYN=1，ACK=0，seq=x），并进入到 SYN-SENT 状态，等待服务器确认

##### 第二次握手

第二次握手实际上是分两部分来完成的，即 SYN+ACK（请求和确认）报文

- 服务端收到了客户端的请求，向客户端回复一个确认信息（ACK=1，ack=x+1）
- 服务端再向客户端发送一个 SYN 包（SYN=1，seq=y）建立连接的请求，此时服务器进入 SYN-RCVD 状态

总结下来发送报文为 （SYN=1，ACK=1，seq=y，ack=x+1）

##### 第三次握手

第三次握手是客户端收到服务器的回复（SYN+ACK报文）。此时，客户端也要向服务器发送确认包（ACK=1，seq=x+1，ack=y+1），此包发送完毕客户端和服务器进入 ESTABLISHED 状态，完成三次握手



前两次握手的特点

- SYN 都设置为1

- 数据部分的长度都为0

- TCP头部的长度一般是32字节

  - 固定部分：20字节
  - 选项部分：12字节

  双方会交换确认一些信息，比如MSS，是否支持SACK，Window Scale（窗口缩放系数）等

  这些数据都放在了TCP头部的选项部分中（12字节）



为什么建立连接的时候要进行三次握手，两次不行吗？

- 主要目的是为了防止server端一直等待，浪费资源

假设client发出的第一个连接请求报文段，因为网络延迟，在连接释放后的某个时间才到达server

本来这是一个早已失效的连接请求，但server收到此失效的请求后，误以为是client再次发出的一个新的连接请求，于是server就向client发出确认报文段，同意建立连接

如果不采用 3次握手，那么只要server端确认，新的连接就建立了

由于现在的client并没有真正想连接服务器，因此不会理睬server的确认，也不会向server发送数据

但server却以为新的连接已经建立，并一直等待client发来数据，这样server的很多资源就白白浪费了



采用“三次握手”的办法可以防止此现象发生

- 例如上述情况，client没有向server的确认发出确认，server由于收不到确认，就知道client并没有要求建立连接



若第三次握手失败了，会怎么处理？

- 此时server的状态为 SYN-RCVD，若等不到client的ACK，server会重新发送 SYN+ACK （SYN=1, ACK=1）包
- 如果server多次重发 SYN+ACK 都等不到client的ACK，就会发送 RST 包，强制关闭连接



#### 释放连接（四次挥手）

当客户端与服务器不再进行通信时，都会以四次挥手的方式结束连接

<img src="./network_img/net_28.png" style="zoom:75%;" />

- FIN-WAIT-1：表示想主动关闭连接

  向对方发送了FIN报文，此时进入到FIN-WAIT-1状态

- CLOSE-WAIT：表示在等待关闭

  - 当对方发送FIN给自己，自己会回应一个ACK报文给对方，此时则进入到CLOSE-WAIT状态

  - 在此状态下，需要考虑自己是否还有数据要发送给对方，如果没有，发送 FIN 报文给对方

- FIN-WAIT-2：只要对方发送ACK确认后，主动方就会处于 FIN-WAIT-2状态，然后等待对方发送FIN报文

- CLOSEING：一种比较罕见的例外状态

  - 表示你发送FIN报文后，并没有收到对方的ACK报文，反而却也收到了对方的FIN报文

  - 如果双方几乎在同时准备关闭连接的话，那么就出现了双方同时发送FIN报文的情况，也即会出现CLOSING状态
  - 表示双方都正在关闭连接

- LAST-ACK：被动关闭一方在发送FIN报文后，最后等待对方的ACK报文

  当收到ACK报文后，即可进入CLOSED关闭状态

- TIME-WAIT：表示收到了对方的FIN报文，并发送出了ACK报文，就等2MSL后即可进入CLOSED状态了

  - 如果FIN-WAIT-1状态下，收到了对方同时带FIN标志和ACK标志的报文时，可以直接进入到TIME-WAIT状态，而无须经过FIN-WAIT-2状态

- CLOSED：关闭状态



##### 第一次挥手

客户端向服务端发送断开TCP连接请求的 FIN 报文，序列号 seq=u，表示要断开TCP连接，此时客户端进入FIN-WAIT-1的状态下

（FIN=1，seq=u）

##### 第二次挥手

当服务端收到客户端发来的断开TCP连接请求后，回复发送ACK报文，表示已经收到断开请求。回复时，序列号 seq=v。由于回复的是客户端发来的请求，所以在客户端请求序号seq=u的情况下加1，得到ack=u+1

（ACK=1，seq=v，ack=u+1）

##### 第三次挥手

服务端在回复完客户端的TCP断开请求后，不会马上进行TCP连接的断开。服务端会先确认断开前，所有传输到客户端的数据是否已经传输完毕。确认传输完毕后才进行断开，向客户端发送 FIN=1，ACK=1，序列号seq=w。由于还是对客户端发来的TCP断开请求回复，所以ack还是ack=u+1

（FIN=1，ACK=1，seq=w，ack=u+1）

##### 第四次挥手

客户端收到服务器发来的TCP断开连接数据包后将进行回复，表示收到断开TCP连接数据包。向服务端发送ACK报文，生成一个序列号seq=u+1。由于回复的是服务器，所以ack字段为w+1

（ACK=1，seq=u+1，ack=w+1）

##### 

- MSL：

  客户端发送ACK后，需要有个TIME-WAIT阶段，等待一段时间后，再真正关闭连接。一般是等待2倍的MSL（Maximum Segment Lifetime，最大分段生成期）

  MSL是TCP报文在Internet上的最长生存时间

  每个具体的TCP实现都必须选择一个确定的MSL值，RFC建议是2分钟
  
  为了防止本次连接中产生的数据包误传到下一次连接中（因为本次连接中的数据包都会在2MSL时间内消失了）
  
  
  
- 为什么要等待2MSL?

  **为的是确认服务器端是否收到客户端发出的ACK确认报文**

  > 当客户端发出最后的ACK确认报文时，并不能确定服务器端能够收到该段报文。所以客户端在发送完ACK确认报文之后，会设置一个时长为2MSL的计时器。MSL指的是Maximum Segment Lifetime：一段TCP报文在传输过程中的最大生命周期。2MSL即是服务器端发出为FIN报文和客户端发出的ACK确认报文所能保持有效的最大时长。

  > 服务器端在1MSL内没有收到客户端发出的ACK确认报文，就会再次向客户端发出FIN报文；
  >
  > - 如果客户端在2MSL内，再次收到了来自服务器端的FIN报文，说明服务器端由于各种原因没有接收到客户端发出的ACK确认报文。客户端再次向服务器端发出ACK确认报文，计时器重置，重新开始2MSL的计时；
  > - 否则客户端在2MSL内没有再次收到来自服务器端的FIN报文，说明服务器端正常接收了ACK确认报文，客户端可以进入CLOSED阶段，完成“四次挥手”。

  **所以，客户端要经历时长为2SML的TIME-WAIT阶段；这也是为什么客户端比服务器端晚进入CLOSED阶段的原因**



- 为什么释放连接的时候，要进行四次挥手？

  > TCP释放连接时之所以需要“四次挥手”,是因为**FIN释放连接报文与ACK确认接收报文是分别由第二次和第三次"挥手"传输的**。为何建立连接时一起传输，释放连接时却要分开传输？
>
  > - 建立连接时，被动方服务器端结束CLOSED阶段进入“握手”阶段并不需要任何准备，可以直接返回SYN和ACK报文，开始建立连接。
  > - 释放连接时，被动方服务器，突然收到主动方客户端释放连接的请求时并不能立即释放连接，因为还有必要的数据需要处理，所以服务器先返回ACK确认收到报文，经过CLOSE-WAIT阶段准备好释放连接之后，才能返回FIN释放连接报文。

  

  TCP是全双工模式

  - 第一次挥手：当主机1发出FIN报文段时。表示主机1告诉主机2，主机1已经没有数据要发送了，但是，此时主机1还是可以接收来自主机2的数据
  - 第二次挥手：当主机2返回ACK报文段时。表示主机2已经知道主机1没有数据发送了，但是主机2还是可以发送数据给主机1的
  - 第三次挥手：当主机2也发送了FIN报文段时。表示主机2告诉主机1，主机2已经没有数据要发送了
  - 第四次挥手：当主机1返回ACK报文段时。表示主机1已经知道主机2没有数据发送了。随后正式断开整个TCP连接



- 三次挥手?

  有时候抓包时，可能只看到三次挥手。这其实是将第2、3次挥手合并了

  当服务端接收到客户端的FIN时，如果服务端后面也没有数据要发送给客户端了，这是服务端就可以将第2、3次挥手合并



## UDP

UDP是无连接的，减少了建立和释放连接的开销

UDP尽最大能力交付，不保证可靠交付

因此不需要维护一些复杂的参数，首部只有8个字节（TCP的首部至少20个字节）

<img src="./network_img/net_1.png" style="zoom:75%;" />

- 源端口号（Source Port）

  占16位，2字节

  表示发送端端口号。该字段是可选项，有时可能不会设置源端口号。没有源端口号的时候该字段的值设置为0，可用于不需要返回的通信中

  

- 目的端口号（Destination Port）

  占16位，2字节

  接收端端口号

  

- UDP长度（Length）

  占16位，首部的长度 + 数据的长度

  

- UDP检验和（Checksum）

  检验和的计算内容：**伪首部 + 首部 + 数据**

  伪首部：仅在计算检验和时起作用，并不会传递给网络层

  <img src="./network_img/net_2.png" alt="net_2" style="zoom:60%;" />

- 端口（Port）

  UDP首部中端口占用2个字节，可以推测出端口号的取值范围是：0 ~ 65535

  客户端的源端口是临时开启的随机端口

  | 协议  |  默认端口号  |
  | :---: | :----------: |
  | HTTP  |   TCP + 80   |
  | HTTPS |  TCP + 443   |
  |  FTP  |   TCP + 21   |
  | MySQL |  TCP + 3306  |
  |  DNS  | UDP/TCP + 53 |
  | SMTP  |   TCP + 25   |
  | POP3  |  TCP + 110   |





# 5. 应用层（Application）

应用层常见协议：

- 超文本传输

  HTTP、HTTPS

- 文件传输

  FTP

- 电子邮件

  SMTP、POP3、IMAP

- 动态主机配置

  DHCP

- 域名系统

  DNS

  

## 域名（Domin Name）

由于IP地址不方便记忆，并且不能表达组织的名称和性质，人们设计出了域名（比如baidu.com）

但实际上，为了能够访问到具体的主机，最终还是得知道目标主机的IP地址。

使用IP地址只占用4字节，可以减少路由器的负担，节省流量



根据级别不同，域名可以分为

- 顶级域名（Top-level Domain，简称TLD）
- 二级域名
- 三级域名
- 。。。



### 顶级域名

- 通用顶级域名（General Top-level Domain，简称gTLD）

  ```
  .com（公司）
  .net（网络机构）
  .org（组织结构）   
  .edu（教育）   
  .gov（政府部门）   
  .int（国际组织）
  ```

- 国家及地区顶级域名（Country Code Top-level Domain，简称ccTLD）

  ```
  .cn（中国）
  .jp（日本）
  .uk（英国）
  ```

- 新通用顶级域名（New Generic Top-level Domain，简称：New gTLD）

  ```
  .vip
  .xyz
  .top
  .club
  .shop
  ```

### 二级域名

二级域名是指顶级域名之下的域名

在通用顶级域名下，它一般指域名注册人的名称，例如google、baidu、microsoft等

在国家及地区顶级域名下，它一般指注册类别的，例如com、edu、gov、net等

![](./network_img/net_30.png)



## DNS

DNS的全程是：Domain Name System，译为：域名系统

利用DNS协议，可以将域名（比如baidu.com）解析成对应的IP地址（比如220.181.38.148）

DNS可以基于UDP协议，也可以基于TCP协议，服务器占用53端口

<img src="./network_img/net_31.png" style="zoom:75%;" />

客户端首先会访问最近的一台DNS服务器（也就是客户端自己配置的DNS服务器）

所有的DNS服务器都记录了DNS根域名服务器的IP地址

上级DNS服务器记录了下一级DNS服务器的IP地址

全球一共13台IPv4的DNS根域名服务器、25台IPv6的DNS根域名服务器



## DHCP

DHCP（Dynamic Host Configuration Protocol），译为：动态主机配置协议

DHCP协议基于UDP协议，客户端是68端口，服务器是67端口

DHCP服务器会从IP地址池中，挑选一个IP地址出租给客户端一段时间，时间到期就回收他们

平时家里上网的路由器就可以充当DHCP服务器

<img src="./network_img/net_32.png" style="zoom:37%;" />

![](./network_img/net_33.png)

分配IP地址的4个阶段

- DISCOVER：发现服务器

  发广播包（源IP是0.0.0.0，目标IP是255.255.255.255，目标MAC地址是FF:FF:FF:FF:FF:FF）

- OFFER：提供租约

  服务器返回可以租用的IP地址，以及租用期限、子网掩码、网关、DNS等信息

  注意：这里可能会有多个服务器提供租约

- REQUEST：选择IP地址

  客户端选择一个OFFER，发送广播包进行回应

- ACKNOWLEDGE：确认

  被选中的服务器发送ACK数据包给客户端

  至此，IP地址分配完毕



注意：

- DHCP服务器可以跨网段分配IP地址吗？（DHCP服务器、客户端不在同一个网段）

  可以借助DHCP中继代理（DHCP Relay Agent）实现跨网段分配IP地址

- 自动续约

  客户端会在租期不足的时候，自动向DHCP服务器发送REQUEST信息申请续约

- 常用命令

  - ipconfig /all 可以看到DHCP相关的详细信息，比如租约过期时间、DHCP服务器地址等
  - ipconfig /release  释放租约
  - ipconfig /renew   重新申请IP地址，申请续约（延长租期）



## HTTP

HTTP（Hyper Text Transfer Protocol）超文本传输协议

- 互联网中应用最广泛的应用层协议之一
- 设计最初目的是：提供一种发布和接收HTML页面的方法，由URI来标识具体的资源

### 发展历史

- 1991年，HTTP/0.9

  只支持GET请求方法获取文本数据（比如HTML文档），且不支持请求头，响应头等，无法向服务器传递太多信息

- 1996年，HTTP/1.0

  支持POST HEAD等请求方法，支持请求头、响应头等，支持更多种数据类型（不再局限于文本数据）

  浏览器的每次请求都需要与服务器建立一个TCP连接，请求处理完成后立即断开TCP连接

- 1997年，HTTP/1.1 （最经典，使用最广泛的版本）

  支持PUT  DELETE等请求方法

  采用持久连接（Connection：keep-alive），多个请求可以共用同一个TCP连接

- 2015年，HTTP/2.0

- 2018年，HTTP/3.0

### 报文格式

![](./network_img/net_34.png)

```
GET / HTTP/1.1
Host: localhost
Connection: keep-alive
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.67 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
If-None-Match: "143-5b5f523376080"
If-Modified-Since: Tue, 08 Dec 2020 14:55:46 GMT

实体主体，若是POST，则此部分放参数（username=123&pwd=456）
```



![](./network_img/net_35.png)

```
HTTP/1.1 200 OK
Date: Tue, 08 Dec 2020 15:01:40 GMT
Server: Apache/2.4.41 (Unix)
Last-Modified: Tue, 08 Dec 2020 14:55:46 GMT
ETag: "143-5b5f523376080"
Accept-Ranges: bytes
Content-Length: 323
Content-Type: text/html

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>

    <p>hello world!!!</p>

    <button onclick="buttonClick()">button</button>
    
</body>

<script>
    function buttonClick() {
        window.location.href = "https://www.baidu.com";
    }
</script>


</html>
```

