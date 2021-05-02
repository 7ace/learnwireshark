# 背景
想到说路由和ARP，来自于两个事情：一是以前的面试，自己总喜欢问候选人一些基本的路由交换知识，譬如局域网下数据包的转发原理，主要想看看对网络基础概念掌握得如何；二是《Wireshark网络分析就这么简单》（林沛满老师著作）中的 **从一道面试题开始说起**  章节，通过 Wireshark 详细阐述了 ARP 的工作过程。也包括在知乎看到的一些关于此方面的问答，就想随便简单说说路由、ARP 的个人理解。

注：此篇文章属于路由交换基础，大拿们可自行略过。如有想交流的朋友可私信给我，我会在第一时间回复，感谢您的关注和支持~

<br/>

# 路由
路由，网络中的术语，一般说的是三层转发，又或者更加通俗的一些解释（详见各网络书籍 ）。


而我理解路由，有点偏工程术语，指的是 **通往目的地的路径，** 着重两方面：

1. **如何选哪条路；**
2. **如何走这条路。**


对应于网络设备，像是三层交换机或路由器，可以理解为：

1. 如何选哪条路 --- 基于静态路由或动态路由方式选取的最佳路径，可能是一条，也可能是多条均衡。   
2. 如何走这条路 --- 知道了哪条路，也就知道了从哪个接口出，根据数据链路层的不同协议，做不同的封装，然后转发包。


对应于 PC 机或是服务器，可以理解为：

1. 如何选哪条路 --- 基于静态路由或网关方式选取的最佳路径，可能是一条，也可能是多条均衡。   
2. 如何走这条路 --- 知道了从哪个网卡出，封装成以太网帧，然后转发包。

<br/>

# ARP
ARP 就更不需要多解释，相信大多数人都了解它的工作原理。作用于上述路由中的如何转发包部分。

但是就像我前面所说，对于一些面试题（结合路由+ARP），并不是所有人都能很清晰的说出完整的数据包交互过程，如下：

![Untitled Diagram.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1613747038959-c822f5c5-c572-4400-887d-8d1c97923169.png#align=left&display=inline&height=276&margin=%5Bobject%20Object%5D&name=Untitled%20Diagram.png&originHeight=276&originWidth=361&size=14005&status=done&style=none&width=361)

<br/>

# 扩展
如果你能很好的说出上题的网络工作原理，那么可以再看看下面这些：


1. Server1 和 Server2 能互通嘛？如果能通，有什么方法？

![Untitled Diagram (1).png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1613747906166-13045835-b0e3-4187-8ceb-523f1cd4f3cc.png#align=left&display=inline&height=273&margin=%5Bobject%20Object%5D&name=Untitled%20Diagram%20%281%29.png&originHeight=273&originWidth=361&size=12148&status=done&style=none&width=361)
也等同于
![Untitled Diagram (2).png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1613747956124-29f30a16-2dbf-4e28-9b8c-7de3c68d5083.png#align=left&display=inline&height=103&margin=%5Bobject%20Object%5D&name=Untitled%20Diagram%20%282%29.png&originHeight=103&originWidth=361&size=4484&status=done&style=none&width=361)

<br/>

2. 加一点交换的东西，同样是1的问题，什么答案？

![Untitled Diagram (4).png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1613748508617-e4fa55a7-c9c6-4173-a05a-6526f2ed9065.png#align=left&display=inline&height=273&margin=%5Bobject%20Object%5D&name=Untitled%20Diagram%20%284%29.png&originHeight=273&originWidth=401&size=12111&status=done&style=none&width=401)

<br/>

3. 换成三层交换机下代理ARP的工作环境，是怎样工作的呢？

<br/>

# 总结
实验环境很简单，搭建并开启 Wireshark 实验起来，相信你会对路由、ARP等有更不一样的理解。


