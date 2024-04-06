![Docker Pulls](https://img.shields.io/docker/pulls/mikusa/docker-php)
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
复制一份compose配置文件：
```bash
cp docker-compose.yml.example docker-compose.yml
```
再复制一份环境变量配置文件：
```bash
cp example.env .env
```
修改一下 `compose` 配置，再修改一下 `.env` 配置中的用户id、mysql密码，就可以启动容器了：
```bash
docker compose up -d
```

## acme.sh 

其实也很简单。启动容器后，进入 acme.sh 容器内部的终端 sh ：
```
docker exec -it acme.sh sh
```

这是一个前台化容器内部终端的指令，你可以直接在此键入命令。后需操作都默认是在 acme.sh 容器内部 操作，否则需要在前面加上 `docker exec acme.sh`。

初次申请，需要先创建 Ze­roSSL 账户，请将 `i@example.com` 替换为你自己的邮箱：
```
acme.sh --register-account -m i@example.com
```
如果网络不行连接不到 Ze­roSSL 的话（具体表现为创建账户或是签发证书的时候卡在 API 连接上），在拥有代理的前提下可以为容器配置 HTTP_PROXY 。在 environment 环境变量中增加两行：
```
environment:
  - TZ=Asia/Shanghai
  - HTTP_PROXY=http://代理的IP:端口
  - HTTPS_PROXY=http://代理的IP:端口
```

假设使用的是腾讯的 DNS­POD，在 DNSPOD 控制台创建好密钥后，导入：
```
export DP_Id="1234"
export DP_Key="sADDsdasdgdsf"
```

若是使用 Cloud­Flare 的 DNS，创建令牌后，导入 CF_Token ：
```
export CF_Token="Y_jpxxxxxxxxxx_qxxxxxxxxxxxxxxxxxxxxxxxxx"
```
如果是阿里云，前往控制台获取 RAM API key 后，将 AccessKey ID 和 AccessKey Secret 导入：
```
export Ali_Key="xxxxxxxx"
export Ali_Secret="xxxxxxxxx"
```
其他服务商的 DNS 还请自行参考官方文档，这里不再赘述。

前期工作准备完毕，开始正式申请证书。提前准备好一个用于 NAS 使用的域名，根域也好子域也罢。例如这里我使用 example.com 申请证书，dns 服务商为 cloudflare，同时申请子域名 *.example.com 的泛域名证书：
```
acme.sh --issue -d example.com -d '*.example.com' --dns dns_cf
```
参考官方文档的步骤，将证书复制到 Ng­inx 的 ssl 目录中。
```
export DEPLOY_DOCKER_CONTAINER_KEY_FILE="/etc/nginx/ssl/example.com/example.com.key"
export DEPLOY_DOCKER_CONTAINER_FULLCHAIN_FILE="/etc/nginx/ssl/example.com/fullchain.cer"

acme.sh --deploy -d example.com --deploy-hook docker
```
在 nginx/ssl 目录下有看到证书文件，就说明证书已经成功复制到了 ng­inx 内了。

如果只计划使用一个域名，那么可以在创建acme.sh 容器之初，就将证书的存放位置固定在环境变量中。

取消这两行注释：
```
       - DEPLOY_DOCKER_CONTAINER_KEY_FILE=${ACME_KEY_FILE}
       - DEPLOY_DOCKER_CONTAINER_FULLCHAIN_FILE=${ACME_FULLCHAIN_FILE}
```
再将 `.env` 环境变量中的

```
ACME_KEY_FILE=/etc/nginx/ssl/example.com/example.com.key
ACME_FULLCHAIN_FILE=/etc/nginx/ssl/example.com/fullchain.cer
```
`example.com` 批量替换为你自己的域名即可。


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

配置完成后，使用命令检测配置是否异常：
```
docker exec nginx nginx -t
```

无报错即可重载配置使之生效：

```
docker exec nginx nginx -s reload
```