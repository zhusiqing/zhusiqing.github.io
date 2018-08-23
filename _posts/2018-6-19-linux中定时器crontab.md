---
layout:     post
title:      linux中定时器crontab
subtitle:   linux,crontab,定时器
date:       2018-06-19
author:     siqing
header-img: img/post-bg-debug.png
catalog: true
tags:
    - centos
    - linux
    - crontab
---

## 修改文文件夹所有者和权限
```bash
# 改变tmp的所有者为root，群组为staff。
chown -hR root:staff tmp
```
## 端口查询和关闭

- 查询指定端口的PID
```bash 
lsof -i tcp:8886
```

- 关闭指定PID的端口
```bash
kill -9 8886
```
- 查询所有端口
```bash
lsof -i -P | grep -i "listen"
```

## 定时器crontab

### 使用

```bash
# 修改crontab文件，如果不存在则创建
crontab -e 
# 显示crontab文件
crontab -l
# 删除文件
crontab -r
# 删除前提醒用户
```

### 参数说明

**[minute] [hour] [day-of-month] [month] [day-of-week] [full-path-to-shell-script]**


| 参数 | 参数值 | 备注 |
| --- | --- | --- |
| minute | 0-59 | |
| hour | 0-23 | |
| day-of-month | 0-31 | |
| month | 1-12 | 1是一月 |
| day-of-week | 0-7 | 周日可以是0或者7|

### 语法说明

| 标识 | 说明 | 备注 |
| --- | --- | --- |
| * | 代表所有的取值范围内的数字 | 就是所有的意思 |
| / | 代表每一定时间间隔 | */10 * * * * 表示每10分执行一次 |
| - | 代表区间范围，是闭区间 | |
| , | 分散的数字 | 可以不连续 |

> * 0-23/2 * * * 表示 在0~23点内每2小时执行一次

### Crontab 示例

1. 在 12:01 a.m 运行，即每天凌晨过一分钟。这是一个恰当的进行备份的时间，因为此时系统负载不大。

``` bash
1 0 * * * /root/bin/backup.sh
```

2. 每个工作日(Mon – Fri) 11:59 p.m 都进行备份作业。

``` bash
59 11 * * 1,2,3,4,5 echo "日志记录" >> /tmp/log.txt
```

下面例子与上面的例子效果一样：

``` bash
59 11 * * 1-5 echo "日志记录" >> /tmp/log.txt
```

3. 每5分钟运行一次命令

``` bash
*/5 * * * * /root/bin/check-status.sh
```

4. 每个月的第一天 1:10 p.m 运行

``` bash
10 13 1 * * /root/bin/full-backup.sh
```

5. 每个工作日 11 p.m 运行。

``` bash
0 23 * * 1-5 /root/bin/incremental-backup.sh
```
