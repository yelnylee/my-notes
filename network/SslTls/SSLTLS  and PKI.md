# SSL/TLS  and PKI---第一章

* SSL/TLS

  * SSL(Secure socket layer) 安全套接字层协议

  * TLS(Translate layer secure)传输层安全协议

    * TLS的四个目标

      * 加密安全

        为通信双方(客户端/服务器端)启用安全通道。

      * 互操作性

        ??

      * 可扩展性

        可以在不改变协议的基础上扩展不同加密基元(算法：RSA/ASE)

      * 效率

        实现上述目标的基础上保持性能成本在可接受的范围内。这需要尽量减少昂贵的加密操作的执行次数，并提供一个会话缓存方案，以避免重复的加密操作仍然在后续的连接中被执行。

* 网络层

  * OSI(open systems interconnection)开发系统互联模型

    | OSI层      | 描述                                                 | 协议示例           |
    | ---------- | ---------------------------------------------------- | ------------------ |
    | 应用层     | 应用程序工作的抽象层                                 | HTTP/SMTP/FTP/IMAP |
    | 表示层     | 数据表示、转换和加密（OS操作系统工作的抽象层）       | SSL/TLS            |
    | 会话层     | 多连接管理（OS操作系统工作的抽象层）                 | -                  |
    | 传输层     | 包或流的可靠传输（OS操作系统工作的抽象层）           | TCP、UDP           |
    | 网络层     | 网络节点间的路由与数据分发（OS操作系统工作的抽象层） | IP、IPSec          |
    | 数据链路层 | 可靠的本地数据连接（LAN）                            | 以太网             |
    | 物理层     | 直接物理数据连接（电缆）                             | CAT5               |
    
    网络通信协议的分层清晰的划分了网络通信的概念。
    
    不同层次的协议可以加入通信或者从通信中移除，一种底层协议可以服务于多种上层协议。
    
    SSL/TLS是这一原则在实践中的一个重要示例。当不需要加密时，可以将TLS从模型中去掉，并不影响网络通信

