<div align="center"><h1>自建机场</h1></div>

## 一. 购买服务器

### 2.1 常规VPS

对于常规 VPS，主要是一个云服务提供商，大的原厂商都提供免费使用和赠金，所以，可以白嫖一段时间。

- [AWS LightSail](https://lightsail.aws.amazon.com/) 是一个非常便宜好用的服务，最低配置一个月 $3.5 美金，流量不限，目前的Zone不多，推荐使用日本，新加坡或美国俄勒冈（支持银联卡）。现对 2021/8/7 之后使用 LightSail 的用户提供3个月的免费试用。
- [Microsoft Azure](https://azure.microsoft.com/zh-cn/) 提供免费一年的服务（B1S实例），而且每个月有 100G 的免费流量，并赠送200刀赠金。（需要国际信用卡）
- [Google Cloud Platform](https://cloud.google.com/) 提供免费试用，赠送300刀赠金（需要国际信用卡）
- [Oracle Cloud](https://www.oracle.com/cloud/free/) 两台VPS无限期使用，可选美日韩等地（需要国际信用卡）
- [RackNerd](https://my.racknerd.com/aff.php?aff=1134) 价格在一年 10 刀-20 刀（如下所示）。支持支付宝、银联。购买的时候，有个 Location 选项，可以选择机房位置，后面有相应的 IP，你可以测试一下 ping 值，选择最低的那个。
- [AWS EC2](https://aws.amazon.com/cn/) 香港、日本或韩国申请个免费试用一年的EC2 VPS （支持银联卡）
- [Linode](https://www.linode.com) 买个一月USD5刀的VPS
- [Vultr](https://www.vultr.com?ref=7609494) 上买一个日本的VPS，一个月5刀 - 6刀 （可以支付宝）
---
## 二. 申请域名

[免费域名](https://blog.csdn.net/weixin_46378344/article/details/129825320) 缺点审核慢约 1天～1个月，不需要实名，建议使用

[腾讯域名](https://console.dnspod.cn/dns/list) 收费最低7块，速度快 2～4 小时审核完成，缺点需要实名，可以暂时使用

---
## 三. Docker服务

首先，你要安装一个Docker CE 服务，这里你要去看一下docker官方的安装文档：

- [CentOS 上的 Docker CE 安装](https://docs.docker.com/install/linux/docker-ce/centos/)
- [Ubuntu 上的 Docker CE 安装](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- [菜鸟教程 Docker CE 安装](https://www.runoob.com/docker/ubuntu-docker-install.html) 建议参考教程非常详细

---
## 四. 申请证书

### 1. 安装 certbot
```shell
$ sudo apt install certbot -y
```
### 2. 安装 certbot
```shell
$ sudo certbot certonly --standalone
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator standalone, Installer None
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): 输入你的邮箱

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.3-September-21-2022.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: A

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: N
Please enter in your domain name(s) (comma and/or space separated)  (Enter 'c'
to cancel): 输入你的域名
```
---
## 五. 用 Gost 设置 HTTPS 服务

[gost](https://github.com/ginuerzh/gost) 是一个非常强的代理服务，它可以设置成 HTTPS 代理，然后把你的服务伪装成一个Web服务器，**我感觉这比其它的流量伪装更好，也更隐蔽。这也是这里强烈推荐的一个方式**。

### 1.创建 gost.sh Docker 自动搭建部署本
```shell
$ vi gost.sh
```
### 2. gost.sh 文件内容如下
- 这个脚本，你需要配置：域名(`DOMAIN`), 用户名 (`USER`), 密码 (`PASS`) 和 端口号(`PORT`) 这几个变量。
- 关于 gost 的参数， 你可以参看其文档：[Gost Wiki](https://docs.ginuerzh.xyz/gost/)，我设置一个参数 `probe_resist=code:404` 意思是，如果服务器被探测，或是用浏览器来访问，返回404错误，也可以返回一个网页（如：`probe_resist=file:/path/to/file.txt` 或其它网站 `probe_resist=web:example.com/page.html`）
```shell
#!/bin/bash
# gost.sh

# 下面的四个参数需要改成你的
DOMAIN="YOU.DOMAIN.NAME"
USER="username"
PASS="password"
PORT=443

BIND_IP=0.0.0.0
CERT_DIR=/etc/letsencrypt
CERT=${CERT_DIR}/live/${DOMAIN}/fullchain.pem
KEY=${CERT_DIR}/live/${DOMAIN}/privkey.pem
sudo docker run -d --name gost \
    -v ${CERT_DIR}:${CERT_DIR}:ro \
    --net=host ginuerzh/gost \
    -L "http2://${USER}:${PASS}@${BIND_IP}:${PORT}?cert=${CERT}&key=${KEY}&probe_resist=code:404&knock=www.google.com"
```
## 六. 使用服务端 gost 代理

---
### 1. PC 端全平台
- [根据自己操作系统下载](https://github.com/ginuerzh/gost/releases)
- 把下载的
- 进入文件所在目录打开终端，代理如下
```shell
# linux
# 需要在点击 系统设置 -> 网络 -> 网络代理 -> 手动
# HTTP proxy 设置为 127.0.0.1 端口 7890
# HTTPS proxy 设置为 127.0.0.1 端口 7890
# -----------------------------------------------------------------------------
# 你需要配置：域名(`YOU.DOMAIN.NAME`), 用户名 (`USER`), 密码 (`PASS`) 这几个变量。
$ sudo chmod +x gost-linux-amd64
$ ./gost-linux-amd64 -L=:7890 -F=http2://USER:YOU.DOMAIN.NAME:443
```
### 2. 手机 
- iPad IOS （推荐小火箭）
  - ShadowRocket：打开app，点击右上角`+`号添加服务器，配置如下几项即可（其余条目采用默认值即可）
    - Type: `HTTPS`
    - Address: `$DOMAIN`
    - Port: `443`
    - User: `$USER`
    - Password: `$PASS`
- Android
  - [Clash for Android](https://github.com/Kr328/ClashForAndroid)
  - Shadowsocks + ShadowsocksGostPlugin
    - 安装Shadowsocks (Google Play Store或[github页面](https://github.com/shadowsocks/shadowsocks-android))
    - 下载[ShadowsocksGostPlugin](https://github.com/segfault-bilibili/ShadowsocksGostPlugin)
    - 配置
      - Server: `$DOMAIN`
      - Remote Port: `443`
      - Password: `gost`
      - Encrypt Method: `RC4-MD5`
      - Plugin: 选择ShadowsocksGostPlugin
      - Configure: `-F https://$USER:$PASS@#SS_HOST:#SS_PORT`

      其中`$DOMAIN`, `$USER`, `$PASS`分别为服务端的域名/子域名, 用户名和密码。

---

- 参考 [左耳朵](https://github.com/haoel/haoel.github.io )