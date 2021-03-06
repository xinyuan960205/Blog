



## 一、UDP

- 用户数据报协议 UDP（User Datagram Protocol）是无连接的，尽最大可能交付，没有拥塞控制，面向报文（对于应用程序传下来的报文不合并也不拆分，只是添加 UDP 首部），支持一对一、一对多、多对一和多对多的交互通信。
- UDP特性：
  - **提供无连接服务**。
  - **在传送数据之前不需要先建立连接**。
  - 传送的数据单位协议是 UDP 报文或用户数据报。
  - **对方的运输层在收到 UDP 报文后，不需要给出任何确认。**
  - 虽然 UDP 不提供可靠交付，但在某些情况下 UDP 是一种最有效的工作方式。
  - UDP 支持一对一、一对多、多对一和多对多的交互通信。
  - UDP 的首部开销小，只有 8 个字节，比 TCP 的 20 个字节的首部要短
- 使用场景（一般用于即时通信）
  - QQ语音 
  - QQ视频 
  - TFTP（简单文件传输协议）
- 基于UDP的协议
  - RIP(路由选择协议)
  - DNS

### 1.1 UDP首部格式

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/UDP%E9%A6%96%E9%83%A8.webp)

首部字段只有 8 个字节，包括源端口、目的端口、长度、检验和。12 字节的伪首部是为了计算检验和临时添加的。

请注意，虽然在 UDP 之间的通信要用到其端口号，**但由于 UDP 的通信是无连接的，因此不需要使用套接字**。

### 1.2 UDP校验和

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/UDP%E6%A0%A1%E9%AA%8C%E5%92%8C.webp)



## 二、TCP

- 传输控制协议 TCP（Transmission Control Protocol）是面向连接的，提供可靠交付，有流量控制，拥塞控制，提供全双工通信，面向字节流（把应用层传下来的报文看成字节流，把字节流组织成大小不等的数据块），每一条 TCP 连接只能是点对点的（一对一）。
- 使用场景（TCP 一般用于文件传输、发送和接收邮件、远程登录等场景）
  - 浏览器，用的HTTP
  - FlashFXP，用的FTP 
  - Outlook，用的POP、SMTP 
  - Putty，用的Telnet、SSH 
  - QQ文件传输
- 基于TCP的协议
  - SMTP
  - TELNET
  - HTTP
  - FTP

### 2.1 TCP首部格式

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/TCP%E9%A6%96%E9%83%A8%E6%A0%BC%E5%BC%8F.webp)

- **序号** seq：用于对字节流进行编号，例如序号为 301，表示第一个字节的编号为 301，如果携带的数据长度为 100 字节，那么下一个报文段的序号应为 401。
- **确认号**ack ：期望收到的下一个报文段的序号。例如 B 正确收到 A 发送来的一个报文段，序号为 501，携带的数据长度为 200 字节，因此 B 期望下一个报文段的序号为 701，B 发送给 A 的确认报文段中确认号就为 701。
- **数据偏移** ：指的是数据部分距离报文段起始处的偏移量，实际上指的是首部的长度。
- **确认 ACK** ：当 ACK=1 时确认号字段有效，否则无效。TCP 规定，在连接建立后所有传送的报文段都必须把 ACK 置 1。
- **同步 SYN** ：在连接建立时用来同步序号。当 SYN=1，ACK=0 时表示这是一个连接请求报文段。若对方同意建立连接，则响应报文中 SYN=1，ACK=1。
- **终止 FIN** ：用来释放一个连接，当 FIN=1 时，表示此报文段的发送方的数据已发送完毕，并要求释放连接。
- **窗口** ：窗口值作为接收方让发送方设置其发送窗口的依据。之所以要有这个限制，是因为接收方的数据缓存空间是有限的。

#### TCP首部选项中时间戳选项

在TCP选项字段中为TCP预留有时间戳功能，不管在网络层面还是应用层面，TCP时间戳往往被大家认为是一个系统行为，并忽略其存在。其实在某些环境下，TCP时间戳同样可以成为大家在时延问题troubleshooting定位时的一个利器。

