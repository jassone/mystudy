## 一、普通网络加解密弊端
### 1、首先想到的对称加密
![B54113BE-2023-4234-8548-A86B32872216.png](https://pic.imgdb.cn/item/616e0b182ab3f51d91616610.png)

对称加密
* 服务端用同一个key,失去了加密的意义，相当于明文
* 不同的客户端、服务器数量庞大，所以双方都需要维护大量的密钥，维护成本很高
* 因每个客户端、服务器的安全级别不同，密钥极易泄露

### 2、再看看非对称加密
![5ED48197-8F43-4AA6-904E-D03F7FE31000.png](https://pic.imgdb.cn/item/616e0f582ab3f51d91647f46.png)

非对称加密
* 客户端到服务端是安全的(因为只有服务端才能用私钥解开数据，但是客户端可以伪造加密数据)，
* 服务端到客户端是不安全的(因为黑客可以拦截到数据并解密出来，但是黑客不能将数据再次加密)

### 3、那对称加密+非对称加密怎么样呢
![D6507D24-CE76-4333-8524-9A505D250B71.png](https://pic.imgdb.cn/item/616e41dc2ab3f51d91866e16.png)
对称加密+非对称加密
* 客户端如何获取公钥
* 这时候黑客可以在中间代理(获取到key1和key3)，获取明文数据，就是**中间人问题**

所以这时候如果能确认访问的服务端就是真实的服务端，那么就完美了，这时候就要用到SSL证书。 

## 二、SSL/TLS
SSL/TLS是一个安全通信框架，上面可以承载HTTP协议或者SMTP/POP3协议/Telent协议等。

SSL（Secure Socket Layer，安全套接字层）：1994年为 Netscape 所研发，SSL 协议位于 TCP/IP 协议与各种应用层协议之间，为数据通讯提供安全支持。

TLS（Transport Layer Security，传输层安全）：其前身是 SSL，它最初的几个版本（SSL 1.0、SSL 2.0、SSL 3.0）由网景公司开发，1999年从 3.1 开始被 IETF 标准化并改名，发展至今已经有 TLS 1.0、TLS 1.1、TLS 1.2 三个版本。SSL3.0和TLS1.0由于存在安全漏洞，已经很少被使用到。TLS 1.3 改动会比较大，目前还在草案阶段，目前使用最广泛的是TLS 1.1、TLS 1.2。

目前TLS几乎已经替代了SSL，只是习惯叫法还是为SSL。

SSL基于TCP，SSL不是简单地单个协议，而是两层协议；
* SSL记录协议（SSL Record Protocol）：它建立在可靠的传输协议（如TCP）之上，为高层协议提供数据封装、压缩、加密等基本功能的支持。
* SSL握手协议（SSL Handshake Protocol）：它建立在SSL记录协议之上，用于在实际的数据传输开始前，通讯双方进行身份认证、协商加密算法、交换加密密钥等。

SSL协议两个重要概念，SSL会话，SSL连接；SSL连接是点到点的连接，而且每个连接都是瞬态的，每一个链接都与一个会话关联。SSL会话是一个客户端和一个服务器之间的一种关联，会话由握手协议（Handshake Protocol）创建，所有会话都定义了一组密码安全参数，这些安全参数可以在多个连接之间共享，会话可以用来避免每一个链接需要进行的代价高昂的新的安全参数协商过程。

## 三、HTTPS
![v2-54ff04e1b0cc698f08f76d6356f59fac_1440w.png](https://pic.imgdb.cn/item/616ec5e62ab3f51d91f151b1.png)
如上图所示 HTTPS 相比 HTTP 多了一层 SSL/TLS。即HTTPS是HTTP的加密版，底层使用的加密协议是SSL。

HTTPS并非是应用层的一种新协议，只是http通信接口部分用SSL/TLS协议代替而已。通常HTTP直接和TCP通信，当用SSL时，就变成了先和SSL通信，再由SSL和TCP通信。

HTTPS=非对称加密+对称加密+HASH+CA.

### 1、加密方式的选择
共享密钥加密 + 对称密钥加密

HTTPS采用共享密钥加密和公开密钥加密混合的加密方式，在交换密钥对环节使用公开密钥加密方式（防止被监听泄漏密钥）加密共享的密钥，在随后的通信过程中使用共享密钥的方式使用共享的密钥进行加解密。

### 2、认证方式实现
CA证书申请下发认证流程-图1
![v2-7c78935389af46e197e96d9cd91c06dd_1440w.jpe](https://pic.imgdb.cn/item/616f62ad2ab3f51d91619fd2.jpg)

CA证书申请下发认证流程-图2
![04d88fe98c310ea629dd1588b001629c.png](https://pic.imgdb.cn/item/616f5fe42ab3f51d91604215.jpg)

##### 2-1 服务器的运营人员向CA机构提交证书签名请求 (CSR, Certificate Signing Request)。

![3740434386-5afbfa765758b_fix732.png](https://pic.imgdb.cn/item/616f6c7b2ab3f51d916774f0.png)

1）证书申请者(一般是公司、个人开发者等）需要使用加密算法(大部分是RSA算法）来生成**私钥**，其中私钥保存在服务器上，不提供给任何人。

2）使用私钥生成证书签名请求 (CSR, Certificate Signing Request)【CSR中附带有公钥】，需要提供一些申请者相关的信息(域名、拥有者，**公钥**等信息），发送给CA机构请求他们的签名，经过他们签名才能成为被信任的证书。

3）CA机构对申请者进行验证(如组织是否存在、企业是否合法，是否拥有域名的所有权等），验证通过后，将会在证书中添加部分信息(证书颁发机构、有效期、指纹/hash算法、签名算法），**形成新的证书**。(CA机构也是使用其自身的私钥来签名其他证书的）

4）对新的证书进行签名，步骤如下：
* 对新证书按照指纹/hash算法进行hash值计算
* **使用CA机构的私钥**对计算出来的hash值按照签名算法进行加密，得出的密文就是数字签名；
* 将数字签名放在证书的最后面【签名完成】，得到**完整的证书**，并返回给申请者；
* 申请者获取签名证书，保证自己的HTTPS服务被信任。

证书基本包括但不限于的内容如下：
* 证书的有效期
* 公钥
* 证书所有者（Subject）
* 签名所使用的算法
* 指纹以及指纹算法【hash算法】
* Version Number，版本号。
* Serial Number，序列号。
* 颁发机构
* 数字签名

##### 2-2 客户端识别证书
![1701189870-5afc1294cdfb5_fix732.png](https://pic.imgdb.cn/item/616f6cda2ab3f51d9167b60b.png)

1) 浏览器访问某https服务，带上浏览器自身支持的一系列Cipher Suite（密钥算法套件，后文简称Cipher）[C1,C2,C3, …]发给服务器(算法套件包括非对称算法、对称算法、hash算法）；

2) 服务器接收到浏览器的所有Cipher后，与自己支持的套件作对比，如果找到双方都支持的Cipher(双方套件中都能支持的最优先算法），则告知浏览器，并返回自己的证书；

3) 浏览器确认和服务端后续的密文通信的加密算法，同时对证书进行认证；

4) 使用证书中的认证机构(可能是二级、三级机构）的**公钥**(系统、浏览器事先准备好的）**按照证书中的签名算法**进行解密(认证机构使用私钥加密的密文只能通过该认证机构的公钥进行解密），解密获取hash值；

5) 使用证书中的hash/摘要算法对证书信息(证书签名除外的信息[服务器公钥、有效期等]）hash值计算，和4中解密的hash值进行对比，如果相等，表示证书值得信任；这时候客户端明确两件事：1，认证服务器发来的是真实有效的证书(没有被篡改)；2，证书中包含的公钥是值得信赖的。

6) 客户端然后验证证书相关的域名信息、有效时间等信息；

### 3、证书信任的方式
1. 操作系统和浏览器内置
    每个操作系统和大多数浏览器都会内置一个知名证书颁发机构的名单，会内置信任CA的证书信息(包含公钥)，如果CA不被信任，则找不到对应CA的证书，证书也会被判定非法。

2. 证书颁发机构
    CA（ Certificate Authority，证书颁发机构）是被证书接受者（拥有者）和依赖证书的一方共同信任的第三方。

3. 手动指定证书
    所有浏览器和操作系统都提供了一种手工导入信任证书的机制。至于如何获得证书和验证完整性则完全由你自己来定。

### 1、HTTPS交互大致流程
![v2-5e2241fae8b593ff7f3b3a308ef81c10_1440w.png](https://pic.imgdb.cn/item/616e691a2ab3f51d91a52ff6.png)

如上图所示
1. 在第 ② 步时服务器发送了一个SSL证书给客户端。

2. 客户端在接受到服务端发来的SSL证书时，会对证书的真伪进行校验。

3. 验证通过后，浏览器就可以读取证书中的公钥

4. 使用上面的公钥将对称加密秘钥加密，发送给服务端协商，服务端再回复给客户端协商可行，然后这个对称加密秘钥将用于后面加密的秘钥。
  
### 2、HTTPS交互详细流程
![4156566902-5afc21f239e2b_fix732.png](https://pic.imgdb.cn/item/61717bc92ab3f51d91f8dbc2.png)


```
Client                                               Server

ClientHello:HandShake       -------->
                                                ServerHello:Handshake
                                               Certificate*:Handshake
                                         ServerKeyExchange*:Handshake
                                        CertificateRequest*:Handshake
                             <--------      ServerHelloDone:Handshake
Certificate*:Handshake
ClientKeyExchange:Handshake
CertificateVerify*:Handshake
[ChangeCipherSpec]
Finished:Handshake           -------->
                                                   [ChangeCipherSpec]
                                                   [Encrypted_handshake_message]
                             <--------             Finished:Handshake
Application Data             <------->               Application Data
```

1. 客户端->服务端。客户端（通常是浏览器）先向服务器发出加密通信的请求(使用443端口)。

     1-1 client Hello 
    （1）支持的协议版本列表(加密套件(Cipher Suite))，比如TLS 1.1，TLS 1.2等。
    （2）支持的加密算法列表(Cipher Suites)，比如RSA公钥加密。
    （3）支持的压缩算法列表(Compression Methods)
    （4） 一个客户端生成的随机数 random_C，稍后用于生成"对话密钥"。
    （5） 其他。

2. 服务端->客户端。服务器收到请求,然后响应(可能一次也可能分多次)

    2-1 server Hello
（1） 确认选定的加密协议版本，比如TLS 1.2版本。如果浏览器与服务器支持的版本不一致，服务器关闭加密通信。
（2） 确认选定的加密算法，比如RSA公钥加密。
（3） 确认选定的压缩方法
（4） 服务器生成的随机数random_S，稍后用于生成"对话密钥"。

    2-2 Certificate(返回的证书内容，包括公钥，签名等)
    
    2-3 Server Key Exchange(返回密钥交换算法所需要的额外参数)
    2-4 Server Hello Done(服务器已经将所有预计的握手消息发送完毕)

3. 客户端->服务端。客户端收到证书后进行验证+生产随机数

    此时客户端已经获取全部的计算协商密钥需要的信息：两个明文随机数 random_C 和 random_S 与自己计算产生的 Pre-master secret，计算得到协商密钥。
    master_secret=Fuc(random_C, random_S, Pre-Master secret)。
    
    3-1 Client Key Exchange (合法性验证通过后，客户端计算产生随机数字 Pre-master secret，并用证书公钥加密，发送给服务器)
    3-2 Change Cipher Spec (客户端通知服务器后续的通信都采用协商的通信密钥和加密算法进行加密通信，参数协商完成)
    3-3 Finished（encrypted_handshake_message，结合之前所有通信参数的 hash 值与其它相关信息生成一段数据，采用协商密钥 session key 与算法进行加密，然后发送给服务器用于数据与握手验证）

 ##### Pre-Master Secret
     (1）Pre-Master Secret是在客户端使用RSA或者Diffie-Hellman等加密算法生成的。它将用来跟服务端和客户端在Hello阶段产生的两个随机数结合在一起生成 Master Secret。在客户端使用服务端的公钥对Pre-Master Secret进行加密之后传送给服务端，服务端将使用私钥进行解密得到Pre-Master secret。也就是说服务端和客户端都有一份相同的Pre-Master secret和随机数。
    
      (2）Pre-Master secret前两个字节是TLS的版本号，这是一个比较重要的用来核对握手数据的版本号，因为在Client Hello阶段，客户端会发送一份加密套件列表和当前支持的SSL/TLS的版本号给服务端，而且是使用明文传送的，如果握手的数据包被破解之后，攻击者很有可能串改数据包，选择一个安全性较低的加密套件和版本给服务端，从而对数据进行破解。所以，服务端需要对密文中解密出来对的Pre-Master版本号跟之前Client Hello阶段的版本号进行对比，如果版本号变低，则说明被串改，则立即停止发送任何消息。

4. 服务端->客户端。
    4-1 服务器用私钥解密加密的 Pre-master secret数据，基于之前交换的两个明文随机数 random_C 和 random_S，计算得到协商密钥:master_secret=Fuc(random_C, random_S, Pre-Master secret);
    4-2 计算之前所有接收信息的hash值，然后解密客户端发送的 Encrypted_handshake_message，验证数据和密钥正确性;
    4-3 change_cipher_spec, 验证通过之后，服务器同样发送 change_cipher_spec 以告知客户端后续的通信都采用协商的密钥与算法进行加密通信;
    4-4 encrypted_handshake_message, 服务器也结合所有当前的通信参数信息生成一段数据并采用协商密钥 session key与算法加密并发送到客户端;

5. 客户端。
客户端计算所有接收信息的hash值，并采用协商密钥解密 encrypted_handshake_message，验证服务器发送的数据和密钥，验证通过则握手完成;

6. 加密通信。开始使用协商密钥与算法进行加密通信(**后面和服务器80端口交互，只加密报体**)。

    client/server皆是通过该master secret来产生对称加密的session key，故client知道server端的session key，server端也知道client端的session key，为此能够发送/接收加密的信息。

    这里的session key， 包括6个部分:

    | 秘钥名称 | 秘钥作用 |
    | --- | --- |
    |client_write_MAC_key[SecurityParameters.mac_key_length]  | 客户端发送数据使用的摘要MAC算法 |
    | server_write_MAC_key[SecurityParameters.mac_key_length] | 服务端发送数据使用摘要MAC算法 |
    |client_write_key[SecurityParameters.enc_key_length]  |  客户端数据加密，服务端解密|
    |server_write_key[SecurityParameters.enc_key_length]  |  服务端加密，客户端解密|
    |  client_write_IV[SecurityParameters.fixed_iv_length]| 初始化向量，运用于分组对称加密|
    | server_write_IV[SecurityParameters.fixed_iv_length] |  初始化向量，运用于分组对称加密|

1. 然后再后续的交互中就使用session key和MAC算法的秘钥对传输的内容进行加密和解密。

    具体的步骤是先使用MAC秘钥对内容进行摘要，然后把摘要放在内容的后面使用session key再进行加密。对于客户端发送的数据，服务器端收到之后，需要先使用client_write_key进行解密，然后使用client_write_MAC_key对数据完整性进行验证。服务器端发送的数据，客户端会使用server_write_key和server_write_MAC_key进行相同的操作。


### 相关疑问
1. HTTPS解决了什么问题

    1）内容加密，防止中间人攻击（man-in-the-middle attacks）
    2）身份认证(防止钓鱼网站)，防止DNS劫持
    3）数据完整性检查，防止内容被第三方冒充或者篡改

1. 中间人有可能篡改该证书吗？

    假设中间人篡改了证书的原文，由于他没有CA机构的私钥，所以无法得到此时加密后签名，无法相应地篡改签名。浏览器收到该证书后会发现原文和签名解密后的值不一致，则说明证书已被篡改，证书不可信，从而终止向服务器传输信息，防止信息泄露给中间人。

1. 中间人有可能把证书掉包吗？

    假设有另一个网站B也拿到了CA机构认证的证书，它想劫持网站A的信息。于是它成为中间人拦截到了A传给浏览器的证书，然后替换成自己的证书，传给浏览器，之后浏览器就会错误地拿到B的证书里的公钥了，这确实会导致上文“中间人攻击”那里提到的漏洞？

    其实这并不会发生，因为证书里包含了网站A的信息，包括域名，浏览器把证书里的域名与自己请求的域名比对一下就知道有没有被掉包了。

1. 签名为什么需要加密计算的hash值，hash值已经是单向不可逆的运算了？

    因为虽然hash值是单向的，但是计算hash的算法和内容都是公开的，如果不进行加密，那么由于其他人可以修改证书内容，根据hash算法重新计算hash，这样就会出现安全漏洞，所以使用加密的密文才是安全的。

1. 为什么制作数字签名时需要hash一次？

    似乎hash有点多余，把hash过程去掉也能保证证书没有被篡改。

    最显然的是性能问题，前面我们已经说了非对称加密效率较差，证书信息一般较长，比较耗时。而hash后得到的是固定长度的信息（比如用md5算法hash后可以得到固定的128位的值），这样加解密就快很多。

    当然也有安全上的原因。

1. 每次进行HTTPS请求时都必须在SSL/TLS层进行握手传输密钥吗？

    每次请求都经历一次密钥传输过程非常耗时，那怎么达到只传输一次呢？

    **服务器**会为每个浏览器（或客户端软件）维护一个session ID，在TLS握手阶段传给浏览器，浏览器生成好密钥传给服务器后，服务器会把该密钥存到相应的session ID下，之后浏览器每次请求都会携带session ID，服务器会根据session ID找到相应的密钥并进行解密加密操作，这样就不必要每次重新制作、传输密钥了！
    
    还有一个session ticket 需要服务器和客户端都支持， 二者对比，主要是保存协商信息的位置与方式不同，类似与http中的session与cookie。
    二者都存在的情况下，(nginx 实现)优先使用 session_ticket。

1. CA机构认证的作用？

    可以作为全球有限的权威认证，经过其签名的证书都可以视为可信任的，所以他们的私钥必须要保证不被泄露，如果泄露的话需要及时的进行吊销。
    
1. 为什么要有随机数，为什么在客户端生成？

    随机数是作为后续整个密文加解密的关键秘钥，只有获取这个随机数的人才可以看到通信的内容，保证通信的安全；通过客户端产生是因为会话的发起者是用户端，为了保证用户端的唯一，以及保证服务端和客服端的会话内容不被篡改，如果是服务端来生成的话，第三方可以通过公钥来解密服务端加密的随机数，存在不安全因素。
    
1. 证书验证完成后，为什么客户端需要和服务端互相发送一次签名信息？

    证书验证完成以后，需要传递一个随机数，使用公钥、私钥进行非对称加解密，后面在发送内容消息的前面是为了验证通过随机数进行对称加解密，保证双方获取的数据正确性。
    
1. 为什么后面数据传输使用对称加密
    首先，非对称加密的加解密效率是非常低的，而 http 的应用场景中通常端与端之间存在大量的交互，非对称加密的效率是无法接受的；

    另外，在 HTTPS 的场景中只有服务端保存了私钥，一对公私钥只能实现单向的加解密，所以 HTTPS 中内容传输加密采取的是对称加密，而不是非对称加密。
    
1. charls抓取https的原理
    charls充当了中间人，并给你下发证书，且需要你信任这证书。charls和服务器交互部分，就是常规的https流程。





 
