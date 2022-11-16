# 实验拓扑
![[attachments/Pasted image 20221018174246.png]]
> **实验前准备配置**
> 首先根据实验给出的接口编址配置好路由器接口 IP。
> ![[attachments/Pasted image 20221018175228.png]]

# 配置 OSPF 路由协议
## 基础配置
R1 中执行命令如下。
``` text
// 以路由器环回地址作为路由器 ID
ospf router-id 10.0.1.1

// 依次配置不同的区域包含的网段
area 0
network 10.0.123.0 0.0.0.255
network 10.0.1.1 0.0.0.0

area 1
network 10.0.14.0 0.0.0.255

area 2
network 10.0.15.0 0.0.0.255
```

R2 中执行命令如下。
``` text
ospf router-id 10.0.2.2 
area 0
network 10.0.123.0 0.0.0.255
network 10.0.2.2 0.0.0.0
```
R3 配置命令类似。

R4 中执行命令如下。
``` text
ospf router-id 10.0.4.4
area 1
network 10.0.14.0 0.0.0.255
network 10.0.4.4 0.0.0.0
```
R5 配置命令类似。

## 测试总结
### 检查基础配置成功
查看 R1 邻居情况
![[attachments/Pasted image 20221018221228.png]]

查看接口 g0/0/0 的 OSPF 详细配置，如下图所示。
![[attachments/Pasted image 20221018121443.png]]

查看接口 s0/0/1 的 OSPF 详细配置，如下图所示。
![[attachments/Pasted image 20221018102200.png]]

### 观察 OSPF 邻居建立过程
关闭点对点网络接口，执行命令如下。
``` text
interface Serial 0/0/1
shutdown
interface Serial 0/0/0
shutdown
```
关闭后，检查 R1 的邻居状态，如下图所示。可以看出仅剩 area 0 中的邻接关系。
![[attachments/Pasted image 20221018120403.png]]

在 R1 上重启 OSPF 进程，观察 R1 和 R2 间邻接关系建立过程。执行命令如下。
``` text
debugging ospf packet
reset ospf process
```

输出过程信息如下。
``` text
// Down -> Init
Oct 18 2022 12:05:07-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[18]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.123.2, Neighbor
Event=HelloReceived, NeighborPreviousState=Down, NeighborCurrentState=Init)

// Init -> 2Way
Oct 18 2022 12:05:07-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[19]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.123.2, Neighbor
Event=2WayReceived, NeighborPreviousState=Init, NeighborCurrentState=2Way)

// 2Way -> ExStart
Oct 18 2022 12:05:07-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[20]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.123.2, Neighbor
Event=AdjOk?, NeighborPreviousState=2Way, NeighborCurrentState=ExStart)

// ExStart -> Exchange
Oct 18 2022 12:05:07-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[23]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.123.2, Neighbor
Event=NegotiationDone, NeighborPreviousState=ExStart, NeighborCurrentState=Excha
nge)

// Exchange -> Loading
Oct 18 2022 12:05:07-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[25]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.123.2, Neighbor
Event=ExchangeDone, NeighborPreviousState=Exchange, NeighborCurrentState=Loading
)

// Loading -> Full
Oct 18 2022 12:05:07-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[27]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.123.2, Neighbor
Event=LoadingDone, NeighborPreviousState=Loading, NeighborCurrentState=Full)
```

### 邻居和邻接关系的区别
更改 R1、R2 的选举优先级，使之不参与选举。分别在 R1、R2 中执行命令如下。
``` text
// R1
int g0/0/0
ospf dr-priority 0

// R2
int g0/0/0
ospf dr-priority 0
```

重启 R1、R2 OSPF 进程，执行命令如下。
``` text
// R1
reset ospf process

// R2
reset ospf process
```

在 R3 中检查 OSPF 邻居配置，如下图所示。
此时 R3 为 DR，R3 和 R1、R2 的链路状态 State 都为 Full，表示是邻接关系。
![[attachments/Pasted image 20221018122051.png]]

在 R1 中检查 OSPF 邻居配置，如下图所示。
此时 R1 为 DRothers，R1 和 R2 的链路状态 State 都为 2-Way，表示是邻居关系。
![[attachments/Pasted image 20221018122311.png]]

### 观察 OSPF 邻居建立过程
重启 R1 的 OSPF 进程，执行命令如下。
``` text
// R1
debugging ospf packet
reset ospf process
```

