# Godlike 自动续期脚本

# ⭐ 觉得有用？给个 Star 支持一下！
> 注册地址：[https://godlike.host/cart-free](https://godlike.host/cart-free)  
> ⚠️忘记密码？在此重置：[https://panel.godlike.host/auth/password](https://panel.godlike.host/auth/password)

自动为 **Godlike Host** 免费 Minecraft 服务器续期 90 分钟的 GitHub Actions 脚本。支持多账号、Cookie / 密码自动切换、Cookie 自动回写、多种代理协议，并通过 Telegram 发送带截图的成功/失败通知。

## ✨ 功能特性

- ✅ 多账号支持（最多 5 个）
- ✅ 智能登录：优先使用持久化 Cookie，失效自动回退至 Playwright 模拟登录
- ✅ Cookie 自动维护：密码登录成功后自动将 Cookie 加密回写至 GitHub Secrets，下次可直接复用
- ✅ 续期保护：检测服务器是否已过期，过期则跳过续期并通知
- ✅ 支持多种代理协议（VLESS / VMess / Trojan / Shadowsocks / SOCKS5）
- ✅ Telegram 通知带截图（成功 / 失败均有完整页面截图）
- ✅ 工作流日志脱敏，TG 通知显示真实账号/服务器信息
- ✅ 错误快照自动上传至 Artifacts 便于调试


## 📋 前置要求

### 1. GitHub Secrets 配置

在仓库 `Settings` → `Secrets and variables` → `Actions` 中添加以下 Secrets：

| Secret 名称 | 必填 | 说明 | 示例 |
|------------|:---:|------|------|
| `GODLIKE_1` | ✅ | 主账号，格式见下方 | `admin@example.com-----MyPassword` |
| `GODLIKE_2` ~ `GODLIKE_5` | ❌ | 额外账号（有多个账号时填写） | 同上 |
| `TG_BOT_TOKEN` | ❌ | Telegram Bot Token | `1234567890:AAE...` |
| `TG_CHAT_ID` | ❌ | 接收通知的 Chat ID | `123456789` |
| `REPO_TOKEN` | ❌ | 用于自动回写 Cookie 的 GitHub PAT | `ghp_xxxxxxxxxxxx` |
| `PROXY_NODE` | ❌ | 代理节点链接 | 见下方代理格式 |

### 2. GODLIKE 账号格式

每个 Secret 的格式为：`用户名/邮箱-----密码`

```
admin@example.com-----MyP@ssw0rd
myusername-----AnotherPassword
```

首次运行时只需提供用户名和密码。脚本会在登录成功后自动将 Cookie 追加到末尾，变为：
```
admin@example.com-----MyP@ssw0rd-----base64编码的cookie列表
```

后续运行将优先使用 Cookie 登录，减少浏览器开销。

### 3. 代理节点格式（可选）

支持以下代理协议，留空则使用直连：

| 协议 | 示例 |
|------|------|
| VLESS | `vless://uuid@host:port?type=ws&security=tls&sni=example.com` |
| VMess | `vmess://eyJhZGQiOiIxLjIuMy40IiwidiI6IjIiLCJwc...` |
| Trojan | `trojan://password@host:port?type=ws&sni=example.com` |
| Shadowsocks | `ss://YWVzLTI1Ni1nY206cGFzc3dvcmQ=@host:port` |
| SOCKS5 | `socks5://user:pass@host:port` 或 `socks5://host:port` |

### 4. Telegram 通知配置（可选）

1. 向 [@BotFather](https://t.me/BotFather) 发送 `/newbot` 创建 Bot，获取 Token。
2. 向 [@userinfobot](https://t.me/userinfobot) 发送任意消息，获取你的 Chat ID。
3. 将 Token 和 Chat ID 填入上述 Secrets。


## 🚀 使用方法

### 方法 1：定时自动运行

Fork 仓库并配置 Secrets 后，工作流默认每 3 小时运行一次（UTC 时间），自动完成续期。

如果需要修改频率，编辑 `.github/workflows/godlike.yml`：

```yaml
schedule:
  - cron: '0 */3 * * *'      # 每3小时
```

常用 cron 表达式：
- `0 */3 * * *` — 每 3 小时

### 方法 2：手动触发（GitHub 网页）

1. 进入仓库的 `Actions` 页面
2. 选择 `Godlike 续期` 工作流
3. 点击 `Run workflow`
4. 点击绿色的 `Run workflow` 按钮

### 方法 3：API 调用（可用于面板/定时任务）

```bash
curl -X POST \
  -H "Authorization: Bearer ghp_xxxxxxxxxxxxxxxxxxxx" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/你的用户名/你的仓库名/actions/workflows/Godlike_Renew.yml/dispatches \
  -d '{"ref":"main"}'
```

一行版：
```bash
curl -s -X POST "https://api.github.com/repos/你的用户名/你的仓库名/actions/workflows/Godlike_Renew.yml/dispatches" -H "Accept: application/vnd.github+json" -H "Authorization: Bearer ghp_xxxxxxxxxxxxxxxxxxxx" -d '{"ref":"main"}'
```


## 🐛 常见问题

### 1. 登录失败

**原因**：
- 账号密码错误（忘记密码可[在此重置](https://panel.godlike.host/auth/password)）
- “Through login/password” 切换按钮未出现（页面结构已变）
- 网络/代理问题

**解决**：
- 检查 `GODLIKE_X` Secret 格式是否正确
- 查看 Actions 日志中的截图（在 Artifacts 中下载）
- 尝试配置代理（`PROXY_NODE`）
- 登录页可能会更新，若持续失败可提交 Issue

### 2. 续期失败

**原因**：
- 服务器已过期（无法续期）
- 广告按钮未加载（页面改版或代理问题）
- 续期冷却期内

**解决**：
- 检查截图确认具体原因
- 如果冷却，脚本会在下次运行时自动重试
- 更新脚本中的按钮选择器（参考 Issue 讨论）

### 3. Telegram 通知未收到

**原因**：
- Bot Token 或 Chat ID 错误
- 你未向 Bot 发送过消息（需先在 Telegram 中向 Bot 发送 `/start`）
- 消息被 Telegram 限制

**解决**：
- 在 Telegram 中向 Bot 发送 `/start`
- 验证 Secrets 配置是否正确
- 检查 Actions 日志中的错误信息

### 4. Cookie 回写失败

**原因**：
- 缺少 `REPO_TOKEN` 或 Token 权限不足
- `REPO_TOKEN` 未勾选 `workflow` 和 `repo` 权限

**解决**：
- 创建具有 `repo` 和 `workflow` 权限的 GitHub Personal Access Token (classic) 并填入 `REPO_TOKEN`
- 若不关心回写，可忽略该警告，脚本仍正常运行

### 5. 截图在哪里查看？

- 截图仅在失败时自动上传（需在工作流中配置 `if: failure()`）。
- 在 Actions 运行完成后，向下滚动至 `Artifacts` 区域，下载 `error-screenshots` 压缩包即可查看所有截图。


## 🔒 安全建议

1. ✅ 使用 **GitHub Secrets** 存储所有敏感信息
2. ✅ `REPO_TOKEN` 仅授予 `repo` 和 `workflow` 权限
3. ✅ 日志中账号/服务器均自动脱敏，TG 通知为私人可见
4. ✅ 定期更换密码并更新 Secret


## 📄 许可证

MIT License

---

**⚠️ 免责声明**：本脚本仅供学习交流使用，使用者需遵守 Godlike Host 的服务条款。因使用本脚本造成的任何问题，作者不承担任何责任。
