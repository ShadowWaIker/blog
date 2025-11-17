---
title: 飞牛 OS 配置 certbot 实现无公网 IP 环境自动续签并更新 SSL 证书
author: ShadoWalker
type: post
date: 2023-06-19T19:17:00+08:00
tags:
  - 笔记
---

#### 1. 开启 ssh 功能

![开启 ssh 功能](/static/images/2025/11/fn/ScreenShot_2025-11-17_104815_393.png "开启 ssh 功能")

#### 2. 使用 SSH 连接到飞牛 NAS

```bash
ssh 飞牛用户名@飞牛IP
```
密码为飞牛用户名对应的密码

#### 3. 安装 certbot 和需要的插件

```bash
sudo apt install certbot python3-certbot python3-certbot-dns-cloudflare
```

#### 4. 导入自动替换证书的 post-hook 脚本

```bash
sudo vim /etc/letsencrypt/renewal/域名.post.sh
```

写入以下内容

```bash
echo "证书已更新，开始替换系统证书 ..."

echo "备份原文件 ..."
mv /usr/trim/var/trim_connect/ssls/域名/随机数/域名.crt /usr/trim/var/trim_connect/ssls/域名/随机数/域名.crt-
mv /usr/trim/var/trim_connect/ssls/域名/随机数/域名.key /usr/trim/var/trim_connect/ssls/域名/随机数/域名.key-
mv /usr/trim/var/trim_connect/ssls/域名/随机数/fullchain.crt /usr/trim/var/trim_connect/ssls/域名/随机数/fullchain.crt-

echo "替换证书 ..."
cp /etc/letsencrypt/live/fn.rebeta.cn/cert.pem /usr/trim/var/trim_connect/ssls/域名/随机数/域名.crt
cp /etc/letsencrypt/live/fn.rebeta.cn/privkey.pem /usr/trim/var/trim_connect/ssls/域名/随机数/域名.key
cp /etc/letsencrypt/live/fn.rebeta.cn/fullchain.pem /usr/trim/var/trim_connect/ssls/域名/随机数/fullchain.crt

echo "修改证书权限 ..."
chmod 755 /usr/trim/var/trim_connect/ssls/域名/随机数/域名.crt
chmod 755 /usr/trim/var/trim_connect/ssls/域名/随机数/域名.key
chmod 755 /usr/trim/var/trim_connect/ssls/域名/随机数/fullchain.crt

# 获取新证书的到期日期并更新数据库中的证书有效期
NEW_EXPIRY_DATE=$(openssl x509 -enddate -noout -in "/usr/trim/var/trim_connect/ssls/域名/随机数/域名.crt" | sed "s/^.*=\(.*\)$/\1/")
NEW_EXPIRY_TIMESTAMP=$(date -d "$NEW_EXPIRY_DATE" +%s%3N)  # 获取毫秒级时间戳
echo "新证书的有效期到: $NEW_EXPIRY_DATE"

# 更新数据库中的证书有效期
echo "更新数据库中的证书有效期 ..."
psql -U postgres -d trim_connect -c "UPDATE cert SET valid_to=$NEW_EXPIRY_TIMESTAMP WHERE domain='fn.rebeta.cn'"

# 重启服务
echo "正在重启相关服务 ..."
systemctl restart webdav.service
systemctl restart smbftpd.service
systemctl restart trim_nginx.service
echo "服务重启完成！"
```

#### 5. 使用 certbot 自动申请证书并配置自动续期

```bash
certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /cloudflare配置文件路径/cloudflare.ini \
  --post-hook /etc/letsencrypt/renewal/域名.post.sh \
  -d 域名
```

#### 5. 下载证书
生成的证书位于 __/etc/letsencrypt/live/域名__ ，使用 ftp 工具或 scp 命令下载证书

#### 6. 在飞牛中添加证书

![在飞牛中添加证书](/static/images/2025/11/fn/ScreenShot_2025-11-17_115203_373.png "在飞牛中添加证书")

#### 7. 在终端获取证书存放位置。

```bash
cat /usr/trim/etc/network_cert_all.conf
```
certificate 字段中 /usr/trim/var/trim_connect/ssls/域名/随机数/域名.crt 这串路径中的 __/usr/trim/var/trim_connect/ssls/域名/随机数/__ 就是证书存放位置。

#### 8. 修改自动更新证书脚本

```bash
sudo vim /etc/letsencrypt/renewal/域名.post.sh
```

将其中的证书路径替换为步骤 7 中获取的证书存放路径。

#### 8. 测试是否可以自动更新证书

```bash
sudo certbot renew --force-renewal
```

执行完毕后刷新浏览器，查看证书的到期时间是否有变化。
