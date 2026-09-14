# 领克 APP 自动签到 · 青龙面板自用版

> ⚠️ **来源与授权声明（请先读）**
>
> 本项目 **fork 自 [spritekite/lynk_auto_sign](https://github.com/spritekite/lynk_auto_sign-20260709)（青龙面板专用版 v2）**。
> 原作者采用 **「MIT 源码 + 单独 License 运行锁」双授权**：代码可自由修改，但脚本运行需向作者申请 License（捐赠后 RSA 签发、机器指纹绑定、有过期日、带反篡改检测）。
>
> **本仓库已移除 License 校验逻辑**，目的仅是**个人学习 / 自用**，让自己能在青龙面板每天自动签到。
> - ✅ **尊重原作者**：本仓库未移除、也未隐藏任何签到 / 分享 / 通知的业务逻辑，仅删掉了「运行门槛」这一段。原作者持续维护的网关凭据与接口适配（这部分是开放代码）你仍可正常使用。
> - ❌ **请勿倒卖、请勿在公开渠道分发去锁版本**——这会直接损害原作者持续维护的动力，也违背其明确声明。
> - 💡 若长期使用，建议**支持原作者**；或改用同为 MIT、无运行锁的 [zcc0077/lynkco-daily](https://github.com/zcc0077/lynkco-daily) 作为更干净的替代方案。

## 功能一览
- 领克 APP 每日自动签到（连续 7 / 30 / 85 / 365 天进度、成长等级与成长值查询）
- 签到任务进度查询
- H5 分享链接生成（手动发微信给好友点击，主账号 +5 能量体）；可选自动分享（需配置 B 账号）
- 多通道 Markdown 推送：优先复用青龙面板「通知」；也可另配企业微信 / 钉钉 / 飞书 / Telegram / Server 酱 / PushPlus / Bark
- `accessToken` 本地缓存（与 App 行为一致，避免每次强制 refresh）

## 2026-09 签到 403 修复说明
领克已将真正执行签到的接口从 `/up/api/v1/user/sign` 改为 `/up/api/v1/user/sign/upgrade`，
并要求 **App 原生签名**（含 `Content-MD5`）。旧 H5 `AppKey` 调用签到会返回
`403 Unauthorized Consumer`（查询能量体/连续天数等接口仍可用 H5 签名）。

本脚本已内置原生 `AppKey` / `AppSecret`（与 App 内常量一致，非个人隐私）；一般无需额外配置。
若日后再次失效，可用环境变量覆盖：

| 变量 | 说明 |
|---|---|
| `LYNK_NATIVE_APP_KEY` | 原生签名 AppKey |
| `LYNK_NATIVE_APP_SECRET` | 原生签名 AppSecret |

今日是否已签改查 `/up/api/v1/user/sign/day/info`（`signStatus=1` 表示已签）。

### 网络 / DNS 失败（`Failed to resolve` / `NameResolutionError`）
若日志出现无法解析 `app-services.lynkco.com.cn` 或 `app-api-gw-toc.lynkco.com`，属于**青龙容器/宿主机出网或 DNS 问题**，不是 token 失效。可尝试：
1. 在容器内执行 `nslookup app-api-gw-toc.lynkco.com` / `ping` 确认能否解析
2. 为 Docker/青龙配置可靠 DNS（如 `223.5.5.5`、`8.8.8.8`）并重启容器
3. 确认未拦截上述域名的代理/防火墙

脚本对 DNS/连接抖动会自动重试（可用 `LYNK_HTTP_RETRIES` / `LYNK_HTTP_TIMEOUT` 调整）。

## 依赖
- Python 3.8+
- `requests`（脚本检测到缺失时会自动 `pip install requests`）

> 原脚本的 `cryptography` 依赖**仅用于 License 验签**，去锁后已不再需要，可放心不装。

## 使用说明（青龙面板）

### 1. 部署脚本
青龙 → **脚本管理** → 新建脚本，粘贴 `ql_lynk.py` 全文并保存；或上传文件到容器内 `/ql/scripts/`。

### 2. 安装依赖
青龙 → **依赖管理** → 添加 Python 依赖 `requests`（或进容器 `pip install requests`）。

### 3. 配置凭证
两种方式，任选其一：
- **A. 改脚本**：编辑文件顶部 `USER_CONFIG` 块，填 `USER_REFRESH_TOKEN` 与 `USER_DEVICE_ID`。
- **B. 环境变量（推荐）**：在青龙「环境变量」中添加：

| 变量 | 说明 | 必填 |
|---|---|---|
| `LYNK_REFRESH_TOKEN` | 主账号 refreshToken（`bearer<uuid>`，约 28 天有效） | ✅ |
| `LYNK_DEVICE_ID` | 设备 ID（与登录设备一致，避免被挤下线） | ✅ |
| `LYNK_TOKEN_B` | B 账号 refreshToken（逗号分隔，启用自动分享时填） | 可选 |
| `LYNK_SHARE_CONTENT_ID` | 分享文章 ID（默认热门 ID） | 可选 |
| `LYNK_AUTO_SHARE` | `1`/`true` 启用自动分享（默认 false，仅生成 URL） | 可选 |
| `LYNK_NATIVE_APP_KEY` / `LYNK_NATIVE_APP_SECRET` | 原生签到签名密钥（一般无需改，脚本已内置） | 可选 |
| `LYNK_APP_VERSION` / `LYNK_DEVICE_TYPE` | App 版本 / 设备类型（默认内置） | 可选 |
## 推送通知

### 方式 A：脚本顶部 USER_CONFIG（推荐，不用环境变量）

编辑 `ql_lynk.py` 顶部：

```python
USER_USE_QL_NOTIFY = False          # 不想走青龙内置通知时改 False
USER_PUSH_BARK_URL = "https://api.day.app/你的Key/"   # 或只填设备码
# 也可填企业微信 / 钉钉 / 飞书 / Telegram / Server酱 / PushPlus
```

### 方式 B：青龙「系统设置 → 通知」

脚本默认会调用青龙 `notify.send`。注意 Python 通知变量名是 **`BARK_PUSH`**（不是本脚本的 `PUSH_BARK_URL`）。  
若日志出现 `无推送渠道`，说明 `notify.py` 没读到渠道变量——改用方式 A 更省事。

日志 `青龙通知: OK (Bark)` 表示走了面板内置推送；`Bark: OK` 表示脚本直推 Bark。

关闭青龙内置通知：`USER_USE_QL_NOTIFY = False`，或环境变量 `LYNK_USE_QL_NOTIFY=0`。

> 说明：青龙 `notify.py` 默认会请求「一言」`v1.hitokoto.cn`，该站 SSL/出网失败会导致整次推送失败。本脚本默认关闭一言（等价于 `HITOKOTO=false`）；若仍要一言，设 `LYNK_HITOKOTO=1`。

### 方式 C：环境变量 `PUSH_*`（可选）

| 变量 | 说明 | 必填 |
|---|---|---|
| `PUSH_WECOM_WEBHOOK` | 企业微信机器人 webhook | 可选 |
| `PUSH_DINGTALK_WEBHOOK` | 钉钉机器人 webhook | 可选 |
| `PUSH_FEISHU_WEBHOOK` | 飞书机器人 webhook | 可选 |
| `PUSH_TG_BOT_TOKEN` / `PUSH_TG_CHAT_ID` | Telegram | 可选 |
| `PUSH_SERVERCHAN_KEY` | Server 酱 SendKey | 可选 |
| `PUSH_PUSHPLUS_TOKEN` | PushPlus Token | 可选 |
| `PUSH_BARK_URL` | Bark 推送 URL | 可选 |
| `LYNK_USE_QL_NOTIFY` | `0`/`false` 关闭青龙内置通知（默认启用） | 可选 |

**如何抓取 `refreshToken` 与 `deviceId`**：
1. 手机抓包（Charles / Fiddler / 小黄鸟等），过滤域名 `app-services.lynkco.com.cn`。
2. 找到 `mobileCodeLogin`（验证码登录）或 `login/refresh` 的响应 / 请求。
3. `refreshToken` 形如 `bearer` + 一串 UUID，从响应里复制完整值。
4. `deviceId` 在登录请求 URL 的 `deviceId` 参数里，复制相同值。
> refreshToken 约 28 天有效，过期需重新抓包；青龙环境下脚本会自动回写更新后的 token。

### 4. 定时任务
青龙 → **定时任务** → 新建：
- 命令：`python3 /ql/scripts/ql_lynk.py`
- 定时：`0 9 * * *`（每天 9 点，避开 0 点风控）

### 5. 命令行直接运行
```bash
python3 ql_lynk.py
```
输出为 Markdown，可对接青龙自带通知或上方各推送渠道。

## 免责声明
本项目仅供**个人学习与技术研究**使用，使用者须遵守领克 APP 用户协议及相关法律法规。因使用本脚本导致的账号风险由使用者自行承担。请尊重原作者知识产权，**勿用于商业倒卖或公开分发去锁版本**。
