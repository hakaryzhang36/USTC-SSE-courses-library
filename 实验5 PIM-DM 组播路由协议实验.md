# 实验拓扑

![[attachments/Pasted image 20221026155840.png]]
# 基础配置
> **Basic Configure**
> 首先根据实验给出的接口编址配置好设备。
> ![[attachments/Pasted image 20221026160051.png]]
> 

**检测 R1 与 R3 之间的连通性**
R1 执行命令如下。
```
ping -c 1 20.0.13.3
```
输出截图如下。
![[attachments/Pasted image 20221026160437.png]]

# 配置 IGP
## 配置 OSPF
在每台路由器上配置 OSPF 路由协议，并通告直连网段。各路由器中执行命令如下。
```
// R1
ospf 1
area 0
network 10.0.12.0 0.0.0.255
network 20.0.13.0 0.0.0.255
network 30.0.14.0 0.0.0.255
network 172.16.1.0 0.0.0.255

// R2
ospf 1
area 0
network 10.0.12.0 0.0.0.255
network 10.0.2.0 0.0.0.255 

// R3
ospf 1
area 0
network 20.0.13.0 0.0.0.255
network 20.0.2.0 0.0.0.255

// R4
ospf 1
area 0
network 20.0.2.0 0.0.0.255
network 30.0.14.0 0.0.0.255 

// R5
ospf 1
area 0
network 20.0.2.0 0.0.0.255
network 30.0.2.0 0.0.0.255
```

## 配置结果
### 查看各路由器 OSPF 邻居关系和路由表
**R1**
ospf 邻居关系，如下图所示。
![[attachments/Pasted image 20221026162141.png]]
路由表，如下图所示。
![[attachments/Pasted image 20221026164951.png]]

**R2**
ospf 邻居关系，如下图所示。
![[attachments/Pasted image 20221026165115.png]]
路由表，如下图所示。
![[attachments/Pasted image 20221026165135.png]]

**R3**
ospf 邻居关系，如下图所示。
![[attachments/Pasted image 20221026165323.png]]
路由表，如下图所示。
![[attachments/Pasted image 20221026165344.png]]

**R4**
ospf 邻居关系，如下图所示。
![[attachments/Pasted image 20221026165410.png]]
路由表，如下图所示。
![[attachments/Pasted image 20221026165427.png]]

**R5**
ospf 邻居关系，如下图所示。
![[attachments/Pasted image 20221026165452.png]]
路由表，如下图所示。
![[attachments/Pasted image 20221026165511.png]]

# 配置 PIM-DM
## 开启组播功能
在各路由器中开启组播功能，执行命令如下。
```
multicast routing-enable
```

在**各路由器接口**下配置 PIM-DM 模式，执行命令如下。
```
pim dm
```

## 使能 IGMP
在 R2-R5 的**用户侧接口**下使能 IGMP。执行命令如下。
```
igmp enable
```

## 配置结果
### 查看各路由器 PIM 邻居关系
**R1**
![[attachments/Pasted image 20221026170301.png]]

**R2**
![[attachments/Pasted image 20221026170408.png]]

**R3**
![[attachments/Pasted image 20221026170438.png]]

**R4**
![[attachments/Pasted image 20221026170519.png]]

**R5**
![[attachments/Pasted image 20221026170546.png]]

### 查看各个路由器的 IGMP 接口信息
**R2**
![[attachments/Pasted image 20221026175203.png]]

**R3**
![[attachments/Pasted image 20221026175800.png]]

**R4**
![[attachments/Pasted image 20221026175821.png]]

**R5**
![[attachments/Pasted image 20221026175848.png]]

# 配置组播服务器
## Ensp 全局设置
组播服务器 Source-1 需要使用 VLC 播放视频的方式来发送组播。首先，在网上下 载并安装一个 VLC 软件，然后，在 eNSP 软件的主界面中点击右上方工具栏的设置按 钮。在“工具设置”页面中设置 VLC 软件路径，如下图所示。
![[attachments/Pasted image 20221026200353.png]]

## 配置组播服务器
文件路径为本地一个视频文件，注意需要通过 浏览 指定视频文件，不能将视频路径直接粘贴到输入框。
![[attachments/Pasted image 20221026201036.png]]

## 配置测试
### 查看 R2 发出的剪枝数据包
**注意**：需要在第一次组播之前，对 R2 g0/0/0 接口抓包，否则可能抓取不到剪枝数据包。

**启动组播**
在 MCS 组播服务器点击“运行”，或对设备右键点击“播放”，如果出现 VLC 播放器窗口播放指定视频，则表示组播开始。

> 由于 PC-1 并没有在第一时间加入该 组播组观看视频，所以 R2 没有收到 IGMP 加入消息，于是 R2 会认为自己没有连接任 何组成员。因此，R2 在收到 R1 发送的组播数据包后，会向 R1 发送剪枝消息。R1 收到剪枝消息后，会立即停止向 R2 发送组播数据包。

在 R2 g0/0/0 接口出抓包结果如下图所示。
![[attachments/Pasted image 20221026200619.png]]
PIM 数据包类型为 Join/Prune，其中组播组 Group 0 字段中 Num Prunes == 1 表示 R2 下的网段可以在组播树中剪枝。

### 查看 R1 的组播路由表
正在组播过程中，R1 组播路由表如下。
![[attachments/Pasted image 20221026193057.png]]

### PC-1 加入组播
**停止组播**，关闭组播服务器的 VLC 窗口。

