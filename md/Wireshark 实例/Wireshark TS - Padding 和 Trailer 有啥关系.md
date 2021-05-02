# 背景
惯例先说下背景，最起先是来自于一个 ARP 数据包分析，单说起 ARP 数据包组成是比较简单，但如下图所示，还是有些 Wireshark 技术点可以琢磨下，像是 Padding 和 Trailer 区别、相应的字节数大小、FCS 问题等等。

<br/>

![截图_20210227203603.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1614429377326-ac3df82d-34a4-4c4e-9c93-8885fde69445.png#align=left&display=inline&height=215&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%9B%BE_20210227203603.png&originHeight=215&originWidth=729&size=12764&status=done&style=none&width=729)

前期也查询过很多资料或是网上咨询，但一直还是有些疑问，刚好最近和 NPM 厂商的兄弟相互讨论了下，感觉对此又有了更深的理解。

<br/>

# 分析
先从最简单的 ARP 数据包说起，以前的文章提到过：
> 14 字节 ( Ethernet II 首部长度 ) + **46 字节** ( 数据字段最小长度要求 ) + 4 字节 ( FCS ) = 64 字节
> 28 字节 ( ARP 请求或应答 ) + 18 字节 (** Padding 填充数据** ) = 46 字节

因此 Padding 数据部分为 18 字节长度，如下图所示。

![](https://cdn.nlark.com/yuque/0/2021/png/2777842/1614432848675-e7ec177b-081c-42cc-afb5-f90b4fd53729.png#align=left&display=inline&height=470&margin=%5Bobject%20Object%5D&originHeight=470&originWidth=946&size=0&status=done&style=none&width=946)

回到问题图：

![截图_20210227203603.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1614429377326-ac3df82d-34a4-4c4e-9c93-8885fde69445.png#align=left&display=inline&height=215&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%9B%BE_20210227203603.png&originHeight=215&originWidth=729&size=12764&status=done&style=none&width=729)

因含有 802.1Q 字段的缘故，判断数据包取自 trunk 端口，而 802.1Q 字段为 4 字节长度，考虑到以太网帧 60 字节最小长度，Wireshark 解析会认为全 0 连续填充 (Padding) 的长度只需要 14 字节即可达到 18 字节的需求，即：
> 14 字节 ( Ethernet II 首部长度 ) + 28 字节 ( ARP 请求或应答 ) + 4 字节（ 802.1Q ）+ 14 字节 (** **Padding 填充数据 )  = 60 字节

而额外多出的数据标记成 Trailer（4字节）。

<br/>

# 原理
在 wireshark_dissector_packet-eth.c 中部分说明如下：
> - padding to meet the minimum 64 byte frame length.
> - Add an Ethernet trailer - which, for some captures, might be the FCS rather than a pad-to-60-bytes trailer.
> - Add the padding to the tree, unless it should be treated as part of the trailer and therefor be handed over to (one of) the ethernet-trailer dissectors.
> - There can not have been padding when the length of the frame (including the trailer) is less than 60 bytes.
> - any frame that big needs no trailer, as there's no need to pad an Ethernet packet past 60 bytes.
> - Calculate the amount of padding needed for a minimum sized frame.

可以认为 **Trailer 是包含 Padding 的**，什么时候解析成 Padding ？一般是当数据帧长度小于 60 字节，需要进行全 0 填充以达到 60 字节最小长度时的部分会被认为是 Padding。
**而额外多出的部分，需结合 FCS （4字节）判断有无的情况下，确认是否属于 Trailer **。再加上 Wireshark 强大的协议解析能力（ethernet-trailer dissectors），再确认属于哪一种 Trailer.
> - Call all ethernet trailer dissectors to dissect the trailer if we actually have a trailer.
> - No luck with the trailer dissectors, so just display the extra bytes as general trailer



Wireshark 解析器协议 Ethernet 部分，其中有三个相关选项：

1. **Assume padding for short frames with trailer（ Never、Zeros、Any）**

_Never_ 选项是不考虑探测任何 Padding，在以太网 payload 之后的任何字节都被视为 Trailer。
_Zeros (默认) _选项是为以太网帧最小长度连续填充全 0 字节的部分被视做 Padding，之外增加的部分被视为 Trailer.
_Any_ 选项是为以太网帧最小长度填充任意字节的部分被视做 Padding，之外增加的部分被视为 Trailer.
**

2. **Fixed ethernet trailer length**


默认值 0 。考虑到 TAP 或是类似负载均衡的代理设备，可能在 Payload 之后 FCS 之前增加固定长度的 trailer（像是高精度时间戳等等）时，可调整该值进行具体分析。
> If there're some bytes left over, it could be a combination of:
> - padding to meet the minimum 64 byte frame length
> - an FCS, if present
> - information inserted by TAPs or other network monitoring equipment.

**

3. **Assume packets have FCS**

默认不勾选。一般网卡都会剥离 FCS，所以 Wireshark 数据包分析时一般都没有 FCS 部分。可能存在 DPDK 技术下所抓取的数据包包含 FCS 的情况下，分析时可勾选打开该选项。

<br/>

# 参考
wireshark_dissector_packet-eth.c
wireshark_dissectorpacket-ethertype.c


