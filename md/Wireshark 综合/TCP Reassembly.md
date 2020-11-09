# TCP Reassembly

**1. TCP Reassembly**

- Wireshark 支持跨越多个 TCP Segment 重组 PDU
- TCP Segment，基于 TCP 之上的协议大包因为 MTU、MSS 等引起的 TCP 分段
- PDU(Protocol Data Unit)，通俗的叫做 "packet"
- TCP Reassembly 只有在捕捉到完整数据包且对包的所有校验和有效的情况下工作

<br/>

**2. Preferences 协议首选项**

**TCP**

```
Allow subdissector to reassemble TCP streams
```

注：仅仅开启 TCP 这个选项不够，还需要同样开启协议特定重组选项，如 HTTP、DNS 等协议。

<br/>

**HTTP**

`Reassemble HTTP headers spanning multiple TCP segments`

`Reassemble HTTP bodies spanning multiple TCP segments`

`Reassemble chunked transfe-coded bodies`

**DNS**

`Reassemble DNS messages spanning multiple TCP segments`

**SSH**

`Reassemble SSH buffers spanning multiple TCP segments`

<br/>

**3. SSH实例：**

**TCP + SSH 选项双开**
6，8-11 帧（5个重组 TCP Segments ），6，8-10 帧显示 TCP segment of a reassembled PDU ，11 帧会显示重组 TCP 的汇总信息[
](https://postimg.cc/CnHtTSqc)![TCP-SSH.png](https://cdn.nlark.com/yuque/0/2020/png/2777842/1604925583418-477bab21-0e68-446f-b9b6-565fcdb00da4.png#align=left&display=inline&height=234&margin=%5Bobject%20Object%5D&name=TCP-SSH.png&originHeight=234&originWidth=1301&size=37828&status=done&style=none&width=1301)

**SSH 选项单开**
6，8-11 帧，显示 Unreassembled packet ，TCP 不允许解析器重组 TCP 流
![SSH.png](https://cdn.nlark.com/yuque/0/2020/png/2777842/1604925637699-07814dec-5eb0-4f0e-8432-b9ad638a0e4f.png#align=left&display=inline&height=237&margin=%5Bobject%20Object%5D&name=SSH.png&originHeight=237&originWidth=1304&size=38794&status=done&style=none&width=1304)

**TCP 选项单开**
6，8-11 帧，显示 Malformed Packet ，TCP 上层协议 SSH 未开启重组信息识别
![TCP.png](https://cdn.nlark.com/yuque/0/2020/png/2777842/1604925714353-91559fdf-275e-477b-8915-f11f683a7590.png#align=left&display=inline&height=238&margin=%5Bobject%20Object%5D&name=TCP.png&originHeight=238&originWidth=1298&size=38804&status=done&style=none&width=1298)

<br/>

**参考 Wireshark WiKi :**
[https://wiki.wireshark.org/TCP_Reassembly](https://wiki.wireshark.org/TCP_Reassembly)

