# macOS-Anti-Censorship-Shield

如果不开放，就毁灭它---一个用于在 macOS 15+ / macOS 27+ (Golden Gate) 系统中彻底切断、阻断国行特供版 Apple Intelligence 资格审查（`eligibilityd`）以及国内主流大模型/云服务（阿里通义千问、DeepSeek、百度文心一言、腾讯混元、字节豆包）底层网络连接与行为追踪的硬核隐私加固指南。

**[Warning]我已经做出了升级版本无需解锁任何限制且不会失效，如果苹果继续无底线侵犯用户使用自由，我将不得不公开教程废除苹果资格审查**

If Apple refuses transparency and user choice, the community will continue to study, challenge, and ultimately dismantle these restrictions - A hardcore privacy-hardening guide for macOS 15+ / macOS 27+ (Golden Gate), designed to completely sever and block Apple Intelligence eligibility enforcement (eligibilityd), while preventing network communications, telemetry collection, behavioral tracking, and cloud interactions associated with privacy-invasive AI and cloud service providers, including Alibaba Qwen, DeepSeek, Baidu ERNIE, Tencent Hunyuan, and ByteDance Doubao.


**[Warning]**

I have already developed a more robust understanding of the Apple Intelligence eligibility enforcement mechanism. Users in China deserve the same transparency, autonomy, and control over their devices as users anywhere else in the world.

Transparency builds trust. Restrictions invite resistance.

---

## 背景说明

在测试过程中，我尝试调整系统安全配置（CSR/SIP）时遇到如下错误：

"Failed to update security configuration for 'Macintosh HD': Manifest verification failed"

查阅社区讨论后发现，部分 Apple Silicon（M 系列）设备用户即使降低安全级别，仍然无法完成相关配置修改，最终只能尝试通过 DFU 模式连接另一台正常运行的 Mac 进行完整恢复。该现象似乎并非个例。

值得注意的是，Apple 近期也在 iPhone 上引入了更便捷的恢复机制（例如通过硬件按键进入恢复流程）。因此我推测，Apple 可能有意让 Beta 测试用户更多地接触和验证系统恢复、刷机及设备救援流程。不过这只是个人猜测，并无直接证据支持。

目前 Apple Intelligence 相关组件已经出现在系统 CLI 工具中（例如 fm --help 可见相关功能入口），但实际调用时仍会受到资格审查机制限制。

## Background

During testing, I encountered the following error while attempting to modify system security settings (CSR/SIP):

"Failed to update security configuration for 'Macintosh HD': Manifest verification failed"

After reviewing community reports, I found that some Apple Silicon (M-series) users experience the same issue. In certain cases, lowering the security level does not resolve the problem, and the only available recovery path could be trying is restoring the device through DFU mode using another functioning Mac. This does not appear to be an isolated incident.

It is also noteworthy that Apple has recently introduced more accessible recovery workflows on iPhone devices. As a result, I speculate that Apple may be encouraging Beta users to exercise and validate recovery, restore, and device rescue procedures more frequently. This is only a personal hypothesis and is not supported by direct evidence.

At present, Apple Intelligence-related components are already visible within system CLI tools (for example, references can be observed via "fm --help"), yet actual functionality remains restricted by eligibility enforcement mechanisms.

---

## ⚠️ 重要警告：小版本更新失效说明 (OTA Updates Warning)

在进行系统升级或小版本更新（例如从 **macOS 27.0** 升级到 **macOS 27.1**）时，系统底层的重构机制会对本项目的防线产生不同的影响。请务必仔细阅读以下说明：

## ⚠️ Important Warning: Minor Version Update Impact (OTA Updates Warning)

When performing system upgrades or minor updates (for example, from **macOS 27.0** to **macOS 27.1**), low-level system refactors may affect the defenses implemented by this project in different ways. Please read the following notes carefully.

### 1. 每次系统小版本更新后【必须重新运行】的部分

* **`/etc/hosts` 拦截**：系统更新时，苹果官方会用干净的默认文件完全重写覆盖 `/etc/hosts`。升级后，所有写入的本地域名重定向会彻底失效。
* **PF 防火墙（`/etc/pf.conf`）**：`/etc` 目录属于系统级根目录，OTA 升级会重置防火墙主配置文件。你配置的底层物理 IP 网段拦截规则会被洗掉。
* **应急对策**：系统升级重启后，请务必返回本项目，重新执行【方案 B】和【方案 C】。

### 1. Parts that MUST be re-run after each minor system update

