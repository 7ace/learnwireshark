# Ping
众所周知，_**Ping**_ 命令是利用 ICMP 协议来检测网络连通状态和时延测量的一个基础网络命令，而基于 _**Ping**_ 所发展出来的衍生命令也是繁多，包括：hrping、hping、tcping、smokeping 等等，功能上较基础 _**Ping**_ 命令相应有所扩展，在不同的场景上也是应用较多。

<br/>

# TCPing
其中 _**TCPing**_ 命令就是网络中一般用来检查端口存活的一个命令，操作类似于 "Ping"，但是通过 TCP 端口工作的。

_**TCPing**_ 通过建立到网络主机的一个连接来模拟基于 TCP 上的 "Ping" ，测量_ **SYN**_ 和 _**SYN/ACK**_ 之间的时间。


操作系统默认无此命令工具
![QQ截图20210509192608.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1620560503304-11f9b4be-2227-4f42-9b9c-4f5a2ab98c45.png#align=left&display=inline&height=408&margin=%5Bobject%20Object%5D&name=QQ%E6%88%AA%E5%9B%BE20210509192608.png&originHeight=408&originWidth=860&size=23365&status=done&style=none&width=860)


在 C:\Windows\System32 下放置 tcping.exe 程序后
![QQ截图20210509194353.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1620560683478-6803c887-4ad7-4ea1-ada1-193fc13b69e5.png#align=left&display=inline&height=229&margin=%5Bobject%20Object%5D&name=QQ%E6%88%AA%E5%9B%BE20210509194353.png&originHeight=229&originWidth=489&size=8695&status=done&style=none&width=489)
不加任何参数时，默认检查的端口是 80 ，端口检测开放以及测量时间如图。


简单的端口参数应用，443 端口同样开放。
![QQ截图20210509194735.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1620560862496-67a9bdd0-398a-4d20-aa53-809411a4a40c.png#align=left&display=inline&height=204&margin=%5Bobject%20Object%5D&name=QQ%E6%88%AA%E5%9B%BE20210509194735.png&originHeight=204&originWidth=472&size=7099&status=done&style=none&width=472)


> 详细用法不再赘述，可自行搜索。
> [https://www.elifulkerson.com/projects/tcping.php](https://www.elifulkerson.com/projects/tcping.php)

<br/>

# Wireshark 解析 TCPing
通过 Wireshark 抓包解析 TCPing 程序，可发现对应的 4 次 probe，以 TCP 三次握手建连，最后以 FIN 包结束连接。 成功的三次握手也证明对端 443 端口的开放。
![QQ截图20210509200027.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1620561640334-910babf1-c380-4725-baa9-22b065238134.png#align=left&display=inline&height=342&margin=%5Bobject%20Object%5D&name=QQ%E6%88%AA%E5%9B%BE20210509200027.png&originHeight=342&originWidth=1102&size=58787&status=done&style=none&width=1102)

<br/>

# TCPing 未解之谜
以上罗里吧嗦了半天，文章重点其实是想说碰到的一个奇怪现象。。。😅  百思不得其解，以此文记录之，留待有缘的一天解谜之。 🤣


在公司办公网一台主机偶然 TCPing 了一台业务网服务器的 80 端口，TCPing 结果显示 80 端口开放，而实际上本地主机包括 Wireshark 以及 netstat 检查根本没有建立相关的 TCP 连接，而且在防火墙日志上可看到 TCPing SYN 的包被丢弃（往目标服务器所在区域会经过防火墙，并未开放相关策略），种种验证结果与 TCPing 端口 open 结果完全不相符。


Tcping 目标服务器 80 端口，结果显示 Port is open（赤裸裸的假象）。但 netstat 检查结果仅显示 SYN_SENT 状态，未建立连接。
![截图_20210509201601_meitu_1_meitu_4.jpg](https://cdn.nlark.com/yuque/0/2021/jpeg/2777842/1620564245432-31124a1b-9fbf-4841-bfe5-45bf69e17e0c.jpeg#align=left&display=inline&height=303&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%9B%BE_20210509201601_meitu_1_meitu_4.jpg&originHeight=303&originWidth=543&size=54328&status=done&style=none&width=543)


Wireshark 抓包结果同样仅显示 SYN 包（4次）发送，无响应，未正常建立连接。
![截图_20210509201710_meitu_2.jpg](https://cdn.nlark.com/yuque/0/2021/jpeg/2777842/1620563902281-c55d7cd4-1b60-4779-a6ab-4269bb1fa8bb.jpeg#align=left&display=inline&height=260&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%9B%BE_20210509201710_meitu_2.jpg&originHeight=260&originWidth=1198&size=242974&status=done&style=none&width=1198)
> 忽略源端口不一样，netstat 和 wireshark 取的结果非同一次。



