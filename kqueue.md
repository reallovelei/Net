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
kevent(int kq, const struct kevent *changelist, int nchanges, struct kevent *eventlist, int nevents, const struct timespec *timeout);
```
先来说说kevent结构体的字段  
ident: 标记事件的描述符, socketfd, filefd, signal.

flags: 是在调用时指定过滤器更改行为，在返回时额外给出条件。  

flag     | 描述      |  更改  |  返回
-------- | --------- |  ----- | ------   
EV_ADD   | 增设事件  |   是   |  
EV_CLEAR | 用户获取后复位事件状态  |   是   | 
EV_DELETE| 删除事件    |   是   | 
EV_DISABLE| 禁用事件但不删除    |   是   | 
EV_ENABLE | 启用之前被禁用的事件    |   是   | 
EV_ONESHOT | 触发一次后删除事件    |   是   | 
-------- | --------- |  ----- | ------   
EV_EOF | 发生EOF条件   |  | 是
EV_ERROR | 发生错误:errno值在data成员中 |  | 是

filter: 指定的过滤器类型
flag     | 描述      
-------- | ---------   
EVFILT_AIO | 异步I/O事件  
EVFILT_PROC | 进程exit,fork,exec事件  
EVFILT_READ | 描述符刻度, 类似select  
EVFILT_WRITE | 描述符可写,类似select  
EVFILT_SIGNAL | 收到信号  
EVFILT_TIMER | 周期性或一次性的定时器  
EVFILT_VNODE | 文件修改和删除事件  

udata: 用户指定的数据

第一个参数kq 就是kqueue的返回值。
