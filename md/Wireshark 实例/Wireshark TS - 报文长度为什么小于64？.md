# 问题背景
来自于知乎的几个问题，都是有关于 ARP 和 64 字节方面，主要如下：

1. [ARP报文长度为什么可以小于64？](https://www.zhihu.com/question/365469180)

2. [wireshark下抓取arp报文长度不全为42B？](https://www.zhihu.com/question/268902484)

3. [为什么Wireshark抓到的ARP包长度小于64?](https://www.zhihu.com/question/348389934)

<br/>

# 问题分析
以下就上述三个问题简单分析一下，首先是问题 1 和 2 ：


![](https://cdn.nlark.com/yuque/0/2021/png/2777842/1613568896725-4dcf6120-f149-4a1a-a155-b24883881d9a.png#align=left&display=inline&height=941&margin=%5Bobject%20Object%5D&originHeight=478&originWidth=720&size=0&status=done&style=none&width=1417)


_**为什么可以小于 64 ？**_
64 字节的说法，我想大家应该都知道是什么样的组成。
> 14 字节 ( Ethernet II 首部长度 ) + 46 字节 ( 数据字段最小长度要求 ) + 4 字节 ( CRC ) = 64 字节

因此数据字段的最小长度是 46 字节，这意味着如果是 ARP 数据包，则 46 字节的组成如下
> 28 字节 ( ARP 请求或应答 ) + 18 字节 ( Padding 填充数据 ) = 46 字节



**_为什么其中一个 ARP 报文长度显示是 42 ？_**
原因是 Wireshark 的抓包方式（或者说是原理，最后附简图，详细的可以看下官网说明）和位置，Wireshark 抓包位置如果是在本地，那么对于本地产生所发出的数据包，是在进网卡之前所抓取的包，而填充数据以及 CRC 一般是由网卡硬件/驱动程序完成，所以 42 字节的组成并不包含填充数据和 CRC 部分。
> 14 字节 ( Ethernet II 首部长度 ) + 28 字节 ( ARP 请求或应答 ) = 42 字节



**_为什么另一个 ARP 报文长度显示是 60 ？_**
综合以上，就很好理解了，60 是来自对方的 ARP 响应包，包含了填充数据 (对方网卡完成)，但不含 CRC 部分 (本地网卡剥离，可以说普通网卡基本都会剥离它，Wireshark 看不到 CRC 部分)。
> 14 字节 ( Ethernet II 首部长度 ) + 46 字节 ( 数据字段最小长度要求 )  = 60 字节

<br/>

# 问题扩展
**来自于问题 3 ，56 字节的 ARP 数据包 ？？？ **
**
![](https://cdn.nlark.com/yuque/0/2021/png/2777842/1613572479530-ca4edc54-42f0-437c-afb8-bb699c689881.png#align=left&display=inline&height=470&margin=%5Bobject%20Object%5D&originHeight=470&originWidth=946&size=0&status=done&style=none&width=946)


和正常的 60 字节的 ARP 数据包，做下对比


![](https://cdn.nlark.com/yuque/0/2021/png/2777842/1613572658107-c107c9de-4710-450c-9342-b50f068e0cd2.png#align=left&display=inline&height=470&margin=%5Bobject%20Object%5D&originHeight=470&originWidth=946&size=0&status=done&style=none&width=946)


之前回答问题大概分析了下，不过提问人并没有进一步反馈抓包环境，也就没有进一步求证了，权当以下是标准答案了  😎 


**_为什么 ARP 报文长度显示是 56 ？_**
对比看了下源mac，一个apple，一个huawei，应该是两个设备网卡支持 802.1Q 标记与否的区别。apple 支持，所以发出来的 arp 数据包，就包含有 4 字节 的 802.1Q 标记，填充字段为 14 字节。中间过了个路由交换设备，去除 802.1Q 标记，但不会再补充填充字段。所以 56 字节组成是：
> 14 字节 ( Ethernet II 首部长度 ) + 42 字节 ( 28 字节 ARP 数据包 + 14 字节填充数据 )  = 56 字节

<br/>

# 问题总结
当然不仅仅是 ARP 报文，对于以太网帧 64 或者 60 字节长度来说，数据字段最小长度要求始终是 46 字节 ，再根据不同的以太网类型和长度，决定是否需要 Padding 。

**附：Wireshark function blocks**


![](https://cdn.nlark.com/yuque/0/2021/png/2777842/1613576577570-d41975b4-0209-4238-a2da-cf1da59481bb.png#align=left&display=inline&height=659&margin=%5Bobject%20Object%5D&originHeight=659&originWidth=461&size=0&status=done&style=none&width=461)


