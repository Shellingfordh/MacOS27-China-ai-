# macOS-Anti-Censorship-Shield

一个用于在 macOS 15+ / macOS 27+ (Golden Gate) 系统中彻底切断、阻断国行特供版 Apple Intelligence 资格审查（`eligibilityd`）以及国内主流大模型/云服务（阿里通义千问、百度文心一言、腾讯混元、字节豆包）底层网络连接与行为追踪的硬核隐私加固指南。

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
