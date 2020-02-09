# DNS域名系统

## 一、DNS域名系统

### 1.1 DNS介绍

DNS 是一个分布式数据库，提供了主机名和 IP 地址之间相互转换的服务。这里的分布式数据库是指，每个站点只保留它自己的那部分数据。

域名具有层次结构，从上到下依次为：根域名、顶级域名、二级域名。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/%E5%9F%9F%E5%90%8D%E7%BB%93%E6%9E%84.jpg)

### 1.2 DNS传输

DNS 可以使用 UDP 或者 TCP 进行传输，使用的端口号都为 53。大多数情况下 DNS 使用 UDP 进行传输，这就要求域名解析器和域名服务器都必须自己处理超时和重传从而保证可靠性。在两种情况下会使用 TCP 进行传输：

- 如果返回的响应超过的 512 字节（UDP 最大只支持 512 字节的数据）。
- 区域传送（区域传送是主域名服务器向辅助域名服务器传送变化的那部分数据）。

### 1.3 域名的解析过程

#### 1.3.1 两种查询方式（递归、迭代）

- **主机向本地域名服务器的查询一般都是采用递归查询**。如果主机所询问的本地域名服务器不知道被查询域名的 IP 地址，那么本地域名服务器就以 DNS 客户的身份，向其他根域名服务器继续发出查询请求报文。
- **本地域名服务器向根域名服务器的查询通常是采用迭代查询。**当根域名服务器收到本地域名服务器的迭代查询请求报文时，要么给出所要查询的 IP 地址，要么告诉本地域名服务器：“你下一步应当向哪一个域名服务器进行查询”。然后让本地域名服务器进行后续的查询。

**从客户端到本地DNS服务器是属于递归查询，而DNS服务器之间就是的交互查询就是迭代查询**。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/dns%E8%A7%A3%E6%9E%90.jpg)

#### 1.3.2 解析过程

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/%E6%B5%8F%E8%A7%88%E5%99%A8DNS%E8%A7%A3%E6%9E%90%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg)

1. 浏览器输入地址，然后浏览器这个进程去调操作系统某个库里的gethostbyname函数(例如，Linux GNU glibc标准库的gethostbyname函数)，然后呢这个函数通过网卡给DNS服务器发UDP请求，接收结果，然后将结果给返回给浏览器。

- 我们在用chrome浏览器的时候，其实会先去浏览器的dns缓存里头查询，dns缓存中没有，再去调用gethostbyname函数。 

2. gethostbyname函数在试图进行DNS解析之前首先检查域名是否在本地 Hosts 里，如果没找到再去本地DNS解析器缓存上查
3. 如果hosts与本地DNS解析器缓存都没有相应的网址映射关系，首先会找TCP\/IP参数中设置的首选DNS服务器，在此我们叫它本地DNS服务器，此服务器收到查询时，如果要查询的域名，包含在本地配置区域资源中，则返回解析结果给客户机，完成域名解析，此解析具有权威性。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/%E6%9C%AC%E6%9C%BADNS.jpg)

4. 如果本地DNS服务器本地区域文件与缓存解析都失效，则根据本地DNS服务器的设置(是否设置转发器)进行查询，如果未用转发模式，本地DNS就把请求发至13台根DNS，根DNS服务器收到请求后会判断这个域名(.com)是谁来授权管理，并会返回一个负责该顶级域名服务器的一个IP。本地DNS服务器收到IP信息后，将会联系负责.com域的这台服务器。这台负责.com域的服务器收到请求后，如果自己无法解析，它就会找一个管理.com域的下一级DNS服务器地址(qq.com)给本地DNS服务器。当本地DNS服务器收到这个地址后，就会找qq.com域服务器，重复上面的动作，进行查询，直至找到www.qq.com主机。
5. 如果用的是转发模式，此DNS服务器就会把请求转发至上一级DNS服务器，由上一级服务器进行解析，上一级服务器如果不能解析，或找根DNS或把转请求转至上上级，以此循环。不管是本地DNS服务器用的是转发，还是根提示，最后都是把结果返回给本地DNS服务器，由此DNS服务器再返回给客户机。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/DNSProcess.png)

#### 1.3.3 查看解析过程

linux下一个`dig`命令，以显示解析域名的过程

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/img/dns%E5%91%BD%E4%BB%A4.png)

- `www.tmall.com`对应的真正的域名为`www.tmall.com.`。末尾的`.`称为根域名，因为每个域名都有根域名，因此我们通常省略。

  根域名的下一级，叫做"顶级域名"（top-level domain，缩写为TLD），比如`.com、.net`；

  再下一级叫做"次级域名"（second-level domain，缩写为SLD），比如`www.tmall.com`里面的`.tmall`，这一级域名是用户可以注册的；

  再下一级是主机名（host），比如`www.tmall.com`里面的`www`，又称为"三级域名"，这是用户在自己的域里面为服务器分配的名称，是用户可以任意分配的。

- 第一行就是说`www.tmall.com`有一个别名是`www.tmall.com.danuoyi.tbcache.com`。后面几行就是这个`www.tmall.com.danuoyi.tbcache.com`地址的真实IP。就是一个淘宝的cdn环境

### 常见面试问题

- DNS在域名的解析上用UDP还是TCP？
- 讲讲域名解析为什么用UDP？区域复制为什么用TCP？
- 讲讲DNS是怎么做域名解析的？
- 为什么要用CNAME，也就是使用CNAME有什么好处？



## 参考

- https://zhuanlan.zhihu.com/p/79350395
- https://troywu0.gitbooks.io/hadoop/dns.html
- https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247484322&idx=8&sn=9dda601c5387ed04fbd7bfd83fda0da1&chksm=ebd742a3dca0cbb52d4993e0b4c38199520a685b5b966e7c43679c310c14dc5b535551876e79&token=620000779&lang=zh_CN&scene=21###wechat_redirect
- https://github.com/CyC2018/CS-Notes/blob/master/notes/计算机网络%20-%20应用层.md
- 