# IO

## 跨平台的实现方式

- java为每个平台开发不同的虚拟机
- c调用不同的系统函数

c库函数:
文件描述符(整型值):索引到对应磁盘文件
文件读写指针
IO缓冲区(内存地址)

## 虚拟地址空间

从4G到0的顺序:
- linux kernel
  - 内存管理
  - 进程管理
  - 设备驱动管理
  - VFS虚拟文件系统
- 环境变量
- 命令行参数
- 栈空间(从大地址到小地址)
- 共享库
- 堆空间(从小地址到大地址)
- .bss 未初始化的全局变量
- .data 已初始化的全局变量
- .text 代码段
- 受保护的地址(0-4k):例如NULL, nullptr

虚拟地址空间通过操作系统的MMU(内存管理单元)映射到真实地址


# linux系统调用

- 打开文件:
  - int open(const char *pathname, int flags);
    - 打开一个已存在的文件,返回文件描述符,失败则返回-1
    - pathname: 要打开的文件路径
    - flags:对文件的操作权限:O_RDONLY, O_WRONLY, O_RDWR等
  - int open(const char *pathname, int flags, mode_t mode);
    - 创建一个新的文件,返回文件描述符,失败则返回-1
    - flags:对文件的操作权限和其他设置
      - 必选:O_RDONLY, O_WRONLY, O_RDWR等
      - 可选:O_CREAT 文件不存在,创建新文件
    - mode:八进制的数字,表示创建出的新的文件的操作权限,比如:0777 (全权限),umask会抹去某些权限,最终权限为mode & ~umask
- 关闭文件:
  - int close(int fd)
    - 关闭文件
    - fd:文件描述符

- 读取文件:
  - ssize_t read(int fd, void *buf, size_t count);
    - fd:文件描述符,open得到的
    - buf:放数据的地方,数组的地址 (传出参数)
    - count:数组的大小
    - 返回值:
      - 成功:
        - 大于0:返回实际读取到的字节数
        - =0:文件已经读取完了
      - 失败:-1,并设置errno

- 写入文件:
  - ssize_t write(int fd, const void* buf, size_t count);
    - fd:文件描述符,open得到的
    - buf:要往磁盘写入的数据
    - count:要写的数据实际的大小
    - 返回值:
      - 成功:实际写入的字节数
      - 失败:-1,并设置errno

- lseek函数
  - off_t lseek(int fd, off_t offset, int whence)
    - fd:文件描述符
    - offset:偏移量
    - whence:
      - SEEK_SET:设置文件指针的偏移量;
      - SEEK_CUR:设置偏移量:当前位置+offset的值
      - SEEK_END:设置偏移量:文件大小(文件末尾)+offset的值 
    - 返回文件指针最终位置


- stat函数
  - int stat(const char* pathname, struct stat* statbuf)
    - pathname:操作的文件路径
    - statbuf:结构体变量,传出参数,用于保存获取到的文件信息
    - 返回值:
      - 成功:0
      - 失败:-1,并设置errno

- lstat函数
  - int lstat(const char* pathname, struct stat* statbuf)
    - pathname:操作的文件路径
    - statbuf:结构体变量,传出参数,用于保存获取到的文件信息
    - 返回值:
      - 成功:0
      - 失败:-1
    - 获取软连接文件的信息

- access函数
  - int access(const char* pathname, int mode)
    - pathname:判断文件路径
    - mode:
      - R_OK:判断是否有读权限
      - W_OK:判断是否有写权限
      - X_OK:判断是否有执行权限
      - F_OK:判断文件时候存在
    - 返回值:
      - 成功:0
      - 失败:-1

- chmod函数
  - int chmod(const char* pathname, mode_t mode)
    - pathname:文件路径
    - mode:需要修改的权限,八进制的值
    - 返回值:
      - 成功:0
      - 失败:-1

- truncate函数
  - int truncate(const char * path, off_t length)
    - path:文件路径 
    - length: 最终文件变成的大小
    - 返回值:
      - 成功:0
      - 失败:-1
    - 缩减或者扩展文件尺寸

- 目录操作函数:
  - int mkdir(const char* pathname, mode_t mode)
  - int rmdir(cosnt char* pathname)
  - int rename(const char* oldpath, const char* newpath)
  - int chdir(cosnt char * path) 更改当前路径
  - char* getcwd(char * buf, size_t size) 获取当前路径 


- 目录遍历函数
  - DIR* opendir(const char* name)
  - struct dirent* readdir(DIR* dirp) 读取目录中的信息
  - int closedir(DIR* dirp)


- 文件描述符
  - int dup(int oldfd)
    - 作用:复制一个新的文件描述符,指向同一个文件
    - 返回值:
      - 成功:返回文件描述符
      - 失败:-1
  - int dup2(int oldfd, int newfd)
    - 重定向 关闭newfd指向的文件,将newfd指向oldfd指向的文件
  
  - linux重定向命令:
    - 输出重定向:
      - \> :会覆盖目标的原有内容。当文件存在时会先删除原文件，再重新创建文件，然后把内容写入该文件；否则直接创建文件。
      - \>\> : 会在目标原有内容后追加内容。当文件存在时直接在文件末尾进行内容追加，不会删除原文件；否则直接创建文件。
    - 输入重定向:<

- fcntl函数
  - int fcntl(int fd, int cmd, ...)
    - 作用:
      - 复制文件描述符
      - 获取或修改文件状态标记
    - fd: 文件描述符
    - cmd:对文件描述符进行如何操作
      - F_DUPFD:复制文件描述符,得到一个新的文件描述符(返回值)
      - F_GETFL:获取指定的文件描述符状态flag
      - F_SETFL:设置文件描述符文件状态flag
        - 必选:O_RDONLY, O_WRONLY, O_RDWR等
        - 可选:O_APPEND 追加数据; O_NONBLOCK 设置成非阻塞 