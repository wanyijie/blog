【sysctl.conf工作原理】

sysctl命令被用于在内核运行时动态地修改内核的运行参数，可用的内核参数在目录/proc/sys中。它包含一些TCP/IP堆栈和虚拟内存系统的高级选项， 这可以让有经验的管理员提高引人注目的系统性能。用sysctl可以读取设置超过五百个系统变量。

【sysctl.conf设置】

#优化TCP
vi /etc/sysctl.conf
#禁用包过滤功能 
net.ipv4.ip_forward = 0  
#启用源路由核查功能 
net.ipv4.conf.default.rp_filter = 1  
#禁用所有IP源路由 
net.ipv4.conf.default.accept_source_route = 0  
#使用sysrq组合键是了解系统目前运行情况，为安全起见设为0关闭
kernel.sysrq = 0  
#控制core文件的文件名是否添加pid作为扩展
kernel.core_uses_pid = 1  
#开启SYN Cookies，当出现SYN等待队列溢出时，启用cookies来处理
net.ipv4.tcp_syncookies = 1  
#每个消息队列的大小（单位：字节）限制
kernel.msgmnb = 65536  
#整个系统最大消息队列数量限制
kernel.msgmax = 65536  
#单个共享内存段的大小（单位：字节）限制，计算公式64G*1024*1024*1024(字节)
kernel.shmmax = 68719476736  
#所有内存大小（单位：页，1页 = 4Kb），计算公式16G*1024*1024*1024/4KB(页)
kernel.shmall = 4294967296  
#timewait的数量，默认是180000
net.ipv4.tcp_max_tw_buckets = 6000  
#开启有选择的应答
net.ipv4.tcp_sack = 1  
#支持更大的TCP窗口. 如果TCP窗口最大超过65535(64K), 必须设置该数值为1
net.ipv4.tcp_window_scaling = 1  
#TCP读buffer
net.ipv4.tcp_rmem = 4096 131072 1048576
#TCP写buffer
net.ipv4.tcp_wmem = 4096 131072 1048576   
#为TCP socket预留用于发送缓冲的内存默认值（单位：字节）
net.core.wmem_default = 8388608
#为TCP socket预留用于发送缓冲的内存最大值（单位：字节）
net.core.wmem_max = 16777216  
#为TCP socket预留用于接收缓冲的内存默认值（单位：字节）  
net.core.rmem_default = 8388608
#为TCP socket预留用于接收缓冲的内存最大值（单位：字节）
net.core.rmem_max = 16777216
#每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目
net.core.netdev_max_backlog = 262144  
#web应用中listen函数的backlog默认会给我们内核参数的net.core.somaxconn限制到128，而nginx定义的NGX_LISTEN_BACKLOG默认为511，所以有必要调整这个值
net.core.somaxconn = 262144  
#系统中最多有多少个TCP套接字不被关联到任何一个用户文件句柄上。这个限制仅仅是为了防止简单的DoS***，不能过分依靠它或者人为地减小这个值，更应该增加这个值(如果增加了内存之后)
net.ipv4.tcp_max_orphans = 3276800  
#记录的那些尚未收到客户端确认信息的连接请求的最大值。对于有128M内存的系统而言，缺省值是1024，小内存的系统则是128
net.ipv4.tcp_max_syn_backlog = 262144  
#时间戳可以避免序列号的卷绕。一个1Gbps的链路肯定会遇到以前用过的序列号。时间戳能够让内核接受这种“异常”的数据包。这里需要将其关掉
net.ipv4.tcp_timestamps = 0  
#为了打开对端的连接，内核需要发送一个SYN并附带一个回应前面一个SYN的ACK。也就是所谓三次握手中的第二次握手。这个设置决定了内核放弃连接之前发送SYN+ACK包的数量
net.ipv4.tcp_synack_retries = 1  
#在内核放弃建立连接之前发送SYN包的数量
net.ipv4.tcp_syn_retries = 1  
#开启TCP连接中time_wait sockets的快速回收
net.ipv4.tcp_tw_recycle = 1  
#开启TCP连接复用功能，允许将time_wait sockets重新用于新的TCP连接（主要针对time_wait连接）
net.ipv4.tcp_tw_reuse = 1  
#1st低于此值,TCP没有内存压力,2nd进入内存压力阶段,3rdTCP拒绝分配socket(单位：内存页)
net.ipv4.tcp_mem = 94500000 915000000 927000000   
#如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间。对端可以出错并永远不关闭连接，甚至意外当机。缺省值是60 秒。2.2 内核的通常值是180秒，你可以按这个设置，但要记住的是，即使你的机器是一个轻载的WEB服务器，也有因为大量的死套接字而内存溢出的风险，FIN- WAIT-2的危险性比FIN-WAIT-1要小，因为它最多只能吃掉1.5K内存，但是它们的生存期长些。
net.ipv4.tcp_fin_timeout = 15  
#表示当keepalive起用的时候，TCP发送keepalive消息的频度（单位：秒）
net.ipv4.tcp_keepalive_time = 30  
#对外连接端口范围
net.ipv4.ip_local_port_range = 2048 65000
#表示文件句柄的最大数量
fs.file-max = 102400</pre>

最后记得刷新立即生效
```/sbin/sysctl -p```

【limits.conf工作原理】
limits.conf 文件实际是 Linux PAM（插入式认证模块，Pluggable Authentication Modules）中 pam_limits.so 的配置文件，而且只针对于单个会话。
当用户访问服务器，服务程序将请求发送到PAM模块，PAM模块根据服务名称在`/etc/pam.d`目录下选择一个对应的服务文件，然后根据服务文件的内容选择具体的PAM模块进行处理。

【limits.conf设置】

1）暂时生效，适用于通过 `ulimit` 命令登录 shell 会话期间

ulimit -SHn 65535

2）永久生效，通过将一个相应的 ulimit 语句添加到由登录 shell 读取的文件之一（例如 ~/.profile），即特定于 shell 的用户资源文件；或者通过编辑`/etc/security/limits.conf`

#比如添加到/etc/profile
echo ulimit -SHn 65535 >> /etc/profile
source /etc/profile

#修改最大进程和最大文件打开数限制
vi /etc/security/limits.conf
* soft nproc 11000
* hard nproc 11000
* soft nofile 655350
* hard nofile 655350</pre>
