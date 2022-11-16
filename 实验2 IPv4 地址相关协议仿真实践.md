# VLAN 间路由的实现
## 网络拓扑
![[attachments/Pasted image 20220930174131.png]]

## 配置交换机 S1
### 配置 VLAN
执行以下命令。
```
sys
sy S1

# 创建两个 vlan 网段
vlan batch 10 20

# 配置 vlan10 网段 ip 地址
int vlanif 10
ip add 192.168.1.0 255.255.255.0

# 配置 vlan20 网段 ip 地址
int vlanif 20
ip add 192.168.2.0 255.255.255.0
```

### 配置接口
**1 配置 g0/0/1 接口**
执行以下命令。
```
int g0/0/1
port link-type access
port default vlan 10
```

**2 配置 g0/0/2 接口**
执行以下命令。
```
int g0/0/2
port link-type access
port default vlan 10
```

**3 配置 g0/0/3 接口**
执行以下命令。
```
int g0/0/3
port link-type access
port default vlan 20
```

## 配置主机
**1 配置主机 PC-1**
![[attachments/Pasted image 20220929101821.png]]

**2 配置主机 PC-2**
![[attachments/Pasted image 20220929101835.png]]

**3 配置主机 PC-3**
![[attachments/Pasted image 20220929101853.png]]

## 配置结果
**交换机配置**
![[attachments/Pasted image 20220929102617.png]]
![[attachments/Pasted image 20220929102715.png]]

## 测试总结
### PC-1 -> PC-2
**ping 成功截图**
![[attachments/Pasted image 20220929102928.png]]

### PC-1 -> PC-3
**ping 成功截图**
![[attachments/Pasted image 20220929103816.png]]

