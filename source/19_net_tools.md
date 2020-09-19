---
title: 网络工具
---

## telnet

使用命令是 `telnet [domainname or ip] [port]` 检查端口是否打开。

## netcat

netcat 简称 nc。

### 用 nc 来当聊天服务器

`nc -l` 命令可以快速的启动一个 tcp server 监听某个端口。

1. 在服务器（16.155.194.49）命令行输入 `nc -l 9090`。
2. 客户端机器的终端中输入 `nc 10.211.55.5 9090`。此时两台机器建立了一条 tcp 连接，可以聊天了。

### 查看远程端口是否打开

使用命令 `nc -zv [host or ip] [port]`。

- `-z` 表示不发送任何数据包，tcp 三次握手完后自动退出进程。
- `-v` 参数则会输出更多详细信息（verbose）。

## netstat

### 列出套接字

- `netstat -a` 输出所有的套接字，包括监听的和未监听的套接字。
- `netstat -at` 只列出 TCP 套接字。也可也用 `--tcp`。
- `netstat -au` 只列出 UDP 套接字。也可也用 `--udp`。
- `netstat -l` 列出处于 LISTEN 状态的连接，也可也用 `--listening`。
- `netstat -ltn` 常用端口都被映射为了名字，比如 22 端口输出显示为 ssh，8080 端口被映射为 webcache。大部分情况下，我们并
不想 netstat 帮我们做这样的事情，可以加上 `-n` 禁用。
- `netstat -ltnp` `-p`可以显示连接归属的进程信息。
- `netstat -i` 可以列出网卡信息。

## tcpdump

Linux 发行包都预装了 tcpdump，如果没有预装，可以用 `yum install -y tcpdump` 来进行安装。

`tcpdump -i any` 显示经过网卡的数据包，如果只想查看 eth0 网卡经过的数据包，就可以使用 `tcpdump -i eth0`。

- `-i` 指定一个网卡
- `any` 表示任意。

### 过滤主机：host

`tcpdump -i any host 10.211.55.2` 表示只查看 ip 为 `10.211.55.2` 的网络包，这个 ip 可以是源地址也可以是目标地址。

### 过滤源地址、目标地址：src、dst

如果只想抓取主机 `10.211.55.10` 发出的包

```sh
tcpdump -i any src 10.211.55.10
```

如果只想抓取主机 `10.211.55.10` 收到的包

```sh
tcpdump -i any dst 10.211.55.1
```

### 过滤端口：port

查看 80 端通信的数据包

```sh
tcpdump -i any port 80
```

只抓取 80 端口收到的包，可以加上 dst

```sh
tcpdump -i any dst port 80
```

### 过滤指定端口范围内的流量

比如抓取 21 到 23 区间所有端口的流量

```sh
tcpdump portrange 21-23
```

### 禁用主机与端口解析：-n 与 -nn

如果不加 `-n` 选项，tcpdump 会显示主机名，比如下面的 `test.ya.local` 和 `c2.shared`：

```sh
09:04:56.821206 IP test.ya.local.59915 > c2.shared.ssh: Flags [P.], seq 397:433, ack 579276, win 2048, options [nop,nop,TS val 1200089877 ecr 435612355], length 36
```

加上 `-n` 选项以后，主机名都被替换成了 ip

```sh
tcpdump -i any -n
10:02:13.705656 IP 10.211.55.2.59915 > 10.211.55.10.ssh: Flags [P.], seq 829:865, ack 1228756, win 2048, options [nop,nop,TS val 1203228910 ecr 439049239], length 36
```

但是常用端口还是会被转换成协议名，比如 ssh 协议的 22 端口。如果不想 tcpdump 做转换，可以加上 `-nn`，这样就不会解析端口了，输出
中的 ssh 变为了 22

```sh
tcpdump -i any -nn
10:07:37.598725 IP 10.211.55.2.59915 > 10.211.55.10.22: Flags [P.], seq 685:721, ack 1006224, win 2048, options [nop,nop,TS val 1203524536 ecr 439373132], length 36
```

### 过滤协议

只查看 udp 协议，可以直接使用下面的命令

```sh
tcpdump -i any -nn udp

10:25:31.457517 IP 10.211.55.10.51516 > 10.211.55.1.53: 23956+ A? www.baidu.com. (31)
10:25:31.490843 IP 10.211.55.1.53 > 10.211.55.10.51516: 23956 3/13/9 CNAME www.a.shifen.com., A 14.215.177.38, A 14.215.177.39 (506)
```

### 抓取指定个数报文

使用 `-c number` 命令可以抓取 number 个报文后退出。在网络包交互非常频繁的服务器上抓包比较有用，可能运维人员
只想抓取 1000 个包来分析一些网络问题，就比较有用了。

```sh
tcpdump -i any -nn port 80  -c 5
```

上面的命令只抓取 5 个报文。

### 数据报文输出到文件

`-w` 选项用来把数据报文输出到文件，比如下面的命令就是把所有 80 端口的数据输出到文件

```sh
tcpdump -i any port 80 -w test.pcap
```

生成的 pcap 文件就可以用 wireshark 打开进行更详细的分析了

也可以加上 `-U` 强制立即写到本地磁盘，性能稍差。

### 显示绝对的序号

默认情况下，tcpdump 显示的是从 0 开始的相对序号。如果想查看真正的绝对序号，可以用 `-S` 选项。

```sh
# 没有 -S 时的输出，可以看到 seq 是从 0 开始
tcpdump -i any port 80 -nn

12:12:37.832165 IP 10.211.55.10.46102 > 36.158.217.230.80: Flags [P.], seq 1:151, ack 1, win 229, length 150
12:12:37.832272 IP 36.158.217.230.80 > 10.211.55.10.46102: Flags [.], ack 151, win 16384, length 0

# 有 -S 时的输出, seq 不是从 0 开始
tcpdump -i any port 80 -nn -S

12:13:21.863918 IP 10.211.55.10.46074 > 36.158.217.223.80: Flags [P.], seq 4277123624:4277123774, ack 3358116659, win 229, length 150
12:13:21.864091 IP 36.158.217.223.80 > 10.211.55.10.46074: Flags [.], ack 4277123774, win 16384, length 0
```

### 运算符

tcpdump 可以用运算符 `and`（或 `&&`）、`or`（或 `||`）、`not`（或 `!`）来组合出任意复杂的过滤器

抓取 ip 为 `10.211.55.10` 到端口 3306 的数据包

```sh
tcpdump -i any host 10.211.55.10 and dst port 3306
```

抓取源 ip 为 `10.211.55.10`，目标端口除了 22 以外所有的流量

```sh
tcpdump -i any src 10.211.55.10 and not dst port 22
```