R1 和 R2 间输出过程信息如下。
``` text
// Down -> Init
Oct 18 2022 18:27:20-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[20]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.123.2, Neighbor
Event=HelloReceived, NeighborPreviousState=Down, NeighborCurrentState=Init)

// Init -> 2Way
Oct 18 2022 18:27:20-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[21]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.123.2, Neighbor
Event=2WayReceived, NeighborPreviousState=Init, NeighborCurrentState=2Way)

// 选举过程中保持2Way
Oct 18 2022 18:27:20-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[22]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.123.2, Neighbor
Event=AdjOk?, NeighborPreviousState=2Way, NeighborCurrentState=2Way)

// 选举过程中保持2Way
Oct 18 2022 18:27:24-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[25]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.123.2, Neighbor
Event=AdjOk?, NeighborPreviousState=2Way, NeighborCurrentState=2Way)
```


### 观察 OSPF 邻居建立过程（点对点网络）
在 R1 中，关闭 g0/0/0 接口，启用 s0/0/0 和 s0/0/1 接口。执行命令如下。
``` text
interface Serial 0/0/1
undo shutdown
interface Serial 0/0/0
undo shutdown
interface GigabiEthernet0/0/0
shutdown
```

检查邻居状态，如下图所示。
![[attachments/Pasted image 20221018171645.png]]

重启 R1 的 OSPF 进程，执行命令如下。
``` text
// R1
debugging ospf packet
reset ospf process
```

R1 和 R4 间输出过程信息如下。
``` text
// Down -> Init
Oct 18 2022 17:18:45-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[59]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.14.4, NeighborE
vent=HelloReceived, NeighborPreviousState=Down, NeighborCurrentState=Init)

// Init -> ExStart
Oct 18 2022 17:18:45-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[60]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.14.4, NeighborE
vent=2WayReceived, NeighborPreviousState=Init, NeighborCurrentState=ExStart)

// ExStart -> Exchange
Oct 18 2022 17:18:45-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[61]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.14.4, NeighborE
vent=NegotiationDone, NeighborPreviousState=ExStart, NeighborCurrentState=Exchan
ge)

// ExChange -> Loading
Oct 18 2022 17:18:45-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[62]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.14.4, NeighborE
vent=ExchangeDone, NeighborPreviousState=Exchange, NeighborCurrentState=Loading)

// Loading -> Full
Oct 18 2022 17:18:45-08:00 R1 %%01OSPF/4/NBR_CHANGE_E(l)[63]:Neighbor changes ev
ent: neighbor status changed. (ProcessId=1, NeighborAddress=10.0.14.4, NeighborE
vent=LoadingDone, NeighborPreviousState=Loading, NeighborCurrentState=Full)
```

由于 R1、R4 是点对点网络，不用进行 DR 选举。

### 观察 OSPF 链路状态数据库同步过程
重启 R1 的 OSPF 进程，执行命令如下。
``` text
// R1
debugging ospf packet
reset ospf process
```

R1 和 R4 的链路状态数据库信息，如下图所示。
![[attachments/Pasted image 20221018184912.png]]
![[attachments/Pasted image 20221018184956.png]]
对比后可以发现，R1、R4 数据库中关于 area 1 的链路信息是一致的。

同步过程中，R1 的 s0/0/0 接口抓包结果如下。
**报文1**：R1 向 R2 发送 DB Description 报文，包含
- 路由器链路状态信息（Router-LSA）
    - R1
- R1 连接网段链路状态信息（Summary-LSA）
    - 10.0.15.0 网段
    - 10.0.5.5 网段
    - 10.0.1.1 网段（该网段属于 area 0，故需要记录为 R1 area 2 下的 Summary-LSA）
![[attachments/Pasted image 20221018191811.png]]

R2 接收到后，将原本没有的 LSA 存入 LSDB，然后再将自己的 LSDB 信息广播出去。

**报文2**：R2 向 R1 发送 DB Description 报文，包含
- 路由器链路状态信息（Router-LSA）
    - R1
    - R2
- R1 连接网段链路状态信息（Summary-LSA）
    - 10.0.1.1 网段
    - 10.0.15.0 网段
    - 10.0.5.5 网段
    （10.0.4.4 网段属于 area 1 的内部子网，故不记录为 Summary-LSA，而是记录为 Router-LSA）
![[attachments/Pasted image 20221018193526.png]]

**报文3**：R1 向 R2 发送 LS 请求报文，获取 LSDB 中没有的 LSA
![[attachments/Pasted image 20221018193617.png]]

# 解答题
1. 在 OSPF 广播型网络中的 DR 与 BDR 之间需要建立 OSPF 邻接关系吗？为什么？
    需要，因为 BDR 也要接收 LSA 才能实现备份，如果二者不是邻接关系则会导致 DR 的 LSA 无法发往 BDR。

2. 按照 OSPF 的网络设计要求，不同普通区域 Area 之间的通信必须经骨干区域 Area 0 中转才能实现，这种要求的出发点是什么？
    因为OSPF区域间路由使用距离矢量计算最优路由，严格的区域层次结构可以避免路由回环导致的“无穷计数”问题。