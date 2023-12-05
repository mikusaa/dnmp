## 自用 dnmp

Docker+Nginx+MySql+PHP，即「DNMP」。

此外，还包括了 acme.sh+phpMyAdmin+Watchtower，不需要可以注释掉。

因为长久以来使用 LNMP 的习惯，在转向 Docker 后，按自己的需求~~随便整~~编辑了一版 dnmp。

离不开 [Kong's Blog](https://jktu.cc/) 大佬的帮助。

~~一把梭真的很快乐！~~

## 这玩意要怎么用？

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
docker network create dnmp
```
复制一份配置文件：
```bash
cp docker-compose.yml.example docker-compose.yml
```
修改一下配置，就可以启动容器了：
```bash
docker compose up -d
```
如果机器很好的话，等待个把分钟编译完 PHP，容器就能正常启动并使用了。
