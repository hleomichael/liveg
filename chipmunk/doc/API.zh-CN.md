# C API 参考（中文）

udpxy 暴露的一组公共头文件与接口。使用时包含相应头文件，并与源码一同编译/链接。

- `udpxy.h`：通用常量与类型
- `uopt.h`：程序选项 API
- `ctx.h`：服务端/客户端上下文与统计
- `netop.h`：网络 socket 辅助函数
- `dpkt.h`：分组/数据流处理
- `rtp.h`：RTP 解析
- `rparse.h`：HTTP 请求解析
- `ifaddr.h`：接口与地址工具
- `prbuf.h`：内存打印缓冲
- `mkpg.h`, `statpg.h`：状态页生成/模板
- `util.h`：通用工具（IO、日志、环境、时间）

## udpxy.h
- 错误码：`ERR_PARAM`, `ERR_REQ`, `ERR_INTERNAL`
- 尺寸限制：`IPADDR_STR_SIZE`, `PORT_STR_SIZE`, `MAX_CMD_LEN`, `MAX_PARAM_LEN`, `MAX_TAIL_LEN`, `ETHERNET_MTU`
- 超时：`RLY_SOCK_TIMEOUT`, `SRVSOCK_TIMEOUT`, `SSEL_TIMEOUT`, `DHOLD_TIMEOUT`
- 标志：`flag_t`, `uf_TRUE`, `uf_FALSE`

## uopt.h
- 结构体 `udpxy_opt`：守护进程选项（日志/详细、缓冲、超时、客户端上限、内容类型等）
- 函数：
  - `int init_uopt(struct udpxy_opt* uo);`
  - `void free_uopt(struct udpxy_opt* uo);`
  - `void set_verbose(flag_t* verbose);`
  - 若定义 `UDPXREC_MOD`：
    - `int init_recopt(struct udpxrec_opt* ro);`
    - `void free_recopt(struct udpxrec_opt* ro);`
    - `void fprint_recopt(FILE* stream, struct udpxrec_opt* ro);`

示例：
```c
struct udpxy_opt opt; init_uopt(&opt);
opt.max_clients = 50; opt.is_verbose = uf_TRUE;
```

## ctx.h
- `struct client_ctx`：单客户端状态（PID、地址、tail 等）
- `struct server_ctx`：监听 socket、组播接口、客户端数组、超时
- `struct tps_data`：吞吐累计
- `struct srv_request`：解析后的 `/cmd/params/tail`
- 函数：
  - `int init_server_ctx(struct server_ctx*, size_t max, const char* laddr, uint16_t lport, const char* mifc_addr);`
  - `void free_server_ctx(struct server_ctx*);`
  - `int find_client(const struct server_ctx*, pid_t pid);`
  - `int add_client(struct server_ctx*, pid_t cpid, const char* maddr, uint16_t mport, int sockfd);`
  - `int delete_client(struct server_ctx*, pid_t cpid);`
  - `void tpstat_init(struct tps_data*, int setpid);`
  - `void tpstat_update(struct server_ctx*, struct tps_data*, ssize_t nbytes);`
  - `int tpstat_read(struct server_ctx*);`

## netop.h
- 监听与组播：
  - `int setup_listener(const char* ipaddr, int port, int* sockfd, int bklog);`
  - `int setup_mcast_listener(struct sockaddr_in* saddr, struct sockaddr_in* maddr, const struct in_addr* mifaddr, int* mcastfd, int sockbuflen);`
  - `void close_mcast_listener(int msockfd, const struct in_addr* mifaddr, const struct in_addr* s_in_addr);`
  - `int set_multicast(int msockfd, const struct in_addr* mifaddr, const struct in_addr* s_in_addr, char* opname);`
  - `int renew_multicast(int msockfd, const struct in_addr* mifaddr, const struct in_addr* s_in_addr);`
- Socket 调优/超时：
  - `int set_timeouts(int rsock, int ssock, u_short rsec, u_short rusec, u_short ssec, u_short susec);`
  - `int set_sendbuf(int sockfd, const size_t len);`
  - `int set_rcvbuf(int sockfd, const size_t len);`
  - `int get_sendbuf(int sockfd, size_t* len);`
  - `int get_rcvbuf(int sockfd, size_t* len);`
  - `int set_nblock(int fd, int set);`
- Socket 信息：
  - `int get_sockinfo(int sockfd, char* addr, size_t alen, int* port);`
  - `int get_peerinfo(int sockfd, char* addr, size_t alen, int* port);`

