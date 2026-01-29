ql-docker.py 可以单下载直接上传到你armv8机子上 运行（我是n1盒子刷了软路由成功运行）这个自动利用F2a  arm32位没弄了手上没有32位的机子了 玩客云

ql-docker-plus.py  这个能看到款额

青龙面板使用

注意：如果是docker容器创建的青龙，请使用whyour/qinglong:debian镜像，latest（alpine）版本可能无法安装部分依赖
# ClawCloud 自动保活脚本 (Selenium ARM64 版)

专为 **青龙面板 (Docker ARM64/AMD64)** 环境设计的 ClawCloud 自动登录保活脚本。

## ✨ 核心功能

*   **多账号支持**：支持配置无限个账号，顺序执行保活。
*   **Cookie 复用**：优先使用本地缓存 Cookie，减少登录频率，降低风控风险。
*   **自动 2FA 验证**：
    *   **强制密钥模式**：支持 TOTP (Authenticator) 两步验证，需配置 `totp_secret` 密钥，实现全自动无人值守登录。
    *   注：为保证稳定性，已移除交互式验证功能。
*   **多渠道通知**：
    *   **Telegram**: 支持发送文本汇总和异常截图。
    *   **微信**: 支持自定义 API (如 wxpush) 推送通知。
*   **智能代理 (国内环境优化)**：
    *   自动检测 `CLAW_PROXY` 或 `HTTP_PROXY` 环境变量。
    *   检测到代理时，自动配置浏览器和网络请求走代理，并自动处理 Docker 容器内的 `localhost` 连接问题，确保国内网络下稳定运行。
*   **环境适配**：针对 ARM64 (树莓派/M1) 和 AMD64 环境下的 Chromium/Chrome 路径进行了自动适配。

## 🛠️ 环境要求

*   **环境**: 青龙面板 (推荐 v2.10+)
*   **依赖**:
    *   Python 3.8+
    *   Chromium / Chrome 浏览器
    *   ChromeDriver
*   **Python 库** (青龙面板依赖管理 -> Python 中添加):
    ```logo
    selenium
    requests
    loguru
    pyotp
    ```
*   **系统包** (青龙面板依赖管理 -> Linux 中添加):
    ```bash
    chromium
    chromium-chromedriver
    ```
    *(注：部分镜像可能直接集成了 chrome，视具体环境而定)*

### 💻 SSH 手动安装依赖 (推荐)

如果您熟悉 SSH，可以直接进入容器安装，速度更快且更稳定。

1.  **进入青龙容器**
    ```bash
    docker exec -it qinglong bash
    # 注意: 'qinglong' 是您的容器名，如果不确定请使用 docker ps 查看
    ```

2.  **安装 Linux 系统依赖 (Alpine 环境)**
    ```bash
    apk add chromium chromium-chromedriver
    ```
    *(如果是 Debian/Ubuntu 环境，请使用 `apt-get install chromium-driver`)*

3.  **安装 Python 依赖**
    ```bash
    pip3 install selenium requests loguru pyotp
    ```

## ⚙️ 环境变量配置

请在青龙面板的「环境变量」中添加以下配置：

### 1. 账号配置 (必须)

| 变量名 | 描述 | 格式 |
| :--- | :--- | :--- |
| `CLAW_ACCOUNTS` | 账号列表 | `账号----密码----2FA密钥` |

*   **多账号**：用 `&` 符号连接。
*   **备注支持**：支持在账号后加 `#备注`，脚本会自动忽略。

**示例**：
```bash
# 单账号
user@gmail.com----password123----JBSWY3DPEHPK3PXP

# 多账号 (带备注)
user1@gmail.com#主号----pass1----SECRET1&user2@qq.com#小号----pass2----SECRET2
```

### 2. 代理配置 (国内用户推荐)

| 变量名 | 描述 | 示例 |
| :--- | :--- | :--- |
| `CLAW_PROXY` | 代理地址 | `http://192.168.1.5:7890` |
| `HTTP_PROXY` | 系统代理 (备选) | `http://192.168.1.5:7890` |

*   **作用**：启用后，浏览器登录 GitHub/ClawCloud 以及发送 Telegram 消息都会走此代理。
*   **无需配置**：如果你是国外 VPS，可不填。

### 3. 通知配置 (可选)

| 变量名 | 描述 | 说明 |
| :--- | :--- | :--- |
| `TG_BOT_TOKEN` | Telegram Bot Token | 机器人 Token |
| `TG_CHAT_ID` | Telegram Chat ID | 接收消息的用户 ID |
| `WECHAT_API_URL` | 微信推送 API | 自定义 GET/POST 接口地址 |
| `WECHAT_AUTH_TOKEN` | 微信推送 Token | 接口鉴权 Token |

## 🚀 运行说明

1.  将脚本 `clawcloud_arm64.py` 添加到青龙面板的脚本库或直接上传。
2.  添加定时任务：
    *   命令：`task clawcloud_arm64.py`
    *   定时：`0 10 * * *` (建议每天运行一次，避开高峰期)
3.  点击运行日志，查看执行情况。