* **`/etc/hosts` interception**: During system updates, Apple may overwrite `/etc/hosts` with a clean default file. After an upgrade, any local hostname redirects you added will be completely lost.
* **PF firewall (`/etc/pf.conf`)**: The `/etc` directory is part of the system root; OTA upgrades can reset the main firewall configuration. Low-level IP subnet block rules you added will be wiped.
* **Emergency action**: After the system upgrades and restarts, return to this project and re-run Plan B and Plan C.

### 2. 系统更新后【依然保持有效】的部分

* **代理软件规则（Clash / Surge）**：由于配置文件存储在用户目录（User Space）中，完全不受系统升级影响。**这是最推荐的一劳永逸方案**。
* **系统诊断与行为追踪关闭（`defaults write`）**：这些属于用户自定义偏好设置（User Preferences），升级时系统会自动保留用户数据，无需重新运行。

### 2. Parts that remain effective after system updates

* **Proxy app rules (Clash / Surge)**: Because their configuration files live in user space, they are not affected by system upgrades. **This is the most recommended long-term solution.**
* **Disabling system diagnostics and behavior tracking (`defaults write`)**: These are user preference settings; they are preserved across upgrades and do not need to be re-applied.

---

## 🛠️ 防线构建方案（三选一）

## 🛠️ Defense Construction Plans (Choose one of three)

### 方案 A：利用代理软件（Clash / Surge）实施全域拦截【最推荐・抗升级】
将以下规则直接复制到你的 **Clash** 或 **Surge** 自定义配置文件的 `[Rule]` 或 `rules:` 最顶部。此方案在系统小版本升级后依然有效。

### Plan A: Use proxy apps (Clash / Surge) for system-wide blocking [Recommended · Upgrade-resistant]
Copy the following rules directly to the top of your custom configuration's `[Rule]` or `rules:` in **Clash** or **Surge**. This approach remains effective after minor system updates.

```text
# ======= 核心进程阻断 =======
PROCESS-NAME,eligibilityd,REJECT

# ======= DeepSeek (深度求索) =======
DOMAIN-KEYWORD,deepseek,REJECT

# ======= 阿里系 (Ali / 通义千问) =======
DOMAIN-KEYWORD,qwen,REJECT
DOMAIN-KEYWORD,alicdn,REJECT
DOMAIN-KEYWORD,aliyuncs,REJECT
DOMAIN-KEYWORD,alibaba,REJECT

# ======= 百度系 (Baidu / 文心一言) =======
DOMAIN-KEYWORD,baidu,REJECT
DOMAIN-KEYWORD,bdstatic,REJECT
DOMAIN-KEYWORD,baidubce,REJECT
DOMAIN-KEYWORD,yiyan,REJECT

# ======= 腾讯系 (Tencent / 混元) =======
DOMAIN-KEYWORD,tencent,REJECT
DOMAIN-KEYWORD,hunyuan,REJECT
DOMAIN-KEYWORD,myqcloud,REJECT
DOMAIN-KEYWORD,qq.com,REJECT

# ======= 字节跳动系 (ByteDance / 豆包) =======
DOMAIN-KEYWORD,doubao,REJECT
DOMAIN-KEYWORD,volces,REJECT
DOMAIN-KEYWORD,bytedance,REJECT

```

```text
# ======= CORE PROCESS BLOCK =======
PROCESS-NAME,eligibilityd,REJECT

# ======= DeepSeek =======
DOMAIN-KEYWORD,deepseek,REJECT

# ======= Aliyun / Qwen =======
DOMAIN-KEYWORD,qwen,REJECT
DOMAIN-KEYWORD,alicdn,REJECT
DOMAIN-KEYWORD,aliyuncs,REJECT
DOMAIN-KEYWORD,alibaba,REJECT

# ======= Baidu =======
DOMAIN-KEYWORD,baidu,REJECT
DOMAIN-KEYWORD,bdstatic,REJECT
DOMAIN-KEYWORD,baidubce,REJECT
DOMAIN-KEYWORD,yiyan,REJECT

# ======= Tencent =======
DOMAIN-KEYWORD,tencent,REJECT
DOMAIN-KEYWORD,hunyuan,REJECT
DOMAIN-KEYWORD,myqcloud,REJECT
DOMAIN-KEYWORD,qq.com,REJECT

# ======= ByteDance =======
DOMAIN-KEYWORD,doubao,REJECT
DOMAIN-KEYWORD,volces,REJECT
DOMAIN-KEYWORD,bytedance,REJECT

```

---

### 方案 B：内核级物理防御 - macOS PF 防火墙【系统升级后需重建】

