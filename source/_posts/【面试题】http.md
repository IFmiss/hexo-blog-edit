---
title: 【面试题】http
date: 2021-03-29 18:48:50
categories: http
tags: [http, 面试]
---

### 常用 http 请求码

- **301** （永久移动） 请求的网页已永久移动到新位置。 服务器返回此响应（对 GET 或 HEAD 请求的响应）时，会自动将请求者转到新位置，url 会换成新的。
- **302** （临时移动） 服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求，url 不会变。
- **304** （未修改） 自从上次请求后，请求的网页未修改过。 服务器返回此响应时，不会返回网页内容。数据从缓存获取。
- **400** （错误请求） 服务器不理解请求的语法。
- **401** （未授权） 请求要求身份验证。 对于需要登录的网页，服务器可能返回此响应。
- **403** （禁止） 服务器拒绝请求。
- **404** （未找到） 服务器找不到请求的资源。
- **405** （方法禁用） 禁用请求中指定的方法。
- **500** （服务器内部错误） 服务器遇到错误，无法完成请求。
- **501** （尚未实施） 服务器不具备完成请求的功能。例如，服务器无法识别请求方法时可能会返回此代码。
- **502** （错误网关） 服务器作为网关或代理，从上游服务器收到无效响应。
- **503** （服务不可用） 服务器目前无法使用（由于超载或停机维护）。
- **504**（网关超时） 服务器作为网关或代理，但是没有及时从上游服务器收到请求。
- **505** （HTTP 版本不受支持） 服务器不支持请求中所用的 HTTP 协议版本。

### 强缓存与协商缓存

- **强缓存**（按照优先级排序）
  - **cache-control**（HTTP/1.1 中定义）
    ```nginx
    add_header    Cache-Control  max-age=3600;
    ```
    > 设置 Cache-Control 则会覆盖 `expires` 设置
  - **expires**（HTTP/1.0 中定义）
    expires 设置资源缓存的时间
    ```nginx
    # 设置一天后过期  s: 秒， m: 分， h: 时， d: 天
    expires 1d;
    ```
- **协商缓存**
  第一次请求数据时，服务器会返回缓存标识和数据资源，客户端会将缓存标识与数据保存在本地
  第二次请求数据时，会将缓存标识发送给服务器，服务器根据标识判断，如果可以使用缓存，则返回 304 状态码，浏览器使用缓存的数据内容
  - **Etag / If-None-Match**
    Etag：服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器决定）
    If-None-Match：再次请求服务器时，会将标识发送给服务器，服务器发现 `If-None-Match` 与被请求资源的标识一致时，则返回 304，并从本地缓存获取资源，否则重新获取新数据
    > 缺点： 针对不需要浏览器更新的修改（如注释）Etag 依然会更新
  - **Last-Modified / If-Modified-Since**
    Last-Modified：服务器在响应请求时，告诉浏览器资源的最后修改时间。
    If-Modified-Since：下次请求时会带上此前的资源修改时间，

### Http 和 http2 的区别

- **二进制传输**
  http2 采用二进制传输，相对于 http 的文本传输安全性高
- **多路复用**
  http 一个连接只能提交一个请求，http2 可以执行多个请求，可以降低连接的数量，提高网络的吞吐量。
  同一个 TCP 连接，同一时刻可以传输多个 HTTP 请求。
- **头部压缩**
  HTTP2 gzip 与 compress 对头部进行压缩，客户端和服务端之间维护了一个头部索引表，通过 id 就可以进行头部的信息传输，缩小头部容量，提升传输效率。
- **HTTP2 支持服务器推送**
  http2 可以主动推送资源到客户端，避免客户端花过多时间逐个请求，降低响应时间。

### 永久性重定向（301）和临时性重定向（302）对 SEO 有什么影响

- 301 重定向可促进搜索引擎优化效果。
- 302 重定向可影响搜索引擎优化效果。

### 301 与 302 的区别
- 对于301请求，浏览器是默认给一个很长的`缓存`。而302是不缓存的。
- 旧地址A的资源不可访问了(永久移除), 重定向到网址B，搜索引擎会抓取网址B的内容，同时将网址保存为B网址。而302是资源仍然可以访问，只是重定向临时从地址A转到了B，此时搜索引擎会抓B的内容，但是网址还是 A 的，所以不利于 `SEO`
- 尽量使用301跳转，以防止`网址劫持`

### 接口如何防刷

- 网关控制流量洪峰，对在一个时间段内出现流量异常，可以拒绝请求
- 源 ip 请求个数限制。对请求来源的 ip 请求个数做限制
- http 请求头信息校验；（例如 host，User-Agent，Referer）
- 人机验证，验证码，短信验证码，滑动图片形式
- 拦截器特定信息校验，例如：token

### 为什么 HTTP1.1 不能实现多路复用

