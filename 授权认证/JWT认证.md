## JWT认证
## 一、什么是JWT
在介绍JWT之前，我们先来回顾一下利用token进行用户身份验证的流程：

* 客户端使用用户名和密码请求登录
* 服务端收到请求，验证用户名和密码
* 验证成功后，服务端会签发一个token，再把这个token返回给客户端
* 客户端收到token后可以把它存储起来，比如放到cookie中
* 客户端每次向服务端请求资源时需要携带服务端签发的token，可以在cookie或者header中携带
* 服务端收到请求，然后去验证客户端请求里面带着的token，如果验证成功，就向客户端返回请求数据

这种基于token的认证方式相比传统的session认证方式更节约服务器资源，并且对移动端和分布式更加友好。其优点如下：

* 支持跨域访问：cookie是无法跨域的，而token由于没有用到cookie(前提是将token放到请求头中)，所以跨域后不会存在信息丢失问题
* 无状态：token机制在服务端不需要存储session信息，因为token自身包含了所有登录用户的信息，所以可以减轻服务端压力
* 更适用CDN：可以通过内容分发网络请求服务端的所有资料
* 更适用于移动端：当客户端是非浏览器平台时，cookie是不被支持的，此时采用token认证方式会简单很多
* 无需考虑CSRF：由于不再依赖cookie，所以采用token认证方式不会发生CSRF，所以也就无需考虑CSRF的防御

而JWT就是上述流程当中token的一种具体实现方式，其全称是JSON Web Token。

通俗地说，JWT的本质就是一个字符串，它是将用户信息保存到一个Json字符串中，然后进行编码后得到一个JWT token，并且这个JWT token带有签名信息，接收后可以校验是否被篡改，所以可以用于在各方之间安全地将信息作为Json对象传输。JWT的认证流程如下：

* 首先，前端通过Web表单将自己的用户名和密码发送到后端的接口，这个过程一般是一个POST请求。建议的方式是通过SSL加密的传输(HTTPS)，从而避免敏感信息被嗅探
* 后端核对用户名和密码成功后，将包含用户信息的数据作为JWT的Payload，将其与JWT Header分别进行Base64编码拼接后签名，形成一个JWT Token，形成的JWT Token就是一个如同lll.zzz.xxx的字符串
* 后端将JWT Token字符串作为登录成功的结果返回给前端。前端可以将返回的结果保存在浏览器中，退出登录时删除保存的JWT Token即可
* 前端在每次请求时将JWT Token放入HTTP请求头中的Authorization属性中(解决XSS和XSRF问题)
* 后端检查前端传过来的JWT Token，验证其有效性，比如检查签名是否正确、是否过期、token的接收方是否是自己等等
* 验证通过后，后端解析出JWT Token中包含的用户信息，进行其他逻辑操作(一般是根据用户信息得到权限等)，返回结果

