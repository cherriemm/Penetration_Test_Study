# ICMP 重定向攻击



## 原理



ICMP (Internet Control Message Protocol) ：Internet 控制报文协议，用于在 IP 主机和 路由器之间传递控制消息(控制消息指网络是否通、主机是否可达、路由器是否可用等)

![image-20241114192636632](https://s2.loli.net/2024/11/14/VjSKzq6MPCGxite.png)



**ICMP 重定向**

在某些特定情况下，路由器在检测到主机使用非优化路由时，会向主机发送一个ICMP重定向报文，使主机的路由改变

![image-20241114192817351](https://s2.loli.net/2024/11/14/k7PSv2XE8FYHcdA.png)

![image-20241114192949499](https://s2.loli.net/2024/11/14/z62cJKvp5gGFjSE.png)



**攻击模型**

![image-20241114193017324](https://s2.loli.net/2024/11/14/neLMc6hoWdFDRxj.png)





## 网络信息



win10 `ip：192.168.137.133` 

<img src="https://s2.loli.net/2024/11/04/zJhXWRFQ6Gg87Lj.png" alt="image-20241104191933701" style="zoom:33%;" />



kali ip：`192.168.137.130`

<img src="https://s2.loli.net/2024/11/04/ZDRvw6hgGWCTYpl.png" alt="image-20241104192111798" style="zoom: 50%;" />

网关ip：`192.168.137.1 `





**测试kali和 windows10 连通性：**

win10 ping kali:

![image-20241104203539700](https://s2.loli.net/2024/11/04/P3jAstv9oFTLcVJ.png)

kali ping win10: ![image-20241104205305914](https://s2.loli.net/2024/11/04/XgErlQLFuMG9KJY.png)



## 攻击



### 正常指定 -i 和  -g

1. kali 使用`netwox 86 -g 192.168.137.130 -i 192.168.137.1` 将win10网关重定向到kali 的ip 地址

- `-g` : 重定向到的新网关的IP地址（Kali攻击机的IP地址）
- `-i`：受攻击主机的网关的IP地址



2. Win10 抓取流量如下：可以看到win10 收到来自网关 192.168.137.1 的重定向报文

![image-20241109085031623](https://s2.loli.net/2024/11/09/SrEK6FohtuVwRnZ.png)



分析上面的 ICMP 重定向报文：

- ICMP 报文的 `Type =5, code = 1` ，表示为主机重定向报文
-  `Gateway Address = 192.168.137.130` 字段值为 kali ip ，即将 win10 网关重定向到 kali 的ip地址

![image-20241109085754506](https://s2.loli.net/2024/11/09/m4MYbtwD9lIds3a.png)





此时win 10 ping 不通 baidu，无法上网

![image-20241109094707313](https://s2.loli.net/2024/11/09/fkO1ArxyCdELta5.png)



可以看到此时网关的下一跳被设置为 kali ip：(即网关被重定向到 kali ip)

![image-20241109094053802](https://s2.loli.net/2024/11/09/DwzekJLG8SWrjxg.png)



### 不指定 -i 选项

**如果把netwox里面的命令中-i 及后面的内容删除，观察此时是否能上网？**

从抓取的报文可知，攻击机 kali (192.168.137.130) 直接将重定向报文发送给受攻击主机 win10(192.168.137.133)

![image-20241109091104020](https://s2.loli.net/2024/11/09/sWcQoI2b1KwPFgm.png)

此时win10可以上网,，这说明没有指定被攻击主机的网关 IP 地址时，攻击无法生效，Windows 10 仍然按照正常的路由将数据包发送给其默认网关，从而能够正常访问 Internet。



### 将 -g 指定为虚假 ip



如果把netwox命令中的-g指定的ip地址改为一个不在该网段中的虚假ip地址，观察此时能否上网，说明了什么？

![image-20241109100146677](https://s2.loli.net/2024/11/14/tzgk5BuSQ6I4xmq.png)



可以看到网关的下一跳被设置为虚假ip地址

![image-20241109100354617](https://s2.loli.net/2024/11/09/kUDqBJ7hTZA9I2W.png)



win10 上网情况：部分数据包正常传输(ping通的部分)，部分数据包不可达(无法访问目标主机)，表明攻击并没有完全阻止数据包的正常传输，部分数据包遵循正常的路由。

![image-20241109100456925](https://s2.loli.net/2024/11/09/IVJskhtX9Wb8cUY.png)



## 识别出ICMP重定向攻击报文

**在Kali Linux能够攻击成功时，观察Wireshark抓取的ICMP重定向报文，思考理论上有没有办法识别出ICMP重定向攻击报文**

1. 检查源 IP 地址：正常的 ICMP 重定向报文应该来自当前主机的网关。如果源 IP 地址不是网关地址或者来自一个不可信的地址，可能是攻击报文。

2. 比较多个重定向报文：如果短时间内收到大量来自不同源地址或针对不同目标的重定向报文，可能是攻击行为。



## 防御 ICMP 重定向攻击

防御 ICMP 重定向攻击的方法包括：

1. 禁用 ICMP 重定向：在 Windows 10 中，可以通过修改注册表或使用组策略编辑器来禁用 ICMP 重定向。这样，即使收到伪造的重定向报文，系统也不会更改路由表。
2. 使用防火墙：配置防火墙来过滤掉可疑的 ICMP 重定向报文。可以设置规则只允许来自可信网关的重定向报文通过。



打开防火墙后，win10虚拟机可以正常上网

![image-20241109102033500](https://s2.loli.net/2024/11/09/t3bPml5KGHFWCvn.png)