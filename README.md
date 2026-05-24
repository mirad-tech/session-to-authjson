# Session → Auth.json 转换器

![纯前端](https://img.shields.io/badge/纯前端-无服务器-blue) ![安全](https://img.shields.io/badge/安全-不上传数据-green)

将 ChatGPT 网页端的 Session JSON 转换为 OpenAI Codex CLI 所需的 `auth.json` 格式，实现免手机验证启动 Codex。

> **纯前端本地转换，不上传任何数据，不存储任何 token。**

## 原理

```
ChatGPT 网页端                              Codex CLI
┌────────────────┐                    ┌──────────────────┐
│ 邮箱 + 密码登录 │                    │ 读取 auth.json    │
│       ↓        │                    │       ↓          │
│   手机号验证 ✓  │                    │  token 有效？     │
│       ↓        │    ── 格式转换 ──→  │  ├ 是 → 直接使用  │
│ 获得 session   │                    │  └ 否 → 触发登录  │
│ (accessToken)  │                    │    (含手机验证)   │
└────────────────┘                    └──────────────────┘
```

**核心逻辑：**

- Codex CLI 启动时会检查本地 `auth.json`，如果其中包含有效的登录状态，就**直接复用**，不会进入登录流程
- ChatGPT 网页端登录成功后，浏览器已经持有通过手机验证的 `accessToken`
- 本工具只是把网页端的 token **从 Session 格式转换为 auth.json 格式**
- **没有破解、没有绕过验证** — 只是将同一个合法 token 从 A 格式转为 B 格式

## 使用方法

### 第一步：获取 ChatGPT Session JSON

1. 在浏览器中登录 [chatgpt.com](https://chatgpt.com)（正常完成所有验证流程）
2. 登录成功后，在同一浏览器中访问：

```
https://chatgpt.com/api/auth/session
```

3. 页面会显示一段 JSON，形如：

```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6Ikp...",
  "account": {
    "id": "user-xxxxxxxxxxxxxxxxxxxxxxxx",
    "email": "your@email.com",
    ...
  },
  ...
}
```

4. 全选复制这段 JSON

### 第二步：转换

1. 用浏览器打开 `session-to-authjson.html`
2. 将复制的 JSON 粘贴到左侧「源 JSON」输入框
3. 点击「转换」按钮（或按 `Ctrl + Enter`）
4. 右侧会显示转换后的 `auth.json` 内容：

```json
{
  "last_refresh": "2026-05-25T00:00:00.000000Z",
  "tokens": {
    "access_token": "eyJhbG...",
    "account_id": "user-xxx...",
    "id_token": "eyJhbG...",
    "refresh_token": "eyJhbG..."
  }
}
```

### 第三步：部署到 Codex CLI

1. 点击右上角的「下载」按钮，保存 `auth.json` 文件
2. 将 `auth.json` 放到 Codex CLI 的配置目录：

| 操作系统 | 路径 |
|---------|------|
| Windows | `%APPDATA%\codex\auth.json` |
| macOS   | `~/.config/codex/auth.json` |
| Linux   | `~/.config/codex/auth.json` |

3. 启动 Codex CLI，它会读取 `auth.json` 中的 token，不再触发登录流程

## 功能特性

| 功能 | 说明 |
|------|------|
| 粘贴转换 | 从剪贴板粘贴 Session JSON，一键转换 |
| 打开文件 | 支持直接打开 `.json` 文件 |
| 复制结果 | 一键复制转换后的 auth.json |
| 下载文件 | 直接下载为 `auth.json` 文件 |
| 压缩模式 | 勾选后输出单行压缩 JSON |
| 快捷键   | `Ctrl + Enter` 快速转换 |
| 纯前端   | 无服务器、无网络请求、无数据存储 |

## 转换映射

本工具从 Session JSON 中提取以下字段：

```
Session JSON                    auth.json
─────────────────               ─────────────────
accessToken          →          tokens.access_token
                     →          tokens.id_token
                     →          tokens.refresh_token
account.id           →          tokens.account_id
(自动生成)            →          last_refresh
```

## 安全说明

> [!IMPORTANT]
> `accessToken` 是你的 OpenAI 账号登录凭证，等同于密码。

- ✅ 本工具**完全在浏览器本地运行**，不发送任何网络请求
- ✅ 不使用 `localStorage`、`sessionStorage`、`cookie` 存储任何数据
- ✅ 关闭页面后所有数据清零
- ✅ 可以断网使用，直接双击 HTML 文件打开即可
- ⚠️ 生成的 `auth.json` 包含敏感 token，请**妥善保管**
- ⚠️ 不要将 `auth.json` 提交到 Git 或分享给他人
- ⚠️ token 有时效性，过期后需要重新从网页端获取

## 常见问题

### Q: 这是在破解或绕过 OpenAI 的验证吗？

**不是。** 你仍然需要在 ChatGPT 网页端正常完成所有验证（包括手机验证）。本工具只是把已经验证通过的 token 转换格式，让 Codex CLI 能直接使用。

### Q: 转换后提示缺少字段怎么办？

确保你访问的是 `https://chatgpt.com/api/auth/session`，并且已经登录。返回的 JSON 必须包含 `accessToken` 和 `account.id` 两个字段。

### Q: token 多久过期？

ChatGPT 的 `accessToken` 通常有效期较短（几小时到几天不等）。过期后需要重新从网页端获取并再次转换。

### Q: 支持哪些浏览器？

任何现代浏览器均可：Chrome、Edge、Firefox、Safari。无需安装任何依赖。
