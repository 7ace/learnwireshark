# Checksum Offload

#### CheckSum

一些网络协议使用校验和来确保数据的完整性，校验和也被称为冗余校验。

校验和用于确保数据传输或存储的数据部分的完整性，它基本上是一个数据部分的计算汇总。发送方将计算数据的校验和，并与校验和一起传送数据。接收方使用与发送方相同的算法计算接收数据的校验和。如果校验和不匹配，则会发生错误，接收方将丢弃数据包。

<br/>

#### Offload

对于 offload 特性，大多数操作系统都支持多种形式的网络卸载，包括在 TCP/IP 协议栈中进行的 IP 分片、TCP 分段、重组、checksum 校验等操作，会转移到网卡硬件中进行，而不是 CPU 上，这样可以降低系统 CPU 的消耗，提高处理性能。

<br/>

#### CheckSum Offload

CheckSum Offload 实际上就是是将 TCP/UDP/IP 校验和工作交给了网卡硬件完成，以节约系统的 CPU 资源。

譬如：以太网发送网卡计算以太网 CRC32 校验和，接收网卡验证这个校验和。如果接收到的校验和错误，Wireshark 甚至不会看到数据包，因为以太网网卡会在内部丢弃数据包。

<br/>

#### TCP/UDP/IP 校验和

- TCP 校验和计算三部分：TCP 头部、TCP 数据和 TCP 伪头部。TCP 校验和是必须的。
- UDP 校验和计算三部分：UDP 头部、UDP 数据和 UDP 伪头部。UDP 校验和是可选的。
- IP 校验和只计算检验 IP 数据报的首部，但不包括 IP 数据报中的数据部分。

TCP/IP 协议栈不会自己计算校验和，而是简单地将一个空的校验和字段(零或随机填充)交给网卡硬件。

<br/>

**参考文档**：
[https://www.wireshark.org/docs/wsug_html_chunked/ChAdvChecksums.html](https://www.wireshark.org/docs/wsug_html_chunked/ChAdvChecksums.html)

