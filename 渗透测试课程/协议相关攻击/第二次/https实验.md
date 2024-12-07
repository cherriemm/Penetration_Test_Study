# https 实验



## 原理

![image-20241114205445012](https://s2.loli.net/2024/11/14/Knhz8SR3LJkiHo1.png)



### http

![image-20241114205506510](https://s2.loli.net/2024/11/14/JcbexPpy1qksw79.png)



### https

https = http + SSL/TLS

•**SSL:** **Secure Socket Layer** **安全套接层**

•**TLS:** **Transport Layer Security** **传输层安全协议**



**加密方式：**

![image-20241114205646125](https://s2.loli.net/2024/11/14/qpAvW5yflcgsHw2.png)



**PKI（公钥基础设施）**

- 使用公钥技术和数字签名来保证信息安全

- 由公钥密码算法、数字证书（Certificate）、CA（Certificate Authority）证书颁发机构、RA（Registration Authority）证书注册机构组成



**实现功能**

- 身份验证：证书

- 数据完整性：完整性校验 MD5/SHA

- 数据机密性：非对称加密算法

- 操作不可否认/抵赖性：数字签名

![image-20241114205904162](https://s2.loli.net/2024/11/14/rgCcH7Y6ij5wJtU.png)

![image-20241114205929568](https://s2.loli.net/2024/11/14/etZ14PU7l6hdFyH.png)



![image-20241114205949260](https://s2.loli.net/2024/11/14/dNbcJTpsVwu8ZYt.png)





## TLSv1.2

抓取与 ip地址为 117.85.69.236 的 TLSv1.2 报文

![image-20241113213941425](https://s2.loli.net/2024/11/13/fceaolutpHLqX1d.png)





### 交互流程



#### Client Hello



![image-20241113214608378](https://s2.loli.net/2024/11/13/I6LO2r98W4tvdkn.png)

客户端发起连接请求，向服务器发送 Client Hello 消息。此消息包含以下重要信息：

- **客户端支持的协议版本**：表明客户端支持 TLSv1.2 协议

- **随机数（Client Random）**：一个 32 字节的随机值，用于后续密钥生成等操作。
- **会话 ID（Session ID）**：如果客户端希望恢复之前的会话，则会包含相应的会话 ID；若为新会话，则会话 ID 为空(此处是新会话，故会话长度为空)

- **密码套件列表（Cipher Suites）**：客户端支持的加密算法组合列表，例如 TLS_RSA_WITH_AES_128_CBC_SHA256 等，每个密码套件包含密钥交换算法、加密算法、消息认证码（MAC）算法等信息。





####  Server Hello



![image-20241113214723758](https://s2.loli.net/2024/11/13/3SIrpz5yonQHNJD.png)

服务器收到客户端的 Client Hello 消息后，选择合适的参数并回复 Server Hello 消息。该消息包含：

- **选择的协议版本**：确认使用 TLSv1.2 协议。
- **随机数（Server Random）**：服务器生成的 32 字节随机值，与客户端随机数一起用于后续密钥生成。
- **会话 ID**：如果服务器同意恢复会话，则返回对应的会话 ID；若不同意或无法恢复，则返回一个新的会话 ID。(此处是新会话，故会话长度为空)
- **选择的密码套件**：从客户端提供的密码套件列表中选择一个服务器支持且认为安全合适的密码套件。



#### Certificate

![image-20241113215006591](https://s2.loli.net/2024/11/13/pz4eaCkTxKl7Vvr.png)

服务器向客户端发送数字证书，具体内容如下：

![image-20241113215255782](https://s2.loli.net/2024/11/13/dmtbuw9lUogFxAQ.png)

证书中包含服务器的公钥等信息。客户端将使用此证书验证服务器的身份，确保通信对方是可信的



#### Server Key Exchange

![image-20241113215504497](https://s2.loli.net/2024/11/13/KGfLEbmYx86MSdN.png)

如果所选的密码套件需要服务器进行密钥交换，服务器会在此步骤发送密钥交换参数。这些参数用于与客户端共同生成会话密钥。例如，在 Diffie - Hellman 密钥交换中，服务器会发送 Diffie - Hellman 公钥等相关参数



#### Server Hello Done

![image-20241113215616476](https://s2.loli.net/2024/11/13/iGtDu6dQW4YXUMB.png)

服务器发送 Server Hello Done 消息，表示服务器的 Hello 阶段结束，等待客户端的响应。



#### Client Key Exchanged

![image-20241113215734773](https://s2.loli.net/2024/11/13/raHAgkWNTlsZ5ot.png)

根据服务器选择的密码套件和密钥交换方式，客户端进行相应的密钥交换操作。客户端根据服务器提供的 Diffie - Hellman 参数计算自己的 Diffie - Hellman 公钥并发送给服务器，同时结合服务器的 Diffie - Hellman 公钥计算出共享密钥



**`Change Cipher Spec`消息**

![image-20241113215955081](https://s2.loli.net/2024/11/13/PrjYLOsEaVu5NnQ.png)

客户端发送一个 Change Cipher Spec 消息，表示后续的通信将使用新协商的加密套件和密钥进行加密



## TLSv1.3



抓取与 ip地址为 117.21.204.45 的 TLSv1.3 报文

![image-20241113221441302](https://s2.loli.net/2024/11/13/oH4tXGPdkaMqrme.png)



### 交互流程



#### Client Hello

![image-20241113221652441](https://s2.loli.net/2024/11/13/xqeVFAkSvaPnw41.png)

- 在 TLSv1.3 中，可用的密码套件相比 TLSv1.2 有所减少，主要选择更安全和高效的算法。





#### Server Hello

![image-20241113221806294](https://s2.loli.net/2024/11/13/ybPcg4aSFtlk9jK.png)

- TLSv1.3 的密钥协商机制更简洁：服务器在回复中直接确定加密套件和其他相关参数，不再有单独的 Server Key Exchange 消息（在大多数情况下） 
- 优化了握手过程：服务器可以在回复中直接包含完成握手所需的所有信息（如选择的加密套件、证书、加密的扩展信息和 Finished 消息等
- 服务器发送的加密扩展（Encrypted Extensions）消息对扩展信息进行加密，增强了安全性和隐私性。





## 比较



与 TLSv1.2 相比，TLSv1.3 的握手过程有以下显著特点：

- **握手轮次减少**：

  TLSv1.3 通过优化握手协议，服务器可以在回复中直接包含完成握手所需的所有信息,  减少了握手过程中的往返次数，从而降低了延迟。

- **安全性增强**：

  TLSv1.2 支持的密码套件种类较多，包括一些可能存在潜在安全风险的较旧算法组合，如基于早期 RSA 和一些较简单的哈希算法的密码套件。TLSv1.3 只支持更安全和现代的密码套件，如基于椭圆曲线密码学（ECC）和更安全的哈希算法的组合，淘汰了很多安全性较低的密码套件。

- **隐私保护更好**：

  TLSv1.3 减少了握手过程中不必要的信息泄露，提高了通信双方的隐私保护。例如，TLSv1.3 在早期的握手阶段就开始对扩展等信息进行加密传输，防止攻击者获取这些信息来推测通信内容或进行其他恶意攻击。而TLSv1.2 一些扩展信息在握手前期可能不加密