- TCP时间戳字段存放的内容注解

  发送方在发送报文段时把当前时钟的时间值放入时间戳字段，接收方在确认该报文段时把时间戳字段值复制到时间戳回送回答字段。因此，发送方在收到确认报文后，可以准确计算出RTT。

  下面我们以一个例子理解timestamp和timestamp echo字段中存放的内容：假设a主机和b主机之间通信，a主机向b主机发送一个报文段，那么：

  1. timestamp字段中存放的内容：a主机向b主机发送报文s1，在s1报文中timestamp存储的是a主机发送s1时的内核时刻ta1。

  2. timestamp echo字段中存放的内容：b主机收到s1报文并向a主机发送含有确认ack的报文s2，在s2报文中，timestamp为b主机此时的内核时刻tb，而timestamp echo字段为从s1报文中解析出的ta1.

  这些时刻具体的作用，我们继续向下看，将结合TCP时间戳选项的功能进行说明。

- TCP时间戳选项的功能：

1. 计算往返时延RTT：

   当a主机接收到b主机发送过来的确认ack报文s2时，a主机此时内核时刻为ta2.

   a主机从s2报文的timestamp echo选项中可以解析出该确认ack确认的报文的发送时刻为ta1.

   那么：RTT＝接收ack报文的时刻－发送报文的时刻＝ta2 －ta1.

   ta2和ta1都来自a主机的内核，所以不需要在tcp连接的两端进行任何时钟同步的操作。

2. 防止回绕的序号：

   我们知道TCP报文的序列号只有32位，而没增加2^32个序列号后就会重复使用原来用过的序列号。假设我们有一条高速网络，通信的主机双方有足够大的带宽涌来快速的传输数据。例如1Gb/s的速率发送报文，则不到35秒报文的序号就会重复。这样对TCP传输带来混乱的情况。而采用时间戳选项，可以很容易的分辨出相同序列号的数据报，哪个是最近发送，哪个是以前发送的。

### 2.2 TCP三次握手

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/TCP%E7%9A%84%E4%B8%89%E6%AC%A1%E6%8F%A1%E6%89%8B.png)

假设 A 为客户端，B 为服务器端。

- 首先 B 处于 LISTEN（监听）状态，等待客户的连接请求。
- A 向 B 发送连接请求报文，SYN=1，ACK=0，选择一个初始的序号 x。
- B 收到连接请求报文，如果同意建立连接，则向 A 发送连接确认报文，SYN=1，ACK=1，确认号为 x+1，同时也选择一个初始的序号 y。
- A 收到 B 的连接确认报文后，还要向 B 发出确认，确认号为 y+1，序号为 x+1。
- B 收到 A 的确认后，连接建立。

#### **三次握手的原因**

两种原因：

1. 第三次握手是为了防止失效的连接请求到达服务器，让服务器错误打开连接。

   客户端发送的连接请求如果在网络中滞留，那么就会隔很长一段时间才能收到服务器端发回的连接确认。客户端等待一个超时重传时间之后，就会重新请求连接。但是这个滞留的连接请求最后还是会到达服务器，如果不进行三次握手，那么服务器就会打开两个连接。如果有第三次握手，客户端会忽略服务器之后发送的对滞留连接请求的连接确认，不进行第三次握手，因此就不会再次打开连接。

2. 让服务器知道自己有发送的能力。

#### 为什么要传回 SYN

接收端传回发送端所发送的 SYN 是为了告诉发送端，我接收到的信息确实就是你所发送的信号了。

> SYN 是 TCP/IP 建立连接时使用的握手信号。在客户机和服务器之间建立正常的 TCP 网络连接时，客户机首先发出一个 SYN 消息，服务器使用 SYN-ACK 应答表示接收到了这个消息，最后客户机再以 ACK(Acknowledgement[汉译：确认字符 ,在数据通信传输中，接收站发给发送站的一种传输控制字符。它表示确认发来的数据已经接受无误。 ]）消息响应。这样在客户机和服务器之间才能建立起可靠的TCP连接，数据才可以在客户机和服务器之间传递。

#### 传了 SYN,为啥还要传 ACK

双方通信无误必须是两者互相发送信息都无误。传了 SYN，证明发送方到接收方的通道没有问题，但是接收方到发送方的通道还需要 ACK 信号来进行验证。

### 2.3 TCP四次挥手

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/TCP%E7%9A%84%E5%9B%9B%E6%AC%A1%E6%8C%A5%E6%89%8B.jpg)

以下描述不讨论序号和确认号，因为序号和确认号的规则比较简单。并且不讨论 ACK，因为 ACK 在连接建立之后都为 1。

