## MQ 最核心的三个价值

### 1.异步

### 2.解耦

### 3.削峰

## MQ 真正难的地方：分布式可靠性

  

  

## Kafka （ 一个分布式、可持久化的事件日志系统 ）

### 发布订阅模型

订阅 ： Consumer 在**逻辑**上订阅 Topic，在实际运行中消费分配给自己的 Partition；无论这些 Partition 分布在哪个 Broker 上，都可以通过**元数据**定位并读取。

  

### 集群

备份机制 Leader Follower 。 Follower会周期地向 Leader 发送同步请求。

​  

### Kafka高可用（ 当系统中某些节点发生故障时，整个系统仍然能够继续提供服务）

多 Broker 集群 + Partition 副本 + Leader 故障切换

副本管理：

1\. LEO：Log End Offset

日志末尾位置，表示这个副本已经写到了哪里。

​  

​  

2\. ISR：In-Sync Replicas （写确认、Leader选举）

（动态机制：只要求当前仍然保持同步的副本共同参与可靠写入，同时允许落后的副本暂时退出这个集合。）

因为当前 ISR 包含 Broker 1 和 Broker 2，`acks=all` 要求 ISR 中的副本确认这次写入。

Leader 将等待 ISR 中的所有副本确认后再做出应答

ISR 中的副本都是与 Leader 同步的副本

​  

​  

3\. HW：High Watermark

高水位，表示哪些消息已经完成复制提交，可以安全地对普通消费者开放。

​  

​  

​  

4\. Eligible Leader Replicas（ELR）机制

​  

​  

管理整个集群的角色：Controller

​  

​  

## 为什么这么快

### 性能抽象： 网络 磁盘 复杂度

### 解决方案抽象：并发 压缩 批量 缓存 算法

## 操作系统：

#### 顺序写

磁盘 HDD（过时了）

**顺序写主要减少机械寻址成本**

​  

固态硬盘 SSD（底层机制：Flash Translation Layer（FTL，闪存转换层））：

**顺序追加主要通过批处理、减少随机覆盖和优化 I/O 模式提高效率，同时可能降低写入放大**

**​**  

1.SSD 可能需要将修改后的数据写到新的物理位置，并把旧页标记为无效。之后垃圾回收时，还可能涉及有效数据搬迁、块擦除等操作。

2.写入放大（Write Amplification）：操作系统要求写入一定数量的数据，但 SSD 内部实际执行了更多的数据写入。

相比之下，连续、大批量写入通常更容易提高吞吐效率，减少部分映射管理和垃圾回收压力。

  

  

#### 零拷贝

`内核 → Java → 内核 → 网卡`

`内核 Page Cache → 网卡`

DMA = Direct Memory Access，直接内存访问

Page Cache（页缓存）是操作系统内核利用 RAM 缓存文件数据的机制

​  

## 网络模型

| 名称 | 英文全称 | 核心特点 |
| --- | --- | --- |
| BIO | Blocking I/O | 阻塞式 I/O |
| NIO | New I/O | 提供 Channel、Buffer、Selector 等机制，支持非阻塞 I/O |
| AIO | Asynchronous I/O | 异步 I/O，通过完成通知获取操作结果 |

### Reactor线程模型

1.  Acceptor：接受新连接，并将其交给某个 Processor。
2.  Processor：将 Socket 注册到自己的 Selector，监听可读、可写等事件。
3.  Processor：发现客户端发来请求，读取网络数据，解析请求并加入 RequestChannel。
4.  Handler：从队列取出请求，执行 Kafka 相应的处理逻辑。
5.  Processor：取得处理后的响应，通过对应连接发送给客户端。

  

  

### 批量传输与压缩消息

1.合成消息块 2.压缩算法 （提高吞吐量）

​