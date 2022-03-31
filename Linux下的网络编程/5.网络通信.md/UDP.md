# UDP通信

无连接的传输协议，该协议称为用户数据报协议（UDP，User Datagram Protocol）。UDP 为应用程序提供了一种无需建立连接就可以发送封装的 IP 数据包的方法  

非连接的协议，传输数据之前源端和终端不建立连接， 当它想传送时就简单地去抓取来自应用程序的数据，并尽可能快地把它扔到网络上。 在发送端，UDP传送数据的速度仅仅是受应用程序生成数据的速度、 计算机的能力和传输带宽的限制； 在接收端，UDP把每个消息段放在队列中，应用程序每次从队列中读一个消息段。  

- 函数:
  - ssize_t sendto(int sockfd, const void* buf, size_t len, int flags, const struct sockaddr* dest_addr, socklen_t addrlen)
    - 参数:
      - sockfd:通信的fd
      - buf:要发送的数据
      - len:发送的数据的长度
      - flags:0
      - dest_addr:通信的另外一端的地址(发送端)
      - addr_len:dest_addr的大小
    - 返回值:
      - 成功:返回已发送的字节的数量
      - 失败:-1
  - ssize_t recvfrom(int sockfd, const void* buf, size_t len, int flags, const struct sockaddr* src_addr, socklen_t addrlen)
    - 参数:
      - sockfd:通信的fd
      - buf:要发送的数据
      - len:发送的数据的长度
      - flags:0
      - src_addr:通信的另外一端的地址(接收端),不需要可以指定为NULL
      - addr_len:dest_addr的大小
    - 返回值:
      - 成功:返回已接收的字节的数量
      - 失败:-1
- 广播
  - 向子网中多台计算机发送消息,并且子网中所有的计算机都可以接受到发送方发送的消息,每个广播消息都包含一个特殊的ip地址(主机地址为全1)
  - 只能在局域网中使用
  - 客户端需要绑定服务器广播使用的端口,才可以接受广播消息
  - int setsockopt(int sockfd, int level,int optname, const void* optval, socketlen_t optlen )
    - 参数:
      - sockfd:文件描述符
      - level:SOL_SOCKET
      - optname:SO_BROADCAST
      - optval:int类型的值,为1表示允许广播
      - optlen: optval的大小
- 组播(多播)
  - 单播地址标识单个ip接口,广播地址标识某个子网内的所有ip接口,多播地址标识一组ip接口,多播数据报只由对它感兴趣的接口接收,即运行相应多播会话应用系统的主机上的接口接收,另外不同于广播只限于局域网使用,多播既可以用于局域网,也可以用于因特网
    - 客户端需要加入多播组才能接收到多播的数据
    - 组播地址:224.0.0.0~239.255.255.255
      - 局部链接多播地址:224.0.0.0~224.0.0.255,是为路由协议和其他用途保留的地址,路由器并不转发用于此范围的ip包
      - 预留多播地址:
          - 224.0.1.0~224.0.1.255:公用组播地址,可用于因特网,使用前需要申请
          - 224.0.2.0~238.255.255.255:用户可用组播地址(临时组地址),全网范围内有效
      - 本地管理组播地址:239.0.0.0~239.255.255.255,可供组织内部使用,类似于私有ip地址,不能用于因特网,可限制多播范围