- A 发送连接释放报文，FIN=1。
- B 收到之后发出确认，此时 TCP 属于半关闭状态，B 能向 A 发送数据但是 A 不能向 B 发送数据。
- 当 B 不再需要连接时，发送连接释放报文，FIN=1。
- A 收到后发出确认，进入 TIME-WAIT 状态，等待 2 MSL（最大报文存活时间）后释放连接。
- B 收到 A 的确认后释放连接。

#### **四次挥手的原因**

任何一方都可以在数据传送结束后发出连接释放的通知，待对方确认后进入半关闭状态。当另一方也没有数据再发送的时候，则发出连接释放通知，对方确认后就完全关闭了TCP连接。

#### **CLOSE-WAIT** 

这个状态是为了让服务器端发送还未传送完毕的数据，传送完毕之后，服务器会发送 FIN 连接释放报文。

#### **TIME_WAIT**

- 存在的意义

  客户端接收到服务器端的 FIN 报文后进入此状态，此时并不是直接进入 CLOSED 状态，还需要等待一个时间计时器设置的时间 2MSL。这么做有两个理由：

  1. 确保最后一个确认报文能够到达。如果 B 没收到 A 发送来的确认报文，那么就会重新发送连接释放请求报文，A 等待一段时间就是为了处理这种情况的发生。

  2. 等待一段时间是为了让本连接持续时间内所产生的所有报文都从网络中消失，使得下一个新的连接不会出现旧的连接请求报文。

- 查看TIME-WAIT状态的链接数量

  ```
  通过 ``"netstat -anp | grep TIME_WAIT | wc -l"` `命令查看数量
  netstat -an |grep "TIME_WAIT" 查看处于Time_wait状态的连接详细情况
  ```

- 为什么会TIME-WAIT过多？解决方法是怎样的？

  原因：根据TCP协议定义的3次握手断开连接规定,发起socket主动关闭的一方 socket将进入TIME_WAIT状态,TIME_WAIT状态将持续2个MSL(Max Segment Lifetime),在Windows下默认为4分钟,即240秒,TIME_WAIT状态下的socket不能被回收使用. 具体现象是对于一个处理大量短连接的服务器,如果是由服务器主动关闭客户端的连接,将导致服务器端存在大量的处于TIME_WAIT状态的socket, 甚至比处于Established状态下的socket多的多,严重影响服务器的处理能力,甚至耗尽可用的socket,停止服务. TIME_WAIT是TCP协议用以保证被重新分配的socket不会受到之前残留的延迟重发报文影响的机制,是必要的逻辑保证.
  解决方法：可以通过调整系统内核参数来解决

  ```
  打开 sysctl.conf 文件，修改以下几个参数：
  [root@web01 ~]``# vim /etc/sysctl.conf
  net.ipv4.tcp_tw_reuse = 1
  net.ipv4.tcp_tw_recycle = 1
  net.ipv4.tcp_timestamps = 1
  net.ipv4.tcp_syncookies = 1
  net.ipv4.tcp_fin_timeout = 30
  [root@web01 ~]``# sysctl -p
  ```

  推荐仔细看这篇文章https://www.cnblogs.com/kevingrace/p/9988354.html

### 2.4 可靠传输

理想的传输条件有以下两个特点：

- (1) 传输信道不产生差错。
- (2) 不管发送方以多快的速度发送数据，接收方总是来得及处理收到的数据。

然而实际的网络都不具备以上两个理想条件**。必须使用一些可靠传输协议，在不可靠的传输信道实现可靠传输。**

#### TCP 协议如何保证可靠传输

1. 应用数据被分割成 TCP 认为最适合发送的数据块。
2. TCP 给发送的每一个包进行编号，接收方对数据包进行排序，把有序数据传送给应用层。
3. **校验和：** TCP 将保持它首部和数据的检验和。这是一个端到端的检验和，目的是检测数据在传输过程中的任何变化。如果收到段的检验和有差错，TCP 将丢弃这个报文段和不确认收到此报文段。
4. TCP 的接收端会丢弃重复的数据。
5. **流量控制：** TCP 连接的每一方都有固定大小的缓冲空间，TCP的接收端只允许发送端发送接收端缓冲区能接纳的数据。当接收方来不及处理发送方的数据，能提示发送方降低发送的速率，防止包丢失。TCP 使用的流量控制协议是可变大小的滑动窗口协议。 （TCP 利用滑动窗口实现流量控制）
6. **拥塞控制：** 当网络拥塞时，减少数据的发送。
7. **ARQ协议：** 也是为了实现可靠传输的，它的基本原理就是每发完一个分组就停止发送，等待对方确认。在收到确认后再发下一个分组。ARQ包括停止等待ARQ协议和连续ARQ协议。
8. **超时重传：** 当 TCP 发出一个段后，它启动一个定时器，等待目的端确认收到这个报文段。如果不能及时收到一个确认，将重发这个报文段。

