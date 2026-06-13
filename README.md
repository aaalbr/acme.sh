# 🔐 acme.sh

一个纯 Shell 编写的 ACME 协议客户端，用于自动化申请和续期 SSL 证书。

[![Shellcheck](https://github.com/acmesh-official/acme.sh/workflows/Shellcheck/badge.svg)](https://github.com/acmesh-official/acme.sh/actions/workflows/Shellcheck.yml)

---

## ✨ 特性

- 🐚 纯 Shell 实现，无 Python 依赖
- 🔑 支持 ECDSA、RSA、通配符证书
- ⚡ 自动签发、自动续期
- 👤 不需要 root 权限
- 🌍 支持 IPv6、Docker

📚 [官方 Wiki](https://github.com/acmesh-official/acme.sh/wiki)

---

## 🚀 安装

### 方式一：从 Gitee 安装（国内推荐）

```bash
git clone https://gitee.com/aaalbr/acme.sh.git
cd acme.sh
./acme.sh --install -m your-email@example.com
```

### 方式二：从官方源安装

```bash
curl https://get.acme.sh | sh -s email=your-email@example.com
# 或
wget -O - https://get.acme.sh | sh -s email=your-email@example.com
```

安装完成后，关闭并重新打开终端，或执行 `source ~/.bashrc` 使命令生效。

---

## 📖 使用指南

### 1. Webroot 模式签发（适用于有 web 服务器）

```bash
acme.sh --issue -d example.com -w /home/wwwroot/example.com
```

### 2. Standalone 模式签发（需要 80 端口空闲）

```bash
acme.sh --issue --standalone -d example.com
```

### 3. DNS 手动模式签发（适用于 NAT 环境、无公网 IP）

```bash
# 第一步：获取 TXT 记录值
acme.sh --issue --dns -d example.com --yes-I-know-dns-manual-mode-enough-go-ahead-please
```

输出示例：
```
Add the following TXT record:
Domain: _acme-challenge.example.com
TXT value: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

```bash
# 第二步：去域名 DNS 后台添加 TXT 记录，等待生效

# 第三步：完成签发
acme.sh --renew -d example.com --yes-I-know-dns-manual-mode-enough-go-ahead-please
```

> ⚠️ DNS 手动模式无法自动续期，续期时需要重新添加 TXT 记录。建议使用 DNS API 模式。

### 4. DNS API 模式签发（自动续期，推荐）

```bash
# 以 Cloudflare 为例
export CF_Token="your-cloudflare-api-token"
acme.sh --issue --dns dns_cf -d example.com -d '*.example.com'
```

📚 [支持的 DNS API 列表](https://github.com/acmesh-official/acme.sh/wiki/dnsapi)

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
| `acme.sh --list` | 列出所有证书 |
| `acme.sh --info -d example.com` | 查看证书信息 |
| `acme.sh --renew -d example.com` | 手动续期证书 |
| `acme.sh --renew -d example.com --force` | 强制续期 |
| `acme.sh --remove -d example.com` | 停止证书续期 |
| `acme.sh --upgrade` | 升级 acme.sh |
| `acme.sh --upgrade --auto-upgrade` | 启用自动升级 |

---

## 🔧 续期机制

- 证书默认每 **30 天** 自动续期一次
- 通过 cron 任务自动运行（安装时自动设置）
- 续期后会自动执行 `--reloadcmd` 重载 Web 服务

---

## 🐳 Docker 使用

```bash
docker run --rm -it \
  -v "$(pwd)/out":/acme.sh \
  -e CF_Token="your-token" \
  neilpang/acme.sh --issue --dns dns_cf -d example.com
```

📚 [Docker 指南](https://github.com/acmesh-official/acme.sh/wiki/Run-acme.sh-in-docker)

---

## 📌 常见问题

### 1. 安装时提示 `curl: (35) OpenSSL SSL_connect: Connection reset by peer`

国内服务器访问 GitHub raw 被干扰，请使用 Gitee 镜像安装。

### 2. 签发证书时提示 `pending` 或超时

DNS 记录未生效，等待几分钟后重试 `acme.sh --renew -d example.com`。

### 3. 证书续期后网站仍显示旧证书

检查 Web 服务（Nginx/Apache）是否重载了配置，`--reloadcmd` 是否正确。

---

## 📄 许可证

GPLv3

---

**项目地址**：
- Gitee 镜像：https://gitee.com/aaalbr/acme.sh
- GitHub 上游：https://github.com/acmesh-official/acme.sh
