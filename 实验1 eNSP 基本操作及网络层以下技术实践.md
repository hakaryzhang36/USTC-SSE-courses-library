# IP 相关命令

## 配置路由器接口 IP

**1 配置 g0/0/0 接口**

执行命令如下。

```text
# 进入系统视图
sys
# 设置主机名称
sy R1
# 进入接口视图
int g0/0/0
# 配置 ip
ip address 10.0.1.254 255.255.255.0

```

输出结果如下。

![](attachments/image_JgoH2IId9n.png)

**2 同理配置 g0/0/1 接口。**

输出结果如下。

![](attachments/image_0fSv24ZWvP.png)

## 查看接口与 IP 摘要信息

**1 查看接口配置**

执行命令如下。

```text
display ip interface brief
```

![](attachments/image_ti8wB4v79I.png)

**2 查看路由表信息**

执行命令如下。

```text
display ip routing-table
```

![](attachments/image_bf_TVa8tpf.png)

# WAN 接入配置

## 网络拓扑

根据参考配置修改连线。

![](attachments/image_c0SpIYvpHM.png)

## 配置路由器 R1

### 配置接口

**1 配置 Ethernet 0/0/1 接口**

尝试配置 R1 的 Ethernet 0/0/1 接口 IP。

```纯文本
int e0/0/1
ip address 10.0.1.254 255.255.255.0
```