HTTP/1.1 不是二进制传输，而是通过文本进行传输。由于没有流的概念，在使用并行传输（多路复用）传递数据时，接收端在接收到响应后，并不能区分多个响应分别对应的请求。

### 简单介绍一下 3 次握手 🤝 ，为什么是 3 次握手？

![三次握手](https://www.daiwei.site/static/blog/【面试题】http/三次握手.png)

SYN 表示建立连接
FIN 表示关闭连接
ACK 表示响应

- 【SYN_SENT 状态】客户端发送 `SYN` 包（`SYN` = 1）到服务器，随机产生 `seq number` = 12345 数据包到服务器，服务器由 `SYN` = 1 知道客户端要求建立连接。**（客户端：我要连接你）**
- 【SYN_RECV 状态】服务端接收到客户端发来的 `SYN` 以及 `seq`, 确认联机信息，向客户端发送 `ack number` = 客户端 `seq` + 1, `SYN` = 1, `ACK` = 1, 并随机产生一个服务端的 `seq` = 54321 的包 **（服务器：好的，你来连吧）**
- 【ESTABLISHED 状态】客户端收到后检查 `ack` 的值是否是客户端发送的 `seq` + 1 的值, 如果正确的话，客户端会再次发送 `ack number` = 服务器的 `seq` + 1, `ACK` = 1, 服务器收到确认的 `seq` 与 `ACK` = 1 则连接建立。**（客户端：好的，我来了）**

**为什么是 3 次握手**

三次是最少的安全次数，两次不安全，四次浪费资源；

### 简单介绍一下 4 次挥手 🙋‍♂️ ，为什么是 4 次挥手？

![四次挥手](https://www.daiwei.site/static/blog/【面试题】http/四次挥手.png)

SYN 表示建立连接
FIN 表示关闭连接
ACK 表示响应

- 客户端向 Server 发送 `FIN` = 1 , `seq` = u（等于前面已经传送过来的数据的最后一个字节的序号加 1），表示客户端关闭连接，然后进入 【FIN_WAIT_1 状态】，等待服务端返回 `ACK` 包，此后客户端不能再向 Server 发送数据，但能读取数据。
- Server 端收到客户端的连接释放报文, 发出确认报文 `ACK` = 1, seq = v, ack = u + 1。进入【CLOSE-WAIT 状态】
- 客户端收到 Server 的确认请求后，客户端进入 【FIN_WAIT_2 状态】等待 Server 发送 FIN 包。
- Server 将最后的数据发送完毕后，发送 FIN 释放报文，`FIN`=1，`ack`=u+1, `ACK` = 1, `seq` = w, 此时服务器进入【LAST-ACK 最后确认状态】此后 Server 既不能发送数据，也不能读取数据。
- 客户端收到服务端的 `FIN` 包，必须发出确认，`ACK` = 1, `seq` = u = 1, `ack` = w + 1, 此时客户端进入 【TIME-WAIT 等待状态】**注意此时 TCP 连接还没有释放**，必须经过**2MS**L（最长报文段寿命）的时间后，当客户端撤销相应的 TCB 后，才进入 CLOSED 状态。
- 服务器只要收到了客户端发出的确认，立即进入 CLOSED 状态。同样，撤销 TCB 后，就结束了这次的 TCP 连接。可以看到，服务器结束 TCP 连接的时间要比客户端早一些。

**为什么是 4 次挥手**

因为 TCP 是全双工通信的

### CDN 是什么？为什么要用？

CDN 的全称是 `Content Delivery Network`，是建立并覆盖在承载网之上，由分布在不同区域的边缘节点服务器群组成的**分布式网络**。

**为什么要用**

- 解决因分布，带宽，服务器性能带来的访问延迟问题。使用户可就近取得所需内容，解决网络拥挤的状况，提高用户访问网站的响应速度和成功率。
- 尽可能少的减少资源在转发，传输，链路抖动等情况下顺利保障信息的连贯性。
- 可以保护网站安全，CDN 的负载均衡和分布式存储技术，可以加强网站的可靠性。应对大部分的互联网攻击事件。

### CDN 有哪些静态资源优化加载速度的机制？

- **资源调度**：根据用户接入网络 IP 寻找距离用户最优路径的服务器。调度的方式主要有 DNS 调度，http 302 调度等等。
- **缓存策略和数据检索**：CDN 服务器使用高效的算法和数据结构，快速检索资源和更新读取缓存。
  - 网络优化: OSI 七层模型进行优化，达到网络优化的目的。
  - L1 物理层：硬件设备升级提高速度
  - L2 数据链路层：寻找更快的网络戒掉，确保 Lastmile 尽量短。
  - L3 路由层：路径优化，寻找两点最优路径
  - L4 传输层：协议 TCP 优化，保持长连接，TCP 快速打开
  - L7 应用层：静态资源压缩，请求合并。

### PWA 是什么？对于 PWA 有什么理解？

PWA（progressive web apps，渐进式 web 应用）
**像网站快捷，又像 QQ 微信一样离线在本地运行**

- Manifest 实现添加至主屏幕
- service worker 实现离线缓存
- message push 消息推送功能（国内用不了）

### http3.0

- http2 中始终在使用 TCP 协议，所以无论如何都还是会有 tcp 传输层的对头阻塞问题，而 http3 彻底解决对头阻塞的问题
- 彻底放弃使用 TCP 协议，而使用基于 UDP 的 QUIC 协议
- UDP 协议不需要 3 次握手和 4 次挥手，所以天然比 TCP 协议快。
- 离开了 TCP 的底层重传，流之间没有依赖，包之间没有依赖，真正实现了多路复用，告诉传输数据。
  ....

### http 请求报文组成

**请求报文**

- 请求行
  - 请求方法
  - http 协议版本
- 请求头 `Content-Type`, `Cookie` ...
- 空行 （请求头之后是一个空行，通知服务器以下不再有请求头）
- 请求体

**响应报文**

- 状态行 状态码
- 响应头
- 空行
- 响应体

### E-tag 保存在哪里，怎么匹配，谁匹配

服务器端生成，根据 INode，MTime，Size 三个属性通过算法来实现，并输出成 hex 的格式，通过 与 `If-None-Match`匹配判断是否 304 还是重新加载资源

### 计算机网络层次结构

![osi七层模型](https://www.daiwei.site/static/blog/【面试题】http/osi七层模型.jpeg);

- **第一层: 物理层**
  建立、维护、断开物理连接。
- **第二层: 数据链路层**
  建立逻辑连接、进行硬件地址寻址、差错校验等功能。
- **第三层: 网络层**
  进行逻辑地址寻址，实现不同网络之间的路径选择。
  **协议有**: ICMP IGMP IP（IPV4 IPV6）
- **第四层: 传输层**
  定义传输数据的协议端口号，以及流控和差错校验。
  **协议有**: TCP UDP，数据包一旦离开网卡即进入网络传输层
- **第五层: 会话层**
  建立、管理、终止会话。（在五层模型里面已经合并到了应用层）
- **第六层: 表示层**
  数据的表示、安全、压缩。（在五层模型里面已经合并到了应用层）
  **格式有**: JPEG、ASCll、EBCDIC、加密格式等
- **第七层: 应用层**
  网络服务与最终用户的一个接口
  **协议有**: HTTP FTP TFTP SMTP SNMP DNS TELNET HTTPS POP3 DHCP

### 四层网络协议
- 网络接口层
- 网间层， 互连网络层
- 传输层
- 应用层


### TCP 和 UDP 的区别？TCP 如何确保数据正确性？TCP 头包含什么？ 属于哪一层？

**TCP 和 UDP 的区别**

- **TCP 必须先建立连接**（如打电话需要先拨号建立连接），**UDP 是无连接**的，即发送数据之前不需要建立连接。
- **TCP 提供可靠服务**，数据 无差错，不丢失，不重复，但是 **UDP 无法保证**。
- **TCP 点对点，一对一**， **UDP 支持一对一，一对多，多对多**。
- TCP 逻辑通信是**全双工的可靠信道**，**UDP 则是不可靠信道**。
- **TCP 由拥塞控制和流量控制保证数据传输的安全性**；**UDP 无拥塞控制**，网络拥塞不影响发送速率。
- TCP 首部开销大，20 个字节；UDP 首部仅有 8 个字节（源端口、目的端口、数据长度、校验和）

**TCP 如何确保数据正确性**

- 检验和
- 连接管理机制
- 确认应答 + 序列号
- 流量控制
- 拥塞控制

**TCP 头包含什么**

- 16 位端口号
- 32 位序列号
- 32 位确认号
- 4 位头部长度
- 16 位窗口大小
- TCP 头部选项

**TCP 属于哪一层**

传输层

> TCP 效率要求相对低，但对准确性要求相对高的场景 文件传输（准确高要求高、但是速度可以相对慢）、接受邮件、远程登录。
> UDP 效率要求相对高，对准确性要求相对低的场景。QQ 聊天、在线视频、网络语音电话（视频、实时通信）

### 什么文件用强缓存，什么文件用协商缓存

**强缓存**

确定文件一直不会变化的资源

**协商缓存**

如果没有修改过，那么就从缓存中读取文件，如平时的代码都是用的协商缓存，有变化则更新。

**不使用缓存**

所有资源从服务器获取。

### `no-store` 与 `no-cache` 的区别

- **`no-store`**「永远都不要在客户端存储资源」，每次永远都要从原始服务器获取资源
- **`no-cache`** 可以在客户端存储资源，但每次都「必须去服务器做新鲜度校验」，来决定从服务器获取最新资源 (200) 还是从客户端读取缓存 (304)，即所谓的协商缓存

### 如何删除 cookie

expires 设置过期时间。

### cookie 与 session 的区别

背景：HTTP 协议是无状态的协议。一旦数据交换完毕，客户端与服务器端的连接就会关闭，再次交换数据需要建立新的连接。这就意味着服务器无法从连接上跟踪会话

- `cookie` 存储在客户端，`session` 存储在服务端
- `session` 不能区分路径, `cookie` 可以通过 path 设置路径
- 安全性来说，`session` 更可靠一点
- `session` 会在一定时间内保存在服务器上。当访问增多，会比较占用你服务器的性能
- 单个 `cookie` 保存的数据不能超过 4K，很多浏览器都限制一个站点最多保存 20 个 `cookie`

### 对称加密

即信息的发送方和接收方使用同一个密钥去加密和解密数据。

- 算法公开
- 加密和解密速度快
- 适合于对大数据量进行加密

算法有: DES、3DES、TDEA、Blowfish、RC5 和 IDEA

其加密过程如下：**明文 + 加密算法 + 私钥 => 密文**
解密过程如下： **密文 + 解密算法 + 私钥 => 明文**

### 非对称加密

非对称加密与对称加密相比，其安全性更好, 非对称加密使用一对密钥，即公钥和私钥，且二者成对出现。私钥被自己保存，不能对外泄露。公钥指的是公共的密钥，任何人都可以获得该密钥。用公钥或私钥中的任何一个进行加密，用另一个进行解密。

**被公钥加密过的密文只能被私钥解密**，过程如下：
**明文 + 加密算法 + 公钥 => 密文， 密文 + 解密算法 + 私钥 => 明文**
**被私钥加密过的密文只能被公钥解密**，过程如下：
**明文 + 加密算法 + 私钥 => 密文， 密文 + 解密算法 + 公钥 => 明文**

算法有: RSA、Elgamal、Rabin、D-H、ECC

### 简述 https 原理，以及与 http 的区别

**HTTPS 协议 = HTTP 协议 + SSL/TLS 协议**，在 HTTPS 数据传输的过程中，需要用 SSL/TLS 对数据进行加密和解密，需要用 HTTP 对加密后的数据进行传输，由此可以看出 HTTPS 是由 HTTP 和 SSL/TLS 一起合作完成的。

- SSL 的全称是 Secure Sockets Layer，即安全套接层协议
- TLS 的全称是 Transport Layer Security

HTTPS 为了兼顾安全与效率，**同时使用了对称加密和非对称加密**。**数据是被对称加密传输的**, 为了确保能把该密钥安全传输到服务器端，**采用非对称加密对该密钥进行加密传输**，总的来说，对数据进行对称加密，对称加密所要使用的密钥通过非对称加密传输。

**https 请求过程**
![https 请求过程](https://www.daiwei.site/static/blog/【面试题】http/https.webp)

1. 浏览器**发起往服务器的 443 端口发起请求**，请求**携带了浏览器支持的加密算法和哈希算法**。
2. 服务器收到请求，**选择浏览器支持的加密算法和哈希算法**。
3. **服务器下将数字证书返回给浏览器**，这里的数字证书可以是向某个可靠机构申请的，也可以是自制的。
4. 浏览器验证证书合法性。
5. 生成一个随机数 R，并使用网站公钥对 R 进行加密。将 R 传送给服务度
6. 服务度通过**私钥解密得到 R**
7. 服务器**以 R 为密钥使用了对称加密算法加密网页内容**并传输给浏览器。
8. 浏览器**以 R 为密钥使用之前约定好的解密算法**获取网页内容。

### webpack 动态加载 js 原理

- 根据 `installedChunks` 检查是否加载过该 `chunk`
- 假如没加载过，则发起一个 `JSONP` 请求去加载 `chunk`
- 设置一些请求的错误处理，然后返回一个 `Promise`。

### 扫码登录的原理

- 网页端请求登陆二维码
  随机生成一个 uuid（这个 uuid 作为页面的唯一标识符），并且会将这个 uuid 当做一个键值对的 key 存入后台 Redis。必须设置一个过期时间

- 浏览器轮训
  当浏览器拿到二维码信息后，解析其中的 uuid，并拿这个 uuid 不断去后台轮询是否已经登陆成功，登陆成功则跳转，否则知道二维码过期。

- 手机扫描，将 token、uuid 提交给服务器。
  用户的 userId 和提交过来的 uuid 当做一个键值对（uudi-userId）存入 Redis

- 浏览器端轮询成功
  依据 Redis 中存在 uuid-userId 键值对。如果这个键值对已经存在，说明手机端已经扫码登陆过。
