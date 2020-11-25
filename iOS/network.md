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

    当ACK=1时，确认号字段才有效

  - **PSH** （Push）

    

  - **RST** （Reset）

    当RST=1时，表明连接中出现严重差错，必须释放连接，然后再重新建立连接

  - **SYN**（Synchronization）

    当SYN=1，ACK=0时，表明这是一个建立连接的请求

    若对方同意建立连接，则回复SYN=1，ACK=1

  - **FIN**（Finish）

    当FIN=1时，表明数据已经发送完毕，要求释放连接

    

  

- 窗口  (Window Size)

  占2字节

  这个字段有流量控制功能，用以告知对方下一次允许发送的数据大小（字节为单位）

  

- 检验和  (Header and Data Checksum)

  占2字节

  跟UDP一样，TCP检验和的计算内容：**伪首部 + 首部 + 数据**



- 紧急指针  (Urgent Pointer)

  占2字节

  

- 选项 （Options）

  最多40字节



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

<img src="./network_img/net_26.png" style="zoom:50%;" />

- CLOSED: 客户端处于关闭状态
- LISTEN：服务端处于监听状态，等待客户端连接
- SYN-SENT：表示客户端已经发送 SYN 报文，等待服务端的第二次握手
- SYN-RCVD：表示服务端接收到了 SYN 报文，当收到客户端的 ACK 报文后，它会进入 ESTABLISHED 已连接状态
- ESTABLISHED：表示连接已经建立



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



