# frp

[![Build Status](https://circleci.com/gh/fatedier/frp.svg?style=shield)](https://circleci.com/gh/fatedier/frp)
[![GitHub release](https://img.shields.io/github/tag/fatedier/frp.svg?label=release)](https://github.com/fatedier/frp/releases)
[![Go Report Card](https://goreportcard.com/badge/github.com/fatedier/frp)](https://goreportcard.com/report/github.com/fatedier/frp)
[![GitHub Releases Stats](https://img.shields.io/github/downloads/fatedier/frp/total.svg?logo=github)](https://somsubhra.github.io/github-release-stats/?username=fatedier&repository=frp)

[README](README.md) | [中文文档](README_zh.md)

frp 是一个专注于内网穿透的高性能的反向代理应用，支持 TCP、UDP、HTTP、HTTPS 等多种协议，且支持 P2P 通信。可以将内网服务以安全、便捷的方式通过具有公网 IP 节点的中转暴露到公网。


## 为什么使用 frp ？

通过在具有公网 IP 的节点上部署 frp 服务端，可以轻松地将内网服务穿透到公网，同时提供诸多专业的功能特性，这包括：

* 客户端服务端通信支持 TCP、QUIC、KCP 以及 Websocket 等多种协议。
* 采用 TCP 连接流式复用，在单个连接间承载更多请求，节省连接建立时间，降低请求延迟。
* 代理组间的负载均衡。
* 端口复用，多个服务通过同一个服务端端口暴露。
* 支持 P2P 通信，流量不经过服务器中转，充分利用带宽资源。
* 多个原生支持的客户端插件（静态文件查看，HTTPS/HTTP 协议转换，HTTP、SOCK5 代理等），便于独立使用 frp 客户端完成某些工作。
* 高度扩展性的服务端插件系统，易于结合自身需求进行功能扩展。
* 服务端和客户端 UI 页面。

## 如何使用



### 服务端配置

**下载frp库并解压安装**

```
wget https://github.com/lamfire/frp/releases/download/0.58.0/frp_0.58.0_linux_amd64.tar.gz
tar -zcvf frp_0.58.0_linux_amd64.tar.gz
cd frp_0.58.0_linux_amd64
cp frps /usr/local/bin

```

快速启动服务端

```
frps -p 6000 --dashboard-port 6600 --vhost-http-port 6080 --vhost-https-port 6443
```

说明：服务端口=6000，管理后台端口=6600，http代理端口=6080



**验证服务端是否启动成功**

```
打开网站，登录名和密码均为‘admin’,能进入则服务端启动成功
http://[your-host]:6600/
```

#### 通过配置文件

```sh
vim /etc/frps.toml

bindPort = 6000
vhostHTTPPort = 6080
vhostHTTPSPort = 6443
auth.token = "[your-token]"
webServer.addr = "0.0.0.0"
webServer.port = 6600
webServer.user = "admin"
webServer.password = "admin"


#run
frps -c /etc/frps.toml &
```

#### 安装服务

```bash
cat >> /etc/systemd/system/frps.service <<EOF
[Unit]
Description=FRPS service
After=network.target
 
[Service]
Type=simple
ExecStart=/usr/local/bin/frps -c /etc/frps.toml
Restart=always

[Install]
WantedBy=multi-user.target
EOF

#启动服务
systemctl daemon-reload
systemctl enable frps
systemctl start frps
```



### 客户端配置

#### **下载frp库并解压安装**

```
wget https://github.com/lamfire/frp/releases/download/0.58.0/frp_0.58.0_linux_amd64.tar.gz
tar -zcvf frp_0.58.0_linux_amd64.tar.gz
cd frp_0.58.0_linux_amd64
cp frpc /usr/local/bin
cp frpc.toml /etc/
```

#### 配置代理

```
vim /etc/frpc.toml

serverAddr = "[your-host]"
serverPort = 6000
auth.token = "[your-token]"

[[proxies]]
name = "web-demo1"
type = "http"
customDomains = ["sh.xxx.com"]
localIP = "192.168.1.214"
localPort = 8080
```

以上配置了1个http代理，访问本地网络中192.168.1.214的8080端口服务



#### 启动客户端

```
/usr/local/bin/frpc -c /etc/frpc.toml &
```



#### 配置域名解析hosts

```
101.24.135.180  sh.xxx.com
```



#### 访问代理

```
http://sh.xxx.com:6080
```



#### 开机启动

```bash
cat >> /etc/systemd/system/frpc.service <<EOF
[Unit]
Description=FRPC service
After=network.target
 
[Service]
Type=simple
ExecStart=/usr/local/bin/frpc -c /etc/frpc.toml
Restart=always

[Install]
WantedBy=multi-user.target
EOF

#启动服务
systemctl daemon-reload
systemctl enable frpc
systemctl start frpc
```



![zsxq](/doc/pic/zsxq.jpg)
