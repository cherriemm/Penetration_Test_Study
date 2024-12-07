# DNS欺骗



## 原理

将域名和IP地址的相互映射关系存放在一个分布式的数据库中

![image-20241114213639832](https://s2.loli.net/2024/11/14/9alpCYoOW6PFG4q.png)



**DNS 欺骗原理**

- 攻击者冒充DNS服务器，为被攻击主机提供错误的IP地址

- 基于ARP攻击

- 本质上属于MITM(中间人攻击)
  - 网络层通过ARP欺骗实现身份伪造
  - 应用层通过软件/脚本发送伪造虚假DNS响应报文

![image-20241114213847957](https://s2.loli.net/2024/11/14/aZhVMykpdle9OPg.png)



## 配置网络信息

设置Kali Linux主机、Windows Server 2016服务器与Windows 10在同一个可以上网的网段

kali ip地址：192.168.139.129

![image-20241114095348018](https://s2.loli.net/2024/11/14/68eNw5ZaYGbscjH.png)



windows 10 ip地址：192.168.139.130

![image-20241114095433026](https://s2.loli.net/2024/11/14/YaDlf1qnTibmuFP.png)



windows server 2016 ip地址：192.168.139.131

![image-20241114095315565](https://s2.loli.net/2024/11/14/HIJrnhzjUgmeixt.png)



连通性测试：

kali ping win10 和 windows server 2016：

![image-20241114095615730](https://s2.loli.net/2024/11/14/W7XfZg2RAesHCp3.png)



win 10 ping kali 和 windows server 2016：

![image-20241114095708710](https://s2.loli.net/2024/11/14/BSvMl4ow5dKqjGP.png)



windows server 2016 ping kali 和 win10：

![image-20241114095757894](https://s2.loli.net/2024/11/14/rJwW6FtzuOXbKMp.png)



## ettercap 开始攻击



编辑 ettercap 工具执行 dns 欺骗时的参数文件：

![image-20241114100432267](https://s2.loli.net/2024/11/14/aAwKIyFDNeHXiqu.png)



在文件中编辑以下内容并保存：

IPv4地址的域名 A 错误的IPv4地址（Windows Server 2016的IP地址，或者一个第三方网站的IP地址）

![image-20241114100408382](https://s2.loli.net/2024/11/14/4nLkvu15ZQoaMKz.png)



![image-20241114210242190](https://s2.loli.net/2024/11/14/vMYVOedpDxjHBAb.png)



点击 scan for hosts 扫描 与kali同一网段的其他主机：

![image-20241114210521234](https://s2.loli.net/2024/11/14/kVN958GJd3bcjQs.png)



点击 Hosts Lists 列出主机列表：

![image-20241114210619932](https://s2.loli.net/2024/11/14/lXB9ysPjJvHfi4K.png)



查看 Kali 网段的网关地址：192.168.139.2

![image-20241114210657095](https://s2.loli.net/2024/11/14/uFyKwZsXD6PAvfb.png)



右键，将 win10 列为 Target1 ，网关列为 Target2

![image-20241114210839532](https://s2.loli.net/2024/11/14/GImjzT5vkZwVOhE.png)

![image-20241114210935150](https://s2.loli.net/2024/11/14/MDv9e81sfKnUtuh.png)





### ARP 欺骗

选择 Mitm -> ARP posisoning 对目标主机和网关进行 ARP 欺骗

![image-20241114211024322](https://s2.loli.net/2024/11/14/KPli8M1jFwQXWuC.png)

分析：Man in the middle攻击，Kali 会向 Win 10 和网关发送虚假的 ARP 数据包。

对Win10，Kali 会发送 ARP 响应包，声称自己是网关，让 Win10 更新其 ARP 缓存表，将原本网关对应的 MAC 地址替换为 Kali 的 MAC 地址。

同时，对于网关，Kali Linux 也会发送 ARP 响应包，声称自己是 Win 10，让网关更新其 ARP 缓存表，将原本 Win 10 对应的 MAC 地址替换为 Kali 的 MAC 地址。



Kali 的 MAC 地址：

![image-20241114211516676](https://s2.loli.net/2024/11/14/y1qCm4aWviYNLF3.png)

win10 中查看 arp 缓存表：网关MAC地址已经被修改为 Kali MAC 地址，攻击成功

![image-20241114211228184](https://s2.loli.net/2024/11/14/gXMWZTAd2OuvbRi.png)





### dns 欺骗

Plugins -> Manage the plugins

![image-20241114211604431](https://s2.loli.net/2024/11/14/AkCMs3xTtcSpjuX.png)



选择1dns_spoof

![image-20241114211638776](https://s2.loli.net/2024/11/14/Qx814gviPMURhFD.png)



win10 以http 方式访问 www.baidu.com

![image-20241114211854347](https://s2.loli.net/2024/11/14/wIZpmKyzTe41Lil.png)



#### 打开Wireshark抓包



##### **http包：**

![image-20241114212244673](https://s2.loli.net/2024/11/14/ym5pgQI3MnG2zfR.png)

http包中目的ip地址为Windows Server 2016 IP 地址，说明 dns 欺骗成功，百度的域名被解析为 Windows Server 2016 IP地址，因此实际显示的内容为 Windows Server2016提供的页面

![image-20241114212328752](https://s2.loli.net/2024/11/14/SeEYUOdlXc4xFhm.png)



##### dns 包

win 10 请求 www.baidu.com 域名对应ip的dns请求报文：

![image-20241114221327477](https://s2.loli.net/2024/11/14/N8uYocMGeED3RgS.png)

- **通过 ARP 欺骗，原网关地址 192.168.139.2 被解析为 KALI 的MAC 地址**



**kali返回给win10的 dns 响应报文：**

![image-20241114212923415](https://s2.loli.net/2024/11/14/fNy1tVqvEr2iFsO.png)

- 将 百度的域名解析为 192.168.139.131，使得win10 访问到的是 Windows server 2016 IIS服务器提供的页面



且此时 win10可以正常访问网络(Kali作为中间人，将win10请求的内容发送给真实网关，再将真实网关的响应消息返回给 win10)

![image-20241114221615178](https://s2.loli.net/2024/11/14/osg5HSu9AcWDmkh.png)





#### https访问

win10以 https访问目标域名，无法访问

![image-20241114221826665](https://s2.loli.net/2024/11/14/oqPORJcKgi9GuTx.png)

查看此时的dns请求和响应报文：

请求报文：

![image-20241114222839973](https://s2.loli.net/2024/11/14/ZPaVMTtnem1zd9H.png)

响应报文：

![image-20241114222906277](https://s2.loli.net/2024/11/14/mzOJLYR1oyG2EM3.png)

可以发现dns请求和响应报文与 http 是相同的，但是由于 IIS 服务器没有开启 https 服务(即 443 端口不开放)，因此无法访问到对应资源



#### 添加证书



**在服务器上IIS里开启https服务，为其开启443端口，并为其添加一个证书**



安装证书服务：

![image-20241114223909124](https://s2.loli.net/2024/11/14/Uy4nrcSLEBfIWlC.png)



选择服务器证书：

![image-20241115102231685](https://s2.loli.net/2024/11/15/JY3AVREaGbtw8Wo.png)

创建自签名证书：

![image-20241115103112625](https://s2.loli.net/2024/11/15/mpcIKV8feWBw1sl.png)

创建的证书如下：

![image-20241115103207393](https://s2.loli.net/2024/11/15/cHi43svonNWU9gC.png)

绑定网站到 https，开放443端口：

![image-20241115103620353](https://s2.loli.net/2024/11/15/VmRZuzGsUJrK43j.png)





win10以 https 访问 www.baidu.com

![image-20241115104915913](https://s2.loli.net/2024/11/15/2ZEubDLMCgyxvqs.png)



报文分析：

![image-20241115105623012](https://s2.loli.net/2024/11/15/bcSKJj4nozfmyD9.png)

windows server 2016向 win10发送 Server Hello 报文，包括 Certificate 等信息



但由于证书不安全，网关会发送一个 Encrypted Alert ，表示证书验证失败

![image-20241115110329842](https://s2.loli.net/2024/11/15/24TuzIb7wycOBmi.png)
