```yaml
# compose.yaml
services:
  app:
    restart: unless-stopped
    image: fatedier/frps:v0.60.0
    logging:
      driver: local
    ports:
      - "20000-20002:20000-20002"
    volumes:
      - ./frps.toml:/frps.toml
    command: -c /frps.toml
```

```toml
# frps.toml
bindPort = 20000
auth.token = "xxxxx"

vhostHTTPPort = 20001
subdomainHost = "xxx.xxx.xxx"

webServer.port = 20002
webServer.addr = "0.0.0.0"
```

```ini
# frpc.service
[Unit]
Description = frp client
After = network-online.target
Wants = network-online.target

[Service]
Type = simple
Restart = always
RestartSec = 5s
ExecStart = /srv/frp/frpc -c /srv/frp/frpc.toml

[Install]
WantedBy = multi-user.target
```

```toml
# frpc.toml
serverAddr = "x.x.x.x"
serverPort = 20000
auth.token = "xxxxx"

user = "tjg"
log.level = "error"

[[proxies]]
name = "ssh"
type = "stcp"
secretKey = "xxxxx"
allowUsers = ["kita"]
localIP = "127.0.0.1"
localPort = 22

[[proxies]]
name = "web"
type = "http"
localPort = 3000
subdomain = "tjg-web"
```

```toml
# config.toml
serverAddr = "x.x.x.x"
serverPort = 20000
auth.token = "xxxxx"

user = "kita"

[[visitors]]
name = "tjg"
type = "stcp"
serverUser = "tjg"
serverName = "ssh"
secretKey = "xxxxx"
bindAddr = "127.0.0.1"
bindPort = 3011
```
