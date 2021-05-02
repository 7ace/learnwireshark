# 问题描述
我司业务网广域网双专线（联通+电信）对接 AWS ，日常联通主用，电信备用。某日业务中心反馈线路业务 Ping 有丢包，网络进行检测后发现联通线路丢包严重，遂正常流程切换至电信备线路运行，之后报故障至联通。联通经过两三天鼓捣后，反馈线路已恢复，周末网络回切后，业务反馈发现一系列问题，主要如下：
a. 业务数据积压；
b. 业务运维跳板机至 AWS 服务器，新 SSH 访问卡机，老 SSH 连接正常；
c. ...
线路继切换至电信备线路后，故障现象消失，但一切换至联通主线路后，则继续发生上述问题。

<br/>

# 问题处理
就所反馈的问题来说，因为不了解业务，所以业务方面的种种异象，不好研究分析，只有从 SSH 访问方面试着下手。

简单整理了拓扑，如下：

![未命名文件.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1619442563362-0d09b16b-bc9d-4d62-8523-f17e9bd6873d.png#align=left&display=inline&height=910&margin=%5Bobject%20Object%5D&name=%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6.png&originHeight=910&originWidth=516&size=49939&status=done&style=none&width=516)
> 简化了架构，实际上 R1 和 R2 分属于双数据中心，下连内部区域实际也为两个同类区域。

依次在广域网设备上检查了 BGP、OSPF 路由，包括来回端的 Ping 和 Trace ，都无明显问题，也一度怀疑双中心线路切换造成了来回路径不一致，穿过了不同的防火墙造成的问题，经过详细排查，也一一排除。在问题联通线路上，SSH 访问卡在某个画面的故障现象仍然必现。
```shell
[C:\~]$ ssh 10.1.1.1

Connecting to 10.1.1.1:22...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.
```
大概是上面这种提示，一直卡着，不跳用户名和密码输入提示框。


光看现象，简直抓瞎，只能抓包试着分析一下。

<br/>

# 问题分析
因为 SSH 协议较为简单，且问题现象较为明确，则在问题复现时直接抓包分析。

**SSH 问题第一次（运维跳板机处抓包）**

![截图_20210426214846.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1619444977374-d4e1b5ae-cef9-4a40-a091-be282cf32021.png#align=left&display=inline&height=422&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%9B%BE_20210426214846.png&originHeight=422&originWidth=1409&size=67191&status=done&style=none&width=1409)
                                                                                图1


三次握手以及 SSH 初始均正常，无法弹出用户名和密码框时的抓包信息如下：

![截图_20210426214911.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1619445033451-918af6c9-ab7e-4d06-a951-e02797128ec6.png#align=left&display=inline&height=636&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%9B%BE_20210426214911.png&originHeight=636&originWidth=1493&size=115218&status=done&style=none&width=1493)
图2

_33 分组 _提示 **TCP Previous segment not captured**，TCP上一分段( **Seq 3050** )未捕捉到，可能是真的未收到，也可能是抓包问题没捕获到。紧接着运维跳板机（客户端）192.168.1.1 一直请求 Seq 3050 分段未果（ TCP Dup ACK 34 #）, 之后一直服务器无回应。


**SSH 问题第二次（运维跳板机处抓包）**
尝试第二次抓问题包，对比看是否是类似现象。

![截图_20210426220225.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1619445757705-e602e27e-c200-40d9-872b-0513489977fb.png#align=left&display=inline&height=502&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%9B%BE_20210426220225.png&originHeight=502&originWidth=1499&size=87953&status=done&style=none&width=1499)
图3

问题依旧，仍然是在 _37 分组 _提示 **TCP Previous segment not captured**，仍然是 TCP 上一分段( **Seq 3050** )未捕捉到。额。好吧，又刚好是分段 3050 丢失，差不多可以判断认为是对端发了，但是本端没收到。


**SSH 正常（运维跳板机处抓包）**
正常线路运行时 SSH 的抓包，对比有问题的时候，看具体是哪个环节出了问题。就在苦苦人肉看包的同时，一同事在边上幽幽的来了一句：估计是运营商问题，Ping 哪哪哪小包就通，大包就不通，MTU 问题了。

额。MTU，好吧，点醒了我们，继续看数据包，果然是这样。

![截图_20210426221730.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1619446711749-98cd4eaa-f9e3-4625-819d-7ccfc56cfc1d.png#align=left&display=inline&height=319&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%9B%BE_20210426221730.png&originHeight=319&originWidth=1202&size=49629&status=done&style=none&width=1202)
图4

<br/>

**联通问题线路时：**
图2中 分组32（长度122）、分组33（长度326），中间丢失分段；
图3中 分组36（长度122）、分组37（长度326），中间丢失分段；
**电信正常线路时：**
图4中 分组35（长度122）、分组37（长度326），中间分组36（**长度1514**）。

至此实锤确认联通线路问题，在运营商中间设备存在 MTU 设置小于 1500 的问题，这样 SSH 也包括业务在内，AWS 服务器正常发送数据字段长度 len = 1460 的包时，即大小在 1500 长度的数据包时，走到运营商设备上，因为 IP 字段 Don't fragment 设置的原因造成数据包丢包，之后引起一系列问题。
![截图_20210426223139.png](https://cdn.nlark.com/yuque/0/2021/png/2777842/1619447565908-9138d553-0ec9-4824-bfde-83ae48ff00bc.png#align=left&display=inline&height=219&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%9B%BE_20210426223139.png&originHeight=219&originWidth=581&size=12428&status=done&style=none&width=581)

<br/>

# 问题总结
总结，没啥好总结，切换至问题线路后，业务有问题一切都找运营商 🤣

文章收尾时，正好同事反馈联通处理完毕后业务恢复正常，Over。
