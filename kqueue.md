## kqueue
FreeBSD 引入了kqueue接口。(mac os是FreeBSD的吗)  
在开始之前得弄清楚如下几个东西。  
struct event  
kevent()  
kqueue

struct event 是kevent()操作的最基本的事件结构。  
kevent()是一个系统调用syscall,  
kqueue 则是freebsd内核中的一个事件队列kernel queue.  
kevent()相当于kqueue的用户接口,是对Kqueue进行添加、删除操作的用户接口。  
看看2个函数
```c
//上面涉及到一个结构体kevent 在<sys/event.h>中   
struct kevent {
    uintptr_t   ident;      /* identifier for this event */ 
    int16_t     filter;     /* filter for event */
    uint16_t    flags;      /* general flags */
    uint32_t    fflags;     /* filter-specific flags */
    intptr_t    data;       /* filter-specific data */
    void        *udata;     /* opaque user data identifier */
};
kqueue()
```

先来说说kevent结构体的字段  
ident 标记事件的描述符, socketfd, filefd, signal.用于存储kqueue的唯一标识, 如果你想给一个事件添加一个文件描述符的话, ident成员就应当被设置成目标描述符的值。  

filter: 指定你希望内核用于ident成员的过滤器类型

filter     | 描述      
-------- | ---------   
EVFILT_AIO | 异步I/O事件  
EVFILT_PROC | 进程exit,fork,exec事件  
EVFILT_READ | 检测描述符什么时候可读, 类似select  
EVFILT_WRITE | 检测描述符什么时候可写,类似select  
EVFILT_SIGNAL | 收到信号  
EVFILT_TIMER | 周期性或一次性的定时器  
EVFILT_VNODE | 检测文件系统上文件修改和删除事件  


flags: 是在调用时指定过滤器更改行为，在返回时额外给出条件。(告诉内核应当对该事件完成哪些操作和处理哪些必要的标志，返回时，flags可用于保存错误条件)  

flag     | 描述      |  更改  |  返回
-------- | --------- |  ----- | ------   
EV_ADD   | 增设事件  |   是   |  
EV_CLEAR | 用户获取后复位事件状态  |   是   | 
EV_DELETE| 删除事件    |   是   | 
EV_DISABLE| 禁用事件但不删除    |   是   | 
EV_ENABLE | 启用之前被禁用的事件    |   是   | 
EV_ONESHOT | 触发一次后删除事件    |   是   | 
EV_EOF | 读取方已经关闭了连接   |  | 是  
EV_ERROR | 发生错误:errno值在data成员中 |  | 是  

fflags: 用于指定想让内核使用的特定于过滤器的标志。返回时，fflags成员可用于保存特定于过滤器的返回值。  

data: 用于保存任何特定于过滤器的数据。 (NULL呢 不保存吗)  

udata: 并不由kqueue使用，kqueue会把它的值不加修改的透传。可被进程用来发送信息甚至是一个函数给它自己，用于一些依赖事件检测的场合。(用户指定的数据)  

再来看下kevent()
```c
kevent(int kq,   
const struct kevent *changelist,   
int nchanges,  
struct kevent *eventlist,  
int nevents,  
const struct timespec *timeout);
```
第一个参数kq 就是kqueue的返回值。  
changelist 是一个大小为nchanges的kevent结构体数组。changelist参数用于注册获修改事件，并且将在从kqueue读出事件之前得到处理。  
nchanges 是指changelist有几个kevent。  
eventlist 是一个大小为nevents的kevent结构体数组。(kevent通过把事件放在eventlist参数中来向调用进程返回事件[如果需要的话,eventlist和changelist可以指向同一个数组]).  
nevents 是指eventlist有几个kevent.  
timeout kevent期待的超时时间。为NULL,kevent将阻塞,直至有事件发生为止。不为NULL，则kevent将阻塞到超时为止。  

#### 代码部分
[见示例代码](https://github.com/reallovelei/Net/blob/master/server.c)

