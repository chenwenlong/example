
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

## mpstat常用使用说明
-- -P ALL 观察所有CPU使用数据，每5秒统计一次，总输出20次
$ mpstat -P ALL 5 10



## pidstat常用使用说明
1.统计各个进程的CPU使用数据
-- -u 显示各个进程的CPU使用统计数据，
$ pidstat -u 1

2.统计各个进程的上下文切换数据
-- -w 显示各个进程的上下文切换数据，-t 进一步显示线程的指标数据（与-w一起即显示线程的上下文统计数据）
$pidstat -wt

3.统计周期
-- -u 显示各个进程的CPU使用统计数据，每2秒统计一次，总输出3次
$ pidstat -u 2 3


## stress、sysstat使用说明：
### （模拟CPU密集型进程）测试CPU高运算引起的CPU占用率爆高：
-- --cpu 1 表示压满1个CPU资源
$ stress --cpu 1 --timeout 600
此处的压测，主要影响系统计数器：
us(top)、load average(top/uptime)、%CPU(top)
%usr(mpstat)
%usr(pidstat)、%CPU(pidstat)
主要消耗用户态CPU。
1.通过uptime观察到系统平均负载较高；
2.通过mpstat观察到用户态CPU使用率很高，而iowait为0，说明进程是CPU密集型。进程使用CPU密集导致系统平均负载变高、CPU使用率变高。
3.通过pidstat查看到是stress进程导致CPU使用率较高。


### （模拟IO密集型进程）测试IO wait引起的CPU占用率爆高：
-- -i 表示调用sync
$ stress -i 1 --timeout 600
此处的压测，主要影响系统计数器：
sy（top）、%CPU（top）
%sys(mpstat)
%system(pidstat)、%CPU(pidstattop)
主要消耗内核态CPU。

OR
-- -I 表示是调用sync，--hdd表示读写临时文件
$ stress-ng -i 1 --hdd 1 --timeout 600
此处的压测，更多的是影响系统计数器：
wa（top）、load average(top/uptime)，轻微影响sy（top）
%iowait（mpstat），轻微影响%sys（mpstat）；
不影响%wait（pidstat），轻微影响%system（pidstat）。
pidstat内的%wait属于CPU就绪队列等待CPU的时间，不包含IO等待时间（即不可中断睡眠时间），所以此计数器值不会被IO压测所影响。
系统平均负载很高，CPU使用率低，但iowait很高，一直在等待IO处理，说明进程是IO密集型。进程频繁进行IO操作，导致系统平均负载很高而CPU使用率不高。


### （大量进程争用CPU资源场景）测试进程过多产生的CPU资源竞争，造成%wait计数器爆高。
-c 产生8个进程；
$ stress-ng -c 8 --timeout 600
此处的压测，更多的是影响系统计数器：
us（top）、load average(top/uptime)、%CPU
%usr（mpstat）
%wait（pidstat）、%usr（pidstat）、%CPU（pidstat）
分析说明：
1.由于进程数较高超过CPU内核数，造成CPU超载（load average计数器值很高）。
2.通过mpstat观察到用户态CPU使用率很高，iowait为0，说明进程是CPU密集型或者进程间存在较高CPU争用。
3.通过观察pidstat的%wait计数器很高，说明进程间存在争用，系统中存在大量进程在等待使用CPU（进程数超过CPU内核数，较多的进程处于就绪队列阻塞着等待CPU时间）。


### （模拟CPU上下文频繁切换）测试CPU上下文切换高引起的CPU占用率爆高
-- --threads 指定线程数，--time 指定运行时间（秒为单位），threads 测试类似为threads，run 执行命令为执行正式测试。
$ sysbench --threads=10 --time=300 threads run
此处的压测，更多的是影响系统计数器：
sy（top）、load average(top/uptime)、%CPU，轻微影响us（top）
r（vmstat）、cs（vmstat）、in（ vmstat）
cswch/s（pidstat）、nvcswch/s（pidstat），轻微影响usr（pidstat）

pidstat输出所有进程的所有线程的统计数据，这样情况下输出会过多，此时可以指定进程统计输出
比如：
-- -p 指定输出117697进程的上下文统计数据（包含线程）
$ pidstat -wt -p 117697 1

1.通过vmstat观察到系统存在上百万的实时的上下文切换，以及几十万的实时中断数。证明系统中存在大量的上下文切换，消耗过多的系统内核态CPU；
2.通过pidstat发现sysbench进程的上下文切换数异常；
3.进一步指定监控进程sysbench，通过pidstat发现sysbench的上下文切换数异常的高，比较匹配系统层级上的上下文切换数，基本认定是这个进程造成的问题。
4.基于第一点发现的中断数异常高的问题，通过watch监控中断数ss，发现是RES Rescheduling interrupts，主要是重调度中断引起的问题。5756

## vmstat常用使用说明
1.统计周期
-- 每1秒统计一次，总输出5次
$ vmstat 1 5
