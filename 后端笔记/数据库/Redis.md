## redis基础

### 缓存 cache

**空间换时间** 用更多的存储空间来存储一些可能重复使用或计算的数据，从而减少数据的重新获取或计算的时间。

​  

**索引**：索引是一种将数据库表中的某些列或字段按照一定的排序规则组织成一个单独的数据结构，需要额外占用空间，但可以大大提高检索效率，降低数据排序成本。  
**数据库表字段冗余**：将经常联合查询的数据冗余存储在同一张表中，以减少对多张表的关联查询，进而提升查询性能，减轻数据库压力。  
**CDN**（内容分发网络）：将静态资源分发到多个不同的地方以实现就近访问，进而加快静态资源的访问速度，减轻服务器以及带宽的负担。

​  

本地缓存

**Caffeine**

​  

分布式缓存

​  

多级缓存

方案： Cache Aside（旁路缓存）模式

​  

## 常见的缓存更新策略

### 1.Cache Aside Pattern（旁路缓存模式）

为什么是删除cache （更直接 有些数据的cache不一定会长期被访问到）

  

先删除cache 再写入db？ 很容易导致不一致的问题

先写入db 再删除 ？ 可能性很小 cache写入速度很快

  

缺陷：1.第一次查询 肯定没有cache （提前录入）

2.频繁写的操作 会一直触发删除cache的操作 （更新cache）

​  

​  

### **2.Read/Write Through Pattern（读写穿透）**

Read-Through Pattern 实际只是在 **Cache-Aside Pattern 之上进行了封装**。Read Through Pattern 是 cache 服务自己来写入缓存的，这对客户端是透明的。

  

### 3.Write Behind Pattern（异步缓存写入）

  
Write Behind Pattern 和 Read/Write Through Pattern 很相似，两者都是由 **cache 服务来负责 cache 和 db 的读写。**  
  
但是，两个又有很大的不同：Read/Write Through 是**同步更新** cache 和 db，而 Write Behind 则是只更新缓存，不直接更新 db，而是改为**异步批量的方式来更新** db。

## redis集群

Redis 默认采用异步复制：Master 不需要等待 Replica 完成复制，就能向客户端确认写入成功。

这意味着客户端收到 `OK`，只能说明主节点执行了写操作，**不能说明数据已经安全地复制到其他节点。**

​  

### 1.主从复制（全量复制 追加）

  

#### RDB （redis database） 快照机制

​  

涉及操作系统的 COW（Copy-On-Write，写时复制）

**如果主进程在生成 RDB 期间修改了大量共享内存页，操作系统会通过 COW 为被修改的内存页创建副本**

​  

**Partial Resynchronization （psync）**

-   **Replication ID****：识别数据复制历史。**
-   **Replication Offset****：识别复制进度。**
-   **Replication Backlog：保存一段近期的复制数据流，供断线重连时补发。**

**​**  

#### AOF（Append Only File，追加日志文件

### 2.哨兵集群 （Redis Sentinel）

Redis Sentinel 实现 Redis 集群高可用，**只是在主从复制实现集群的基础下，多了一个 Sentinel 角色**来帮助我们监控 Redis 节点的运行状态并自动实现故障转移。

​  

**部署多个 Sentinel 实例**，形成监控与协调体系 。 **不承担普通 Redis 的业务数据存储职责**

SDOWN → ODOWN → Leader 选举 → Replica 晋升 → 重新配置复制关系

​  

​  

### 3.Redis Cluster 切片

#### 横向扩展

1.  **存储容量**：一台机器放不下，就分散到多台机器。
2.  **读写吞吐量**：所有写请求不再集中在一个 Master 上，可以由多个 Master 分担。

​  

#### 16384 个哈希槽

  

#### **MOVED 和 ASK：保证扩缩容期间仍能访问数据**

  

#### 故障转移（master之间检查 replic选举）

Gossip 故障检测 → Master 多数派确认 FAIL → Replica 发起选举 → Master 投票授权 → Replica 晋升并接管槽位。