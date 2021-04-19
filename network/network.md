# network
## tcp
### OSI七层模型
物理层、数据链路层、网络层(ip)、传输层(TCP/UDP)、会话层、表示层、应用层; 会话层表示层应用层在五层协议里统称为应用层;

### tcp三次握手四次挥手
三次握手
- client发送SYN包到server请求连接,请求序号seq=x;
- server返回ACK包作为应答,ack=x+1, seq=y;
- client再发送ACK包表示建立连接, seq=x+1, ack=y+1;

四次挥手:
- 主动关闭方发送FIN包, 进入`FIN_WAIT_1`状态, 等待ack应答;
- 被动关闭方发送ACK包应答(交付EOF), 进入`CLOSE_WAIT`状态;
- 此时server仍然可以向client发送数据,client在收到server的FIN包前都会处理数据;(半关闭情况, 在调用shutdown api时出现; 但是一般程序都是调用close api直接关闭程序);
- 主动关闭方收到ACK应答, 进入`FIN_WAIT_2`状态;
- 被动关闭方发送FIN包, 进入`LAST-ACK`状态,等待最后的ack到来;
- 主动关闭方收到FIN包后确认关闭,回复ACK包, 进入2MSL的`TIME_WAIT`状态;
- 被动关闭方收到ACK包,确认关闭;

四次挥手报文丢失的情况:
- 1. 主动关闭方发送FIN丢包, 很久收不到ACK应答, 主动关闭方重发FIN;
- 2. 被动关闭方收到了FIN, 回复ACK丢包; 此时被动关闭方会卡在`CLOSE_WAIT`状态,等待主动发送方重发FIN包;
- 3. 被动关闭方结束`CLOSE_WAIT`(在go里面调用`conn.Close()`), 发送FIN包丢包,超时重发;
- 4. 主动关闭方收到FIN包,回复ACK丢包, 此时主动关闭方会等待2MSL的`TIME_WAIT`,防止ACK丢包导致被动关闭方未关闭重发FIN包;

### 如果第二次握手（SYN ACK）报文丢失会出现什么问题?
client会重传SYN包;

### 四次挥手中的包中网络中丢失，tcp会怎么处理？
FIN包丢失: 没有收到ack包会重发;
ack包丢失: 接收方必须在TIME_WAIT状态等待2倍MSL时间, 等待ACK重发;

### time-wait的作用; TimeWait和CloseWait原因
time-wait:
- 出现在主动关闭方;
- 保证被动关闭方在2MSL内重发FIN包能及时回复ACK;

closewait:
- 出现在被动关闭方;(server)
- 并发请求太多, 服务器没能跟上并发速度;
- 被动关闭方未及时释放端口资源;(比如没有 `defer conn.Close`)
- 大量客户端关闭了连接;(客户端存在超时机制、服务端可能负载过高导致响应耗时变长)

### 讲下tcp的快速重传和拥塞机制
- 拥塞:接收方网络资源繁忙,因未及时响应ACK导致发送方重传大量数据;
- 拥塞机制(四个拥塞算法): 慢启动和拥塞避免、快速重传与快速恢复;

- 慢开始: 发送方传送速度指数增长;
- 达到某个阈值, 开始进入拥塞避免阶段,加法增长传送速度; 
- 如果遇到拥塞(收到三个重复的ACK), 执行快速重传/快速恢复, 拥塞窗口减小一半, 将丢包的数据包重新发送一遍;然后再进入拥塞避免状态

### tcp粘包/拆包问题怎么处理？
- 粘包: 多个请求合并到一个报文里面;
- 拆包: 一个请求拆分到多个报文里面;
- 应用程序写入数据大于socket缓冲区大小导致拆包;小于缓冲区大小,导致粘包
- MSS最大报文长度 TCP长度-TCP头部长度 > MSS 进行拆包;
- 接收方法不及时读取套接字的缓冲区数据,发生粘包;

解决办法:
- 定义消息头,先从头读取包长度,再读包内容;
- 设置定长消息, 每次读取定长内容,长度不够就填充固定补位字符;
- 设置消息边界, 一般使用`\n`;
- 使用更复杂的协议, 如json、protobuf;

### TCP为什么是三次握手四次挥手
- 三次握手其实是将server的ack和syn包合并了，加快效率；
- TCP需要seq序列号来做可靠重传或接收，而避免连接复用时无法分辨出seq是延迟或者是旧链接的seq，因此需要三次握手来约定确定双方的ISN(初始seq序列号）
- 四次挥手是需要考虑到被动关闭方可能数据没有发完（半关闭状态）,因此被动关闭方的FIN包和ACK不能合并在一起发;

## http
### HTTPS 和 HTTP 有什么区别
- https是http的安全版本, http协议的数据传输是明文的,https使用了SSL/TLS协议进行了加密处理;
- https与http的连接方式不一样, 默认端口也不一样,http是80,https是443;

> HTTPS其实是有两部分组成：HTTP + SSL / TLS，也就是在HTTP上又加了一层处理加密信息的模块。服务端和客户端的信息传输都会通过TLS进行加密，所以传输的数据都是加密后的数据。 对称加密算法加密数据+非对称加密算法交换密钥+数字证书验证身份=安全

### https解决了什么问题，怎么解决的？说下https的握手过程。
握手过程
- client向443端口发起请求，明文向服务器协商关于加密的信息;
- server返回协商结果,包括协议版本、加密套件、证书(公钥)等;
- client通过受信任的CA验证证书合法性;
- client随机生成RSA公私钥以及会话密钥,使用服务器的公钥加密发给服务器;
- server用私钥解密，拿到client的会话密钥, 用客户端的公钥加密会话密钥发给客户端;
- client解出server的会话密钥;
- 双方都拿到会话密钥,可以加密数据了;

### http/2与http1.1的区别
- http/2采用二进制格式而非文本格式;
- http/2使用一个连接可以实现多路复用;
- 使用报头压缩,http/2降低了开销;
- http/2让服务器可以响应主动"推送"到客户端缓存

## socket
### 打开一个socket发生了什么?怎么写一个socket服务器?
- `Socket = IP 地址 + 端口 + 协议`,组成一个唯一标识，用来标识一个通信链路;
- Socket 其实是对 TCP/IP 进行了高度封装，屏蔽了很多网络细节.
- websocket: 只是一种持久化协议；
- websocket握手： header头发送:
    ```
    Connection: Upgrade
    Upgrade: websocket
    Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
    Sec-WebSocket-Protocol: chat, superchat
    Sec-WebSocket-Version: 13
    ```
    回应：
    ```
    Connection: Upgrade
    Upgrade: websocket
    Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
    Sec-WebSocket-Protocol: chat
    ```
- 最显著的区别是socket是流式数据，需要判断消息的头尾，解决粘包/拆包的问题；websocket是单纯的发消息，收消息.

### 长连接写过吗? 你们全都用的rpc 请求吗?讲一下grpc 和ws

