# udpxy Documentation

- See CLI usage and examples: [CLI.md](./CLI.md)
- See public C API reference: [API.md](./API.md)

## Overview
udpxy is a UDP-to-HTTP multicast relay daemon. It forwards UDP multicast (or SSM) traffic to HTTP clients and provides a small set of administrative HTTP endpoints.

## Quickstart
- Build: `make` in `chipmunk/`
- Run daemon: `./udpxy -p 4022 -m 192.168.1.1`
- Relay a channel: `curl http://127.0.0.1:4022/udp/224.0.2.26:1234/ > stream.ts`
- Show status page: open `http://127.0.0.1:4022/status/` in a browser
- Record traffic: create a symlink `ln -s udpxy udpxrec`, then run `./udpxrec -c 224.0.2.26:1234 out.mpg`