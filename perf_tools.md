
## 【压测工具集合】
stress：Linux 压力测试工具；
sysbench：Linux压力基准测试，包含线程上下文切换测试；
sysstat：系统性能检测工具集，包括pidstat、mpstat、sar等；
vmstat：内存、CPU、swap、IO等数据监控工具；
perf：代码函数级性能追踪工具；

## 工具安装：
stress、sysbench、
systat、perf、vmstat

sysstat基于源码安装：
https://github.com/sysstat/sysstat
1.解压安装包。
2.进入安装目录。
3.执行配置命令：./configure
4.执行编译和安装命令：sudo make && make install；


stress基于源码安装：
https://github.com/ColinIanKing/stress-ng
1.解压安装包。
2.进入安装目录。
3.执行配置命令：./configure
4.执行编译和安装命令：sudo make && make install；

sysbench基于源码安装：
https://github.com/akopytov/sysbench#installing-from-binary-packages
1.解压安装包。
2.进入安装目录。
3.执行配置命令：./configure --without-mysql    —— 暂不需要mysql支持
4.执行编译和安装命令：sudo make && make install；


vmstat基于yum命令安装：
```
$ yum install vmstat
```

perf基于yum命令安装：
```
$ yum install perf
```
