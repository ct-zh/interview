# 源码分析 目录
目录中分为api、doc、include、lib、src、misc、test这7个原始目录，编译后还会生成bin、pkg，2个目录
- api：对应工具"go tool api"相应源码 src/cmd/api;
- doc：文档目录，如果要编译，或者了解一些版本信息，可以看一下这个目录中html内容;
- include：依赖C头文件, go1.5实现自举以后, 不需要这个目录了;
- lib：依赖库文件;
- src：源代码;
- misc：一些杂项脚本都在这里面;
- test：测试;
- bin：go、godoc、gofmt等等;
- pkg：生成对应系统动态连接库，以及对应系统的工具命令都在该目录，如cgo、api、yacc等等;


## reference
> [如何优雅的使用GDB调试Go](https://mp.weixin.qq.com/s/xfDydcpRCmX1dR5FybI0Rw)
