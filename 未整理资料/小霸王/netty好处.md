## netty好处

1. 高并发架构
   - 网络模型:nio非阻塞模型 
   - 线程模型:reactor模型
2. 高性能
   - 动态分配byteBuf
   - 网络参数优化:接收缓冲区 发送缓冲区 noDelay
3. 高可靠
   -  网络连接断开
   - 网络链路探查
   - nioEventLoop线程池(单线程)容错的处理try cacth
   - epoll空轮序的bug
   - tailContext byteBuf内存自动释放
4. 可扩展
   - pipeline中handler调用链
   - 序列化协议
   - 定制网络通信协议
5. nio拆包和粘包问题
   -  编码器比如MessageToByteEncoder
   - 解码器:
     - 分隔符:DelimiterBasedFrameDecoder 
     - 长度域::LengthFieldBasedFrameDecoder(rocketmq中解码器)