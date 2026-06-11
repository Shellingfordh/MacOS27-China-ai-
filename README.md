# macOS-Anti-Censorship-Shield

一个用于在 macOS 27+ (Golden Gate) 系统中彻底切断、阻断国行特供版 Apple Intelligence 资格审查（`eligibilityd`）以及国内主流大模型/云服务（阿里通义千问、百度文心一言、腾讯混元、字节豆包）底层网络连接与行为追踪的硬核隐私加固指南。

---

## ⚠️ 重要警告：小版本更新失效说明 (OTA Updates Warning)

在进行系统升级或小版本更新（例如从 **macOS 27.0** 升级到 **macOS 27.1**）时，系统底层的重构机制会对本项目的防线产生不同的影响。请务必仔细阅读以下说明：

### 1. 每次系统小版本更新后【必须重新运行】的部分
* **`/etc/hosts` 拦截**：系统更新时，苹果官方会用干净的默认文件完全重写覆盖 `/etc/hosts`。升级后，所有写入的本地域名重定向会彻底失效。
* **PF 防火墙（`/etc/pf.conf`）**：`/etc` 目录属于系统级根目录，OTA 升级会重置防火墙主配置文件。你配置的底层物理 IP 网段拦截规则会被洗掉。
* **应急对策**：系统升级重启后，请务必返回本项目，重新执行【方案 B】和【方案 C】。

### 2. 系统更新后【依然保持有效】的部分
* **代理软件规则（Clash / Surge）**：由于配置文件存储在用户目录（User Space）中，完全不受系统升级影响。**这是最推荐的一劳永逸方案**。
* **系统诊断与行为追踪关闭（`defaults write`）**：这些属于用户自定义偏好设置（User Preferences），升级时系统会自动保留用户数据，无需重新运行。

---

## 🛠️ 防线构建方案（三选一）

### 方案 A：利用代理软件（Clash / Surge）实施全域拦截【最推荐・抗升级】
将以下规则直接复制到你的 **Clash** 或 **Surge** 自定义配置文件的 `[Rule]` 或 `rules:` 最顶部。此方案在系统小版本升级后依然有效。

```text
# ======= 核心进程阻断 =======
PROCESS-NAME,eligibilityd,REJECT

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

---

### 方案 B：内核级物理防御 - macOS PF 防火墙【系统升级后需重建】

直接封锁四大巨头最核心的物理 IP 广播网段。即使它们更换全新 AI 域名，只要服务器在这些机房，流量就会在出网卡瞬间被丢弃（Drop）。

1. 创建独立的规则配置文件：

```bash
sudo nano /etc/pf.anchors/com.privacy.blockai

```

2. 粘贴以下内容并保存（`Ctrl+O` 回车，`Ctrl+X` 退出）：

```text
block drop out proto tcp to eligibility.apple.com
block drop out proto tcp to eligibilityd.apple.com
# 阿里云 & 阿里 AI (包含 DashScope)
block drop out proto tcp to 140.205.0.0/16
block drop out proto tcp to 106.11.0.0/16
block drop out proto tcp to 47.92.0.0/14
block drop out proto tcp to 8.210.0.0/16
# 百度云 & 文心一言
block drop out proto tcp to 111.206.0.0/16
block drop out proto tcp to 180.76.0.0/16
block drop out proto tcp to 220.181.0.0/16
# 腾讯云 & 混元
block drop out proto tcp to 119.28.0.0/16
block drop out proto tcp to 150.109.0.0/16
block drop out proto tcp to 129.211.0.0/16
block drop out proto tcp to 43.138.0.0/16
# 火山引擎 & 字节豆包
block drop out proto tcp to 113.108.0.0/16
block drop out proto tcp to 222.126.0.0/16

```

3. 修改主配置文件 `sudo nano /etc/pf.conf`：
* 在 `rdr-anchor "com.apple/*"` 下方添加：`anchor "com.privacy.blockai"`
* 在文件最末尾添加：`load anchor "com.privacy.blockai" from "/etc/pf.anchors/com.privacy.blockai"`


4. 强制启动防火墙并应用规则：

```bash
sudo pfctl -E -f /etc/pf.conf

```

---

### 方案 C：本地流向重定向 - `/etc/hosts`【系统升级后需重建】

通过本地 Hosts 将核心 API 直接拦截路由到本地零地址。

在终端中执行以下命令：

```bash
sudo bash -c 'cat >> /etc/hosts <<EOF

# [Privacy] Block All Domestic AI & Cloud Gateways
127.0.0.1 eligibility.apple.com
127.0.0.1 eligibilityd.apple.com
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

### 1. 关闭系统诊断、分析日志与 Apple 追踪

```bash
# 禁用系统诊断与日志自动提报服务
sudo defaults write /Library/Preferences/com.apple.SubmitDiagInfo SubmitDiagInfo -bool false

# 禁用向苹果发送损毁报告与分析数据
defaults write com.apple.CrashReporter DialogType -string "none"

# 关闭系统内置广告和偏好行为追踪
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

### 3. 防止本地共享或外接盘产生隐私痕迹 `.DS_Store`

```bash
# 禁止在网络共享盘（SMB/AFP）上自动生成 .DS_Store 文件
defaults write com.apple.desktopservices DSDontWriteNetworkStores -bool true

# 禁止在外接设备（U盘/移动硬盘）上自动生成 .DS_Store 文件
defaults write com.apple.desktopservices DSDontWriteUSBStores -bool true

```

---

## 🔍 自检与审计 (Audit)

配置完成后，建议通过以下命令检查防护状态：

1. **检查防火墙状态**：运行 `sudo pfctl -s info` 确认防火墙为 enabled。
2. **检查域名拦截**：运行 `ping qwen.alicdn.com` 确认返回的 IP 为 `127.0.0.1`。
3. **确认最高安全法案**：运行 `csrutil status`，**日常使用务必确保系统完整性保护 (SIP) 为 `enabled` 状态**，以防恶意组件非法提权。

```

```
