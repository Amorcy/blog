---
title: 浅析WebSocket（二）
tags: [前端, WebSocket]
---

在上一篇文章[浅析WebSocket（一）](//amorcy.cc//2018/11/14/webSocket-1/)，我们了解到了WebSocket出现的背景，聪明的工程师们，为了能够实现前后端实时通讯，在WebSocket出现之前，还发明了Comet(Alex Russell发明的词儿)，他指的是一种更高级的Ajax技术（也经常被人们称为“服务端推送”）。Ajax是一种从页面向服务请求数据的技术，而Comet则是一种服务器向页面推送数据的技术，Comet能够让信息近乎实时的被推送到页面上，非常适合处理体育比赛的分数跟股票。它有两种实现方式，一种就是我们之前提到的长轮询，还有一种是HTTP流，感兴趣的同学可以参考[传送门](https://www.ibm.com/developerworks/cn/web/wa-lo-comet/index.html)了解它是如何实现的。
<!-- more -->
---
WebSocket通信协议包含两个高层组件：开发性的HTTP握手用于协商连接参数、二进制消息分帧机制用于支持低开销的基于消息的文本和二进制数据传输。WebSocket 协议尝试在既有HTTP 基础设施中实现双向HTTP 通信，因此也使用HTTP 的80 和443 端口。不过，这个设计不限于通过HTTP 实现WebSocket 通信，未来的实现可以在某个专用端口上使用更简单的握手，而不必重新定义一个协议。WebSocket 协议是一个独立完善的协议，可以在浏览器之外实现。不过，它的主要应用目标还是实现浏览器应用的双向通信。

---
我们首先看下WebSocket的请求头报文与相应头报文来进一步了解WebSocket的高层组件之一：开发性的HTTP握手
> Request Headers

![image](/img/websocket/wsReqH.jpeg)

观察以上面请求头我们可以看到
1. 第4行Connection：HTTP1.1中规定Upgrade只能应用在直连连接中。带有Upgrade头的     HTTP1.1消息必须含有Connection头，因为Connection头的意义就是，任何接收到此消     息的人（通常为代理服务器）都要在转发此消息之前处理掉Connection中指定的域（不转     发此Upgrade域）
2. 第12行Upgrade是HTTP1.1中用于定义转换协议的header域。如果server支持的话，client希望使用建立好的HTTP(TCP)连接，切换到WebSocket协议
3. 第8行Sec-WebSocket-Extensions是客户端用来与服务端协商扩展协议的字段，permessage-deflate表示协商是否使用传输数据压缩，client_max_window_bits表示采用LZ77压缩算法时，滑动窗口相关的SIZE大小
4. 第9行Sec-WebSocket-Key是一个Base64encode的值，这个是客户端随机生成的，用于服务端的验证，服务器会使用此字段组装成另一个key值放在握手返回信息里发送客户端
5. 第10行Sec_WebSocket-Protocol是一个用户定义的字符串，用来区分同URL下，不同的服务所需要的协议，标识了客户端支持的子协议的列表
6. 第11行Sec-WebSocket-Version标识了客户端支持的WebSocket协议的版本列表

> Response Headers

![image](/img/websocket/wsResH.jpeg)

观察以上面请求头我们可以看到
1. 第1行Connection字段value值同样为upgrade, 既升级协议
2. 第6行则定义了升级协议为WebSocket
3. 第3行Sec-WebSocket-Accept，是客户端校验服务端是否支持WebSocket协议的字段，它的生成过程为：
将请求头中的Sec-WebSocket-Key与协议中已定义的一个GUID“258EAFA5-E914-47DA-95CA-C5AB0DC85B11”进行拼接，然后将这个生成的字符串进行SHA1编码，之后生成的字符串进行Base64编码。客户端通过验证服务端返Sec-WebSocket-Accep的值, 来确定两件事情:
- 服务端是否理解WebSocket协议, 如果服务端不理解,那么它就不会返回正确的Sec-WebSocket-Accept，则建立WebSocket连接失败
- 服务端返回的Response是对于客户端的此次请求的,而不是之前的缓存。 主要是防止有些缓存服务器返回缓存的Response

---
以上请求头跟响应头如果主要字段都没有出差错，那么本次连接则从HTTP连接升级到了WebSocket连接了。现在客户端就可以通过WebSocketAPI来接收服务端推送的消息了
```JS
// 创建WebSocket连接
const ws = new WebSocket('ws://WebSocket-example:8081')
const reader = new FileReader()
// 错误处理
ws.onerror = (err) => {
  // todo
}
// 连接建立是调用
ws.onopen = () => {
  // 向server发送消息
  ws.send('connected')
}
// 接收server发送的消息
ws.onmessage = (msg) => {
  // todo
  if (msg.data instanceof Blob) {
    // 处理二进制信息
    reader.readAsText(msg.data, 'UTF-8')
  }
}
//关闭时调用
ws.onclose = () => {
  // todo
}
```
---
之前前端er便可以愉快的接收服务端主动推送的消息啦~~
但是有个问题：很多现有的HTTP 中间设备可能不理解新的WebSocket 协议，而这可能导致各种问题：盲目的连接升级、意外缓冲WebSocket 帧、不明就里地修改内容、把WebSocket 流量误当作不完整的HTTP 通信，等等。这时WSS（**WSS协议表示使用加密信道通信，基于SSL的安全传输，占用与tls相同的443端口**）就提供了一种不错的解决方案，它建立一条端到端的安全通道，这个端到端的加密隧道对中间设备模糊了数据，因此中间设备就不能再感知到数据内容，也就无法再对请求做特殊处理。




