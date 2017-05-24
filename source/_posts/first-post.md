---
title: 浅谈使用Json Web Token和Cookie的利弊
date: 2017-05-22 14:23:41
tags: 
- JWT 
categories: browser
author: CP
---

## Authentication
现在基本上有两种不同的方式来处理服务器端的认证（Authentication）问题。

1. Cookie-Based Authentication:最常用的是利用cookie来存储认证信息，并包含在发送请求中用于服务器端的核查
2. Token-Based Authentication:本文主要介绍的Token为基础的认证方式。

## Json Web Token(JWT)
JWT是一个以JSON为基准的标准规范，通过JSON来传递消息，具体定义可以参见[这里](http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html)
JWT自身会携带所有的信息，包括payload和signature。
JWT常常通过HTTP的header来传送信息，同时也可以通过URL来传送。

<!-- more -->

**JWT的构成**
```
header.payload.signature
```

### Header
Header包含2部分。
typ定义类型为JWT，alg定义加密类型为HS256
``` json
{
    "typ":"JWT",
    "alg":"HS256"
}
```
通过[Base64](https://zh.wikipedia.org/wiki/Base64)的转化得到第一部分
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

### Payload
这部分由我们需要发送的数据内容构成
``` json
{
    "iss": "chengpeng.me",
    "exp": 144444444,
    "user": "CP",
    "email": "chengandpeng@gmail.com"
}
```

其中
**iss**: token的发行者
**exp**: 过期时间，必须在现在时间之后
其他一些声明
**sub**: token的主题
**aud**: token的接受者
**nfb**: 定义不可接受数据的时间
**iat**: JWT的声明时间

转换成Base64后得到第二部分
```
eyJpc3MiOiAiY2hlbmdwZW5nLm1lIiwiZXhwIjogMTQ0NDQ0NDQ0LCJ1c2VyIjogIkNQIiwiZW1haWwiOiJjaGVuZ2FuZHBlbmdAZ21haWwuY29tIn0=
```

### Singnature
第三部分的签名由上面我们得到的header和payload的base64的字符串，通过**.**连接起来，然后使用HS256算法进行加密得到。
``` javascript
var encodedString = base64UrlEncode(header) + "." + base64UrlEncode(payload);
HMACSHA256(encodedString, 'secret');
```

加密时必须定义一个加密字符串（本例中为'secret'），由服务端自行定义，计算后得到的结果是
```
183e996cf3ba65be2d31e6ea3d53d70942ded7a9b89bfdd7e47e38a554c3ff25
```

把这部分内容和前2个结果通过**.**符号接连起来就是我们的JWT了
``` 
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiAiY2hlbmdwZW5nLm1lIiwiZXhwIjogMTQ0NDQ0NDQ0LCJ1c2VyIjogIkNQIiwiZW1haWwiOiJjaGVuZ2FuZHBlbmdAZ21haWwuY29tIn0=.183e996cf3ba65be2d31e6ea3d53d70942ded7a9b89bfdd7e47e38a554c3ff25
```

## JWT vs Cookie
相比传统的Cookie，使用JWT有些什么好处呢?
1. 跨域：JWT允许向任何服务器或者域名使用ajax，因为JWT是包含在http header中传送数据的
2. 减轻服务端负担：比起使用session来保存cookie，JWT自身包含了所有信息，通过解密即可验证
4. 安全性：因为无需使用cookie，就不必担心劫持cookie的CSRF攻击了
5. 移动端：对于无法使用cookie的一些移动端，JWT能够正常使用
