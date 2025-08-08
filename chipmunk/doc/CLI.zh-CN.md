# 命令行用法（中文）

## udpxy

将 UDP 组播/SSM 转发为 HTTP。

- 基本用法：`udpxy [-vTS] [-a <listenaddr>] [-m <mcast_ifc_addr>] [-c <clients>] [-l <logfile>] [-B <sizeK>] [-R <msgs>] [-H <sec>] [-n <nice_incr>] [-M <sec>] -p <port>`

常用选项：
- `-p <port>`：监听端口（必填）
- `-a <listenaddr>`：绑定的 IPv4 地址/接口（默认 `0.0.0.0`）
- `-m <mcast_ifc_addr>`：组播源（接口）IPv4 地址（默认 `0.0.0.0`）
- `-c <clients>`：最大并发客户端数（默认 3，最大 5000）
- `-l <logfile>`：日志文件（默认 stderr）
- `-B <sizeK>`：入站（UDP）socket 缓冲大小，如 `64K`、`1M`（默认 2048 字节）
- `-R <msgs>`：缓冲的最大报文数（-1 表示全部，默认 1）
- `-H <sec>`：在缓冲中保留数据的最长时间（-1 表示无限制，默认 1 秒）
- `-n <nice_incr>`：进程 nice 值增量（默认 0）
- `-M <sec>`：每隔 M 秒刷新组播订阅（0 表示不刷新，默认 0）
- `-v`：详细输出
- `-S`：启用客户端吞吐统计
- `-T`：前台运行（不以守护进程形式）

HTTP 接口：
- `GET /udp/[src@]maddr:port/`：转发流（自动识别 MPEG-TS 或 RTP-over-TS）
- `GET /rtp/[src@]maddr:port/`：按 RTP-over-TS 处理并转发
- `GET /status/`：HTML 状态页
- `GET /restart/`：重启守护进程并断开所有客户端

示例：
- 绑定并监听：`udpxy -p 4022 -m 192.168.1.1`
- 转发：`curl http://127.0.0.1:4022/udp/224.0.2.26:24012/ > stream.ts`
- SSM 转发：`curl http://127.0.0.1:4022/udp/10.0.0.5@232.22.137.57:5057/ > stream.ts`
- 状态页：浏览器打开 `http://127.0.0.1:4022/status/`

常用环境变量：
- `UDPXY_RCV_TMOUT`, `UDPXY_DHOLD_TMOUT`, `UDPXY_SREAD_TMOUT`, `UDPXY_SWRITE_TMOUT`, `UDPXY_SSEL_TMOUT`, `UDPXY_LQ_BACKLOG`, `UDPXY_SRV_RLWMARK`, `UDPXY_SSOCKBUF_NOSYNC`, `UDPXY_DSOCKBUF_NOSYNC`, `UDPXY_TCP_NODELAY`, `UDPXY_HTTP200_FTR_FILE`, `UDPXY_HTTP200_FTR_LN`, `UDPXY_ALLOW_PAUSES`, `UDPXY_PAUSE_MSEC`, `UDPXY_CONTENT_TYPE`

## udpxrec

可编程组播录制工具（通过 `udpxy` 的软链接名 `udpxrec` 调用）。

- 基本用法：`udpxrec [-v] [-b <begin_time>] [-e <end_time>] [-M <maxfilesize>] [-p <pidfile>] [-B <bufsizeK>] [-n <nice_incr>] [-u <seconds_to_wait>] [-m <mcast_ifc_addr>] [-l <logfile>] [-s <ssm_source_addr>] -c <src_addr>:<port> <dstfile>`

选项说明：
- `-c <src_addr>:<port>`：要录制的组播频道（必填）
- `-s <ssm_source_addr>`：可选 SSM 源地址
- `-b <begin_time>`：开始时间 `[+]dd:hh24:mi.ss`（`+` 表示相对时间）
- `-e <end_time>`：结束时间 `[+]dd:hh24:mi.ss`（`+` 表示相对时间）
- `-M <maxfilesize>`：目标文件最大大小（如 `1.5Gb`）
- `-p <pidfile>`：PID 文件（若以守护进程运行必须指定）
- `-B <bufsizeK>`：入站（UDP）socket 缓冲大小
- `-R <msgs>`：缓冲的最大报文数
- `-n <nice_incr>`：进程 nice 值增量
- `-u <seconds_to_wait>`：开始前的进度更新间隔（秒）
- `-m <mcast_ifc_addr>`：组播接口地址
- `-l <logfile>`：日志文件
- `-v`：详细输出

示例：
```
udpxrec -b 15:45.00 -e +2:00.00 -M 1.5Gb -n 2 -B 64K -s 87.141.215.251 -c 224.0.11.31:5050 /opt/video/tv5.mpg
```