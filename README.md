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
- 多通道 Markdown 推送：企业微信 / 钉钉 / 飞书 / Telegram / Server 酱 / PushPlus / Bark
- `accessToken` 本地缓存（与 App 行为一致，避免每次强制 refresh）

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
| `LYNK_APP_VERSION` / `LYNK_DEVICE_TYPE` | App 版本 / 设备类型（默认内置） | 可选 |
| `PUSH_WECOM_WEBHOOK` | 企业微信机器人 webhook | 可选 |
| `PUSH_DINGTALK_WEBHOOK` | 钉钉机器人 webhook | 可选 |
| `PUSH_FEISHU_WEBHOOK` | 飞书机器人 webhook | 可选 |
| `PUSH_TG_BOT_TOKEN` / `PUSH_TG_CHAT_ID` | Telegram | 可选 |
| `PUSH_SERVERCHAN_KEY` | Server 酱 SendKey | 可选 |
| `PUSH_PUSHPLUS_TOKEN` | PushPlus Token | 可选 |
| `PUSH_BARK_URL` | Bark 推送 URL | 可选 |

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
