---
title: 《 Linux 性能优化实战》笔记
tags: []
---

## 01 如何学习 Linux 性能优化
性能分析，就是找出应用或系统的瓶颈，并设法去避免或者缓解它们。包含了一系列的步骤。

- 选择指标评估应用程序和系统的性能
- 为应用程序和系统设置性能目标
- 进行性能基准测试
- 性能分析定位瓶颈
- 优化系统和应用程序
- 性能监控和告警

## 02 基础篇：到底应该怎么理解“平均负载”？
平均负载(Load Average), 是指单位时间内，系统处于可运行状态(Running, Runnable, R) 和不可中断状态(Uninterruptible Sleep, Disk Sleep), 也就是平均活跃进程数。

本节使用的相关命令

```sh
uptime
stress --cpu 1 --timeout 600
watch -d uptime
mpstat -P ALL 5
pidstat -u 5 1
stress -i 1 --timeout 600
stress -c 8 --timeout 600
```

## 04 基础篇：经常说的 CPU 上下文切换是什么意思？（下）
```sh
vmstat 5
pidstat -w 5
pidstat -w -u 1
pidstat -wt 1
watch -d cat /proc/interrupts
```

- 自愿上下文切换变多了，说明进程都在等待资源，有可能发生了 I/O 等其他问题；

- 非自愿上下文切换变多了，说明进程都在被强制调度，也就是都在争抢 CPU，说明 CPU 的确成了瓶颈； 

- 中断次数变多了，说明 CPU 被中断处理程序占用，还需要通过查看 /proc/interrupts 文件来分析具体的中断类型。

## 05 基础篇：某个应用的 CPU 使用率居然达到 100%，我该怎么办？
使用 `cat /proc/stat` 获取系统 CPU 时间统计，单位一般是 10ms. 详见 `man proc`

使用 `cat /proc/[pid]/stat` 获取进程 CPU 时间统计。

平均 CPU 使用率 = 1 - (新空闲时间 - 旧空闲时间) /(新总 CPU 时间 - 旧总 CPU 时间)

`top` 默认使用 3 秒时间间隔，`ps` 默认使用进程整个生命周期，`pistat` 可以每隔 x 秒输出一组数据，共输出 x 组，并计算平均值。

`perf top (-g)` 可用于实时显示占用 CPU 时钟最多的函数或者指令，可以用来查找热门函数。

`perf record` ,  可以保存数据，用 `perf report` 显示。

## 06 基础篇：系统的 CPU 使用率很高，但为啥却找不到高 CPU 的应用？
如果观察到进程的 PID 不断变化，可能进程不断重启，也可能都是短时进程。

用 `pstree | grep ...` 可以查看短时进程调用。

`perf` 在这种情况下仍然可用。

`execsnoop` 可以实时监控短时进程。
