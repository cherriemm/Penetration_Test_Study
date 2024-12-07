# DHCP 欺骗



## 原理



DHCP 协议(Dynamic Host Configuration Protocol) 动态主机配置协议：主要给客户机提供 TCP/IP 参数，包括：IP 地址、子网掩码、网关、DNS、租期



**工作原理**

- 应用层协议，基于UDP
- 主机向服务器 67 号端口发送 DHCP 请求
- 服务器响应给客户机的 68号端口

![image-20241114204645196](https://s2.loli.net/2024/11/14/jM56EZYraDs8XiK.png)





## 配置

![image-20241112201603610](https://s2.loli.net/2024/11/12/xoaMig9Hsh1dXTN.png)



### 设置DHCP 服务器作用域



在Windows 2016 Server上添加DHCP服务器



添加第一个作用域并激活：

![image-20241112205412075](https://s2.loli.net/2024/11/12/tdD3khpywEKWYG2.png)



设置 Kali 和 Win10 侧网段的作用域：

![image-20241112210108187](https://s2.loli.net/2024/11/12/Cv86GaTij45VLlk.png)



添加网关的IP地址为实验环境中路由器的f0/0端口的IP地址

![image-20241112210047064](https://s2.loli.net/2024/11/12/gVJLGk4QEbhvPnN.png)



右击新配好的作用域下的“作用域选项”，选中DNS选项，配置 DNS 服务器 IP 地址

![image-20241112210255198](https://s2.loli.net/2024/11/12/nqDb6l2URpIm5Go.png)





### win10 配置



在Vmware的虚拟网编辑器中，将win10网卡中的“使用本地DHCP服务将IP地址分配给虚拟机”选项勾除：

![image-20241112210415534](https://s2.loli.net/2024/11/12/OpaXRJQLyioPIcU.png)



在Windows 10的网络适配器中把ipv4更改为自动获取IP地址和自动获取DNS服务器：

![image-20241112210518807](https://s2.loli.net/2024/11/12/qJfILKPv3gHtkEi.png)



`ipconfig/renew` 尝试获取 ip地址：此时win10不能获取 IP 地址

![image-20241112211958332](https://s2.loli.net/2024/11/12/WstfFd7okcPRhBA.png)



抓取GNS3上路由器连接win10一侧的链路的流量：

![image-20241112212044392](https://s2.loli.net/2024/11/12/g3tkRTvLPw2fu6O.png)

报文内容具体分析：

![image-20241112212458279](https://s2.loli.net/2024/11/12/tMyxV8SXYNCRwda.png)

- 源IP地址：由于客户端还未获得 IP 地址，因此源 IP 地址此时显示未 0.0.0.0
- 目标IP 地址为 255.255.255.255，表示广播地址，向所有设备广播此请求
- `Client MAC address` : 客户端MAC地址唯一标识该主机，用于服务器识别请求来自哪个设备。此处即为 win10 MAC 地址
- `DHCP Message Type` : `Discover`



### 配置中继

在GNS3中的镜像路由器上为f0/0端口配置中继：

![image-20241112212944982](https://s2.loli.net/2024/11/12/nHyItmfK1Oowe6l.png)



右键禁用再启用Ethernet0网卡

![image-20241112213143340](https://s2.loli.net/2024/11/12/azMUHv2XxkfhQwW.png)

执行 `ipconfig /all` 命令查看，此时win10 已成功分配 ip地址：192.168.1.4， DHCP 服务器ip地址：192.168.2.2，DNS 服务器为 8.8.8.8(由 DHCP 服务器分配)

![image-20241112223422295](https://s2.loli.net/2024/11/12/hjD2ZB7Ovb8HA1N.png)





### 报文分析



#### 路由器左侧(即win10一侧)的dhcp报文：



![image-20241113090953521](https://s2.loli.net/2024/11/13/uUpmSAozl7QOftg.png)

不同类型的dhcp报文分析如下：



##### `Discover`

左侧第一种类型的报文：**`Discover`** ，由 win10 发出，请求 dhcp 服务的报文![image-20241113091317075](https://s2.loli.net/2024/11/13/iIz7VF5p9LtyhkD.png)

- 源ip地址为 0.0.0.0(未分配ip)
- 目的ip地址为 255.255.255.255(服务器ip未知)
- DHCP 报文的 `Client MAC address` 值为 win10 的 MAC 地址，用于标识 win10





##### `Offer`

左侧第二种类型的报文：`Offer` , 由路由器作为中继，发送给win10，作为向客户端提供 IP 地址分配的初步建议

![image-20241113091816777](https://s2.loli.net/2024/11/13/jEK1tnSkFJTriG4.png)

- 由于此时 win10 ip 未分配，路由器实际采用广播的方式发送报文，在 `Client MAC address` 标识出了请求主机的MAC 地址
- 并指明了 `Your Ip address:  192.168.1.4` ,  中继ip为：192.168.1.1,  dhcp服务器ip为 192.168.2.2



##### `Request`

左侧第三种类型的报文：`Request` , 由 win10 发送给 dhcp 服务器(实际上是win10 广播发送到中继路由器)

![image-20241113092004090](https://s2.loli.net/2024/11/13/beGz5SYgCaIL1jX.png)

- 发送 dhcp Request 报文时，客户端还未正式获得ip地址，因此它无法以单播的形式发送报文，可以看到，此报文的源ip地址为 0.0.0.0， 目的ip地址为 255.255.255.255

- 收到 Offer报文后，win10获得了 dhcp服务器的ip地址为 192.168.2.2，并包含在 Request报文

  当客户端收到多个dhcp服务器发送的dhcp Offer报文时，它选择其中一个服务器后，发送的dhcp Request 报文中，应包含所选择的dhcp服务器的标识符，以明确告知该服务器自己接收其提供的ip地址和网络配置，并告知其他未被选中的服务器回收其准备分配的ip地址





##### `ACK`

左侧第四种类型的报文：`ACK` , 由路由器作为中继，发送给win10，作为dhcp服务器确认客户端ip的确认报文

![image-20241113093828758](https://s2.loli.net/2024/11/13/EH5atByJASs9nCv.png)





#### 路由器右侧(即dhcp服务器一侧)的报文：



右侧报文与左侧类似，路由器作为中继转发win10和dhcp服务器的报文

![image-20241113094432895](https://s2.loli.net/2024/11/13/PeDq529cYM3J4sz.png)

- `Discover ` 报文中，源ip地址为中继ip，由于fa0/0 直接中继到 dhcp 服务器ip，因此此处目的ip为 dhcp服务器ip，而不是广播
- `Offer`报文：dhcp服务器将客户请求的ip地址配置返回给中继
- `Request`报文：中继转发客户的请求报文，作为确认选择该dhcp服务器的标识
- `ACK` 报文：dhcp 服务器将确认客户请求的 ACK 报文返回给中继





## 攻击



kali安装 `yersinia` 工具

![image-20241113103742476](https://s2.loli.net/2024/11/13/5Djzp6cu3JsQZag.png)



kali ip地址：

![image-20241113192947583](https://s2.loli.net/2024/11/13/zKIbAx4WcCgjZpQ.png)



测试 kali和 win10连通性如下：

![image-20241113193138090](https://s2.loli.net/2024/11/13/q8Lvk4Y65C1Miaf.png)





`yersinia -G` 

![image-20241113174811192](https://s2.loli.net/2024/11/13/dZtOVx8eRCaBIJq.png)



### sending discover packet





点击“launch attack”，选择sending discover packet

![image-20241113194246810](https://s2.loli.net/2024/11/13/kJXFlIVA1G58aW7.png)



开始攻击：

![image-20241113195155866](https://s2.loli.net/2024/11/13/2xgFjTDX9SefkQB.png)



GNS3 kali连接的链路的报文：可以看到，kali发送大量dhcp Discover 报文，以此来消耗 dhcp服务器的资源，从而合法客户端无法正常获取ip地址

![image-20241113195327380](https://s2.loli.net/2024/11/13/THptvNWiUI6yKLk.png)





GNS3 路由器左侧的报文：

![image-20241113195346783](https://s2.loli.net/2024/11/13/bK4DPSorfsqu5Ti.png)



GNS3 右侧的报文：

![image-20241113201221682](https://s2.loli.net/2024/11/14/fRHvsl8qSycLG3e.png)



此时在Windows 2016 Server中右击“作用域”选择“显示统计信息”，观察该作用域当前分配的IP地址的情况：

![image-20241113201627624](https://s2.loli.net/2024/11/13/gSkvIrhidPlt9A7.png)



### creating DHCP rogue server

在Windows 10上使用 `ipconfig /release`释放已分配的IP地址

![image-20241113202728042](https://s2.loli.net/2024/11/13/B7cveFa3y8QVH1X.png)



选择“creating DHCP rogue server”:

![image-20241113202944782](https://s2.loli.net/2024/11/13/1e6ka9sbZKStuA8.png)



填写相应参数：Start IP和End IP写为非VMnet1中的IP地址，使Kali给Windows 10分配到IP地址不规范：192.166.1.2 - 192.166.1.10 不在 192.168.1.0 子网内

![image-20241113205459208](https://s2.loli.net/2024/11/13/PzyEudqi2SohKQR.png)



`ipconfig /renew` 命令重新获取IP地址

![image-20241113204043504](https://s2.loli.net/2024/11/13/b68rj1DeIQKpzVd.png)



win10 ip地址被设置为kali发送的非法 ip 地址：

![image-20241114094007752](https://s2.loli.net/2024/11/14/kKWmHcdhG8rbBfC.png)





抓取的报文：

![image-20241114092418501](https://s2.loli.net/2024/11/14/NBRmMZULaPIj5gA.png)



Kali 广播发送的虚假 dhcp Offer报文：

![image-20241114091809491](https://s2.loli.net/2024/11/14/WkIjv3UCzFDRHBb.png)





由 win10 发送给 kali 的 Request 报文：`Requested IP Address` 字段的值为 kali发送的虚假dhcp Offer 报文的值，即非法ip地址192.166.1.2

![image-20241114091605684](https://s2.loli.net/2024/11/14/d7VoCbBGcREgWwT.png)



Kali 广播发送的虚假 dhcp ACK 报文

![image-20241114091953231](https://s2.loli.net/2024/11/14/IL5Y781aFln2cqG.png)