直接封锁核心大模型及云厂商最底层的物理 IP 广播网段。即使它们更换全新 AI 域名，只要服务器在这些机房，流量就会在出网卡瞬间被丢弃（Drop）。

### Plan B: Kernel-level physical defense - macOS PF firewall [Requires reapplication after updates]

Directly block core IP subnets of major models and cloud vendors. Even if they change domains, as long as their servers remain in these data center networks, traffic will be dropped at the network egress.

1. 创建独立的规则配置文件：

```bash
sudo nano /etc/pf.anchors/com.privacy.blockai

```

1. Create a separate anchor rules file:

```bash
sudo nano /etc/pf.anchors/com.privacy.blockai

```

2. 粘贴以下内容并保存（`Ctrl+O` 回车，`Ctrl+X` 退出）：

```text
block drop out proto tcp to eligibility.apple.com
block drop out proto tcp to eligibilityd.apple.com

# 0. DeepSeek 专属核心机房网段封锁
block drop out proto tcp to 103.197.68.0/22
block drop out proto tcp to 45.45.148.0/22

# 1. 阿里云 & 阿里 AI (包含 DashScope)
block drop out proto tcp to 140.205.0.0/16
block drop out proto tcp to 106.11.0.0/16
block drop out proto tcp to 47.92.0.0/14
block drop out proto tcp to 8.210.0.0/16

# 2. 百度云 & 文心一言
block drop out proto tcp to 111.206.0.0/16
block drop out proto tcp to 180.76.0.0/16
block drop out proto tcp to 220.181.0.0/16

# 3. 腾讯云 & 混元
block drop out proto tcp to 119.28.0.0/16
block drop out proto tcp to 150.109.0.0/16
block drop out proto tcp to 129.211.0.0/16
block drop out proto tcp to 43.138.0.0/16

# 4. 火山引擎 & 字节豆包
block drop out proto tcp to 113.108.0.0/16
block drop out proto tcp to 222.126.0.0/16

```

2. Paste and save the following content (Ctrl+O, Enter, Ctrl+X to exit):

```text
block drop out proto tcp to eligibility.apple.com
block drop out proto tcp to eligibilityd.apple.com

# 0. DeepSeek core data center IP blocks
block drop out proto tcp to 103.197.68.0/22
block drop out proto tcp to 45.45.148.0/22

# 1. Aliyun & Ali AI (incl. DashScope)
block drop out proto tcp to 140.205.0.0/16
block drop out proto tcp to 106.11.0.0/16
block drop out proto tcp to 47.92.0.0/14
block drop out proto tcp to 8.210.0.0/16

# 2. Baidu Cloud & Wenxin
block drop out proto tcp to 111.206.0.0/16
block drop out proto tcp to 180.76.0.0/16
block drop out proto tcp to 220.181.0.0/16

# 3. Tencent Cloud & Hunyuan
block drop out proto tcp to 119.28.0.0/16
block drop out proto tcp to 150.109.0.0/16
block drop out proto tcp to 129.211.0.0/16
block drop out proto tcp to 43.138.0.0/16

# 4. Volcano Engine & ByteDance
block drop out proto tcp to 113.108.0.0/16
block drop out proto tcp to 222.126.0.0/16

```

3. 修改主配置文件 `sudo nano /etc/pf.conf`：
* 在 `rdr-anchor "com.apple/*"` 下方添加：`anchor "com.privacy.blockai"`
* 在文件最末尾添加：`load anchor "com.privacy.blockai" from "/etc/pf.anchors/com.privacy.blockai"`

3. Modify the main config `sudo nano /etc/pf.conf`:
* Add `anchor "com.privacy.blockai"` under `rdr-anchor "com.apple/*"`
* Add `load anchor "com.privacy.blockai" from "/etc/pf.anchors/com.privacy.blockai"` at the end of the file.

4. 强制启动防火墙并应用规则：

```bash
sudo pfctl -E -f /etc/pf.conf

```

4. Enable pf and apply rules:

```bash
sudo pfctl -E -f /etc/pf.conf

```

---

### 方案 C：本地流向重定向 - `/etc/hosts`【系统升级后需重建】

通过本地 Hosts 将核心 AI 接口直接拦截路由到本地零地址。

在终端中执行以下命令：

