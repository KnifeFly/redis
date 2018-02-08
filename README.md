# Redis 4.0 源码

> 提交截至到 `f17d82961da78933e9311b122a2ac699b3fde0f9`

- 首先编译一个Redis

```bash
make noopt
```

- 我们找一个简单的命令，gdb下断点，把调用栈打出来：

```bash
gdb --quiet ./redis-server
Reading symbols from ./redis-server...done.
(gdb) b pingCommand 
Breakpoint 1 at 0x3897e: file server.c, line 2664.
(gdb) run
Starting program: /home/jiajun/Code/redis/src/redis-server 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
20208:C 08 Feb 10:53:07.890 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
20208:C 08 Feb 10:53:07.890 # Redis version=4.0.8, bits=64, commit=f17d8296, modified=0, pid=20208, just started
20208:C 08 Feb 10:53:07.890 # Warning: no config file specified, using the default config. In order to specify a config file use /home/jiajun/Code/redis/src/redis-server /path/to/redis.conf
20208:M 08 Feb 10:53:07.891 * Increased maximum number of open files to 10032 (it was originally set to 1024).
[New Thread 0x7ffff6dff700 (LWP 20213)]
[New Thread 0x7ffff65fe700 (LWP 20214)]
[New Thread 0x7ffff5dfd700 (LWP 20215)]
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 4.0.8 (f17d8296/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 20208
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

20208:M 08 Feb 10:53:07.892 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
20208:M 08 Feb 10:53:07.892 # Server initialized
20208:M 08 Feb 10:53:07.892 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
20208:M 08 Feb 10:53:07.892 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.
20208:M 08 Feb 10:53:07.892 * Ready to accept connections

Thread 1 "redis-server" hit Breakpoint 1, pingCommand (c=0x7ffff6f18b40) at server.c:2664
2664	    if (c->argc > 2) {
(gdb) info stack
#0  pingCommand (c=0x7ffff6f18b40) at server.c:2664
#1  0x000055555558b8d0 in call (c=0x7ffff6f18b40, flags=15) at server.c:2226
#2  0x000055555558c4a6 in processCommand (c=0x7ffff6f18b40) at server.c:2507
#3  0x000055555559c13b in processInputBuffer (c=0x7ffff6f18b40) at networking.c:1336
#4  0x000055555559c514 in readQueryFromClient (el=0x7ffff6e2f0a0, fd=8, privdata=0x7ffff6f18b40, 
    mask=1) at networking.c:1426
#5  0x000055555558350a in aeProcessEvents (eventLoop=0x7ffff6e2f0a0, flags=11) at ae.c:421
#6  0x0000555555583694 in aeMain (eventLoop=0x7ffff6e2f0a0) at ae.c:464
#7  0x000055555558fed4 in main (argc=1, argv=0x7fffffffde98) at server.c:3887
```

最后一段就是调用栈，然后我们从后往前跳到对应文件的对应行号，找到对应的函数，慢慢来看：

```
#0  pingCommand (c=0x7ffff6f18b40) at server.c:2664
#1  0x000055555558b8d0 in call (c=0x7ffff6f18b40, flags=15) at server.c:2226
#2  0x000055555558c4a6 in processCommand (c=0x7ffff6f18b40) at server.c:2507
#3  0x000055555559c13b in processInputBuffer (c=0x7ffff6f18b40) at networking.c:1336
#4  0x000055555559c514 in readQueryFromClient (el=0x7ffff6e2f0a0, fd=8, privdata=0x7ffff6f18b40, 
    mask=1) at networking.c:1426
#5  0x000055555558350a in aeProcessEvents (eventLoop=0x7ffff6e2f0a0, flags=11) at ae.c:421
#6  0x0000555555583694 in aeMain (eventLoop=0x7ffff6e2f0a0) at ae.c:464
#7  0x000055555558fed4 in main (argc=1, argv=0x7fffffffde98) at server.c:3887
```
