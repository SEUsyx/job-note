# I/O多路复用(I/O多路转接)

IO多路复用使得程序能同时监听多个文件描述符,能够提高程序的性能  
Linux下实现IO多路复用的系统调用主要有select,poll和epoll  

### BIO模型:
- 阻塞等待:
    - 优点:不占用cpu宝贵的时间
    - 缺点:同一时刻只能处理一个操作
- 多线程或多进程处理 
    - 优点:能够同时处理多个操作
    - 缺点:
      - 线程或者进程会消耗资源
      - 线程或进程调度会消耗cpu资源

### NIO模型:程序自己轮询所有的文件描述符,检测是否有新数据到达
- 非阻塞,忙轮询
    - 优点:提高了程序的执行效率
    - 缺点:需要占用更多的cpu和系统资源(每次循环都需要轮询所有的文件描述符)

### IO多路转接技术:将需要检测的文件描述符交给内核,内核检测后将结果交给程序
- select
    - 主要思想:
      - 首先构造一个关于文件描述符的列表,将要监听的文件描述符添加到该列表中
      - 调用一个系统函数(select()),监听该列表中的文件描述符,直到这些描述符中的一个或者多个进行IO操作时,该函数才返回
        - 这个函数是阻塞的
        - 函数对文件描述符的检测是由内核完成的
      - 在返回时,会告诉进程有多少(哪些)操作符要进行IO操作
    - 函数:
      - int select(int nfds, fd_set* readfds, fd_set* writefds, fd_set* exceptfds, struct timeval * timeout);
        - 参数:
          - nfds:委托给内核检测的最大文件描述符的值+1
          - readfds: 要检测的文件描述符的集合,委托内核检测哪些文件描述符的读属性(数据是否发生变化),是一个传入传出参数
          - writefds: 要检测的文件描述的写属性的集合,判断里面有没有空余空间(一般不检测,因为内存满了写函数通常会阻塞)
          - exceptfds:检测发生异常的文件描述符的集合(一般不用)
          - timeout:阻塞时间
            - NULL: 永久阻塞
            - tv_sec =0, tv_usec=0: 不阻塞
            - tv_sec >0, tv_usec>0: 阻塞的时间
        - 返回值:
          - -1:失败
          - \>=0:检测集合中有多少文件描述符发生了变化
      - void FD_CLR(int fd, fd_set* set); 将文件描述符集合中fd对应的标志位置为0
      - int FD_ISSET(int fd, fd_set* set); 判断fd对应的标志位是0还是1
      - void FD_SET(int fd_set* set); 将文件描述符集合中fd对应的标志位置为1
      - void FD_ZERO(fd_set* set);将文件描述符全部初始化为0(一共1024位)
    - selectde的缺点:
      - 1.每次都要将fd_set从用户态拷贝到内核态
      - 2.每次调用select都需要在内核遍历fd_set(O(n)复杂度)
      - 3.支持的文件描述符的数量太小了,默认是1024
      - 4.fd_set集合不能重用,每次都需要重置
- poll
    - 对select的改进(针对3,4条)
    - int poll(struct pollfd* fds, nfds_t nfds, int timeout);
      - 参数:
        - fds:结构体类型
          - ```c++
                struct pollfd{
                    int fd; /委托内核检测的文件描述符
                    short events; /委托内核检测文件描述符的事件 POLL_IN(读事件)
                    short revents; /文件描述符实际发生的事件
                }
            ```
        - nfds:第一个参数数组中最后一个有效元素的下标+1
        - timeout:阻塞时长
          - 0: 不阻塞
          - -1: 阻塞, 当检测到需要检测的文件描述符发生变化时,解除阻塞
          - \>0: 阻塞的时长
        - 返回值:
          - -1:失败
          - \>0:成功,检测集合中有多少文件描述符发生了变化
- epoll
    - 针对select()的1,2,3,4
      - ```c++
            struct eventpoll{
                ...
                struct rb_root rbr; //红黑树头节点,需要检测的文件描述符的信息
                struct list_head rblist; //双向链表头节点,存放检测到数据发生改变的文件描述符的信息
            }
            struct epoll_event{
                unit32_t    events;
                epoll_data_t    data;
            }

            typedef union epoll_data{
                void    *ptr;
                int     fd;
                unit32_t    u32;
                unit64_t    u64;
            }epoll_data_t;
        ```
    - 函数:
      - int epoll_create(int size); 
        - 作用:创建一个新的epoll实例,在内核中创建了一个数据,数据中有两个比较重要的数据成员,一个是需要检测的文件描述符的信息(红黑树结构),另一个是就绪链表,存放检测到数据发生改变的文件描述符的信息(双向链表结构)
        - 参数:
          - size: 目前没有意义,随便写一个大于0的数就行(之前使用的是哈希表实现,所以需要size)
        - 返回值:
          - -1: 失效
          - \>0: 文件描述符,用来操作epoll实例
      - int epoll_ctl(int epfd, int op,int fd, struct epoll_event* event);
        - 作用:对epoll实例进行管理:添加文件描述符信息,删除信息,修改信息
        - 参数:
          - epfd:epoll实例对应的文件描述符
          - op:操作
            - EPOLL_CTL_ADD:添加
            - EPOLL_CTL_MOD:修改
            - EPOLL_CTL_DEL:删除
          - fd:要检测的文件描述符
          - event:检测文件描述符的事件
            - EPOLLIN:读事件
            - EPOLLOUT:写时间
            - EPOLLERR:错误
      -  int epoll_wait(int epfd, struct epoll_event* events, int maxevents, int timeout);
         -  作用:检测函数
         -  参数
            -  epfd: epoll实例对应的文件描述符
            -  events:传出参数,保存了发生了变化的文件描述符的信息
            -  maxevents:第二个参数结构体数组的大小
            -  timeout: 阻塞时间
         - 返回值:
           - 成功:返回发生变化的文件描述符的个数
           - 失败: -1
    - 工作模式:
      - LT模式(水平触发):默认工作模式, 并且同时支持block和no-block socket, 在这种做法中,内核会告诉你一个文件描述符是否就绪,然后你就可以对这个就绪的fd进行IO操作,如果不作任何操作,内核还是会继续通知你
      - ET模式(边沿触发):高速工作方式,只支持no-block Socket. 在这种模式下, 当描述符从未就绪变为已就绪时,内核会通过epoll告诉你, 然后内核就假设你知道文件描述符已经就绪, 并且不会再为那个文件描述符发送更多的就绪通知,直到你做了某些操作导致那个文件描述符不再为就绪状态,但是如果一直不对这个fd进行IO操作(从而导致它再次变为未就绪状态时),内核不会发送更多的通知
        - ET模式很大程度上减少了epoll事件被重复触发的次数,因此效率要比LT模式高,epoll工作在ET模式的时候必须使用非阻塞套接字,以避免由于一个文件描述符 的阻塞读/写操作把处理多个文件描述符的任务饿死
        - 一般配合循环读数据和非阻塞的API