解答题：[交换机的工作过程](#问题：VLAN%20间路由中交换机的工作过程)？

# 基于全局地址池的 DHCP
## 网络拓扑
![[attachments/Pasted image 20220930174257.png]]

## 配置交换机 R1
### 配置地址池
配置地址池 huawei1，执行命令如下。
```
# 使能 DHCP 功能
dhcp enable
# 创建全局地址池
ip pool huawei1

# 配置客户端网关地址
getway-list 192.168.1.254

# 配置不参与自动分配的 IP 地址
excluded-ip-address 192.168.1.250 192.168.1.253

# 修改地址租期
lease day 2 hour 0 minute 0

# 配置地址池将静态指定的DNS配置信息分配给DHCP客户端
# 备注：实验中应该非必需，因为根本没有 DNS 服务器。
dns-list 8.8.8.8
```

配置地址池 huawei2，执行命令如下。
```
ip pool huawei2
gateway-list 192.168.2.254
# 指明地址池的可分配 IP 地址范围
network 192.168.2.0 mask 255.255.255.0
dns-list 8.8.8.8
```

> **配置客户端网关的作用**
> DHCP服务器上配置客户端的网关地址后，客户端会获取到服务器分配的网关地址，并自动生成到该网关地址的缺省路由。之后，客户端才能访问非本网段内的主机。
> 如果DHCP服务器与DHCP客户端处于同一网段，并且DHCP服务器作为客户端网关时，可以不配置此任务。
> 
>**分配 DNS 信息的作用**
>当管理员希望客户端的DNS配置通过DHCP方式动态获取时，可以执行此任务。
>自动获取要求作为DHCP服务器的设备同时作为DHCP客户端（连接远端DHCP服务器的接口配置DHCP客户端功能），**设备从远端DHCP服务器获取DNS服务器IP地址、DNS域名后缀和NetBIOS服务器IP地址配置信息之后，通过地址池的**import**功能，将这些信息再分配给下行的客户端**。例如，某公司的DHCP服务器希望从运营商获取统一的DNS服务器IP地址、DNS域名后缀和NetBIOS服务器IP地址配置信息，同时又将这些信息分配给下行的客户端，此时可以选择自动获取方式。

### 配置接口
**1 配置 g0/0/0 接口**
执行命令如下。
```
interface GigabitEthernet0/0/0
ip address 192.168.1.254 24
# 接口采用全局地址池的 DHCP 功能
dhcp select global
```


**2 配置 g0/0/1 接口**
执行命令如下。
```
interface GigabitEthernet0/0/1
ip address 192.168.2.254 24
dhcp select global
```

## 配置主机
主机全部选择 IPv4 中的 DHCP 功能，勾选自动获取 DNS 服务器地址。

## 测试总结
**PC-1 IP 配置情况**
由于地址池 huawei1 限制了可分配地址范围，PC-1 获得的地址是.249而非.253。
![[attachments/Pasted image 20220929151158.png]]

**PC-2 IP 配置情况**
![[attachments/Pasted image 20220929151225.png]]

可见两台主机都自动获取到了 IP 地址。

**地址池配置情况**
![[attachments/Pasted image 20220929151413.png]]

# 基于接口地址池的 DHCP
## 网络拓扑
![[attachments/Pasted image 20220930174330.png]]

## 配置路由器 R1
执行命令如下。
```
# 使能 DHCP 功能
dhcp enable
```

### 配置接口
**1 配置 g0/0/0 接口**
执行命令如下。
```
int g0/0/0
ip add 192.168.1.254 24

# 配置接口采用接口地址池的 DHCP 服务器功能
dhcp select interface

# 配置不参与自动分配的 IP 地址
dhcp sever excluded-ip-address 192.168.1.1 192.168.1.10

dhcp server lease day 2 hour 0 minute 0
```

**2 配置 g0/0/1 接口**
执行命令如下。
```
interface GigabitEthernet0/0/1
ip address 192.168.2.254 24
dhcp select interface
dhcp server dns-list 8.8.8.8
```

## 配置主机
主机全部选择 IPv4 中的 DHCP 功能，勾选自动获取 DNS 服务器地址

## 测试总结
**PC-1 IP 配置**
![[attachments/Pasted image 20220929145115.png]]

**PC-2 IP 配置**
![[attachments/Pasted image 20220929145256.png]]

**接口地址池配置**
![[attachments/Pasted image 20220929145900.png]]

**解答题：[DHCP 两种地址分配模式的不同？](#问题2：DHCP%20两种地址分配模式的不同)**

# NAT 协议仿真
## 前置知识
Basic NAT 是实现一对一的IP地址转换，而 NAPT 可以实现多个私有IP地址映射到同一个公有IP地址上。由于Basic NAT这种一对一的转换方式并未实现公网地址的复用，不能有效解决IP地址短缺的问题，因此在实际应用中并不常用。

## 网络拓扑
【注意】图中实验示例拓扑和原始拓扑的子网划分不一样，注意 IP 不要配错了。下图为实验示例拓扑。
![[attachments/Pasted image 20220929165048.png]]

## 配置路由器 R1
### 配置 NAT
```
# 新建一个 ACL 规则
acl number 2001
rule 5 permit source 20.1.1.0 0.0.0.255
# 配置公网 IP 地址池
nat address-group 1 202.169.10.50 202.169.10.60
```

**ACL 规则是什么？**

> [华为路由器配置指南-安全-ACL配置](https://support.huawei.com/enterprise/zh/doc/EDOC1100033978/9aca1495)
> 
> 访问控制列表ACL（Access Control List）是由一条或多条规则组成的集合。所谓规则，是指描述报文匹配条件的判断语句，这些条件可以是报文的源地址、目的地址、端口号等。
>
> ACL本质上是一种报文过滤器，规则是过滤器的滤芯。设备基于这些规则进行报文匹配，可以过滤出特定的报文，并根据应用ACL的业务模块的处理策略来允许或阻止该报文通过。

根据文档中的介绍，按照我的理解———— ACL 就是一个规则集，通过设定一系列规则，让设备可以根据规则判断报文的处理方式。

**配置了什么 ACL 规则？**

实验中创建了一个编号为 2001 的基本 ACL。

> **为什么是2001？**
> 
> 这里涉及 ACL 的分类，不同类型的 ACL 可以根据不一样的信息来制定规则。具体参考[ACL的分类](https://support.huawei.com/enterprise/zh/doc/EDOC1100033978/4ae8136)。
> ![[attachments/Pasted image 20220929203522.png]]

然后制定了一个5号规则：允许来自 20.1.1.0/24 网段的报文通过。

更多规则的配置方法可参考[ACL的常用匹配项](https://support.huawei.com/enterprise/zh/doc/EDOC1100033978/7a1ffd8b)。

*根据[华为配置文档](https://support.huawei.com/enterprise/zh/doc/EDOC1100033970/b5466a66)中介绍，仅当 ACL 的 rule 配置为 permit 时，设备允许匹配该规则中指定的源IP地址使用地址池进行地址转换。【存疑】如果不配置 ACL 似乎不能使用 NAT 完成动态地址转换。*

### 配置默认路由
执行命令如下。
```
# 配置默认路由，下一跳 IP 地址是外网
ip route-static 0.0.0.0 0.0.0.0 202.169.10.2
```


### 配置接口
**1 配置 g0/0/0 接口**

g0/0/0 接口是 NAT 的出接口，执行命令如下。
```
interface GigabitEthernet0/0/0
ip address 202.169.10.1 255.255.255.0

# 使能 arp 代理（无作用）
arp-proxy enable

# 配置静态地址转换（10.1.1.1主机不存在，无作用）
nat static global 202.169.10.5 inside 10.1.1.1 netmask 255.255.255.255

# 配置带地址池的NAT Outbound，用于动态地址转换
nat outbound 2001 address-group 1 no-pat
```

**那么，上述命令配置了那些 NAT 相关信息呢？**

>[华为路由器配置指南-NAT业务](https://support.huawei.com/enterprise/zh/doc/EDOC1100033970/c2415af0)
>
>**NAT Outbound 方式**：该方式所用的地址池是用来存放动态NAT使用到的IP地址的集合，在做动态NAT时会选择地址池中的某个地址用做地址转换。

根据上述文字可以看出，实验中使用的是 NAT Outbound 方式配置该接口的公网地址限定在地址池 Group 1 的范围，同时允许转发的报文需要遵循 ACL 2001 的规则。

之后通过该接口的报文将可以转换为公网 IP 访问外网啦。

**小问题：为什么使能 ARP 代理没有用？**

根据[配置文档](https://support.huawei.com/enterprise/zh/doc/EDOC1100069360/84487237)，华为路由器仅支持“属于同一网段但不同物理网络的主机间”用 arp 代理，与一般的 arp 代理定义有区别。但是实验不存在这种情况的两台主机。

那么也就是说，三台主机还是得配置默认网关才行。

**2 配置 g0/0/1 和 g0/0/2 接口**
执行命令如下。
```
interface GigabitEthernet0/0/1
ip address 10.1.1.254 255.255.255.0
interface GigabitEthernet0/0/2
ip address 20.1.1.254 255.255.255.0
```

## 配置路由器 R2
执行命令如下。
```
snmp-agent local-engineid 800007DB03000000000000
snmp-agent
interface GigabitEthernet0/0/0
ip address 202.169.10.2 255.255.255.0

# 配置环回测试接口
interface LoopBack0
ip address 202.169.20.1 255.255.255.0
```

## 配置主机
**配置主机 PC-1**
![[attachments/Pasted image 20220929220117.png]]

**配置主机 PC-2**
![[attachments/Pasted image 20220929220151.png]]

**配置主机 PC-3**
![[attachments/Pasted image 20220929220209.png]]

## 测试总结
**PC-1 -> Server 成功**
![[attachments/Pasted image 20220929222000.png]]

**PC-1 -> Internet 失败**
无法 ping 通，因为设置了 ACL 规则。
![[attachments/Pasted image 20220929221844.png]]

**PC-2 -> Server 成功**
![[attachments/Pasted image 20220929222048.png]]

**PC-2 -> Internet 成功**
![[attachments/Pasted image 20220929222129.png]]

**修改配置，使得 R1 支持 NAPT 协议**

执行命令如下。

```
int g0/0/0
undo nat outbound 2001 address-group 1 no-pat
nat outbound 2001 address-group 1
```

保存重启拓扑后即可测试。

**解析并比较 NAT 和 NAPT 下数据分组的异同**

NAT 协议下的端口映射如下图所示。
![[attachments/Pasted image 20220930164654.png]]
可见 PC-2 每次用不同的端口向 R2 发包时，主机的端口会映射到不同的公网 IP 的相同端口。

NAPT 协议下的端口映射如下图所示。
![[attachments/Pasted image 20220930165418.png]]
可见 PC-2 每次用不同的端口向 R2 发包时，主机的端口会映射到相同的公网 IP 的不同端口。

**解答题：[NAT 和 NAPT 协议细节？](#问题3：NAT%20和%20NAPT%20协议细节)**


# 解答题
## 问题1：VLAN 间路由中交换机的工作过程
**1 同一网段下工作过程**
**g0/0/2 接口抓包**
![[attachments/Pasted image 20220929104231.png]]

**同一网段下通信，交换机工作过程：**

两台主机处在同一局域网下，如上图抓包结果显示，在 ARP 广播中可以看到 PC-2 的 MAC 地址需要回复给 PC-1，即 PC-2 可以收到来自 PC-1 的 ARP 报文。

后续 ICMP 报文可直接封装在以太网帧中，而 PC-1 可以直接在以太网帧的 Destination 字段写入 PC-2 的 MAC 地址。

**2 不同网段下工作过程**
**g0/0/3 接口信息**
![[attachments/Pasted image 20220929105201.png]]

**g0/0/3 接口抓包**
![[attachments/Pasted image 20220929110614.png]]

**不同网段下通信，交换机工作过程：**

两台主机处在不同局域网下，如上图抓包结果显示，在 ARP 广播中 PC-2 的 MAC 地址需要回复到g0/0/3接口，即PC-2不可以收到来自PC-1的ARP报文。

可见 PC-1 的 ARP 广播要找的是 g0/0/1 的接口 MAC 地址，相当于 PC-1 认为 192.168.2.1 不在同一局域网下，那么直接找默认网关转发后续的 ICMP 报文。

后续 ICMP 报文可直接封装在以太网帧中，而 PC-1 可以直接在以太网帧的 Destination 字段写入 g0/0/1 的 MAC 地址。



## 问题2：DHCP 两种地址分配模式的不同
DHCP 分别有全局模式、接口模式两种地址分配方式。

全局模式是在系统视图下配置地址池。设置地址池后需要根据给接口配置合适的 IP 地址，此后接口下的主机尽量分配处于同一网段下的 IP 地址。如果接口未配置IP地址，或者没有和接口地址在相同网段的地址池，客户端无法成功申请 IP 地址。

接口模式是在接口视图下配置地址池。接口下的主机直接按照接口地址池分配地址即可。


## 问题3：NAT 和 NAPT 协议细节
**对 NAT 和 NAPT 协议的细节如对外地址循环分配、重新分配、同一 IP 的分 组首部标识变化、端口号变化等理解。**

NAT 协议下，对地址池采用循环分配的方法，但是当地址池用尽的时候，只能通过回收长时间未使用的公网 IP 的方法来解决。在转发过程中，直接替换 IP 数据报首部的源地址后转发，传输层的端口号不变，并且不同端口对应的公网 IP 不同。

NAPT 协议下，针对 TCP 和 UDP 报文作出了优化，它通过“主机 + 协议 + 端口号”组合作为内网和外网的映射。地址充足的情况下，主机发出的相同传输层协议的报文会被映射到相同的“公网 IP + 随机端口号”。但是当地址不足时，NAPT 协议采用复用公网 IP 但改变端口号的方式完成映射。

当地址池只有1个地址时，不同主机的报文映射情况如下图所示。
![[attachments/Pasted image 20220930171619.png]]

综上来说，NAT 协议本质上并没有改变公网 IP 不足的问题，只是采用了类似分时复用的方法处理内外网映射。NAPT 协议不改变公网 IP 但改变端口的方式解决了公网 IP 不足的问题，但是仅支持 TCP 和 UDP 协议，同时是一种利用传输层的补救措施。