---
title: 了解JSON Web Token(JWT)
date: 2019-08-08 22:04:06
copyright: true
categories: [计算机知识]
tags: ["Spring Boot",JWT]
---

# 1. 前言

 互联网服务离不开用户认证，一般流程是下面这样。

> 1、用户向服务器发送用户名和密码。
>
> 2、服务器验证通过后，在当前对话（session）里面保存相关数据，比如用户角色、登录时间等等。
>
> 3、服务器向用户返回一个 session_id，写入用户的 Cookie。
>
> 4、用户随后的每一次请求，都会通过 Cookie，将 session_id 传回服务器。
>
> 5、服务器收到 session_id，找到前期保存的数据，由此得知用户的身份。

这种模式的问题在于，扩展性（scaling）不好。单机当然没有问题，如果是服务器集群，或者是跨域的服务导向架构，就要求 session 数据共享，每台服务器都能够读取 session。

举例来说，A 网站和 B 网站是同一家公司的关联服务。现在要求，用户只要在其中一个网站登录，再访问另一个网站就会自动登录，请问怎么实现？

一种解决方案是 session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。

另一种方案是服务器索性不保存 session 数据了，所有数据都保存在客户端，每次请求都发回服务器。JWT 就是这种方案的一个代表。

下面介绍 JWT 的使用。

<!--more-->

# 2. 什么是JWT

