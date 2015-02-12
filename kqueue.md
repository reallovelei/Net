## kqueue
FreeBSD 引入了kqueue接口。(mac os是FreeBSD的吗)  
主要有2个函数
```c
kqueue()
kevent(int kq, const struct kevent *changelist, int nchanges, struct kevent *eventlist, int nevents, const struct timespec *timeout);
//上面涉及到一个结构体kevent 在<sys/event.h>中   
struct kevent {
    uintptr_t   ident;      /* identifier for this event */
    int16_t     filter;     /* filter for event */
    uint16_t    flags;      /* general flags */
    uint32_t    fflags;     /* filter-specific flags */
    intptr_t    data;       /* filter-specific data */
    void        *udata;     /* opaque user data identifier */
};
```
先来说说kevent结构体的字段  
flags: 是在调用时指定过滤器更改行为，在返回时额外给出条件。  

flag     | 描述    
-------- | ---------  
EV_ADD   | 增设事件  
EV_CLEAR | 用户获取后复位事件状态

第一个参数kq 就是kqueue的返回值。
