# hello-ctf 周报 #3 素材汇总（2026.3.9–3.14）

本文件夹为第 3 周周报的参考资料与要点归纳，已按主题归类，便于查阅与上传 GitHub。

---

## 一、本周要闻概览

### 1. OpenClaw（“龙虾”）AI 智能体安全风险

- CNCERT 3 月 10 日风险提示，NVDB 跟进预警；多家信托、金融监管严禁/限制使用。
- 核心风险：越权操作、数据泄露、恶意注入、隐私窃取；监管提出“六要六不要”。
- 清华×蚂蚁团队曝光劫持链与“脑控”风险；OpenClaw 从“万能助手”被视为“万能后门”。

### 2. 其他网络安全动态

- **Veeam**：严重 RCE 漏洞（CVE-2026-21666 等），备份服务器成勒索攻击面，建议立即打补丁。
- **美国**：特朗普政府 2026 网络安全战略（3 月 6 日发布）本周解读高峰，强调进攻性网络行动、打击诈骗等。
- CISA iOS 漏洞、LexisNexis / TriZetto 数据泄露等亦有进展。

### 3. CTF 赛事节奏（本周）

- **刚结束**：CodeVinci CTF 2026（3/7–9）、港理工×NuttyShell 网络安全夺旗赛（3/6–8）。
- **预热/报名**：Marine Corps Cyber Games 春季 CTF（5/8 主赛）、ISC2 NJ CTF for Novices（3/21）、SUCTF 2026（3/14–16 倒计时）。
- **当日/当周**：Dark CTF（3/14 06:30 UTC，Jeopardy）、upCTF 2026、picoCTF 相关活动等。

---

## 二、参考来源与链接

### OpenClaw

- CNCERT/新华网：<https://www.news.cn/tech/20260310/959f13d18edb4759ae031a5e30523d23/c.html>
- 新浪财经（六要六不要等）：<https://finance.sina.com.cn/roll/2026-03-10/doc-inhqphnk3170978.shtml>
- 央视/魏亮采访：<https://www.cnr.cn/newscenter/native/gd/20260313/t20260313_527550454.shtml>
- 财联社（金融监管提示）：<https://www.cls.cn/detail/2311139>

### Veeam RCE

- Veeam KB：<https://www.veeam.com/kb4830>（12.x）、<https://www.veeam.com/kb4831>（13）
- BleepingComputer：<https://www.bleepingcomputer.com/news/security/veeam-warns-of-critical-flaws-exposing-backup-servers-to-rce-attacks/>

### 美国网络安全战略

- 白宫 PDF：<https://www.whitehouse.gov/wp-content/uploads/2026/03/President-Trumps-Cyber-Strategy-for-America.pdf>
- 行政令：<https://www.whitehouse.gov/presidential-actions/2026/03/combating-cybercrime-fraud-and-predatory-schemes-against-american-citizens/>
- CSIS 解读：<https://www.csis.org/analysis/what-does-new-cyber-strategy-really-mean>

### CTF 赛事

- Dark CTF：<https://ctftime.org/event/3182>，主办 <https://crack-on.live/>

---

## 三、AI 与 CTF 专题（周报核心）

### 3.1 国际：AI 战绩与基准

- **Cybersecurity AI (CAI)**  
  - 2025 年多场顶级 CTF Rank #1；NeuroGrid 41/45 flags、>5 万美元头奖；Dragos OT 比精英人类快 37%；Cyber Apocalypse、HTB AI vs Humans 等前列。  
  - 论文：<https://arxiv.org/abs/2512.02654>  
  - 中文评述：<https://www.themoonlight.io/zh/review/cybersecurity-ai-the-worlds-top-ai-agent-for-security-capture-the-flag-ctf>

- **Hack The Box 基准报告（2026.3.5）**  
  - NeuroGrid 数据：1337 支人类队 + 156 支 AI 队（958 vs 120 实际参赛）。  
  - 精英 AI 增强团队最多 **4.1 倍** 输出，解题率 27% vs 人类 16%；整体 AI 队解决率约 3.2 倍。  
  - Unite.AI 报道：<https://www.unite.ai/hack-the-box-benchmark-ai-augmented-teams-outperform-human-cybersecurity-analysts/>  
  - HTB 官方报告：<https://www.hackthebox.com/blog/hack-the-box-ai-cybersecurity-benchmark-report>

- **其他**  
  - krauq 博客「CTF is dying because of AI...?」：单人+AI 冲 CTFtime #1，讨论 Pay-to-Win、token 换胜利。  
    <https://blog.krauq.com/post/ctf-is-dying-because-of-ai>  
  - PAIStrike（ScantistAI）：Cyber Apocalypse 全自主参赛 #18。  
  - DiceCTF / AliyunCTF：社区热议 AI 滥用、PayToWin。

### 3.2 国内：规则与现场

- **阿里云 CTF 决赛**  
  - 现场普遍多开 AI 窗口（Claude/GPT/o1 等），边写 exploit 边用 AI 整理思路、生成 payload、debug；已成“公开秘密”，无官方禁令但选手默认全用。

- **强网杯等规则（应对 AI）**  
  - 第九届强网杯线上赛（2025）：**允许**生成式 AI 辅助，但须保留完整 AI 对话记录（导出/截图/视频），writeup 开头附网盘链接；不交或失效按作弊处理。  
  - 规则示例（签到题 readme）：<https://github.com/CTF-Archives/2025-qwbs9-quals>

- **线下设备限制（国赛/长城杯等）**  
  - 设备报备、每人一台、**功率不超过 200W**，防多机/多 GPU AI 农场；3 分钟内功率超标须整改。

### 3.3 社区声音与结论

- **观点碰撞**  
  - 一方：AI 是工具链进化（如 @glzjin），考察“问什么、验证什么、如何组织攻击”；另一方：volume + AI 冲榜、manual CTFing dead。  
  - 日本 CTF 圈研讨会标题：“AI 会超过顶级 CTF 选手吗？”  
  - DARPA AIxCC、Stanford ARTEMIS 等现实攻防案例被反复引用。

- **周报可用结论**  
  - AI 不是“斩杀线”，而是**新门槛**：纯手动会被拉开，AI-fluent + 安全功底者受益最大。  
  - 国内规则在“拖延”：允许 AI 但留痕、限功率，维持公平，但挡不住选手全员 AI 化。  
  - 未来方向：更多“AI 专杀题”、human-in-the-loop 验证、以及**分级/分赛道规则**（如 XSWCTF 2025 决赛 A0/A1/A2 分级：<https://detroit.sustech.edu.cn/games/6>），而非一刀切。

---

## 四、其他参考

- **CAI 论文要点**（main.md 整理）：Jeopardy CTF 对成熟 AI 已成“已解决游戏”；主张转向 A&D 格式；新型架构显著降本（如 98% 成本降低），经济上可行；讨论 OT 安全、局限性、伦理（双重用途、问责、能力扩散）。
- **配图与截图**：DiceCTF PayToWin 讨论、HTB 基准、krauq 等见本文件夹内 `assets` 或周报 pptx 中引用。

---