## 📂 文件结构

*   `clawcloud_arm64.py`: 主脚本文件
*   `cookies_xxx.json`: 脚本自动生成的 Cookie 缓存文件 (自动生成，无需管理)
*   `*.png`: 运行过程中生成的临时截图 (脚本运行结束会自动清理)

## ⚠️ 常见问题

1.  **报错 `Network unreachable`**
    *   请检查是否配置了 `CLAW_PROXY`。国内网络直连 Google/GitHub/Telegram 通常不通。
2.  **报错 `WebDriverException: Session not created`**
    *   通常是 Chrome 和 ChromeDriver 版本不匹配，或未安装 Chromium。请检查青龙面板的 Linux 依赖是否安装了 `chromium` 和 `chromium-chromedriver`。
3.  **2FA 登录失败**
    *   请确保 `totp_secret` 是正确的 Base32 密钥字符串（通常是添加 Authenticator 时显示的密钥）。不要填 6 位动态码。


点击 确定
<img width="527" height="447" alt="image" src="https://github.com/user-attachments/assets/b429dc14-8097-4a3e-aa14-0fe1f365cbc7" />

---

# ☁️ ClawCloud Auto-Login / 自动保活V1.1版

此工作流旨在实现 **每 15 天自动登录一次 ClawCloud (爪云)** 以保持账号活跃。

为了确保自动化脚本顺利运行，**必须**满足以下两个前置条件：

1. ❌ **关闭 Passkey (通行密钥)**：避免脚本无法处理生物识别弹窗。
2. 
3.  ✅ **开启 2FA (双重验证)**：配合脚本中的 PyOTP 自动生成验证码，绕过异地登录风控。
4.  `增加TG接收消息。`
---

## 🛠️ 配置步骤

### 第一步：Fork 本项目
点击页面右上角的 **Fork** 按钮，将此仓库复制到您的 GitHub 账号下。

### 第二步：开启 GitHub 2FA 并获取密钥
脚本需要通过 2FA 密钥自动计算验证码，因此不能只扫二维码，必须获取**文本密钥**。

1. 登录 GitHub，点击右上角头像 -> **Settings**。
2. 在左侧菜单选择 **Password and authentication**。
3. 找到 "Two-factor authentication" 区域，点击 **Enable 2FA**。
4. 选择 **Set up using an app**。
5. **⚠️ 关键步骤**：
   > 页面显示二维码时，**不要直接点击 Continue**。请点击二维码下方的蓝色小字 **"Setup Key"**（或 "enter this text code"）。
6. **复制显示的字符串密钥**（通常是 16 位字母数字组合）。
   * *注意：同时也请用手机验证器 App (如 Google Auth) 扫描二维码或输入密钥，以完成 GitHub 的验证流程。*

7. **⚠️ 记得把Preferred 2FA method选为Authenticator App，否则脚本不生效**
### 第三步：配置 GitHub Secrets
为了保护您的账号安全，请将敏感信息存储在仓库的 Secrets 中。

1. 进入您的 GitHub 仓库页面。
2. 依次点击导航栏的 **Settings** -> 左侧栏 **Secrets and variables** -> **Actions**。
3. 点击右上角的 **New repository secret** 按钮。
4. 依次添加以下 3 个 Secret：

| Secret 名称 | 填入内容 (Value) | 说明 |
| :--- | :--- | :--- |
| `GH_USERNAME` | **您的 GitHub 账号** | 通常是您的登录邮箱 |
| `GH_PASSWORD` | **您的 GitHub 密码** | 登录用的密码 |
| `GH_2FA_SECRET` | **2FA 密钥** | 第二步中复制的那串字符 (请去除空格) |
| `TG_BOT_TOKEN` | **机器人的token**| 
| `TG_CHAT_ID` | **机器人的id**| 
### 第四步：启用工作流权限 (⚠️ 重要)
由于是 Fork 的仓库，GitHub 默认可能会禁用 Actions 以防止滥用。

1. 点击仓库顶部的 **Actions** 选项卡。
2. 如果看到警告提示，请点击绿色的 **"I understand my workflows, go ahead and enable them"** 按钮。

### 第五步：手动测试运行
配置完成后，建议手动触发一次以确保一切正常。

1. 点击 **Actions** 选项卡。
2. 在左侧列表中选择 **ClawCloud Run Auto Login**。
3. 点击右侧的 **Run workflow** 下拉菜单 -> 点击绿色 **Run workflow** 按钮。
4. 等待运行完成，查看日志确保显示 `🎉 登录成功`。

---
**✅ 完成！之后脚本将每隔 15 天自动执行一次保活任务。**
# 项目名

<!-- 徽章区 -->
[![Stars](https://img.shields.io/github/stars/djkyc/clawcloud-auto-login)](https://github.com/djkyc/clawcloud-auto-login)

<!-- Star History -->
## ⭐ Star History
[![Star History Chart](https://api.star-history.com/svg?repos=djkyc/clawcloud-auto-login&type=Date)](https://star-history.com/#djkyc/clawcloud-auto-login&Date)