JWT是 `Json Web Token` 的缩写。它是基于 [RFC 7519](https://link.jianshu.com/?t=https://tools.ietf.org/html/rfc7519) 标准定义的一种可以安全传输的 **小巧** 和 **自包含** 的 JSON 对象。由于数据是使用数字签名的，所以是可信任的和安全的。JWT 可以使用 HMAC 算法对 secret 进行加密或者使用 RSA 的公钥私钥对来进行签名。

# 3. JWT的工作流程

下面是模拟 JWT 的实际工作流程（假设受保护的 API 在 `/protected` 中）

> 1. 用户导航到登录页，输入用户名、密码，进行登录
> 2. 服务器验证登录鉴权，如果该用户合法，根据用户的信息和服务器的规则生成 JWT Token
> 3. 服务器将该 token 以 json 形式返回（不一定要 json 形式，这里说的是一种常见的做法）
> 4. 用户得到 token，存在 localStorage、cookie 或其它数据存储形式中
> 5. 以后用户请求 `/protected` 中的 API 时，在请求的 header 中加入 `Authorization: Bearer xxxx(token)`。此处注意 token 之前有一个7字符长度的 `Bearer`
> 6. 服务器端对此token进行检验，如果合法就解析其中内容，根据其拥有的权限和自己的业务逻辑给出对应的响应结果
> 7. 用户取得结果

# 4. JWT的数据结构

为了更好的理解 JWT 是什么，我们先来看一个 JWT 生成后的样子

```
eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ3YW5nIiwiY3JlYXRlZCI6MTQ4OTA3OTk4MTM5MywiZXhwIjoxNDg5Njg0NzgxfQ.RC-BYCe_UZ2URtWddUpWXIp4NMsoeq2O6UF-8tVplqXY1-CI9u1-a-9DAAJGfNWkHE81mpnR3gXzfrBAB3WUAg
```

上面这一串的字符串就是一个 JWT 了，虽然说很长，但是仔细看的话就可以看到这个 JWT 分成了三个部分。每个部分用了一个 `.` 来分隔，每段都是用 Base64URL 编码的。JWT 的三部分依次如下

>- Header: 头部
>- Payload: 负载
>- Signature: 签名

写成一行就是下面这个样子

```
Header.Payload.Signature
```

## 4.1 Header

第一部分 Header 是一个 JSON 对象，描述 JWT 的元数据，通常是这个样子的

```json
{
    "alg":"HS256",
    "typ":"JWT"
}
```

> `alg` 属性表示签名的算法 (algorithm)，默认是 HMAC SHA256 
>
> `typ` 属性表示这个令牌 (token) 的类型 (type)，JWT 令牌统一写为 JWT

然后将这个 JSON 对象使用 Base64URL 算法转成字符串。



## 4.2 Payload

第二部分 Payload 也是一个 JSON 对象，用来存放实际需要传递的数据。 JWT 规定了7个官方字段供使用

>- iss (issuer)：签发人
>- exp (expiration time)：过期时间
>- sub (subject)：主题
>- aud (audience)：受众
>- nbf (Not Before)：生效时间
>- iat (Issued At)：签发时间
>- jti (JWT ID)：编号

除了这7个官方字段外，用户还可以在这个部分定义自定义字段，如下

```json
{
    "sub":"sgh",
    "test":"这是一个自定义的字段"
}
```

**注意** ，JWT 默认是不加密的，任何人都可以读到，所以不要把如用户密码等秘密信息放在这个部分。

这个 JSON 对象也要使用 Base64URL 算法转成字符串。

## 4.3 Signature

第三个部分 Signature 是对前两个部分的签名，防止数据被篡改。

首先需要指定一个秘钥 (secret)，此秘钥只有服务器知道，不能泄露给用户，然后使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名

```javascript
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用 `.` 分隔，就可以返回给用户。

## 4.4 Base64URL

前面提到，Header 和 Payload 串型化的算法是 Base64URL。这个算法跟 Base64 算法基本类似，但有一些小的不同。

JWT 作为一个令牌（token），有些场合可能会放到 URL（比如 api.example.com/?token=xxx）。Base64 有三个字符`+`、`/`和`=`，在 URL 里面有特殊含义，所以要被替换掉：`=`被省略、`+`替换成`-`，`/`替换成`_` 。这就是 Base64URL 算法。

# 5. JWT的特点

>1. JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。
>
>2. JWT 不加密的情况下，不能将秘密数据写入 JWT。
>
>3. JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。
>
>4. JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
>
>5. JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。
>
>6. 为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。

# 6. JWT的生成和解析

为了简化我们的工作，这里引入一个比较成熟的 JWT 类库，叫 [jjwt]( [https://github.com/jwtk/jjwt](https://link.jianshu.com/?t=https://github.com/jwtk/jjwt) )。这个类库可以用于 Java 和 Android 的 JWT  token 的生成和验证。

首先要导入 jjwt 的坐标

```xml
<!--JWT-->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

导入完 jjwt 的坐标之后, JWT 的生成可以使用下面这样的代码完成

```JAVA
String generateToken(Map<String, Object> claims) {
    return Jwts.builder()
            .setClaims(claims)
            .setExpiration(generateExpirationDate())
            .signWith(SignatureAlgorithm.HS512, secret) //采用什么算法是可以自己选择的，不一定非要采用HS512
            .compact();
}
```

数据声明 (Claim) 其实就是一个 Map，比如我们想放入用户名，可以简单的创建一个 Map 然后 put 进去就可以了。

```JAVA
Map<String, Object> claims = new HashMap<>();
claims.put(CLAIM_KEY_USERNAME, "sgh");
```

解析也很简单，利用 `jjwt` 提供的 parser 传入秘钥，然后就可以解析 token 了。

```java
Claims getClaimsFromToken(String token) {
    Claims claims;
    try {
        claims = Jwts.parser()
                .setSigningKey(secret)
                .parseClaimsJws(token)
                .getBody();
    } catch (Exception e) {
        claims = null;
    }
    return claims;
}
```

JWT 本身没啥难度，但安全整体是一个比较复杂的事情，JWT 只不过提供了一种基于 token 的请求验证机制。但我们的用户权限，对于 API 的权限划分、资源的权限划分，用户的验证等等都不是 JWT 负责的。也就是说，请求验证后，你是否有权限看对应的内容是由你的用户角色决定的。所以我们这里要利用 Spring 的一个子项目 Spring Security 来简化我们的工作。

具体如何操作，请看 [JWT整合Spring-Security](https://shangguanhong.github.io/2019/08/14/JWT整合Spring-Security/)

# 7. 参考资料

1. [重拾后端之Spring Boot（四）：使用 JWT 和 Spring Security 保护 REST API](https://www.jianshu.com/p/6307c89fe3fa)
2. [JSON Web Token 入门教程 - 阮一峰的网络日志](http://www.baidu.com/link?url=gs1xCBsMDfGGT1Mbq646U12Eodu63eiMLBDp3-JnSTb6v0fGFFnUJqbyJSV0D_UGzisIlXVOfGZ6-os7EQWpTsPnLjwgE0ZxqLOaKy22gOC)