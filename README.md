# 🔐 acme.sh

一个纯 Shell 编写的 ACME 协议客户端，用于自动化申请和续期 SSL 证书。

[![Shellcheck](https://github.com/acmesh-official/acme.sh/workflows/Shellcheck/badge.svg)](https://github.com/acmesh-official/acme.sh/actions/workflows/Shellcheck.yml)

---

## ✨ 特性

- 🐚 纯 Shell 实现，无需 Python 依赖
- 🔑 支持 ECDSA、RSA、通配符证书
- ⚡ 自动签发、自动续期
- 👤 不需要 root 权限
- 🌍 支持 IPv6、Docker
- 📦 支持 100+ DNS API 自动验证

📚 [官方 Wiki](https://github.com/acmesh-official/acme.sh/wiki)

---

## 🚀 一行安装

```bash
curl -s https://gitee.com/aaalbr/acme.sh/raw/master/acme.sh | sh -s email=你的邮箱@example.com
```

或使用 wget：

```bash
wget -O - https://gitee.com/aaalbr/acme.sh/raw/master/acme.sh | sh -s email=你的邮箱@example.com
```

安装后关闭并重新打开终端，或执行 `source ~/.bashrc`。

---

## 📖 签发证书

### 方式一：Webroot 模式

适用于已有 web 服务器，需要知道网站根目录路径。

```bash
acme.sh --issue -d example.com -w /home/wwwroot/example.com
```

多个域名同证书：

```bash
acme.sh --issue -d example.com -d www.example.com -d api.example.com -w /home/wwwroot/example.com
```

### 方式二：Standalone 模式

适用于无 web 服务器，需要 80 端口空闲。

```bash
acme.sh --issue --standalone -d example.com
```

### 方式三：DNS 手动模式（适合 NAT VPS）

适用于无公网 IP、无法开放 80/443 端口的环境。

```bash
# 第一步：获取 TXT 记录值
acme.sh --issue --dns -d example.com --yes-I-know-dns-manual-mode-enough-go-ahead-please
```

输出示例：
```
Add the following TXT record:
Domain: _acme-challenge.example.com
Txt value: 9ihDbjYfTExAYeDs4DBUeuTo18KBzwvTEjUnSwd32-c
```

```bash
# 第二步：去域名 DNS 后台添加 TXT 记录，等待 3-5 分钟生效

# 第三步：完成签发
acme.sh --renew -d example.com --yes-I-know-dns-manual-mode-enough-go-ahead-please
```

### 方式四：DNS API 模式（推荐）

自动添加和删除 TXT 记录，支持自动续期。

```bash
# Cloudflare 示例
export CF_Token="your-cloudflare-api-token"
acme.sh --issue --dns dns_cf -d example.com -d '*.example.com'
```

```bash
# 阿里云示例
export Ali_Key="your-access-key-id"
export Ali_Secret="your-access-key-secret"
acme.sh --issue --dns dns_ali -d example.com
```

```bash
# DNSPod（腾讯云）示例
export Tencent_SecretId="your-secret-id"
export Tencent_SecretKey="your-secret-key"
acme.sh --issue --dns dns_tencent -d example.com
```

📚 [所有支持的 DNS API](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)

---

## 📦 安装证书

### 安装到 Nginx

```bash
acme.sh --install-cert -d example.com \
  --key-file       /etc/nginx/ssl/example.com.key \
  --fullchain-file /etc/nginx/ssl/example.com.cer \
  --reloadcmd      "nginx -s reload"
```

### 安装到 Apache

```bash
acme.sh --install-cert -d example.com \
  --cert-file      /etc/apache2/ssl/example.com.cert \
  --key-file       /etc/apache2/ssl/example.com.key \
  --fullchain-file /etc/apache2/ssl/example.com.cer \
  --reloadcmd      "systemctl reload apache2"
```

---

## ⚙️ 常用命令

| 命令 | 说明 |
| :--- | :--- |
| `acme.sh --list` | 列出所有已签发的证书 |
| `acme.sh --info -d example.com` | 查看证书详细信息 |
| `acme.sh --renew -d example.com` | 手动续期证书 |
| `acme.sh --renew -d example.com --force` | 强制续期（忽略有效期） |
| `acme.sh --remove -d example.com` | 停止证书续期并删除 |
| `acme.sh --upgrade` | 升级 acme.sh 到最新版 |
| `acme.sh --upgrade --auto-upgrade` | 启用自动升级 |
| `acme.sh --version` | 查看版本 |
| `acme.sh -h` | 查看帮助 |

---

## 🔧 续期机制

- 证书默认每 **30 天** 自动续期一次
- 安装时自动设置 cron 任务
- 续期后自动执行 `--reloadcmd` 重载 web 服务
- 查看 cron：`crontab -l | grep acme`

手动测试续期：

```bash
acme.sh --renew -d example.com --force --debug
```

---

## 🐳 Docker 使用

```bash
docker run --rm -it \
  -v "$(pwd)/out":/acme.sh \
  -e CF_Token="your-token" \
  neilpang/acme.sh --issue --dns dns_cf -d example.com
```

---

## ❓ 常见问题

### Q: 安装时提示 `curl: (35) OpenSSL SSL_connect: Connection reset by peer`

国内访问 GitHub raw 被干扰，使用本 Gitee 镜像的一行命令安装。

### Q: 签发时提示 `pending` 或超时

DNS 记录还未生效，等待 3-5 分钟后重试 `acme.sh --renew -d example.com`。

### Q: 证书续期后网站仍显示旧证书

检查 web 服务是否成功重载，确认 `--reloadcmd` 命令正确。

### Q: 如何查看证书到期时间？

```bash
acme.sh --info -d example.com | grep -E "Expire|Le_NextRenewTime"
```

### Q: DNS API 模式需要每次更新 API 密钥吗？

不需要，acme.sh 会保存密钥配置，后续自动使用。

---

## 📄 许可证

GPLv3

---

## 🔗 链接

- Gitee 镜像：https://gitee.com/aaalbr/acme.sh
- GitHub 上游：https://github.com/acmesh-official/acme.sh
- 官方 Wiki：https://github.com/acmesh-official/acme.sh/wiki