![image-20210626223811598.png](https://pic.imgdb.cn/item/61dcfa432ab3f51d9151b0d6.png)

## 二、为什么要用JWT
### 1、传统Session认证的弊端
我们知道HTTP本身是一种`无状态`的协议，这就意味着如果用户向我们的应用提供了用户名和密码来进行用户认证，认证通过后HTTP协议不会记录下认证后的状态，那么下一次请求时，用户还要再一次进行认证，因为根据HTTP协议，我们并不知道是哪个用户发出的请求，所以为了让我们的应用能识别是哪个用户发出的请求，我们只能在用户首次登录成功后，在服务器存储一份用户登录的信息，这份登录信息会在响应时传递给浏览器，告诉其保存为cookie，以便下次请求时发送给我们的应用，这样我们的应用就能识别请求来自哪个用户了。

然而，传统的session认证有如下的问题：
* 每个用户的登录信息都会保存到服务器的session中，随着用户的增多，服务器开销会明显增大
* 由于session是存在与服务器的物理内存中，所以在分布式系统中，这种方式将会失效。虽然可以将session统一保存到Redis中，但是这样做无疑增加了系统的复杂性，对于不需要redis的应用也会白白多引入一个缓存中间件
* 对于非浏览器的客户端、手机移动端等不适用，因为session依赖于cookie，而移动端经常没有cookie
* 因为session认证本质基于cookie，所以如果cookie被截获，用户很容易收到跨站请求伪造攻击。并且如果浏览器禁用了cookie，这种方式也会失效
* 前后端分离系统中更加不适用，后端部署复杂，前端发送的请求往往经过多个中间件到达后端，cookie中关于session的信息会转发多次
* 由于基于Cookie，而cookie无法跨域，所以session的认证也无法跨域，对单点登录不适用

### 2、JWT认证的优势
对比传统的session认证方式，JWT的优势是：

* 简洁：JWT Token数据量小，传输速度也很快
* 因为JWT Token是以JSON加密形式保存在客户端的，所以JWT是跨语言的，原则上任何web形式都支持
* 不需要在服务端保存会话信息，也就是说不依赖于cookie和session，所以没有了传统session认证的弊端，特别适用于分布式微服务
* 单点登录友好：使用token进行认证的话， token可以被保存在客户端的任意位置的内存中，不一定是cookie，所以不依赖cookie，不会存在这些问题
* 适合移动端应用：使用Session进行身份认证的话，需要保存一份信息在服务器端，而且这种方式会依赖到Cookie，所以不适合移动端

因为这些优势，目前无论单体应用还是分布式应用，都更加推荐用JWT token的方式进行用户认证

## 三、JWT结构
JWT由3部分组成：标头(Header)、有效载荷(Payload)和签名(Signature)。在传输的时候，会将JWT的3部分分别进行Base64编码后用.进行连接形成最终传输的字符串

```java
JWTString = Base64(Header).Base64(Payload).HMACSHA256(base64UrlEncode(header) + “.” + base64UrlEncode(payload), secret)
```

### 1、Header
JWT头是一个描述JWT元数据的JSON对象，alg属性表示签名使用的算法，默认为HMAC SHA256（写为HS256）；typ属性表示令牌的类型，JWT令牌统一写为JWT。最后，使用Base64 URL算法将上述JSON对象转换为字符串保存

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### 2、Payload
有效载荷部分，是JWT的主体内容部分，也是一个JSON对象，包含需要传递的数据。 JWT指定七个默认字段供选择

* iss：发行人
* exp：到期时间
* sub：主题
* aud：用户
* nbf：在此之前不可用
* iat：发布时间
* jti：JWT ID用于标识该JWT

除以上默认字段外，我们还可以自定义私有字段，一般会把包含用户信息的数据放到payload中，如下例：

```sql
{
  "sub": "1234567890",
  "name": "Helen",
  "admin": true
}
```
请注意，默认情况下JWT是未加密的，因为只是采用base64算法，拿到JWT字符串后可以转换回原本的JSON数据，任何人都可以解读其内容，因此不要构建隐私信息字段，比如用户的密码一定不能保存到JWT中，以防止信息泄露。JWT只是适合在网络中传输一些非敏感的信息

### 3、Signature
签名哈希部分是对上面两部分数据签名，需要使用base64编码后的header和payload数据，通过指定的算法生成哈希，以确保数据不会被篡改。首先，需要指定一个密钥（secret）。该密码仅仅为保存在服务器中，并且不能向用户公开。然后，使用header中指定的签名算法（默认情况下为HMAC SHA256）根据以下公式生成签名

```java
HMACSHA256(base64UrlEncode(header) + “.” + base64UrlEncode(payload), secret)
```

在计算出签名哈希后，JWT头，有效载荷和签名哈希的三个部分组合成一个字符串，每个部分用.分隔，就构成整个JWT对象

注意JWT每部分的作用，在服务端接收到客户端发送过来的JWT token之后：
* header和payload可以直接利用base64解码出原文，从header中获取哈希签名的算法，从payload中获取有效数据
* signature由于使用了不可逆的加密算法，无法解码出原文，它的作用是校验token有没有被篡改。服务端获取header中的加密算法之后，利用该算法加上secretKey对header、payload进行加密，比对加密后的数据和客户端发送过来的是否一致。注意**secretKey只能保存在服务端**，而且对于不同的加密算法其含义有所不同，一般对于MD5类型的摘要加密算法，secretKey实际上代表的是盐值。

## 四、JWT的种类
其实JWT(JSON Web Token)指的是一种规范，这种规范允许我们使用JWT在两个组织之间传递安全可靠的信息，JWT的具体实现可以分为以下几种：

### 1、nonsecure JWT：未经过签名，不安全的JWT

未经过签名，不安全的JWT。其header部分没有指定签名算法。

```json
{
  "alg": "none",
  "typ": "JWT"
}
```
并且也没有Signature部分

### 2、JWS：经过签名的JWT
JWS ，也就是JWT Signature，其结构就是在之前nonsecure JWT的基础上，在头部声明签名算法，并在最后添加上签名。创建签名，是保证jwt不能被他人随意篡改。我们**通常使用的JWT一般都是JWS。**

为了完成签名，除了用到header信息和payload信息外，还需要算法的密钥，也就是secretKey。加密的算法一般有2类：
* 对称加密：secretKey指加密密钥，可以生成签名与验签
* 非对称加密：secretKey指私钥，只用来生成签名，不能用来验签(验签用的是公钥)

JWT的密钥或者密钥对，一般统一称为JSON Web Key，也就是JWK

到目前为止，jwt的签名算法有三种：
* HMAC【哈希消息验证码(对称)】：HS256/HS384/HS512
* RSASSA【RSA签名算法(非对称)】（RS256/RS384/RS512）
* ECDSA【椭圆曲线数据签名算法(非对称)】（ES256/ES384/ES512）
### 3、JWE：payload部分经过加密的JWT

## 五、JWT校验
### 1、方案1：网关统一校验
JWT校验无感知，验签过程无侵入，执行效率低，适用于低并发企业级应用。

![dfsfsffffff.png](https://pic.imgdb.cn/item/61dd91c82ab3f51d91cbabed.png)

第一步向登陆中心发起登陆请求，获取基本信息和token。
![20220111222219.jpg](https://pic.imgdb.cn/item/61dd92ba2ab3f51d91cc614c.jpg)

第一步请求接口时，应用网关到校验中心认证，并返回用户权限数据
![3345gfds.png](https://pic.imgdb.cn/item/61dd9e6f2ab3f51d91d684f8.png)

第三步向真实接口发起请求。

### 2、方案2：网关统一校验应用认证方案
控制更加灵活，有一定代码侵入，代码可以灵活控制，适用于追求性能互联网应用![jhgfdfghjk.png](https://pic.imgdb.cn/item/61dd9f802ab3f51d91d755ca.png)

校验可以用注解来完成
```java@GetMapping("/xxx")//自定义注解，利用AOP做验签@CheckJwt public void xxx(){    //Controller代码}
```

## 六、如何实现续签功能
token如果没有过期时间，会留下”太空垃圾”，后患无穷，续签JWT必须有退出机制，让用户重新登陆。

### 1、不允许改变Token令牌实现续签
![khgfdfghj.png](https://pic.imgdb.cn/item/61de42b22ab3f51d912a1562.png)
* 过期时间的被放到后端Redis存储，可以灵活控制* 同时在生成MD5时加入环境特征，尽量避免人为盗取* 但这也意味着JWT是有状态的，也是一种折中方案

### 2、允许改变JWT实现续签
![lkjhgfdszxcvbn.png](https://pic.imgdb.cn/item/61de44022ab3f51d912b1c08.png)

![lkjhtui.png](https://pic.imgdb.cn/item/61de48f52ab3f51d912eec20.png)

超过60分钟后，两个token都过期了，就提示用户进行登陆。

##### 问：为什么必须要两个refresh_token?为什么不直接设置token一个小时过期，判断还有10分钟过期的时候，生成新的token进行替换？
答：这两个token的职责不一样：
* access_token用于业务系统交互，是最核心的数据。
* refresh_token只用于向认证中心获取新的access_token与refresh_token。
* refresh_token的出现本质解决了在用户超过30分钟后，access_token已经失效，此时access_token被送给认证中心是无法解析的，而refresh_token因为生存时间更长，且主体内容与access_token一致，因此被送达认证中心后可以被正确解析，进而重新生成新的access_token与refresh_token。

### 3、续约时的重发JWT问题解决
* 认证中心设计一个计时Map数据结构* 只记录过去n秒内的原始jwt刷新所生成新jwt数据* 几秒内如果发现同样的jwt在再次请求刷新，就返回相同的新jwt数据。

## 七、项目中使用JWT
目前官网(https://jwt.io/)已经提供了很多语言的SDK，按需下载使用即可。

## 八、其他
### 1、工具
* 解密：https://www.box3.cn/tools/jwt.html

TODO:php实现：https://www.jianshu.com/p/a2efb2c8dcde