#### ARQ协议

**自动重传请求**（Automatic Repeat-reQuest，ARQ）是OSI模型中数据链路层和传输层的错误纠正协议之一。它通过使用**确认**和**超时**这两个机制，在不可靠服务的基础上实现可靠的信息传输。如果发送方在发送后一段时间之内没有收到确认帧，它通常会重新发送。ARQ包括停止等待ARQ协议和连续ARQ协议。

##### 2.4.1 停止等待ARQ协议

**“停止等待”就是每发送完一个分组就停止发送，等待对方的确认。在收到确认后再发送下一个分组。**

停止等待协议有两种情况：

- **无差错情况**
- **出现差错**

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E8%BF%9E%E7%BB%AD%E5%81%9C%E6%AD%A2%E5%8D%8F%E8%AE%AE1.webp)

在接收方 B 会出现两种情况：

1. **B 接收 M1 时检测出了差错，就丢弃 M1，其他什么也不做（不通知 A 收到有差错的分组）。**
2. **M1 在传输过程中丢失了，这时 B 当然什么都不知道，也什么都不做。**

解决：

- A 为每一个已发送的分组都设置了一个**超时计时器**。
- A 只要在超时计时器到期之前收到了相应的确认，就撤销该超时计时器，继续发送下一个分组 M2 。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E8%BF%9E%E7%BB%AD%E5%81%9C%E6%AD%A2%E5%8D%8F%E8%AE%AE2.webp)

值得注意的是：

- **在发送完一个分组后，必须暂时保留已发送的分组的副本，以备重发。**
- **分组和确认分组都必须进行编号。**
- 超时计时器的超时重传时间RTO（Retransmission Time-Out 这里有公式，可以有时间了解补充一下）应当比数据在分组传输的平均往返时间更长一些。

**确认丢失和确认迟到**

- **确认丢失** ：确认消息在传输过程丢失。当A发送M1消息，B收到后，B向A发送了一个M1确认消息，但却在传输过程中丢失。而A并不知道，在超时计时过后，A重传M1消息，B再次收到该消息后采取以下两点措施：1. 丢弃这个重复的M1消息，不向上层交付。 2. 向A发送确认消息。（不会认为已经发送过了，就不再发送。A能重传，就证明B的确认消息丢失）。
- **确认迟到** ：确认消息在传输过程中迟到。A发送M1消息，B收到并发送确认。在超时时间内没有收到确认消息，A重传M1消息，B仍然收到并继续发送确认消息（B收到了2份M1）。此时A收到了B第二次发送的确认消息。接着发送其他数据。过了一会，A收到了B第一次发送的对M1的确认消息（A也收到了2份确认消息）。处理如下：1. A收到重复的确认后，直接丢弃。2. B收到重复的M1后，也直接丢弃重复的M1。

##### 2.4.2 连续 ARQ 协议

- 发送方维持的发送窗口，它的意义是：**位于发送窗口内的分组都可连续发送出去，而不需要等待对方的确认。这样，信道利用率就提高了。**
- **连续 ARQ 协议规定，发送方每收到一个确认，就把发送窗口向前滑动一个分组的位置。**

即不必对收到的分组逐个发送确认，**而是对按序到达的最后一个分组发送确认，这样就表示：到这个分组为止的所有分组都已正确收到了。**

优点：**容易实现，即使确认丢失也不必重传。**
缺点：**不能向发送方反映出接收方已经正确收到的所有分组的信息。**

如果发送方发送了前 5 个分组，**而中间的第 3 个分组丢失了**。这时接收方只能对前两个分组发出确认**。发送方无法知道后面三个分组的下落，而只好把后面的三个分组都再重传一次。**
这就叫做 Go-back-N（回退 N），**表示需要再退回来重传已发送过的 N 个分组。**

具体实现：