## dpkt.h
- 数据流上下文 `struct dstream_ctx` 与标志（`F_DROP_PACKET`, `F_CHECK_FMT`, `F_SCATTERED`, `F_FILE_INPUT`）
- 函数：
  - `const char* fmt2str(upxfmt_t fmt);`
  - `upxfmt_t get_fstream_type(int fd, FILE* log);`
  - `upxfmt_t get_mstream_type(const char* data, size_t len, FILE* log);`
  - `ssize_t read_frecord(int fd, char* data, size_t len, upxfmt_t* stream_type, FILE* log);`
  - `ssize_t write_frecord(int fd, const char* data, size_t len, upxfmt_t sfmt, upxfmt_t dfmt, FILE* log);`
  - `void reset_pkt_registry(struct dstream_ctx* ds);`
  - `void free_dstream_ctx(struct dstream_ctx* ds);`
  - `int init_dstream_ctx(struct dstream_ctx* ds, const char* cmd, const char* fname, ssize_t nmsgs);`
  - `ssize_t read_data(struct dstream_ctx* spc, int fd, char* data, ssize_t data_len, const struct rdata_opt* opt);`
  - `ssize_t write_data(const struct dstream_ctx* spc, const char* data, ssize_t len, int fd);`

## rtp.h
- 检查与剥离 RTP 头：
  - `int RTP_check(const char* buf, size_t len, int* is_rtp, FILE* log);`
  - `int RTP_process(void** pbuf, size_t* len, int verify, FILE* log);`
  - `int RTP_verify(const char* buf, size_t len, FILE* log);`
  - `int RTP_hdrlen(const char* buf, size_t len, size_t* hdrlen, FILE* log);`

## rparse.h
- HTTP 请求解析：
  - `int get_request(const char* src, size_t srclen, char* request, size_t* rqlen);`
  - `int parse_param(const char* s, size_t slen, char* cmd, size_t clen, char* opt, size_t optlen, char* tail, size_t tlen);`
  - `int parse_udprelay(const char* opt, size_t optlen, char* s_addr, size_t s_addrlen, char* addr, size_t addrlen, uint16_t* port);`

## ifaddr.h
- 地址与解析：
  - `int if2addr(const char* ifname, struct sockaddr* addr, size_t addrlen);`
  - `int get_ipv4_address(const char* s, char* buf, size_t len);`
  - `int get_addrport(const char* s, char* addr, size_t len, int* port);`

## prbuf.h
- 内存打印缓冲：
  - `int prbuf_open(prbuf_t* pb, void* buf, size_t n);`
  - `int prbuf_close(prbuf_t pb);`
  - `int prbuf_printf(prbuf_t pb, const char* format, ...);`
  - `size_t prbuf_len(prbuf_t pb);`
  - `void prbuf_rewind(prbuf_t pb);`

## mkpg.h 与 statpg.h
- 基于 `server_ctx` 生成 HTML 状态页：
  - `int mk_status_page(const struct server_ctx* ctx, char* buf, size_t* len, int options);`
  - 选项位：`MSO_HTTP_HEADER`, `MSO_SKIP_CLIENTS`, `MSO_RESTART`
- `statpg.h` 暴露 HTML 模板与常量（单元测试可用）。

## util.h
- 文件/IO：`save_buffer`, `txtf_read`, `write_buf`, `read_buf`, `hex_dump`, `sizecheck`, `buf_overrun`
- 进程/守护：`daemonize`, `make_pidfile`, `set_pidfile`, `set_nice`, `printcmdln`
- 时间/格式：`tmfprintf`, `tmfputs`, `a2time`, `a2size`, `a2int64`, `Zasctime`, `mk_tvstamp`
- 环境/配置：`get_timeval`, `get_flagval`, `get_sizeval`, `get_pidstr`, `get_sysinfo`, `would_block`, `no_fault`, `mk_app_info`

最小示例：请求解析与地址拆分
```c
char rq[256]; size_t rqlen = sizeof(rq);
get_request("GET /udp/224.0.2.26:1234/ HTTP/1.1\r\n\r\n", 40, rq, &rqlen);
char cmd[16], opt[128], tail[256];
parse_param(rq, rqlen, cmd, sizeof(cmd), opt, sizeof(opt), tail, sizeof(tail));
char addr[32]; uint16_t port; char saddr[32];
parse_udprelay(opt, strlen(opt)+1, saddr, sizeof(saddr), addr, sizeof(addr), &port);
```