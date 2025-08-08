# udpxy 文档（中文）

- 查看命令行用法与示例：[`CLI.zh-CN.md`](./CLI.zh-CN.md)
- 查看公开 C API 参考：[`API.zh-CN.md`](./API.zh-CN.md)

## 概述
udpxy 是一个 UDP → HTTP 的组播转发守护进程。它从指定的组播（含 SSM）订阅读取 UDP 流量，并将其转发给请求的 HTTP 客户端，同时提供少量管理类 HTTP 接口。

## 快速开始
- 构建：在 `chipmunk/` 目录执行 `make`
- 运行守护进程：`./udpxy -p 4022 -m 192.168.1.1`
- 转发一个频道：`curl http://127.0.0.1:4022/udp/224.0.2.26:1234/ > stream.ts`
- 查看状态页：在浏览器打开 `http://127.0.0.1:4022/status/`
- 录制流量：先创建软链接 `ln -s udpxy udpxrec`，然后执行 `./udpxrec -c 224.0.2.26:1234 out.mpg`