> TCP 连接的每一端都必须设有两个窗口——一个发送窗口和一个接收窗口。
> TCP 的可靠传输机制用字节的序号进行控制。TCP 所有的确认都是基于序号而不是基于报文段。
> TCP 两端的四个窗口经常处于动态变化之中。
> TCP连接的往返时间 RTT 也不是固定不变的。需要使用特定的算法估算较为合理的重传时间。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E8%BF%9E%E7%BB%ADARQ%E7%AA%97%E5%8F%A3.webp)



连续ARQ协议规定，发送方每收到一个确认，就把发送窗口向前滑动一个分组的位置。例如上面的图（b），当发送方收到第一个分组的确认，就把发送窗口向前移动一个分组的位置。如果原来已经发送了前5个分组，则现在可以发送窗口内的第6个分组。 

接收方一般都是采用累积确认的方式。也就是说接收方不必对收到的分组逐个发送确认。而是在收到几个分组后，对按序到达的最后一个分组发送确认。如果收到了这个分组确认信息，则表示到这个分组为止的所有分组都已经正确接收到了。 

#### TCP的拥塞控制

开环控制方法就是在设计网络时**事先将有关发生拥塞的因素考虑周到，力求网络在工作时不产生拥塞**。

闭环控制方法是基于反馈环路的概念。属于闭环控制的有以下几种措施：

1. (1) **监测网络系统**以便检测到拥塞在何时、何处发生。
2. (2) 将拥塞发生的信息传**送到可采取行动的地方**。
3. (3) **调整网络系统的运行以解决出现的问题**。

TCP 采用基**于窗口的方法进行拥塞控制。该方法属于闭环控制方法。**

- **只要网络没有出现拥塞，拥塞窗口就可以再增大一些，以便把更多的分组发送出去，这样就可以提高网络的利用率**。
- 但只要网络出现拥塞或有可能出现拥塞，**就必须把拥塞窗口减小一些，以减少注入到网络中的分组数，以便缓解网络出现的拥塞。**

拥塞的判断：

1. 重传定时器超时
2. 收到三个相同（重复）的 ACK

TCP拥塞控制算法：

- 慢开始 (slow-start)
- 拥塞避免 (congestion avoidance)
- 快重传 (fast retransmit)
- 快恢复 (fast recovery)

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/TCP%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6%E6%B5%81%E7%A8%8B%E5%9B%BE.webp)

##### 1. 慢开始与拥塞避免

发送的最初执行慢开始，令 cwnd = 1，发送方只能发送 1 个报文段；当收到确认后，将 cwnd 加倍，因此之后发送方能够发送的报文段数量为：2、4、8 ...

注意到慢开始每个轮次都将 cwnd 加倍，这样会让 cwnd 增长速度非常快，从而使得发送方发送的速度增长速度过快，网络拥塞的可能性也就更高。设置一个慢开始门限 ssthresh，当 cwnd >= ssthresh 时，进入拥塞避免，每个轮次只将 cwnd 加 1。

如果出现了超时，则令 ssthresh = cwnd / 2，然后重新执行慢开始。

##### 2. 快重传与快恢复

在接收方，要求每次接收到报文段都应该对最后一个已收到的有序报文段进行确认。例如已经接收到 M1 和 M2，此时收到 M4，应当发送对 M2 的确认。

在发送方，如果收到三个重复确认，那么可以知道下一个报文段丢失，此时执行快重传，立即重传下一个报文段。例如收到三个 M2，则 M3 丢失，立即重传 M3。

在这种情况下，只是丢失个别报文段，而不是网络拥塞。因此执行快恢复，令 ssthresh = cwnd / 2 ，cwnd = ssthresh，注意到此时直接进入拥塞避免。

慢开始和快恢复的快慢指的是 cwnd 的设定值，而不是 cwnd 的增长速率。慢开始 cwnd 设定为 1，而快恢复 cwnd 设定为 ssthresh。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%BF%AB%E9%87%8D%E4%BC%A0%E7%A4%BA%E6%84%8F%E5%9B%BE.png)

#### 滑动窗口和流量控制

**TCP 利用滑动窗口实现流量控制。流量控制是为了控制发送方发送速率，保证接收方来得及接收。** 接收方发送的确认报文中的窗口字段可以用来控制发送方窗口大小，从而影响发送方的发送速率。将窗口字段设置为 0，则发送方不能发送数据。

**流量控制和拥塞控制的区别**

