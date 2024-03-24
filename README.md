![Tianji Visitor](https://status.himiku.com/telemetry/clnzoxcy10001vy2ohi4obbi0/clu5muup80r0jmb4g3fbsx5gm/badge.svg)
# dnmp

Docker+Nginx+MySql+PHP，即「DNMP」。

此外，还包括了 acme.sh+phpMyAdmin+Watchtower，不需要可以注释掉。

因为长久以来使用 LNMP 的习惯，在转向 Docker 后，按自己的需求~~随便整~~编辑了一版 dnmp。

离不开 [Kong's Blog](https://jktu.cc/) 大佬的帮助。

~~一把梭真的很快乐！~~

## 这玩意要怎么用？

### 启动

首先，你需要安装 Docker 和 Docker compose，才能使用这个玩意。

其次，你需要对 Docker、Nginx、PHP、phpMyAdmin 相关配置有一定了解。Docker 可以参考 [Docker 从入门到实践](https://yeasy.gitbook.io/docker_practice/)。

另外，还需要学习一下使用 [acme.sh](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)。

一切准备就绪后，克隆这个仓库：
```bash
git clone https://github.com/mikusaa/dnmp.git
```
进入目录：
```bash
cd dnmp
```
由于指定使用了名为 `dnmp` 的网络，所以咱需要先创建一个 docker 网络：
```bash
docker network create -d bridge dnmp
```
复制一份配置文件：
```bash
cp docker-compose.yml.example docker-compose.yml
```
修改一下配置，就可以启动容器了：
```bash
docker compose up -d
```

### Nginx

nginx 使用的是包含了 `QUIC`、`HTTP/3`、`Google's brotli compression` 的版本，非官方版。具体可以参考[官方文档](https://hub.docker.com/r/macbre/nginx-http3)。

这里我预创建了一个 `ssl.conf` 配置文件，里面包含了基础通用的 SSL 配置，包括启用 `http2` 以及 `quic`，但还是需要手动创建俩文件，一个是 `dhparam.pem`，一个是 `ticket.key` 。

进入 `ssl` 目录：

```bash
cd dnmp/nginx/ssl
```
使用 openssl 创建这两个文件：
```bash
openssl dhparam -out dhparam.pem 2048
openssl rand 80 > ticket.key
```

这样你就可以在配置域名的时候，直接使用 `ssl` 目录下的 `ssl.conf` 和 `quic.conf` 配置 HTTPS 部分了。

```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    listen 443 quic ;
    listen [::]:443 quic ;
    server_name www.himiku.com;
    ssl_certificate_key ssl/himiku.com/himiku.com.key;
    ssl_certificate ssl/himiku.com/fullchain.cer;
    include ssl/ssl.conf;
    include ssl/quic.conf; #鉴于国内网络对quic的支持极差，这一段配置单独列出
    #其他省略
}
```
至于 `nginx.conf` 配置文件，这个容器添加了 [headers-more-nginx-module](https://github.com/openresty/headers-more-nginx-module#readme) 第三方模块用于清除 `headers` 避免遭到攻击什么的。但实际使用过程中发现，原 `nginx.conf` 配置文件中默认添加的

```nginx
more_set_headers "Content-Security-Policy: object-src 'none'; frame-ancestors 'self'; form-action 'self'; block-all-mixed-content; sandbox allow-forms allow-same-origin allow-scripts allow-popups allow-downloads; base-uri 'self';";
```

会导致 Typecho 无法正常使用。故禁用了部分配置。