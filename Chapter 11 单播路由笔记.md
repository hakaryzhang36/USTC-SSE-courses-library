# 路由信息协议 RIP
应用层协议，作为 UDP 的载荷，Port 520。

## 报文格式
以下为一条RIP 路由信息的格式，UDP 载荷中可以同时传递多条 RIP 路由信息。
![[attachments/Pasted image 20221031155907.png]]
- Command：报文类型（请求、响应）
- Family：【待补充】

## 更新路由表过程 Update Routing Table
1. 合法性检验
    IP and Port
2. 操作每条路由信息
    1. metric = min（metric + 1，16）
    2. if not in table，add it
    3. if new next-hop as same as the IP in table，update
    4. if metric is lower, update

## 定时器 Timer
- 周期定时器 Periodic Timer

- 超时定时器 Expiration Timer
    当一段时间接收不到一个网络地址的更新信息，则将该地址置为不可达（不删除）

- 垃圾回收定时器 Garbage Collection Timer

- 抑制定时器 Hold-down Timer
    为了避免 Routing looping 问题

![[attachments/Pasted image 20221031155711.png]]

## RIPv2
## 报文格式
![[attachments/Pasted image 20221031160357.png]]
- 路由标记 Route Tag：【待补充】


# 链路状态路由 Link State Routing
- 每一个节点都会获得域内所有的链路状态信息
- 每一个节点通过 Dijkstra 算法构建自己的路由表（不直接从获取相邻节点获取，不同于距离向量路由）

**距离向量路由 vs 链路状态路由**
距离向量路由：
    - 共享自己的路由信息
    - 发送（单播/广播）到相邻节点
    - 周期更新 + 触发更新
    - 删减路由信息得到路由表
链路状态路由：
    - 共享自己的链路状态
    - 泛洪到所有节点
    - 触发更新
    - 自行计算得到路由表


## 生成路由表流程
1. **装载链路状态包 LSP**
    LSP 数据：
        - node ID
        - list of links
        - sequence number
        - age
    LSP 生成时间
        - 相邻链路发送改变
        - 周期性（非必须）

2. **泛洪转发 LSP**
    为了保证每个节点都有相同的 LSP，需要可靠转发，ACK 重传机制。

3. **构建最短路径树 SPT**

4. **得到路由表**

# 最短路径协议 OSPF
网络层（其实传输层更合理），作为 IP 的数据段，type = 89。

## 分区 Area
**目的：**
- 减少 LSP 广播
- 减少 SPF 计算
- 缩小路由表规模

![[attachments/Pasted image 20221031170411.png]]
Area 0 骨干区，与所有区域邻接，必需。

**如果不能直连 Area 0，则通过虚链路**
![[attachments/Pasted image 20221031171205.png]]

## 链路 Link
![[attachments/Pasted image 20221031171410.png]]

### Transit network 转接网络
在一个区域内，为了减少路由计算的数量，需要减少链路信息，选举出一个指定路由器 Designated Router。

所有路由器逻辑上只和 DR 邻接，也就是逻辑上只有一条域内链路。


## OSPF 分组
### 5 种分组
![[attachments/Pasted image 20221031172744.png]]
- Hello：keep alive
- 数据库描述 DBD： 有新的邻居加入时，向其发送。
- 链路状态请求 LSR：向邻居请求指定链路状态。
- 链路状态更新 LSU：向邻居发送链路状态信息 LSA。
- 链路状态通告 LSAck

### 首部格式
![[attachments/Pasted image 20221031172915.png]]

### 整体格式
![[attachments/Pasted image 20221031173635.png]]


### 链路状态信息 LSA （5种）
一共 5 种状态信息 LSA
- Type 1： Router-LSA（区域内）
- Type 2： Network-LSA（区域内）
- Type 3： Summary—LSA
- Type 4： Summary-LSA
- Type 5： AS-external-LSA

#### 路由器状态 Router-LSA
路由器端口信息，区域内洪泛。

**LSA 分组格式**
![[attachments/Pasted image 20221031192352.png]]

#### 网络状态 Network—LSA
路由器连接的网络信息，一般是边界路由器 ABR 和指定路由器 DR 发送，区域内洪泛。
![[attachments/Pasted image 20221031192709.png]]
![[attachments/Pasted image 20221031192812.png]]

#### 区域汇总状态 Summary-LSA
区域边界路由器 ABR 发出，对应 LSA-Type 为 3
![[attachments/Pasted image 20221031193513.png]]

自治系统边界路由器 ASBR 向自治系统内发送，对应 LSA-Type 为 4
![[attachments/Pasted image 20221031193702.png]]

#### 自治系统外部状态 AS-external-LSA
![[attachments/Pasted image 20221031193532.png]]


### 链路类型与 Link ID 的关系
![[attachments/Pasted image 20221031174044.png]]

## OSPF 流程
1. 确定邻接关系
2. 选举 DR/BDR
3. 交换链路状态信息
4. 获取路由，计算 SPF
5. 维护链路状态信息

## OSPF 状态
### 状态分类
- **Down**
- **Init**：收到 Hello 分组时，建立邻居表。
- **2-Way**：收到邻居回复的 Hello 分组，当中包含自己的信息（不用回复 Hello），建立双向邻居关系
- **ExStart**：通过 Hello 分组，确定 Master 和 Slave。
- **ExChange**：交换 DBD，比较链路数据库 LSDB
- **Loading**：通过 LSR、LSU、LSAck 获得邻居有但自己没有的路由的完整信息。
- **Full**：建立双向邻接关系。

### 状态时序
与 OSPF 操作流程相对应

![[attachments/Pasted image 20221101153811.png]]
![[attachments/Pasted image 20221101153615.png]]

# RIP vs OSPF
![[attachments/Pasted image 20221101155225.png]]

# 路径向量路由 Path Vector Routing
为了减少路由报文的数量，减少路由计算，不记录 Metric 了，只记录路径，用于**外部路由协议**。

对于一个 AS 中，有一个与外部 AS 交换的外部路由器，该路由器首先有一个记录了所属 AS 中所有网络的列表。
![[attachments/Pasted image 20221101164546.png]]

然后通过在 AS 之间交换彼此的可达网络列表，每个外部路由器就可以建立一张路径向量路由表。
![[attachments/Pasted image 20221101161519.png]]

**避免环路**
如果收到的可达性路由中，路径上已有自己所属的 AS，则表示会出现环路，丢弃该信息。




# 边界网关协议 BGP
传输层协议，端口 179，采用路径向量路由，现在主流是 BGP-4。

根据在 AS 内外的不同数据通信，可以分为：
- 内部 BGP 会话
- 外部 BGP 会话

## 自治系统类型划分
- **残桩系统 Stub AS**：连接单个 AS。
- **多归属系统 Multihomed AS**：连接多个 AS，但不允许穿越的数据通信，必然一家大公司。
- **转接系统 Transit AS**


## BGP 报文
### 报文分类
![[attachments/Pasted image 20221101165759.png]]
- Open：type = 1，TCP 长连接保持随时可以传输数据。
- Update：更新路径向量路由。
- Keepalive：配合 Open 长连接，保持。
- Notification：链路状态改变的通知（不可达、故障）

### 报文格式
![[attachments/Pasted image 20221101170215.png]]
- Path Attributes：路径属性，用于评估路径成本
    格式 <type, length, value>


## BGP 生成路由表过程
![[attachments/Pasted image 20221101174357.png]]

# OSPF 和 BGP 的联动问题



S1mple

Hak4ry
Hakary

h4ckT3ch

Hak0mat1ay
Packet

h4ckG3n