- 拥塞控制：
  - **拥塞控制是一个全局性的过程**，涉及到所有的主机、所有的路由器，以及与降低网络传输性能有关的所有因素。
  - 拥塞控制就是**防止过多的数据注入到网络中，使网络中的路由器或链路不致过载。**

- 流量控制：
  - 流量控制往往指**点对点通信量的控制，是个端到端的问题（接收端控制发送端）。**
  - 流量制所要做的就是**抑制发送端发送数据的速率，以便使接收端来得及接收**。

### TCP粘包现象原因和解决方法

TCP粘包就是指发送方发送的若干包数据到达接收方时粘成了一包，从接收缓冲区来看，后一包数据的头紧接着前一包数据的尾。原因可能是发送方也可能是接收方造成的。

- 发送方原因：TCP默认使用Nagle算法，将多次间隔较小、数据量较小的数据，合并成一个数据量大的数据块，然后进行封包。
- 接收方原因：TCP将接收到的数据包保存在接收缓存里，然后应用程序主动从缓存读取收到的分组。这样一来，如果TCP接收数据包到缓存的速度大于应用程序从缓存中读取数据包的速度，多个包就会被缓存，应用程序就有可能读取到多个首尾相接粘到一起的包。

如果多个分组毫不相干，甚至是并列关系，那么这个时候就一定要处理粘包现象了。

- 处理方法：发送方关闭Nagle算法。
- 接收方：接收方没有办法来处理粘包现象，只能将问题交给应用层来处理。应用层循环读取所有的数据，根据报文的长度判断每个包开始和结束的位置。

TCP为了保证可靠传输并减少额外的开销（每次发包都要验证），采用了基于流的传输，基于流的传输不认为消息是一条一条的，是无保护消息边界的。而UDP则是面向消息传输的，是有保护消息边界的，接收方一次只接受一条独立的信息，所以不存在粘包问题。

### TCP 保活机制

首先需要明确一点，为什么需要连接的保活？当双方已经建立了连接，但因为网络问题，链路不通，这样长连接就不能使用了。需要明确的一点是，通过 netstat，lsof 等指令查看到连接的状态处于 `ESTABLISHED` 状态并不是一件非常靠谱的事，因为连接可能已死，但没有被系统感知到，更不用提假死这种疑难杂症了。如果保证长连接可用是一件技术活。

#### 连接的保活：KeepAlive

首先想到的是 TCP 中的 KeepAlive 机制。KeepAlive 并不是 TCP 协议的一部分，但是大多数操作系统都实现了这个机制（所以需要在操作系统层面设置 KeepAlive 的相关参数）。KeepAlive 机制开启后，在一定时间内（一般时间为 7200s，参数 `tcp_keepalive_time`）在链路上没有数据传送的情况下，TCP 层将发送相应的 KeepAlive 探针以确定连接可用性，探测失败后重试 10（参数 `tcp_keepalive_probes`）次，每次间隔时间 75s（参数 `tcp_keepalive_intvl`），所有探测失败后，才认为当前连接已经不可用。

在 Netty 中开启 KeepAlive：

```
bootstrap.option(ChannelOption.SO_KEEPALIVE, true)
```

Linux 操作系统中设置 KeepAlive 相关参数，修改 `/etc/sysctl.conf` 文件：

```
net.ipv4.tcp_keepalive_time=90
net.ipv4.tcp_keepalive_intvl=15
net.ipv4.tcp_keepalive_probes=2
```

**KeepAlive 机制是在网络层面保证了连接的可用性** ，但站在应用框架层面我们认为这还不够。主要体现在三个方面：

- KeepAlive 的开关是在应用层开启的，但是具体参数（如重试测试，重试间隔时间）的设置却是操作系统级别的，位于操作系统的 `/etc/sysctl.conf` 配置中，这对于应用来说不够灵活。
- KeepAlive 的保活机制只在链路空闲的情况下才会起到作用，假如此时有数据发送，且物理链路已经不通，操作系统这边的链路状态还是 `ESTABLISHED`，这时会发生什么？自然会走 TCP 重传机制，要知道默认的 TCP 超时重传，指数退避算法也是一个相当长的过程。
- KeepAlive 本身是面向网络的，并不面向于应用，当连接不可用，可能是由于应用本身的 GC 频繁，系统 load 高等情况，但网络仍然是通的，此时，应用已经失去了活性，连接应该被认为是不可用的。

