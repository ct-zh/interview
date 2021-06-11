### 协议编码适配流程总结
1. 首先需要定义三个结构体：`Option`、`Header`、`Body`;
2. Option 用来进行 客户端服务端的协商，这个Option的编码必须提前统一好，比如json，一般协商内容是：
    - MagicCodo/MagicNumber: 放在第一，用于验证是否是Option数据的唯一标示；
    - 编码器Codec的类型；
    - 超时策略相关数据；

3. Option协商完毕之后，客户端服务端开始正式通信，服务端需要根据编码器类型去初始化一个Codec编码器，Codec要实现如下接口
    - 读header
    - 读body
    - 写功能
    - 关闭连接的功能

4. 然后再编写多种编码类，比如golang提供json、xml、binary、gob、base64之类的编码解码器；要根据协议规范实现decode和encode；然后用一个工厂模式根据Option里Codec的值去初始化对应的编码器；

5. 对于传输的数据，分为header和body；

6. header类似于http的header，要有：
    - Seq请求编号；
    - 部分编码可能 需要body的 length；
    - Code码/错误码；
    - 相关操作，比如需要调用的方法；

7. body就是传输的具体数据; 关于body的长度问题/粘包拆包：
    - 如果编码不是那种有明确界限的编码，需要注意tcp报文的粘包/拆包问题；
    - 解决办法：1. header带上长度；2. 设置定长消息，长度不够用固定字符补位；3. 设置消息边界，如`\n`； 4. 使用json、protobuf这些有明确边界的编码；

8. 补充：
    - 连接可能需要心跳包来保持： 心跳流程;
    - 不管服务器还是客户端，write/read操作最好上bufferio;