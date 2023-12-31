# 套接字（socket）

## socket概览

skbuff可以跟踪数据包的整个生命周期，sock则跟踪一个socket的整个生命周期。

socket是内核中的一个资源实体，利用这个实体可以访问网络。也就是说，网络协议栈对外呈现的并不是一个面向过程的函数调用，在概念上是一个由类生成的一个个socket对象，通过这一个个socket对象，用户可以使用调用对象的方法访问网络。

每个应用程序都会对网络使用一个事务，这个事务可能包含很多个不同的步骤，而任何一个函数调用都只能完成其中的一部分，所以每个事务都需要变量来存储当前的状态。对于网络调用来说，即：监听的哪个端口；该端口是否允许重用；这个事务的所有者是哪个或哪些进程，绑定的是哪个设备，使用的是哪个协议族；源地址和与这个事务有关的缓存、内存、资源（例如可以同时监听的连接数backlog）、超时时间等。这些参数对于每个函数调用来说只是变量，但是对于整个事务来说，就是这个事务存续期间的不可变量，这就是sock结构体存在的意义。综上，sock结构体代表的是网络事务。

### socket类型与接口

socket有很多种类型，分别工作在不同的网络协议层，但是socket对外的函数和数据接口是基本一致的。常见的socket类型有用于直接发送IP数据包的Packet Socket；有用于本机进程通信的UNIX Domain Socket，有用于TCP/UDP通信的socket；有用于虚拟机与主机通信的Virtual Socket；还有用于用户与内核通信的Netlink。不同而定socket在于调用的网络服务不一样，但操作接口是一样的都是

```cpp
int socket(int family, int type, int protocol);
```

所有操作系统为了有效地管理资源，并且能被用户程序有效地索引，都会用数字编号命名资源，称为句柄。我们在用户进程可以使用指针，但是内核与用户空间之间是不能用指针引用的。通过一个句柄的分配和查询，让无论是打开的文件还是生成的socket都可以有唯一的身份ID。

**socket是一个句柄**，对于内核来说，用户必须提供这个句柄才能使用socket编程的第一步。

由于与socket函数相关的调用是通用的，但是用户总是要指定传输协议，因此传输协议就是**第一个参数family**，比如**TCP/IP协议族、OSI协议族或AppleTalk协议族.协议族的不同决定了内核使用的整个协议栈的不同**。**type参数**的设置是因为所有的传输层协议都只有两种：**数据流式和数据包式**。虽然所有的数据都是数据包形式传输的，但是从传输层来看，如果数据包可以有序且不遗漏的到达，那么就是数据流。因此数据流式的协议（如TCP）都会提供额外的传输控制，而数据包式的协议（如UDP）一般只起到了不同端口定位不同程序的作用。因此常见的**type参数**有**SOCK_STREAM和SOCK_DGTAM**两种取值。protocol指明具体的传输协议。

UDP与TCP不同但是两者要有同样的对外接口函数。目前式按照TCP的需求设计接口，UDP使用其中的一部分。

socket是内核中一个拥有状态的对象，刚创建出来的时候默认是closed，当TCP在服务端调用了listen后变为listen状态，之后会随着TCP的连接建立和关闭切换不同的状态。

不管是TCP还是UDP，在服务端都是负责监听连接，因此，需要先选择自己的IP和端口，这个函数是bind

```cpp
int bind(int sockfd, const struct sockaddr *myaddr, socklen_t addlen);
```

第一个是bind的套接字，第二个是地址和端口结构，第三个用来指明长度。ip和端口可以都指定也可以不指定。如果不指定端口，内核随机选取，需要用getspckname()手动获得，不指定IP（指定为INADDR_ANY），内核会等到TCP建立连接或UDP发出数据所使用的IP地址来设定为socket之后的IP地址，但是主socket还是继续使用通配地址。

#### 1. 服务端

```cpp
int listen(int socketfd, int backlog);
```

这是唯一一个只能由由TCP服务端调用的函数，因为这个函数的作用是识别三次握手。socket是内核资源，listen操作也是由内核完成的，内核完成三次握手完全不需要用户程序的参与。

因为内核要完成三次握手，所以要考虑多个用户同时连接的情况。

由于网络环境的不确定，用户不一定回复，回复的时间也不一定。所以内核为监听三次握手的socket维护了两个队列，一个是正在等待客户端最后回复的队列；另一个是已经成功完成三次握手的队列。

已经成功建立的队列如果有了新的条目，内核就会唤醒用户进程将已经建立的socket交给进程。如果正在等待客户端回复的队列满了，内核将无法再继续接收新的TCP SYNC连接请求，此时服务端将不回复（如果回复RST，用户端就会放弃）。通常出现这样情况要么是backlog不够大，要么收到了TCP Flood攻击。

```cpp
int accept(int sockfd, struct sockaddr *cliaddr, socklen_t *addrlen);
```

当客户端connect成功后，服务端就可以得到对应的socket，但是内核不会主动把已经建立好的socket交给用户进程，需要用户进程手动获取，这个就是accept。

如果，内核主动把建立好的socket交给用户进程，那就需要设置一个队列，当有socket建立好之后放在这里，然后通知用户进程有socket好了，这时用户进程就需要检查这个队列，并看看是因为什么原因唤醒的，这样反而增加了难度。

#### 2. 客户端

```c
int connect(int sockfd, const struct sockaddr* servaddr, socklen_t addrlen);
```

connect是客户端发起三次握手的函数，发送SYNC后等待回复的SYNC/ACK，很可能收不到回复，也可能收到目的地址不可达的ICMP。或RST回复，也可能什么都没有收到。对于不同的回复，不同的操作系统有不同的反应，比如什么都收不到就会超时重传，可以设置超时重传的次数。

### Linux socket连接模型

分为三类阻塞、多进程、I/O多路复用