```bash
sudo bash -c 'cat >> /etc/hosts <<EOF

# [Privacy] Block All Domestic AI & Cloud Gateways
127.0.0.1 eligibility.apple.com
127.0.0.1 eligibilityd.apple.com
127.0.0.1 api.deepseek.com
127.0.0.1 deepseek.com
127.0.0.1 qwen.alicdn.com
127.0.0.1 dashscope.aliyuncs.com
127.0.0.1 api.vllm.ai
127.0.0.1 yiyan.baidu.com
127.0.0.1 bcebos.com
127.0.0.1 hunyuan.tencent.com
127.0.0.1 cloud.tencent.com
127.0.0.1 doubao.com
127.0.0.1 volces.com
EOF'

```

---

## 🔒 进阶底层隐私优化（永久生效）

以下优化命令直接写入用户偏好配置文件，**不受系统小版本升级影响，重启依然有效**。

## 🔒 Advanced low-level privacy optimizations (persistent)

The following optimization commands write directly to user preference files and are **not affected by minor system upgrades; they remain effective after reboot**.

### 1. 关闭系统诊断、分析日志与 Apple 追踪

```bash
# 禁用系统诊断与日志自动提报服务
sudo defaults write /Library/Preferences/com.apple.SubmitDiagInfo SubmitDiagInfo -bool false

# 禁用向苹果发送损毁报告与分析数据
defaults write com.apple.CrashReporter DialogType -string "none"

# 关闭系统内置广告和偏好行为追踪
defaults write com.apple.AdLib ADMonetizationEnabled -bool false

```

### 1. Disable system diagnostics, analytics and Apple tracking

```bash
# Disable system diagnostics upload
sudo defaults write /Library/Preferences/com.apple.SubmitDiagInfo SubmitDiagInfo -bool false

# Disable crash reports and analytics
defaults write com.apple.CrashReporter DialogType -string "none"

# Disable built-in ad/behavior tracking
defaults write com.apple.AdLib ADMonetizationEnabled -bool false

```

### 2. 禁止外接设备自动挂载与锁屏安全加固

```bash
# 当 Mac 锁定或睡眠时，断开与未授权硬件的通信（防物理注入）
sudo defaults write /Library/Preferences/com.apple.security.deviceenrollment SUBMIT_DIAGNOSTICS -bool false
sudo defaults write /Library/Preferences/com.apple.frameworks.diskimages skip-verify -bool false

# 强制锁屏后立刻（0秒）要求输入密码
defaults write com.apple.screensaver askForPassword -int 1
defaults write com.apple.screensaver askForPasswordDelay -int 0

```

### 2. Prevent automatic mounting of external devices & tighten lockscreen

```bash
# When Mac is locked or sleeping, disable communication with unauthorized hardware
sudo defaults write /Library/Preferences/com.apple.security.deviceenrollment SUBMIT_DIAGNOSTICS -bool false
sudo defaults write /Library/Preferences/com.apple.frameworks.diskimages skip-verify -bool false

# Require password immediately after lock (0s delay)
defaults write com.apple.screensaver askForPassword -int 1
defaults write com.apple.screensaver askForPasswordDelay -int 0

```

### 3. 防止本地共享或外接盘产生隐私痕迹 `.DS_Store`

```bash
# 禁止在网络共享盘（SMB/AFP）上自动生成 .DS_Store 文件
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool true

# 禁止在外接设备（U盘/移动硬盘）上自动生成 .DS_Store 文件
defaults write com.apple.desktopservices DSDontWriteUSBStores -bool true

```

### 3. Prevent .DS_Store artifacts on shared or external drives

```bash
# Disable .DS_Store creation on network shares
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool true

# Disable .DS_Store creation on USB/external drives
defaults write com.apple.desktopservices DSDontWriteUSBStores -bool true

```

---

## 🔍 自检与审计 (Audit)

配置完成后，建议通过以下命令检查防护状态：

1. **检查防火墙状态**：运行 `sudo pfctl -s info` 确认防火墙为 enabled。
2. **检查域名拦截**：运行 `ping api.deepseek.com` 或 `ping qwen.alicdn.com` 确认返回的 IP 为 `127.0.0.1`。
3. **确认最高安全法案**：运行 `csrutil status`，**日常使用务必确保系统完整性保护 (SIP) 为 `enabled` 状态**，以防恶意组件非法提权。

## 🔍 Self-check & Audit

After configuration, verify protection status with the following commands:

1. **Check the firewall status**: run `sudo pfctl -s info` to confirm pf is enabled.
2. **Check hostname blocking**: run `ping api.deepseek.com` or `ping qwen.alicdn.com` and confirm the returned IP is `127.0.0.1`.
3. **Confirm SIP status**: run `csrutil status`. For everyday use, ensure System Integrity Protection (SIP) is `enabled` to prevent unauthorized privilege escalation.

