# ARP欺骗



## 原理

将一个已知的IP地址解析为MAC地址，从而进行二层数据交互



- `arp -a` ：查看 ARP 映射关系

- `arp -d` ：清除 ARP 缓存

- `netsh -c interface ipv4 add neighbors <idx> <ip address> <MAC address>` ,  静态绑定 ARP 映射关系

  `netsh interface ipv4 show interface` 查询 idx 值



**Case1 ：伪造虚假 ARP 应答报文，造成被攻击主机无法上网**

![image-20241114185625988](https://s2.loli.net/2024/11/14/wxlhBr7CtXc9GzD.png)



**Case2 : 向被攻击主机和网关响应攻击主机真实的 MAC 地址，从而截获被攻击主机的数据 ， 此时被攻击主机可以访问网络**

![image-20241114190011724](https://s2.loli.net/2024/11/14/iAofFBhgmUKlv1y.png)





**Case3 : 伪造 ARP 应答报文，向被攻击主机和其通信主机响应攻击主机真实的 MAC 地址**

![image-20241114190315451](https://s2.loli.net/2024/11/14/UVcQ29eFMg3fnTu.png)





## 环境信息：

Windows 系统下 `ipconfig /all` 查看Mac地址信息：

win10 `ip：192.168.137.133` , `Mac: 00-0c-29-c9-d7-6e`

<img src="https://s2.loli.net/2024/11/04/zJhXWRFQ6Gg87Lj.png" alt="image-20241104191933701" style="zoom:33%;" />

kali ip：`192.168.137.130` , `Mac: 00-0c-29-79-46-7d`

<img src="https://s2.loli.net/2024/11/04/ZDRvw6hgGWCTYpl.png" alt="image-20241104192111798" style="zoom: 50%;" />

网关ip：`192.168.137.1 `, `Mac: 00-50-56-c0-00-00`

<img src="https://s2.loli.net/2024/11/04/8c4mo6kDjICMxHv.png" alt="image-20241104191040349" style="zoom: 25%;" />



## 实施欺骗

win10 查看网关信息：ip：`192.168.137.1 `, `Mac: 00-50-56-c0-00-00`

![image-20241104161055142](https://s2.loli.net/2024/11/04/8iZc54V9edGjLum.png)



开始 ARP 欺骗

kali 执行命令：`arpspoof -i eth0 -t 192.168.137.133 192.168.137.1`

![image-20241104161450198](https://s2.loli.net/2024/11/04/SjAnUm2bykEds9p.png)



然后在 win 10上查看网关 Mac 地址：此时网关的Mac地址与Kali的Mac地址相同

![image-20241104161419533](https://s2.loli.net/2024/11/04/WZYhsH5MluVAey7.png)



打开Wireshark 抓取欺骗攻击流量，查看ARP 协议包，可以看到网关ip被响应为Kali ip地址(`VMware_79:46:7d` 是kali , `VMware_c9:d7:6e`是 win10 )

攻击者发送的响应包将其MAC地址映射到目标IP地址，从而覆盖真实的ARP映射。

![image-20241104170254525](https://s2.loli.net/2024/11/04/28KILxrnUh1EBCA.png)





### 开启IP转发

当前受欺骗主机无法上网

<img src="https://s2.loli.net/2024/11/04/movpbOugL8e3NxA.png" alt="image-20241104170506863" style="zoom: 33%;" />



Kali执行`echo 1 >> /proc/sys/net/ipv4/ip_forward`， 开启IP转发功能后，观察到受欺骗主机可以正常上网

<img src="https://s2.loli.net/2024/11/04/QOcRFPS9fN715go.png" alt="image-20241104170941975" style="zoom: 25%;" />



### 分析数据包

受欺骗主机访问 www.baidu.com 后，通过`nslookup www.baidu.com` 得到百度的ip地址为：183.2.172.185

Wireshark 过滤目的ip为 百度 ip 的包：

![image-20241104171530818](https://s2.loli.net/2024/11/04/RwMYiNq5Io2hlmy.png)



查看包的内容，可以看到报文的目的Mac地址为Kali 的ip地址，即win10 将 kali作为网关，因此欺骗成功且受欺骗机可以正常联网

![image-20241104171719889](https://s2.loli.net/2024/11/04/pPqzHBu6gREV4Dw.png)







## ARP 防御

![image-20241114190442461](https://s2.loli.net/2024/11/14/jv6xReAdCnHi21w.png)



在受欺骗主机上进行 ARP 绑定，绑定网关ip与MAC地址为环境中网关的真实地址进行ARP 防御

![image-20241104191730657](https://s2.loli.net/2024/11/04/gti1sbmRrHNjkWy.png)

![image-20241104191832300](https://s2.loli.net/2024/11/04/X4YleHzbAgkEpZF.png)

可以看到，当前ip 地址与 Mac 地址静态绑定



此时 win 10 可以正常访问网络

![image-20241104184154462](https://s2.loli.net/2024/11/14/NrtLDJZXhGyYo5b.png)



win10 打开Wireshark 抓取流量，此时 win10 与真实网关正常通信，防御成功

![image-20241104194506554](https://s2.loli.net/2024/11/14/ZopA2gHdXWPY5FV.png)





## 截获http流量包



受攻击主机以http访问http://www.uooconline.com网站

![image-20241104194814839](https://s2.loli.net/2024/11/04/RImbCwJ6Qc1qXrg.png)



在Kali Linux上使用Wireshark抓取网络中流量，观察到察受欺骗主机的应用层网络流量

![image-20241104195840478](https://s2.loli.net/2024/11/04/PpTchaXiuLlfwbv.png)



查看 POST 数据包，发现账号信息：

![image-20241104201846138](https://s2.loli.net/2024/11/04/7gickGhQdeTzEfv.png)



密码信息：

![image-20241104201421594](https://s2.loli.net/2024/11/04/KvYoxz2majQduRD.png)



base64解码获得密码：

<img src="https://s2.loli.net/2024/11/04/VqZTalBun1w8cIU.png" alt="image-20241104201540303" style="zoom:33%;" />



## driftnet 图片抓取

kali 使用driftnet命令  `driftnet -i eth0` ,打开driftnet的图片抓取程序，抓取到受欺骗主机在访问http网站时的图片：

![image-20241104202903833](https://s2.loli.net/2024/11/04/jXrhZldgwqWmbvf.png)