我们已经为应用层面的连接保活做了足够的铺垫，下面就来一起看看，怎么在应用层做连接保活。

#### 连接的保活：应用层心跳

如何理解应用层的心跳？简单来说，就是客户端会开启一个定时任务，定时对已经建立连接的对端应用发送请求（这里的请求是特殊的心跳请求），服务端则需要特殊处理该请求，返回响应。如果心跳持续多次没有收到响应，客户端会认为连接不可用，主动断开连接。不同的服务治理框架对心跳，建连，断连，拉黑的机制有不同的策略，但大多数的服务治理框架都会在应用层做心跳

#### 注意和 HTTP 的 KeepAlive 区别对待

- HTTP 协议的 KeepAlive 意图在于连接复用，同一个连接上串行方式传递请求 - 响应数据
- TCP 的 KeepAlive 机制意图在于保活、心跳，检测连接错误。

这压根是两个概念。

## 面试

### TCP 三次握手和四次挥手描述

### 为什么要三次握手

**三次握手的目的是建立可靠的通信信道，说到通讯，简单来说就是数据的发送与接收，而三次握手最主要的目的就是双方确认自己与对方的发送与接收是正常的。**

第一次握手：Client 什么都不能确认；Server 确认了对方发送正常，自己接收正常

第二次握手：Client 确认了：自己发送、接收正常，对方发送、接收正常；Server 确认了：对方发送正常，自己接收正常

第三次握手：Client 确认了：自己发送、接收正常，对方发送、接收正常；Server 确认了：自己发送、接收正常，对方发送、接收正常

所以三次握手就能确认双发收发功能都正常，缺一不可。

### 为什么要传回 SYN

接收端传回发送端所发送的 SYN 是为了告诉发送端，我接收到的信息确实就是你所发送的信号了。

> SYN 是 TCP/IP 建立连接时使用的握手信号。在客户机和服务器之间建立正常的 TCP 网络连接时，客户机首先发出一个 SYN 消息，服务器使用 SYN-ACK 应答表示接收到了这个消息，最后客户机再以 ACK(Acknowledgement[汉译：确认字符 ,在数据通信传输中，接收站发给发送站的一种传输控制字符。它表示确认发来的数据已经接受无误。 ]）消息响应。这样在客户机和服务器之间才能建立起可靠的TCP连接，数据才可以在客户机和服务器之间传递。

### 传了 SYN,为啥还要传 ACK

双方通信无误必须是两者互相发送信息都无误。传了 SYN，证明发送方到接收方的通道没有问题，但是接收方到发送方的通道还需要 ACK 信号来进行验证。

[![TCP四次挥手](https://camo.githubusercontent.com/66e3447783e22d736f7db9c7d121e8eba54f07c0/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392f372f5443502545352539422539422545362541432541312545362538432541352545362538392538422e706e67)](https://camo.githubusercontent.com/66e3447783e22d736f7db9c7d121e8eba54f07c0/68747470733a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f323031392f372f5443502545352539422539422545362541432541312545362538432541352545362538392538422e706e67)

断开一个 TCP 连接则需要“四次挥手”：

- 客户端-发送一个 FIN，用来关闭客户端到服务器的数据传送
- 服务器-收到这个 FIN，它发回一 个 ACK，确认序号为收到的序号加1 。和 SYN 一样，一个 FIN 将占用一个序号
- 服务器-关闭与客户端的连接，发送一个FIN给客户端
- 客户端-发回 ACK 报文确认，并将确认序号设置为收到序号加1

### 为什么要四次挥手

任何一方都可以在数据传送结束后发出连接释放的通知，待对方确认后进入半关闭状态。当另一方也没有数据再发送的时候，则发出连接释放通知，对方确认后就完全关闭了TCP连接。

举个例子：A 和 B 打电话，通话即将结束后，A 说“我没啥要说的了”，B回答“我知道了”，但是 B 可能还会有要说的话，A 不能要求 B 跟着自己的节奏结束通话，于是 B 可能又巴拉巴拉说了一通，最后 B 说“我说完了”，A 回答“知道了”，这样通话才算结束。

上面讲的比较概括，推荐一篇讲的比较细致的文章：<https://blog.csdn.net/qzcsu/article/details/72861891>

### TCP 协议如何保证可靠传输

### TCP,UDP 协议的区别