**配置 PC-1**
PC-1 配置如下图所示。点击“加入”，加入组播组。
![[attachments/Pasted image 20221026203601.png]]

在 R2 g0/0/0 接口出抓包如下图所示。
**Graft 数据包**

![[attachments/Pasted image 20221026203917.png]]

**Graft-Ack 数据包**
![[attachments/Pasted image 20221026204050.png]]

从 Graft 和 Graft-Ack 数据包中可以看到，R2 连接了 Group 0 中一个成员。两个数据包中的信息一致。

> Graft消息以单播方式被发送给多播树上的上游邻居。当上游路由器接收到该 Graft 消息后，会将接收到该消息的接口增加到其出口接口列表中，从而使该接口进入转发状态，并立即向其新下游邻居发送 Graft-ACK 消息。

重新在 MCS 服务器上**启动组播**，在 PC-1 上打开 VLC 可以看到同步播放的视频。其中显示播放源为 rtp://127.0.0.1:16694。
![[attachments/Pasted image 20221026204259.png]]

同时 PC-1 接口出抓取视频流的 UDP 报文如下图所示。
![[attachments/Pasted image 20221026204641.png]]


### 观察 PIM-DM 中 DR 的选举
**修改 R3、R4 接口的 IGMP 版本**
假定 PC-2 将使用 IGMPv1 加入组播组224.1.1.1，为此，在 R3 和 R4 上修改 IGMP 为版本 1。分别在 R3、R4 执行命令如下。
```
// R3
int g0/0/0
igmp version 1

// R4
int g0/0/0
igmp version 1
```

**查看 R3、R4 接口 PIM 配置**
![[attachments/Pasted image 20221026210551.png]]
![[attachments/Pasted image 20221026210605.png]]
> 当一个网段内有多个组播路由器时，由于它们都可以接收到主机发送的成员报告报文，因此只需要选取其中一台组播路由器发送查询报文就足够了，该组播路由器称为IGMP查询器（Querier）。在IGMPv1中，由组播路由协议 PIM 选举出唯一的组播信息转发者（Assert Winner或DR）作为IGMPv1的查询器，负责该网段的组成员关系查询。
> 
> **DR 竞选规则**：
>    1. DR优先级较高者获胜（网段中所有PIM路由器都支持DR优先级）。
>    2. 如果DR优先级相同或该网段存在至少一台PIM路由器不支持在Hello报文中携带DR优先级，则IP地址较大者获胜。

由于 R3、R4 同属一个网段 20.0.2.0/24，需要选出该网段下的唯一的组播信息转发者，根据转发规则二者拥有更大 IP 地址接口的 **R4 被选为 DR**。

**查看 R3、R4 IGMP 配置**
![[attachments/Pasted image 20221026211647.png]]
![[attachments/Pasted image 20221026212107.png]]
从 IGMP 配置中可以看到 IGMP Querier 查询器即 PIM-DM 选举出的 DR 路由器 R4。

### 观察 PIM-DM 中的 Assert 机制

> **Assert 机制说明**
> 由于 R3 和 R4 从上游接收到组播报文后，都会向下游网络转发该组播报文，这样 就会导致下游节点 R5 收到两份完全相同的组播报文。为了避免这种情况的发生、PIM-DM 采用了 Assert 机制来选定一个唯一的转发者，即：对于一个特定的组播组，如果同 一网段上存在多个上游路由器，则这些上游路由器中到组播源的路径开销最小者将被选 举为转发者；**若开销相同，则 IP 地址最大的路由器将被选举为转发者**。只有转发者才 能向该网段转发相应的组播数据报文，其他落选路由器应裁剪掉对应的接口，禁止向该 网段转发相应的组播数据报文。

**重启拓扑，先监听 R5 g0/0/0 接口，再开始组播**，因为 Assert 数据包仅在组播路由器第一次收到重复组播报文时发出。

**R3 发出的 Assert 数据包**
在 R5 g0/0/0 处抓包结果如图下。
![[attachments/Pasted image 20221027154125.png]]

**R4 发向 R5 的视频流**
![[attachments/Pasted image 20221027153344.png]]

**R5 的多播路由**
如下图所示。从图中可以看到 R5 的上游节点是 20.0.2.254 接口的所有者 R4。因为根据 Assert 规则，R5 成为该网段内的唯一组播数据报转发者，所以应当作为 R5 的上游节点。
![[attachments/Pasted image 20221027153521.png]]


# 解答题
1. PIM-DM 中 Assert 机制的作用是什么？ 
    为了避免在一个网段内多个上游路由器转发相同的组播报文。

1. IGMP 是一个路由协议吗？ 
    不是。IGMP 协议的作用是“在接收者主机和与其直接相邻的组播路由器之间建立和维护组播组成员关系”，IGMP 只关心组播路由器之间的关联关系。一个组播路由器可以通过 IGMP 协议想要自己需要转发到哪些接口，需要依靠路由协议得到的路由表决定，IGMP 协议并不能维护路由器到组播成员的路由信息。

2. 给出两层 IGMP Snooping 与三层组播协议的区别和关联点。
    IGMP Snooping，它是运行在二层设备上，通过侦听三层设备和主机之间的 IGMP 报文来生成二层组播转发表，从而管理和控制组播数据报文的转发，实现组播数据报文在二层的按需分发。根据组播 MAC 地址来决定数据帧的转发方向
    
    三层组播协议包括组播组管理协议和组播路由协议，根据三层组播地址，即组播IP地址来进行数据帧的转发。