# 计算机本地端口网络配置及查看
网卡接口、路由表如下图所示。
![[attachments/Pasted image 20221004153435.png]]

路由表各项代表含义：
- 0.0.0.0，默认路由。网关是 WLAN 下的 AP 地址，直接给出。接口为 WiFi 网卡分配得到的地址。
- 114.214.233.255/32，WLAN 下的广播地址。
- 127.x.x.x，分别是环回接口的网段、单播、广播地址，下一跳为127.0.0.1。
- 192.168.56.x，是 VirualBox 虚拟网卡配置的网段，其中.1是该网卡手动配置的地址。
- 224.0.0.0，组播地址。
- 255.255.255.255，广播地址。

# ICMP 调试命令操作及协议解析
**ping 命令的工作原理：**

ping 命令是通过 ICMP 协议中的回送消息的查询报文验证目的地址是否可达的。源主机首先会构建一个 ICMP 回送请求消息数据包。type 字段为8表示 ping request 包。sequence number 字段表示包的序号，用于区分 ping 命令同时发送的多个 ICMP 包。
然后 ICMP 包封装在 IP 数据报中，这里的 type 字段为1表示数据段为 ICMP 协议包。

**tracert 命令的工作原理：**

tracert 命令通过 ICMP 报文的回复报文确定到目的地址的路由。该命令会重复发出到目的地址的 ICMP ping 报文，其中 TTL 字段从 1 开始依次递增。当 TTL 减到0时，中途节点会回复一个 ICMP Time-to-live exceeded 报文，主机就可根据该报文的源地址确定节点地址。

**ping 自带选项字示例**
- **-f，在数据包中设置“不分片”标记。**
执行 `ping -f baidu.com` 命令，wireshark 抓包结果如下图所示。
![[attachments/Pasted image 20221004202003.png]]

- **-r count，在选项字段中记录路由节点，count 为记录地址数。**
执行 `ping -r 7 ustc.edu` 命令，输出信息如下图所示。
![[attachments/Pasted image 20221004203020.png]]

主机发送的 request IP 数据报中，增加了 options 可选字段，其中 type 为7表示当中数据是 Record Route。地址初始化为 0.0.0.0。抓包结果如下图所示。
![[attachments/Pasted image 20221004203337.png]]

主机收到的 reply IP 数据报中，同样增加了 options 可选字段，地址记录了路由信息。抓包结果如下图所示。
![[attachments/Pasted image 20221004203747.png]]

- **-s count，记录每个跃点的 reply 报文的到达时间戳。**
执行 `ping -s 4 ustc.edu` 命令，输出信息如下图所示。
![[attachments/Pasted image 20221004204234.png]]

主机发送的 request IP 数据报中，增加了 options 可选字段，其中 type 为68表示当中数据是地址+时间戳。抓包结果如下图所示。
![[attachments/Pasted image 20221004205501.png]]

主机收到的 reply IP 数据报中，同样增加了 options 可选字段，记录了报文经过的前4个路由的地址和时间戳。抓包结果如下图所示。
![[attachments/Pasted image 20221004205615.png]]

# 路由配置及 ICMP 协议仿真
## 配置路由器
### 配置 Router A
**配置接口 IP 地址**
执行命令如下。
```
int g0/0/0
ip add 202.38.75.33 24

int g0/0/1
ip add 202.38.77.254 24
```

**配置静态路由**
```
ip route-static 202.38.73.0 24 202.38.75.254
ip route-static 202.38.74.0 24 202.38.75.254
```

### 配置 Router B
**配置接口 IP 地址**
执行命令如下。
```
int g0/0/0
ip add 202.38.76.253 24

int g0/0/1
ip add 202.38.73.254 24
```

**配置静态路由**
```
ip route-static 202.38.74.0 24 202.38.76.254
ip route-static 202.38.77.0 24 202.38.76.254
```

### 配置 Router C
**配置接口 IP 地址**
执行命令如下。
```
int g0/0/0
ip add 202.38.75.254 24

int g0/0/1
ip add 202.38.76.254 24

int g0/0/2
ip add 202.38.74.254 24
```

**配置静态路由**
```
ip route-static 202.38.73.0 24 202.38.74.254
ip route-static 202.38.77.0 24 202.38.75.33
```

## 配置主机
主机配置好后，需要先主动与网关 ping 通，这样路由器才能把主机 MAC 写入 ARP 表中。
**A1**
![[attachments/Pasted image 20221006150050.png]]

**A2**
![[attachments/Pasted image 20221006150139.png]]

**B1**
![[attachments/Pasted image 20221006150212.png]]

**C1**
![[attachments/Pasted image 20221006150236.png]]

**C2**
![[attachments/Pasted image 20221006150257.png]]

## 测试总结
### ping 命令
尝试从 A1 向 B1 发送 ICMP ping 报文。

**Route A g0/0/1 接口 IPv4 报文首部**
抓包结果如下图所示。
![[attachments/Pasted image 20221006154538.png]]

**Route C g0/0/0 接口 IPv4 报文首部**
抓包结果如下图所示。
![[attachments/Pasted image 20221006154744.png]]

后续接口处报文分组首部类似，唯一区别就是 TTL 字段经过一个节点都会减一。

## tracert 命令
尝试获得从 A1 到 B1 的路由。输出结果如下图所示。
![[attachments/Pasted image 20221006155529.png]]
前三个输出的 IP 地址是路由器上的报文接收接口地址，而从 IP 地址可以看出依次经过了A、C、B三个节点，最后到达 B1。

根据 tracert 的原理，它发送的报文会将 TTL 从0逐渐增加至到达目的地址的大小。当中途节点将 TTL 减到0的时候，会往源地址发送 Time-to-live exceeded 报文。

测试中 **Route A g0/0/1 接口 IPv4 报文首部** 确实依次接收到 TTL 为1-4的报文（每类报文重复发送了3次）。抓包如下图所示。
![[attachments/Pasted image 20221006172537.png]]
![[attachments/Pasted image 20221006172616.png]]