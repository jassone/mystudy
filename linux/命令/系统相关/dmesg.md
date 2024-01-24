## dmesg

dmesg指令是一个在Linux系统中查看内核日志的实用工具。它允许我们查看系统内核的输出消息，包括引导信息、硬件检测、设备驱动程序和系统错误等。通过使用dmesg指令，我们可以追踪系统启动过程中的事件，排查故障和问题。

有时候可以查看内存溢出问题，比如如下的日志

```sh

# dmesg
[15975.012058] [ 209756]     0 209756 11111138  6623232 59961344        0           984 ffmpeg
[15975.012060] [ 211576]     0 211576     2927       80    61440        0           992 run_logtail.sh
[15975.722451] oom_reaper: reaped process 209756 (ffmpeg), now anon-rss:0kB, file-rss:0kB, shmem-rss:0kB
[15975.009397] ffmpeg invoked oom-killer: gfp_mask=0x6000c0(GFP_KERNEL), nodemask=(null), order=0, oom_score_adj=984
[15975.009399] ffmpeg cpuset=cri-containerd-31c9420554417fab54bcf85cd1628e7e08140567bc63c636d65ad096781746a2.scope mems_allowed=0
[15975.009405] CPU: 7 PID: 209756 Comm: ffmpeg Kdump: loaded Not tainted 4.19.91-27.al7.x86_64 #1
```







