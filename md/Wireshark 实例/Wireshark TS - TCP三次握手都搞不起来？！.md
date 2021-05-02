# 问题描述
某年某月某日， 业务中心反馈外单位至我司业务系统访问不通，初步沟通了下，反馈我司业务系统检查正常，外单位客户端 ping 我司业务系统服务器正常，该系统对接其他外单位业务均正常，怀疑是网络故障。网络经初步内部核查后，确认网络正常，且故障开始发生的时间段并未做过网络变更。
至此聊不出个所以然，进入问题实际分析。


# 问题处理
定义问题、收集信息、抓包分析三板斧，在问题较明确的情况下，收集系统、网络等相关信息后，大概整理了以下拓扑，且同时业务补充反馈客户端访问测试系统也有同样问题。
![未命名文件.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1616599015078-35fd885a-3538-4907-acfd-046c780011ca.png#align=left&display=inline&height=910&margin=%5Bobject%20Object%5D&name=%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.png&originHeight=910&originWidth=516&size=50003&status=done&style=none&width=516)
> 1. 广域网双路由器冗余对接外单位；
> 1. 外部BGP对接，内部OSPF。



本着 **数据包分析 - 我们无法解决看不到的问题**  的原则，进一步抓包验证所说的问题，主要如下

1. Ping 包分析（_客户端抓包_）

_![icmp.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1616591897997-b4f84634-7e5d-4c89-a639-5340a4801723.png#align=left&display=inline&height=219&margin=%5Bobject%20Object%5D&name=icmp.png&originHeight=219&originWidth=1576&size=43759&status=done&style=none&width=1576)_
确实 Ping 来回正常。


2. 业务交互数据包分析（_SW1 和 SW2 镜像流量抓包_）

![截图_20210324213118.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1616592764910-913a0a28-d7f3-4708-974c-450dfb560e0c.png#align=left&display=inline&height=221&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%9B%BE_20210324213118.png&originHeight=221&originWidth=1550&size=45512&status=done&style=none&width=1550)
通过上图，很明显的问题原因就是 **TCP三次握手都没有完成！**，自然业务无法正常通讯。
> 请忽略 Time 列，数据包修改器匿名化处理时自动加的时间。



# 问题分析
故障现象：
客户端在发送 SYN，服务器也响应 SYN/ACK 的情况下，客户端未能回应 ACK，即 TCP 三次握手中的第三个包。 

故障分析：

- 客户端没有收到 SYN/ACK：SYN/ACK 包在返回过程中，在某个节点上被拒绝了，该包未能到达客户端，客户端在1秒后发生 SYN 重传，为可能原因；
- 客户端收到 SYN/ACK 但是无法处理：如果客户端因为性能不足，比如 CPU 利用率高或者连接满等，那么在处理性能不足时，后续请求会进入队列等待。但由于客户端有能力在 1 秒后重新发起 SYN 连接，故可以排除。
- 客户端收到 SYN/ACK ，返回 ACK，但服务端没收到：此种情况不符合现象，持续抓包一次也没抓到过 ACK，且如果客户端收到 SYN/ACK 后会进入 ESTABLISHED 状态，认为连接已建立，会立即发送数据，而数据包中客户端并无后续的数据，故可以排除。



基本可以确认是存在来回路径不一致的现象，通过了 NAT/防火墙的不同节点，造成了丢包。基于拓扑，正好是双节点对接处，进一步分析数据包，可以看到来回数据包的 mac 地址并不一样，一个是 R1 的内网口 mac 地址，一个是 R2 的内网口 mac 地址。
![截图_20210325154715.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1616658439500-57d3f007-76de-4c92-949a-1ad5670bc6bc.png#align=left&display=inline&height=138&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%9B%BE_20210325154715.png&originHeight=138&originWidth=1998&size=36819&status=done&style=none&width=1998)


故障定位：
检查 R1R2 路由，可发现对端客户端的路由此时从 R2 BGP 学来，因为与外联单位对接基本上会存在 NAT/防火墙的节点，所以来回路径不一致造成了业务通讯问题。

# 后续处理
原因反馈给业务，后交由外单位网络检查后确认对端路由 BGP 发布问题，本来应由 R1 处学来的路由，从 R2 处学来，造成了来回路径不一致的通讯问题。最终对端外单位调整 BGP 路由后，业务恢复正常。
