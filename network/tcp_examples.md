## TCP 示例

### 关键词

- ACK 确认 Acknowledgment
- SYN 同步序列编号 Synchronize Sequence Numbers
- RST 重置 Reset
- LISTEN 侦听来自远方的TCP端口的连接请求 LISTENING
- ESTABLISHED 表示两台机器正在传输数据
- CLOSE WAIT 等待主动关闭端发来的连接中断请求的确认
- TIME WAIT 等待足够的时间以确保被动关闭端接收到连接中断请求的确认
- MSL 报文段最大生存时间 Maximum Segment Lifetime
- 客户端：主动连接端、主动关闭端
- 服务端：被动连接端、被动关闭端

### TCP 连接

![三次握手和四次挥手的状态转换](../img/network-2020-05-22-21-50-40.png)

### TCP 状态图

![tcp state diagram](../img/tcp_state_diagram_fixed.svg.png)

### 握手为什么需要三次

- 第一次握手（SYN），主动连接端 C 发送网络包，被动连接端 S 收到，证明 C 的发送能力， S 的接收能力正常
- 第二次握手（SYN+ACK），S 端发包，C 端收到，证明 S 端发送能力，C 端接收能力正常，此时 S 端不清楚 C 端接收能力正常
- 第三次握手（ACK），C 端发包，S 端收到，证明 C 端接收能力正常

### 挥手为什么需要四次

- 关闭连接时，当被动关闭端收到 FIN 报文时，很可能并不会立即关闭 SOCKET
- 所以只能先回复一个 ACK 报文，告诉主动关闭端，"你发的 FIN 报文我收到了"
- 等到被动关闭端所有的报文都发送完了，发送 FIN 报文
- 等待主动关闭端发来的连接中断请求的确认 ACK ，关闭连接

### SYN 泛洪攻击

- 被动连接端的资源分配是在二次握手时分配，主动连接端的资源是在完成三次握手时分配
- 主动连接端在短时间伪造大量不存在的 IP，向被动连接端不断发送 SYN 包
- 被动连接端回复确认包，等待主动连接端确认
- 由于 IP 不存在，因此被动连接端不断重发确认包，直到超时
- 这时，伪造的 SYN 包长时间占用未连接队列
- 正常的 SYN 请求因为队列满被丢弃，导致网络拥塞
- SYN 攻击是典型的 DDoS 攻击

## 参考

- [TCP连接的状态详解以及故障排查 - cloud.tencent.com](https://cloud.tencent.com/developer/article/1347046)
- [三次握手和四次挥手 - juejin.cn](https://juejin.cn/post/6844903958624878606)
- [TCP 状态图 - wikipedia.org](https://en.wikipedia.org/wiki/File:Tcp_state_diagram.png)