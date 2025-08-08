# CLI Usage

## udpxy

Relay UDP multicast/SSM to HTTP.

- Usage: `udpxy [-vTS] [-a <listenaddr>] [-m <mcast_ifc_addr>] [-c <clients>] [-l <logfile>] [-B <sizeK>] [-R <msgs>] [-H <sec>] [-n <nice_incr>] [-M <sec>] -p <port>`

Common options:
- `-p <port>`: Port to listen on (required)
- `-a <listenaddr>`: IPv4 address/interface to bind (default `0.0.0.0`)
- `-m <mcast_ifc_addr>`: IPv4 address/interface for multicast source (default `0.0.0.0`)
- `-c <clients>`: Max concurrent clients (default 3, max 5000)
- `-l <logfile>`: Log file (default stderr)
- `-B <sizeK>`: UDP socket buffer size, e.g. `64K`, `1M` (default 2048 bytes)
- `-R <msgs>`: Max messages buffered (-1 = all, default 1)
- `-H <sec>`: Max seconds to hold data in buffer (-1 = unlimited, default 1)
- `-n <nice_incr>`: Increment process nice (default 0)
- `-M <sec>`: Renew multicast subscription every M seconds (0 to skip, default 0)
- `-v`: Verbose output
- `-S`: Enable client throughput statistics
- `-T`: Do not daemonize (stay in foreground)

HTTP endpoints:
- `GET /udp/[src@]maddr:port/`: relay stream (auto-detect MPEG-TS vs RTP over TS)
- `GET /rtp/[src@]maddr:port/`: relay assuming RTP over TS
- `GET /status/`: HTML status page
- `GET /restart/`: restart daemon and drop clients

Examples:
- Bind and listen: `udpxy -p 4022 -m 192.168.1.1`
- Relay: `curl http://127.0.0.1:4022/udp/224.0.2.26:24012/ > stream.ts`
- SSM relay: `curl http://127.0.0.1:4022/udp/10.0.0.5@232.22.137.57:5057/ > stream.ts`
- Status: open `http://127.0.0.1:4022/status/`

Environment variables:
- `UDPXY_RCV_TMOUT`, `UDPXY_DHOLD_TMOUT`, `UDPXY_SREAD_TMOUT`, `UDPXY_SWRITE_TMOUT`, `UDPXY_SSEL_TMOUT`, `UDPXY_LQ_BACKLOG`, `UDPXY_SRV_RLWMARK`, `UDPXY_SSOCKBUF_NOSYNC`, `UDPXY_DSOCKBUF_NOSYNC`, `UDPXY_TCP_NODELAY`, `UDPXY_HTTP200_FTR_FILE`, `UDPXY_HTTP200_FTR_LN`, `UDPXY_ALLOW_PAUSES`, `UDPXY_PAUSE_MSEC`, `UDPXY_CONTENT_TYPE`

## udpxrec

Programmable multicast recorder (via `udpxy` symlink `udpxrec`).

- Usage: `udpxrec [-v] [-b <begin_time>] [-e <end_time>] [-M <maxfilesize>] [-p <pidfile>] [-B <bufsizeK>] [-n <nice_incr>] [-u <seconds_to_wait>] [-m <mcast_ifc_addr>] [-l <logfile>] [-s <ssm_source_addr>] -c <src_addr>:<port> <dstfile>`

Options:
- `-c <src_addr>:<port>`: Multicast channel to record (required)
- `-s <ssm_source_addr>`: Optional SSM source
- `-b <begin_time>`: Start at `[+]dd:hh24:mi.ss` (use `+` for relative)
- `-e <end_time>`: Stop at `[+]dd:hh24:mi.ss` (use `+` for relative)
- `-M <maxfilesize>`: Max destination file size (e.g., `1.5Gb`)
- `-p <pidfile>`: PID file (required if running as daemon)
- `-B <bufsizeK>`: UDP socket buffer size
- `-R <msgs>`: Max messages buffered
- `-n <nice_incr>`: Increment process nice
- `-u <seconds_to_wait>`: Progress update interval before start
- `-m <mcast_ifc_addr>`: Multicast interface address
- `-l <logfile>`: Log file
- `-v`: Verbose

Example:
```
udpxrec -b 15:45.00 -e +2:00.00 -M 1.5Gb -n 2 -B 64K -s 87.141.215.251 -c 224.0.11.31:5050 /opt/video/tv5.mpg
```