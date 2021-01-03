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

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ecdb0268701c42f7ae1ec5790bb8e8ac~tplv-k3u1fbpfcp-watermark.image)

  

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

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e016ca5b3f7142f7b66ec4ba4eade9b6~tplv-k3u1fbpfcp-watermark.image)

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

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b4f81c8f990d41848298ae493464bb81~tplv-k3u1fbpfcp-watermark.image)



### 透明传输

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9bf8ba4d63fc466e859d1a55e38d6a4d~tplv-k3u1fbpfcp-watermark.image)

使用SOH（Start Of Header）：作为帧开始符

使用EOT（End Of Transmission）：作为帧结束符



数据部分一旦出现了SOH、EOT，就需要进行转义

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d71d606c54aa4a64a76eda6a6dfbbe41~tplv-k3u1fbpfcp-watermark.image)



### 差错检验

FCS：根据 数据部分+首部 计算得出的

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bd78e00d88b543c79b32cada063cf631~tplv-k3u1fbpfcp-watermark.image)



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

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d5aefac5bc84c26b08506ebfab2afa2~tplv-k3u1fbpfcp-watermark.image" style="zoom:75%;" />

- 首部： 目标MAC地址 + 源MAC地址 + 网络类型

- 以太网帧：首部 + 数据 + FCS

- 数据长度至少是：64 - 6 - 6 - 2 - 4 = 46字节

- 当数据部分的长度小于46字节时，数据链路层会在数据的后面加入一些字节填充，接收端会将添加的字节去掉

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a2c56ec2281e4537b63ac8cefc5a0f84~tplv-k3u1fbpfcp-watermark.image" style="zoom:90%;" />

长度总结：

- 以太网帧的数据长度： 46 - 1500 字节
- 以太网帧的长度：64 - 1518字节（目标MAC + 源MAC + 网络类型 + 数据 + FCS）

网卡接收到一个帧，首先会进行差错校验，如果校验通过则接收，否则丢弃



### PPP（Point to Point Protocol）

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8182a398858480d9a4b6421def11120~tplv-k3u1fbpfcp-watermark.image" style="zoom:75%;" />

- Address字段：图中的值是0xFF，形同虚设，点到点信道不需要源MAC，目标MAC地址
- Control字段：图中的值是0x03，目前没什么作用
- Protocol字段：内部用到的协议类型
- 帧开始符、帧结束符：0x7E

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb62817e21fc4fc9b386dee1e143be30~tplv-k3u1fbpfcp-watermark.image" style="zoom:85%;" />

字节填充：

将 0x7E 替换成 0x7D5E

将 0x7D 替换成 0x7D5D



# 3. 网络层（Network）

网络层数据包（IP数据包，Packet）由首部、数据2部分组成

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/983f1ca5196445a981f0eb84f9847f8c~tplv-k3u1fbpfcp-watermark.image)

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

    <img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a84f25e3e709476ab1e1cbff3b697b4c~tplv-k3u1fbpfcp-watermark.image" style="zoom:90%;" />

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