但是发现不能配置。[查阅资料](https://support.huawei.com/enterprise/zh/doc/EDOC1000174016/d254c495 "查阅资料")后知道，**路由器 AR1220 的 Ethernet 接口不全是三层接口，e0/0/1就是一个二层口（不能提供网络层功能和服务）**，我的理解是连上的这个接口的 PC 就相当与连上了一个交换机接口，而不是路由器，所以不能配置 IP。

> 基于接口板的硬件构造，某些形态设备上接口只能作为二层以太网接口，某些形态设备上接口只能作为三层以太网接口，而还有一些接口则比较灵活，可以改变其二三层模式：在二层模式下，该接口作为一个二层以太网接口使用；在三层模式下，该接口作为一个三层以太网接口使用。
>
> AR200系列、AR1220、AR1220V、AR1220W、AR1220VW和AR1220F支持将接口Eth0/0/0～Eth0/0/7从二层模式切换到三层模式。【存疑】

解决办法：

*   方法一：**通过建立 VLAN 实现 E0/0/1 接口的局域网 IP 配置**。[VLAN 基础知识](https://zhuanlan.zhihu.com/p/35616289 "VLAN 基础知识")。

*   方法二：将以太网接口切换到三层模式，根据[华为官方配置文档](https://support.huawei.com/enterprise/zh/doc/EDOC1000174016/d254c495 "华为官方配置文档")。

    *【存疑】我曾试图在AR1220路由器中，尝试将 e0/0/1 接口转换为三层模式，但是失败。*

- 方法三：更换接口

此处先用方法一解决，执行命令如下。

```纯文本
# 进入 vlan10 接口
vlan 10
int vlanif 10

# 配置 vlan IP
ip add 172.16.2.254 24

# 将 e0/0/1 接口划分进 vlan 10
int e0/0/1
port link-type access
port default vlan 10
```


**2 配置 Serial 1/0/0 和 Serial 1/0/1 接口**

执行命令如下。

```纯文本
# 配置 s1/0/0 接口，协议为 PPP
int s1/0/0
link-protocol ppp
ip address 192.168.1.2 255.255.255.0

# 配置 s1/0/1 接口，协议为 HDLC
int s1/0/1
link-protocol hdlc
ip address 192.168.2.2 255.255.255.0

```


### 配置静态路由

在系统视图，执行命令如下。

```纯文本
ip route-static 172.16.1.0 255.255.255.0 192.168.1.1
ip route-static 172.16.3.0 255.255.255.0 Serial1/0/1

```

### 配置PPPoe拨号规则（好像不是必需）

限制只有ip流量可以触发拨号的条件，执行命令如下。

```纯文本
dialer-rule
dialer-rule 1 ip permit
```

### 配置结果

**接口 IP**

![](attachments/image_RP9J2q9Wbn.png)

![[attachments/Pasted image 20220928150705.png]]

**静态路由与 Dialer 规则**

![](attachments/image_RmR6M9HA31.png)

## 配置路由器 R2

### 配置接口

**1 配置 Ethernet 2/0/1 接口**

参考配置中用到的 e0/0/1 接口为 Switch Port（二层接口），我试图将它切换为三层接口，但是失败了。故不得已改用 e2/0/1 三层接口，拓扑中默认也是这个接口。

执行命令如下。

```纯文本
sys
int e2/0/1
ip add 172.16.1.254 24
```

**2 配置 Serial 1/0/0 接口**

执行命令如下。

```纯文本
int s1/0/0  
link-protocol ppp
ip address 192.168.1.1 24
```

### 配置静态路由

执行命令如下。

```纯文本
# 配置默认路由，下一跳发往 R1
ip route-static 0.0.0.0 0.0.0.0 192.168.1.2

```

### 配置结果

**接口 IP**

![](attachments/image_UaqO11K_ML.png)

**路由表**

![](attachments/image_qMlAU5r7g3.png)

## 配置路由器 R3

### 配置接口

**1 配置 Ethernet 2/0/1 接口**

与 R2 类似使用改接口方式。

执行命令如下。

```纯文本
sys
int e2/0/1
ip add 172.16.3.254 24
```

**2 配置 Serial 1/0/1 接口**

执行命令如下。

```纯文本
int s1/0/1 
link-protocol hdlc
ip address 192.168.2.1 24
```

### 配置静态路由

执行命令如下。

```纯文本
# 默认路由，下一跳发往 R1
ip route-static 0.0.0.0 0.0.0.0 S1/0/1

```

### 配置结果

**接口 IP**

![](attachments/image_gezZlDFbQv.png)

**路由表**

![](attachments/image_D7tj0k4rgR.png)

## 配置 PC

### 配置 PC-1

![](attachments/image_vo0oMBTOIX.png)

### 配置 PC-3

![](attachments/image_csuW1543so.png)

## 测试总结

对 R2 Serial1/0/0 接口和 R3 Serial1/0/1 接口进行抓包。

### PC-1 尝试 ping 通 PC-3

![](attachments/image_M6QgVuj0ii.png)

### PPP 协议抓包

R2 Serial1/0/0 接口数据链路层遵循 PPP 协议，抓包结果如下。

![](attachments/image_WhFS6ZwLjG.png)

### HDLC 协议抓包

R3 Serial1/0/1 接口数据链路层遵循 HDLC 协议，抓包结果如下。

![](attachments/image_OLFI_2akFT.png)

### 总结

**问：PPP 协议和 HDLC 协议的区别？**

答：首先，两者都是数据链路层为实现广域网而出现的协议。报文首部的区别：

- PPP 地址字段通常置为 0xff（无意义），HDLC 地址字段 0x0f 同样无意义；
- PPP 控制字段通常置为 0x03，HDLC 控制字段 0x00 表示帧类型为信息帧；
- PPP 协议字段 0x0021 表示数据段为 IP 数据报，HDLC 协议字段则用 0x0800 表示。

# PPP 认证

**为什么要认证？**

PPP 认证为端到端的网络连接提供了安全性保证。以下为[华为官方文档](https://support.huawei.com/enterprise/zh/doc/EDOC1100033737/6c7d0558 "华为官方文档")关于 PPP 认证的介绍。

PPP支持PAP和CHAP认证，CHAP认证是Server端先发起challenge，PAP认证是Client端直接发起认证。Client端和Server端认证方式一致才能认证成功。PAP认证时，密码会在网络中以明文形式传输，存在安全风险。**推荐使用CHAP认证。**

**CHAP认证过程**分为两种情况：认证方配置了用户名和认证方没有配置用户名。推荐使用认证方配置用户名的方式，这样可以对认证方的用户名进行确认。

如图所示，用户希望RouterA对RouterB进行可靠的认证，且对安全的要求较高，只需配置RouterA作为认证方使用CHAP方式认证被认证方RouterB。

![](attachments/image_atzrleWZLw.png)

**根据本次实验的参考配置，使用 CHAP 认证中认证方配置用户名的方式**。

## 网络拓扑
![](attachments/image_XWkp7POPcg.png)

## 配置路由器 R1（被认证方）

### 配置接口

**1 配置 Serial 4/0/0 接口（PPP 协议）**

CHAP 被认证方只需在接口处配置用户名和密码，执行命令如下。

```纯文本
int s4/0/0
link-protocol ppp
# 配置用户名 r1 和密码 Huawei
ppp chap user r1
ppp chap password cipher Huawei
# 配置 IP
ip add 10.0.13.1 24
```

**2 配置 GE 0/0/0 接口**

执行命令如下。

```纯文本
int g0/0/0
ip add 10/0/1/254 24
```

### 配置路由协议

根据参考配置，采用 OSPF 路由协议，执行命令如下。

```纯文本
ospf 1
area 0.0.0.0
# 配置该路由器连接的两个网段
network 10.0.1.0 0.0.0.255
network 10.0.13.0 0.0.0.255
```

### 配置结果

**接口 IP**

![[attachments/Pasted image 20220928153427.png]]

**OSPF 协议**

![[attachments/Pasted image 20220928153543.png]]

## 配置路由器 R3（认证方）

### 配置 aaa 本地认证

此处采用*缺省域下AAA本地认证的配置方式*，我认为是指在路由器全局配置 PPP 认证的用户名和密码。执行命令如下。

```纯文本
aaa
local-user r1 password cipher Huawei
local-user r1 service-type ppp

```

### 配置接口

**1 配置 Serial 4/0/0（PPP 协议）**

执行命令如下。

```纯文本
link-protocol ppp
ip address 10.0.13.3 255.255.255.0
# 配置认证方式为 CHAP
ppp authentication-mode chap 

```

**2 配置 GE 0/0/0 接口**

执行命令如下。

```纯文本
int g0/0/0
ip address 10.0.23.3 255.255.255.0

```

### 配置路由协议

根据参考配置，采用 OSPF 路由协议，执行命令如下。

```纯文本
ospf 1
area 0.0.0.0
# 配置该路由器连接的两个网段
network 10.0.13.0 0.0.0.255
network 10.0.23.0 0.0.0.255
```

### 配置结果

**aaa 全局配置**

![[attachments/Pasted image 20220928153748.png]]

**接口 IP**

![[attachments/Pasted image 20220928153830.png]]

**OSPF 路由协议**

![[attachments/Pasted image 20220928153910.png]]

## 配置路由器 R2

### 配置接口

**1 配置 GE 0/0/0 接口**

执行命令如下。

```纯文本
int g0/0/0
ip address 10.0.23.2 24
```

**2 配置 GE 0/0/1 接口**

执行命令如下。

```纯文本
int g0/0/1
ip address 10.0.2.254 24
```

### 配置路由协议

根据参考配置，采用 OSPF 路由协议，执行命令如下。

```纯文本
ospf 1 
area 0.0.0.0 
network 10.0.2.0 0.0.0.255 
network 10.0.23.0 0.0.0.255 
```

### 配置结果

**接口 IP**

![[attachments/Pasted image 20220928153948.png]]

**OSPF 路由协议**

![[attachments/Pasted image 20220928154021.png]]

## 测试总结

### PC-1 尝试 ping 通 PC-3

根据实验指南的配置一通操作，居然没有任何问题就 ping 通了，着实神奇。比起 Part 2 的 WAN 实验好多了。

![](attachments/image_fzqrrcTD8d.png)

### Challenge 报文抓包

![](attachments/image_71bQtiQANe.png)

### Response 报文抓包

![](attachments/image_Lar7wWi-bG.png)

### Success 报文抓包

Success 报文

![](attachments/image_w_ZqLT_y1I.png)

Failure 报文

![](attachments/image_xED2POwEJ5.png)

### 总结

CHAP 认证过程如下图所示。

![](attachments/image_1vs5u6OZ9n.png)

**问：PPP 认证过程中的定时请求和回应是如何进行的？**

实验中采用的是 CHAP 认证方式。从 Wireshark 抓包结果来看，CHAP 报文通过 Code 字段区分报文类型。

首先由认证方发起 Challenge 报文，当中携带一串随机数据字段 value。

当被认证方接收到 Challenge 报文后回复 Response 报文，当中携带用户名字段 name 和密码字段 value，其中用户名通过明文传输而密码经过 Hash 加密后传输。

认证方收到 Response 报文后，通过验证用户名和密码的结果，向被认证方发送 Success 或 Failure 报文。

# 实验中遇到的问题

- **VRP Console 超时休眠，一直输出#####。**

解决办法如下。

```text
# 进入console唯一端口
[R1]user-interface console 0 
# 设置自动退出超时时间（分钟），0为永久
[R1：ei-ui-console0]idle-timeout 0
```

- **参考配置中用到的 e0/0/1 接口为 Switch Port（二层接口），我试图将它切换为三层接口，但是失败了。**

- **R2 路由表中，192.168.1.0/24 的下一跳地址是路由器自身接口Serial1/0/0 的 IP 地址？不应是下一跳路由器接口的 IP 地址吗？**

![](attachments/image_wT2PYiJW29.png)
