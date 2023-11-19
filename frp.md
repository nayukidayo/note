```toml
# frps.toml
bindPort = 40000
auth.token = "nayukidayo"

vhostHTTPPort = 43210
subdomainHost = "frp.nayuki.top"

webServer.addr = "0.0.0.0"
webServer.port = 43211
```

```toml
# frpc.toml
serverAddr = "remote.nayuki.top"
serverPort = 40000
auth.token = "nayukidayo"

loginFailExit = false
log.level = "error"

[[proxies]]
name = "ssh-01"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 42201

[[proxies]]
name = "web-01"
type = "http"
localPort = 80
subdomain = "web-01"
```