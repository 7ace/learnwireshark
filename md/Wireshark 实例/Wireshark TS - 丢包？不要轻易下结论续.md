# 问题背景
在《[Wireshark TS | 丢包？不要轻易下结论](https://github.com/7ace/learnwireshark/edit/main/md/Wireshark%20%E5%AE%9E%E4%BE%8B/Wireshark%20TS%20-%20%E4%B8%A2%E5%8C%85%EF%BC%9F%E4%B8%8D%E8%A6%81%E8%BD%BB%E6%98%93%E4%B8%8B%E7%BB%93%E8%AE%BA.md)》一文中，提到出现 **TCP Previous segment not captured** 和 **ACKed segment that wasn't caputred** 的一种情形，**可能是交换机镜像会话丢弃了数据包，而不是真实会话发生了数据包丢失，** 所以数据包文件才会显示有丢包，而且也才没有捕捉到数据包重传。总结来说，对于疑似丢包的情况，不要轻易下结论，需要结合上下文，看看是否有相应的乱序、重传或者快速重传等等，再来进一步分析判断问题。

本文再次介绍一种 Wireshark 提示 **TCP Previous segment not captured** 和 **ACKed segment that wasn't caputred** 的情形，来具体分析是否存在丢包现象。

<br/>

# 问题描述
抓包如下，在数据包初始即出现了熟悉的 Bad TCP 提示。
![1.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1622683243637-ef8498c6-c3f2-4b75-9f36-048f8023a85b.png#align=left&display=inline&height=119&margin=%5Bobject%20Object%5D&name=1.png&originHeight=119&originWidth=1635&size=23305&status=done&style=none&width=1635)
单就从前5个数据包来说，筛选出 **_Seq、NextSeq、Ack_** 三列分析 ，Wireshark 的判断并没有任何问题。


数据包2 - **ACKed segment that wasn't caputred** ：从它的角度来说，Ack 2 应该是确认了一个 len 为 1 的数据分段，但是未抓到，所以会提示 TCP ACK确认了一个未曾看到的数据分段。
数据包3 - **TCP Previous segment not captured** ：Seq 2 ，上一个数据包理论上应该 NextSeq 为 2 ，但是未抓到，所以会提示 TCP 上一个分段未捕获。
数据包4同数据包2一样，Ack 63 确认了63之前所有的分段，但实际上仍然缺少一个分段。

因此对 Wireshark 来说，它的判断原则认为在传输方向 _**10.1.1.1 -> 10.1.2.1 **_ 上可能丢了一个 _**Seq 1 , NextSeq 2 , Ack 1**_ 的一个数据分段。

<br/>

# 问题分析
分析阶段，再次添加过滤 _**ip.id**_ 列来进一步分析，可以看到一个奇怪的现象，在传输方向 _**10.1.1.1 -> 10.1.2.1 **_ 上，数据1和数据包3的 ip.id 是连续字段，**0x98a6 -> 0x98a7**，也就是说在此方向上数据包1和3中间并没有额外的数据分段，也不可能存在数据包未捕获到的情况，那么是否是 **Wireshark 错误判断了 **？？？

![2.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1622683264132-99fbf671-909c-424e-b608-56ea59b16644.png#align=left&display=inline&height=117&margin=%5Bobject%20Object%5D&name=2.png&originHeight=117&originWidth=1758&size=25749&status=done&style=none&width=1758)


进一步研究接下来的数据包，发现会有 **TCP Keep-Alive** 和 **TCP Keep-Alive ACK ** 数据包，且业务数据包排列有一定规律。

![丢包？1.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1622802674678-d2e582d5-bceb-4b43-8433-7ff3b7f33c02.png#align=left&display=inline&height=521&margin=%5Bobject%20Object%5D&name=%E4%B8%A2%E5%8C%85%EF%BC%9F1.png&originHeight=521&originWidth=1708&size=118828&status=done&style=none&width=1708)

拿数据包 12-14、15-20 为例，可以看出是 3个业务数据包 + 6个 TCP Keep-ALive 数据包，其中 **TCP Keep-Alive ** 数据包的 Seq 会减 1，也就是数据包15（17、19）Seq 为 123，即 124 - 1。


往后数据包 21-23、24-xx 之后均是相同排列，那么往前呢？数据包 3-5、6-11 应该也是相同规律，同样为 3个业务数据包 + 6个 TCP Keep-ALive 数据包。

至此，数据包 1-2 的类型也可以明显定位了，是一对 **TCP Keep-Alive** 和 **TCP Keep-Alive ACK ** 数据包。因为捕获数据包并未抓到之前的包，Wireshark 无法根据上文判断，所以并未标记成 TCP Keep-Alive 数据包，造成之后的数据包 2-4 进行 tcp.analysis 判断时出现偏差，认为是存在丢包情况，但实际上的原因是 Seq 因为 Keep-Alive 的情况已经减 1，理论上之前未捕获的数据包有一个_** Seq 2 , NextSeq 2 , Ack 1**_ 的一个数据分段。

将 **TCP 选项** _**Relative sequence numbers**_ 关闭后，可以更加直观的对 Seq 进行判断分析，如下：

![丢包？2.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1622862648079-4e6e2ff9-8424-43fe-938c-d50f9350720d.png#align=left&display=inline&height=518&margin=%5Bobject%20Object%5D&name=%E4%B8%A2%E5%8C%85%EF%BC%9F2.png&originHeight=518&originWidth=2001&size=137645&status=done&style=none&width=2001)
实际上应该是一个 _**Seq ****1529774348**** , NextSeq ****1529774348**** , Ack ****516539103**** **_的数据分段。

<br/>

# 问题总结
综上所述，实际上此案例并未发生丢包，不管是实际业务又或是镜像方面均未发生丢包情况，仅仅是因为未抓到之前的数据包，且存在特殊情形（TCP Keep-Alive），综合起来造成 Wireshark 判断分析出现偏差。

所以对于疑似丢包的情况，不要轻易下结论，需要结合上下文的情况综合判断。






