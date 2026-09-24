---
name: trim-cli
description: Sign in to TRIM NAS, manage files, photos and Baidu Netdisk, inspect applications, Docker and storage, and validate operations on a NAS device. Excludes Media APIs.
---

# trim-cli

## CLI 准备

本 Skill 对应 npm 包 `@trimjs/trim-cli@0.1.1`，需要 Node.js 18+。
首次使用前执行 `trim-cli --version`；未安装或版本不匹配时，可安装匹配版本：

```bash
npm install -g @trimjs/trim-cli@0.1.1
```

也可将下文命令的 `trim-cli` 替换为 `npx -y @trimjs/trim-cli@0.1.1`，无需全局安装。
安装 Skill 不会自动安装 CLI，也不会登录 NAS。OAuth 登录需要保持交互式终端。

TRIM NAS / fnOS 的命令行客户端。通过 HTTP API proxy 访问系统业务接口和原生应用 API，
覆盖认证、文件、相册、百度网盘、应用中心、Docker、下载、日志、监控、网络、电源、存储和用户管理。

## 任务路由

先选最具体的入口，再按入口中的 reference 或 workflow 执行：

| 任务 | 入口 |
| --- | --- |
| 登录、session、连接失败、profile | `entries/trim-shared.md`、`reference/_index.md` |
| 文件、finder、共享目录、ACL、上传下载 | `entries/trim-file.md`、`reference/workflows/file-routing.md` |
| 相册搜索、AI 搜图、详情、预览 | `entries/trim-photos.md`、`reference/workflows/photos-routing.md` |
| 百度网盘授权状态、文件、分享链接保存、传输任务、同步任务 | `entries/trim-baidu-netdisk.md`、`reference/baidu-netdisk.md` |
| 应用中心、Docker、下载、日志、监控 | 对应 `entries/` 文件和 `reference/_index.md` |
| 存储写操作 | `entries/trim-storage.md`、`reference/workflows/storage-dangerous-ops.md` |
| 用户、用户组、权限 | `entries/trim-user.md`、`reference/user.md` |
| 真机验证 | `reference/workflows/device-validation.md` |

## 安全规则

- 影视/Media API 当前不属于本 skill；不要探测或调用 CLI 中可能存在的 `media` 命令。
- 登录唯一使用 OAuth 2.0 Authorization Code + PKCE；不要索取、猜测或生成 NAS 用户名密码、
  2FA、旧 token 或手工 access-token 登录命令。
- 固定 OAuth `client_id=YJNMPJUGA9`，按模块申请当前 CLI 所需的 `trim.*` scopes；完整列表见
  `reference/oauth.md`。授权 URL 不含 `redirect_uri`。
- 生产环境只使用交互式 `login`：CLI 生成授权链接并保持等待，用户自行在浏览器登录、确认授权、
  复制一次性 code，再粘贴到 CLI 的 `Authorization code` 提示中。Agent 不代填账号密码、不自动
  点击授权、不读取页面 code，也不把 code 放进命令参数。
- session 保存 access/refresh token、过期时间和 OAuth device 信息；敏感值
  只进入平台安全存储或显式选择的文件后端，不写入普通输出或错误日志。
- access token 临期会自动刷新；`/ogh/ac/w` 或 `/ogh/ac/h/*` 收到 HTTP 401 时会 refresh 并只重试
  一次。连续 401 或 refresh 失败时要求重新 OAuth 登录，不循环、不回退旧登录流程。
- refresh 前会确认当前 profile 的 session 存储可写，并发刷新会复用已写入的较新 token。
- 保存的 session 缺少当前模块 scope（包括旧 `file photo` 授权）时必须重新登录；refresh 不能扩展
  已授权 scope。
- `logout` 清理当前 profile 的 session 和 OAuth pending 文件；不声称服务端令牌已撤销。
- 远程明文 HTTP 必须显式 `--allow-insecure-http`；自签 HTTPS 使用 `--tls-insecure`。
- Agent 或非交互流程执行写命令时一律显式传 `--yes`；高风险存储命令还需要必要的密码预校验。
- 百度账号授权只能在 NAS 的百度网盘应用登录页完成；CLI 只检查状态，不生成授权链接、不接收百度账号密码或 OAuth code。
- 不把 refresh token 或授权 code 发送给无关第三方。

## 连接选项

```text
--host <host>                 NAS 主机，默认 localhost
--port <port>                 端口，显式传入时优先
--profile <name>              独立 session profile
--scheme auto|http|https      传输协议，默认 auto
--allow-insecure-http         允许远程明文 HTTP
--tls-insecure                允许 HTTPS 自签证书
```

未显式传连接参数时，CLI 会复用当前 profile 保存的 host、port、scheme 和 TLS 设置。默认
loopback 为 `http://localhost:5666`；远程 IP 默认 HTTPS 5667，域名默认 HTTPS 443。

## 最小认证流程

```bash
trim-cli --profile home --host <host> --port <port> --scheme https login
trim-cli --profile home login --refresh
trim-cli --profile home logout
```

`login` 会输出授权链接并尝试打开浏览器，然后在当前终端等待 `Authorization code`。用户必须
自行完成浏览器登录和授权，再把页面显示的一次性 code 粘贴回该提示。无法自动打开浏览器时
使用 `login --no-open`，授权步骤仍完全相同。

远程明文设备示例：

```bash
trim-cli --profile home --host <host> --port 5666 --scheme http --allow-insecure-http login --no-open
```

## 常用只读命令

```bash
trim-cli --profile home system info
trim-cli --profile home file ls /vol1
trim-cli --profile home file ls-dir /vol1
trim-cli --profile home file usage 1
trim-cli --profile home photos folders
trim-cli --profile home photos search <keyword> --limit 20
trim-cli --profile home baidu-netdisk status
trim-cli --profile home baidu-netdisk ls /
trim-cli --profile home app list
trim-cli --profile home docker image ls
trim-cli --profile home download ls
trim-cli --profile home monitor cpu
trim-cli --profile home storage pools
trim-cli --profile home user info
```

## 原始请求

系统业务缺少命名命令时使用 `raw`；Photos 原生 HTTP API 使用 `photos request`。`raw` 只有
内置、已审计的只读端点可以省略 `--yes`，其他端点（包括未知端点）一律先确认并显式传
`--yes`。命令仍会校验路径和请求体。
原始请求结果必须同时检查 HTTP 状态和业务响应 code/终态，不能只看进程退出码。

## 输出约定

命名命令默认输出 JSON 计划或业务数据。Photos 搜索的 `total` 才是匹配总数；AI 搜图只有在
响应提供稳定总数时才能报告精确数量，否则只能报告本次返回条数。预览 URL 通常仍需要
已登录浏览器 cookie。

`file ls-dir` 汇总所有目录分段为 JSON 数组；`file usage` 汇总容量分类为 JSON 数组，同一分类身份取最后一次结果而不是累加。

## 进一步阅读

- 总索引：`reference/_index.md`
- OAuth 合约：`reference/oauth.md`
- Photos API：`reference/photos.md`
- 百度网盘：`entries/trim-baidu-netdisk.md`、`reference/baidu-netdisk.md`
- 连接与真机：`entries/trim-shared.md`、`reference/workflows/device-validation.md`