* SSL/TLS协议的历史

  * 1995年SSL3.0由网景公司发布、2015年弃用

  * 1999年在Microsoft的参与中改名为TLS，TLS1.0问世，计划2020年弃用

  * 2006年TLS1.1发布，计划2020年弃用

  * 2008年TLS1.2发布，添加了对已验证加密的支持？？并删除了所有硬编码的安全基元(算法等RSA/ASE),是协议扩展性增强

  * 2018年TLS1.3发布

  * 参考资料

     [https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A#TLS_1.3](https://zh.wikipedia.org/wiki/傳輸層安全性協定#TLS_1.3)

* 密码学

   部署正确的密码能解决安全的三个核心需求：保持秘密（机密性）、验证身份（真实性）、以及保证传输安全（完整性）

  * 构建基块

    * 加密基元（cryptographic primitive）介绍

      * 对称加密（symmetric encryption）又称私钥加密（private-key cryptography）

      * 明文（plaintext）

      * 密钥（cipher）用于加密、解密

        * 密钥空间

        * 序列密码（stream cipher）

          * 密钥序列（keystream）

            ![image-20200320145839304](C:\Users\liguoqi.CORPDOM\AppData\Roaming\Typora\typora-user-images\image-20200320145839304.png)

          * 异或运算（XOR或者^表示，基于二进制位的运算）

            * 参考资料 

              https://blog.csdn.net/jerry99s/article/details/46485417 

          * 缺陷

            知道明文和密文的情况下，推测出部分密钥

        * 分组密码(block cipher)

          * 分组密码模式（block cipher mode） 
          * AES(advanced encryption standard) 128/192/256
          * 缺陷
            * 只能使用它们加密，长度等于加密块大小的数据
            * 分组密码确定的情况下，相同的输入，输出也会相同

        * 填充

          * 用于解决处理数据长度小于加密块大小的数据加密

          * 填充数据不能是随机的，必须遵循某种格式，这样接收方才能发现填充并了解需要丢弃多少字节

          * TLS填充示例

            ![image-20200321003643607](C:\Users\liguoqi.CORPDOM\AppData\Roaming\Typora\typora-user-images\image-20200321003643607.png)

      * 散列函数（Hash function）

        * 密码学上散列的特性
        * 特点是
        * SHA256
        * 消息验证代码（Message Authentication Code/MAC）

    * 分组密码模式

      分组密码模式是为了加密任意长度的数据二设计的密码学方案，是对分组密码的扩展

      * 电码本模式（electronic codebook，ECB）

        * 任意长度数据填充后都是加密块的整数倍，每个加密块进行加密

        * 缺点

          分组加密的特性，输入相同，输出相同

      * 加密块链接模式（cipher block chaining，CBC）

        * 在ECB基础上增加了IV(initialization vector),IV是解密时必须的值

          ![image-20200323023718237](C:\Users\liguoqi.CORPDOM\AppData\Roaming\Typora\typora-user-images\image-20200323023718237.png)

    * 非对称加密（asymmetric encryption/public key cryptography）

      * 非对称加密可以解决密码学中的那些问题？

      * RSA

      * 非对称加密的毒药特性，以毒攻毒

        公钥加密，私钥解

        私钥加密，公钥解

        单一作用在明文上得到的密文，都无法解密

      * ???

    * 数字签名？？

    * 随机数生成？？

  * 协议

    加密基元本身没有什么作用，只有将这些元素组成协议和方案，才能满足复杂的安全需求，

    安全需求：机密性、真实性、完整性

  * 攻击密码

    * 攻击的方式

  * 衡量强度

    * 通过攻破某个加密基元的操作数衡量密码系统的强度

  * 中间人攻击



# 协议 ---第二章

* 记录协议

  * TLS记录

    ![image-20200323151501117](C:\Users\liguoqi.CORPDOM\AppData\Roaming\Typora\typora-user-images\image-20200323151501117.png)

  * TLS协议的四个核心子协议

    握手协议（handshake protocol）、密钥规格变更协议（change cipher spec protocol）、应用数据协议（application data protocol）、警报协议（alert protocol）

* 握手协议

  用于通信双方协商连接参数，并且完成身份验证

* 密钥交换

* 身份验证

* 加密

* 重新协商

  * 部署客户端证书的两种方法
    * 要求连接到网站的所有连接都需要客户端证书，但是这种方式对那些（还）没有证书的使用者并不友好；在连接成功以前，你无法向它们发送任何信息和说明。处理错误的情况同样不可能。
    * 许多操作员会选择允许连接到网站根路径的连接不携带证书，并设计一个需要提供客户端证书的子区域。当用户打算浏览子区域时，服务器发起重新协商请求，要求客户端提供证书。
  * i需要重新协商情形
    * 第二种方式需要重新协商

* 应用数据协议

  * 应用数据协议携带着应用数据，即数据缓冲区，记录层使用当前连接的协商后的安全参数对消息内容进行打包，碎片整理，加密

* 警报协议

* 关闭连接

* 密码操作

* 密码套件

* 扩展

* 协议限制

* 协议间的版本差异

  

  # 公钥基础设施PKI(Public Key In..)---第三章

* 互联网公钥基础设施

  * CA(Certification Authority)证书颁发机构

* 标准

* 证书

  证书是一个包含公钥、订阅人相关信息以及证书颁发者数字签名的数字文件，也就是一个让
  我们可以交换、存储和使用公钥的壳。因此，证书成为了整个PKI体系的基础组成元素

  * 证书的格式？？

     https://amon.org/ca-famats 

  * 自签名证书

    在自签名证书里，使用者
    （subject）和颁发者（issuer）字段的可分辨名称是一样的

* 证书链

* 信赖方

* 证书颁发机构

* 证书生命周期

* 吊销

  * 证书吊销的两种机制

    * CRL(Certification Revocation List)证书吊销列表

      证书吊销列表(什么是证书吊销列表(CRL)？吊销列表起什么作用)，一个单独的文件。该文件包含了 CA 已经吊销的证书序列号(唯一)与吊销日期，同时该文件包含生效日期并通知下次更新该文件的时间，当然该文件必然包含 CA 私钥的签名以验证文件的合法性。

      证书中一般会包含一个 URL 地址 CRL Distribution Point，通知使用者去哪里下载对应的 CRL 以校验证书是否吊销。该吊销方式的优点是不需要频繁更新，但是不能及时吊销证书，因为 CRL 更新时间一般是几天，这期间可能已经造成了极大损失。

    * OCSP(Online Certification Status Protocol)证书状态在线查询协议

       Online Certificate Status Protocol, 证书状态在线查询协议，一个实时查询证书是否吊销的方式。请求者发送证书的信息并请求查询，服务器返回正常、吊销或未知中的任何一个状态。证书中一般也会包含一个 OCSP 的 URL 地址，要求查询服务器具有良好的性能。部分 CA 或大部分的自签 CA (根证书)都是未提供 CRL 或 OCSP 地址的，对于吊销证书会是一件非常麻烦的事情。 

* 弱点

* 根密钥泄露

* 生态系统评估

* 进步

  

  # PKI 攻击

  

  

