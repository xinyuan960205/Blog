## 一、垂直分区

垂直切分是将一张表按列切分成多个表，通常是按照列的关系密集程度进行切分，也可以利用垂直切分将经常被使用的列和不经常被使用的列切分到不同的表中。

在数据库的层面使用垂直切分将按数据库中表的密集程度部署到不同的库中，例如将原来的电商数据库垂直切分成商品数据库、用户数据库等。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E5%9E%82%E7%9B%B4%E5%88%87%E5%88%86.jpg)





## 二、水平分区

水平切分又称为 Sharding，它是将同一个表中的记录拆分到多个结构相同的表中。

当一个表的数据不断增多时，Sharding 是必然的选择，它可以将数据分布到集群的不同节点上，从而缓存单个数据库的压力。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E6%B0%B4%E5%B9%B3%E5%88%87%E5%88%86.jpg)



## 三、读写分离

主服务器处理写操作以及实时性要求比较高的读操作，而从服务器处理读操作。

读写分离能提高性能的原因在于：

- 主从服务器负责各自的读和写，极大程度缓解了锁的争用；
- 从服务器可以使用 MyISAM，提升查询性能以及节约系统开销；
- 增加冗余，提高可用性。

读写分离常用代理方式来实现，代理服务器接收应用层传来的读写请求，然后决定转发到哪个服务器。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB.png)



## 四、主从复制

主要涉及三个线程：binlog 线程、I/O 线程和 SQL 线程。

- **binlog 线程** ：负责将主服务器上的数据更改写入二进制日志（Binary log）中。
- **I/O 线程** ：负责从主服务器上读取二进制日志，并写入从服务器的中继日志（Relay log）。
- **SQL 线程** ：负责读取中继日志，解析出主服务器已经执行的数据更改并在从服务器中重放（Replay）。

![](https://raw.githubusercontent.com/xinyuan960205/pic_resource/master/image/%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6.png)