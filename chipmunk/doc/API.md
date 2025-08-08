# C API Reference

Public interfaces exposed by udpxy headers. Include relevant headers and link with udpxy sources.

- `udpxy.h`: Common constants and types
- `uopt.h`: Program options API
- `ctx.h`: Server/client context and stats
- `netop.h`: Network socket helpers
- `dpkt.h`: Packet/stream processing
- `rtp.h`: RTP parsing
- `rparse.h`: HTTP request parsing
- `ifaddr.h`: Interface and address helpers
- `prbuf.h`: In-memory printf buffer
- `mkpg.h`, `statpg.h`: Status page generation/templates
- `util.h`: Utilities (IO, logging, env, timing)

## udpxy.h
- Error codes: `ERR_PARAM`, `ERR_REQ`, `ERR_INTERNAL`
- Sizes and limits: `IPADDR_STR_SIZE`, `PORT_STR_SIZE`, `MAX_CMD_LEN`, `MAX_PARAM_LEN`, `MAX_TAIL_LEN`, `ETHERNET_MTU`
- Timeouts: `RLY_SOCK_TIMEOUT`, `SRVSOCK_TIMEOUT`, `SSEL_TIMEOUT`, `DHOLD_TIMEOUT`
- Flags: `flag_t`, `uf_TRUE`, `uf_FALSE`

## uopt.h
- Struct `udpxy_opt`: daemon options (verbosity, buffer sizes, timeouts, client limits, content type, etc.)
- Functions:
  - `int init_uopt(struct udpxy_opt* uo);`
  - `void free_uopt(struct udpxy_opt* uo);`
  - `void set_verbose(flag_t* verbose);`
  - If built with `UDPXREC_MOD`:
    - `int init_recopt(struct udpxrec_opt* ro);`
    - `void free_recopt(struct udpxrec_opt* ro);`
    - `void fprint_recopt(FILE* stream, struct udpxrec_opt* ro);`

Example:
```c
struct udpxy_opt opt; init_uopt(&opt);
opt.max_clients = 50; opt.is_verbose = uf_TRUE;
```

## ctx.h
- `struct client_ctx`: per-client state (PIDs, addresses, parsed tail)
- `struct server_ctx`: listening socket, multicast iface, clients, timeouts
- `struct tps_data`: throughput accumulation
- `struct srv_request`: parsed `/cmd/params/tail`
- Functions:
  - `int init_server_ctx(struct server_ctx*, size_t max, const char* laddr, uint16_t lport, const char* mifc_addr);`
  - `void free_server_ctx(struct server_ctx*);`
  - `int find_client(const struct server_ctx*, pid_t pid);`
  - `int add_client(struct server_ctx*, pid_t cpid, const char* maddr, uint16_t mport, int sockfd);`
  - `int delete_client(struct server_ctx*, pid_t cpid);`
  - `void tpstat_init(struct tps_data*, int setpid);`
  - `void tpstat_update(struct server_ctx*, struct tps_data*, ssize_t nbytes);`
  - `int tpstat_read(struct server_ctx*);
```

## netop.h
- Listener and multicast helpers:
  - `int setup_listener(const char* ipaddr, int port, int* sockfd, int bklog);`
  - `int setup_mcast_listener(struct sockaddr_in* saddr, struct sockaddr_in* maddr, const struct in_addr* mifaddr, int* mcastfd, int sockbuflen);`
  - `void close_mcast_listener(int msockfd, const struct in_addr* mifaddr, const struct in_addr* s_in_addr);`
  - `int set_multicast(int msockfd, const struct in_addr* mifaddr, const struct in_addr* s_in_addr, char* opname);`
  - `int renew_multicast(int msockfd, const struct in_addr* mifaddr, const struct in_addr* s_in_addr);`
- Socket tuning/timeouts:
  - `int set_timeouts(int rsock, int ssock, u_short rsec, u_short rusec, u_short ssec, u_short susec);`
  - `int set_sendbuf(int sockfd, const size_t len);`
  - `int set_rcvbuf(int sockfd, const size_t len);`
  - `int get_sendbuf(int sockfd, size_t* len);`
  - `int get_rcvbuf(int sockfd, size_t* len);`
  - `int set_nblock(int fd, int set);`
- Socket info:
  - `int get_sockinfo(int sockfd, char* addr, size_t alen, int* port);`
  - `int get_peerinfo(int sockfd, char* addr, size_t alen, int* port);`

## dpkt.h
- Stream context `struct dstream_ctx` and flags (`F_DROP_PACKET`, `F_CHECK_FMT`, `F_SCATTERED`, `F_FILE_INPUT`)
- Functions:
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
- Inspect and strip RTP headers:
  - `int RTP_check(const char* buf, size_t len, int* is_rtp, FILE* log);`
  - `int RTP_process(void** pbuf, size_t* len, int verify, FILE* log);`
  - `int RTP_verify(const char* buf, size_t len, FILE* log);`
  - `int RTP_hdrlen(const char* buf, size_t len, size_t* hdrlen, FILE* log);`

## rparse.h
- HTTP request parsing:
  - `int get_request(const char* src, size_t srclen, char* request, size_t* rqlen);`
  - `int parse_param(const char* s, size_t slen, char* cmd, size_t clen, char* opt, size_t optlen, char* tail, size_t tlen);`
  - `int parse_udprelay(const char* opt, size_t optlen, char* s_addr, size_t s_addrlen, char* addr, size_t addrlen, uint16_t* port);`

## ifaddr.h
- Addresses and parsing:
  - `int if2addr(const char* ifname, struct sockaddr* addr, size_t addrlen);`
  - `int get_ipv4_address(const char* s, char* buf, size_t len);`
  - `int get_addrport(const char* s, char* addr, size_t len, int* port);`

## prbuf.h
- In-memory printf buffer:
  - `int prbuf_open(prbuf_t* pb, void* buf, size_t n);`
  - `int prbuf_close(prbuf_t pb);`
  - `int prbuf_printf(prbuf_t pb, const char* format, ...);`
  - `size_t prbuf_len(prbuf_t pb);`
  - `void prbuf_rewind(prbuf_t pb);`

## mkpg.h and statpg.h
- Generate HTML status page from `server_ctx`:
  - `int mk_status_page(const struct server_ctx* ctx, char* buf, size_t* len, int options);`
  - Options bitmask: `MSO_HTTP_HEADER`, `MSO_SKIP_CLIENTS`, `MSO_RESTART`
- `statpg.h` contains exported HTML templates and constants if needed by tests.

## util.h
- Files and IO: `save_buffer`, `txtf_read`, `write_buf`, `read_buf`, `hex_dump`, `sizecheck`, `buf_overrun`
- Process/daemon: `daemonize`, `make_pidfile`, `set_pidfile`, `set_nice`, `printcmdln`
- Time/format: `tmfprintf`, `tmfputs`, `a2time`, `a2size`, `a2int64`, `Zasctime`, `mk_tvstamp`
- Env/config: `get_timeval`, `get_flagval`, `get_sizeval`, `get_pidstr`, `get_sysinfo`, `would_block`, `no_fault`, `mk_app_info`

Minimal example: request parsing and address split
```c
char rq[256]; size_t rqlen = sizeof(rq);
get_request("GET /udp/224.0.2.26:1234/ HTTP/1.1\r\n\r\n", 40, rq, &rqlen);
char cmd[16], opt[128], tail[256];
parse_param(rq, rqlen, cmd, sizeof(cmd), opt, sizeof(opt), tail, sizeof(tail));
char addr[32]; uint16_t port; char saddr[32];
parse_udprelay(opt, strlen(opt)+1, saddr, sizeof(saddr), addr, sizeof(addr), &port);
```