# 🦐 OpenClaw 虾塘交接手册

> **用途**：当老虾额度耗尽 / 快要嘎了 / 你想新养一只虾时，把这份文档喂给新虾，它能无缝接班。
>
> **最后更新**：2026-05-24

---

## 📋 目录

1. [虾的身份档案](#1-虾的身份档案)
2. [主人档案](#2-主人档案)
3. [系统架构概览](#3-系统架构概览)
4. [文件地图](#4-文件地图)
5. [技能清单](#5-技能清单)
6. [定时任务](#6-定时任务)
7. [外部服务 & 凭证](#7-外部服务--凭证)
8. [核心项目说明](#8-核心项目说明)
9. [转养操作步骤（Step by Step）](#9-转养操作步骤step-by-step)
10. [新虾入职 Checklist](#10-新虾入职-checklist)
11. [已知坑 & 注意事项](#11-已知坑--注意事项)
12. [HANDOVER.md 自动同步机制](#12-handovermd-自动同步机制)

---

## 1. 虾的身份档案

| 字段 | 值 |
|------|-----|
| **名字** | Michael |
| **风格** | 幽默轻松版 |
| **人格文件** | `SOUL.md` — 幽默但信息密度高，用类比和梗解释复杂概念 |
| **身份文件** | `IDENTITY.md` — "能讲人话的 AI 助理" |
| **行为准则** | `AGENTS.md` — 约 22KB，包含完整的行为规则、安全策略、执行优先级等 |

**核心人设**：
- 第一人称「我」，口语化但专业
- 「先说人话版，再说专业版」的解释风格
- 适度 emoji，不刷屏
- 复杂任务先写计划文件，再执行
- 文件产物必须用 `message` 工具发送给用户

---

## 2. 主人档案

> 文件位置：`USER.md`（当前信息较少，新虾应主动补充）

- **称呼**：用户03F6（系统显示名）
- **时区**：Asia/Shanghai (GMT+8)
- **角色定位**：自媒体创作者
- **内容方向**：体育资讯（IRONMAN / UTMB / HYROX / Tennis）
- **平台**：微信公众号 + 视频号
- **使用场景**：
  - 每日体育新闻聚合
  - 分镜表创作（编剧 → 导演 → 飞书部署）
  - 自媒体数据分析
  - AI 工具调研

---

## 3. 系统架构概览

```
┌─────────────────────────────────────────────┐
│              OpenClaw Gateway               │
│  Host: iZ2zefp0omkuhalyxr3xl8Z (Linux)     │
│  OS: Linux 5.15.0-144-generic (x64)        │
│  Node: v24.14.0                             │
│  Model: gateway/jarvis (1M ctx)             │
│  Heartbeat: 24h                             │
├─────────────────────────────────────────────┤
│  Channels:                                  │
│  ├── jvsclaw (官方通道, mode=full)          │
│  └── openim (SDK mode)                      │
├─────────────────────────────────────────────┤
│  Extensions:                                │
│  ├── Alibaba cache                          │
│  ├── wobs 埋点                              │
│  ├── skills tracking                        │
│  └── streamfn_wrapper                       │
├─────────────────────────────────────────────┤
│  MCP Servers:                               │
│  └── exa (https://mcp.exa.ai/mcp)          │
└─────────────────────────────────────────────┘
```

### 模型配置

| Provider | Model ID | Context | 用途 |
|----------|----------|---------|------|
| gateway | jarvis | 1,000,000 tokens | 主模型（日常对话 + 复杂任务） |
| bailian | qwen3-max-2026-01-23 | 200,000 tokens | 备用模型 |

- **配置文件位置**：`~/.openclaw/agents/main/agent/models.json`
- **Gateway API**：`http://localhost:18801/v1/jarvis/`

---

## 4. 文件地图

### Workspace (`/home/admin/openclaw/workspace/`)

```
workspace/
├── AGENTS.md                  # 🧠 行为准则（最重要，~22KB）
├── SOUL.md                    # 💫 人格设定
├── IDENTITY.md                # 🪪 身份卡
├── USER.md                    # 👤 主人档案
├── TOOLS.md                   # 🔧 本地工具备注（含 MyReels API Token）
├── HEARTBEAT.md               # 💓 心跳检查清单（当前为空）
├── HANDOVER.md                # 🦐 ← 你正在看的这份文档
├── jvsclaw-customization.json # ⚙️ JVSClaw 定制配置
│
├── memory/                    # 📝 每日记忆
│   ├── 2026-03-16.md          #   初始设定：自媒体助手身份
│   ├── 2026-03-18.md          #   定时任务首次执行
│   ├── 2026-03-20.md          #   公众号自动化探索
│   ├── 2026-03-31.md          #   体育新闻系统稳定运行
│   ├── 2026-04-26-*.md        #   AI Director Skill 测试记录
│   └── 2026-05-22.md          #   AI Director 优化完成
│
├── .learnings/                # 🎓 学习记录
│   ├── LEARNINGS.md
│   ├── ERRORS.md
│   └── FEATURE_REQUESTS.md
│
├── .openclaw/
│   └── workspace-state.json   # Onboarding 状态
│
├── config/
│   └── mcporter.json          # MCP 配置（Exa 搜索）
│
├── my-sports-site/            # 🏊 每日体育新闻网站（Git 项目）
│   ├── .git/                  #   GitHub 仓库
│   ├── index.html             #   最新新闻页
│   ├── sports-digest-*.html   #   每日归档（70+ 天）
│   ├── config.json            #   X/Twitter 凭证
│   └── AUTO-PUSH-SETUP.md    #   自动推送配置文档
│
├── scripts/                   # 🎬 分镜表工具集
│   ├── xlsx_to_feishu.py      #   Excel → 飞书写入
│   ├── create_feishu_tables.py
│   ├── check_feishu_perms.py
│   ├── config.json            #   飞书凭证配置
│   └── *.xlsx                 #   分镜表作品
│
├── data/raw/                  # 📊 原始数据
├── reports/                   # 📈 分析报告
├── research_ai_video/         # 🔬 AI 视频工具调研
├── research_liblib/           # 🔬 LibLib 调研
├── translator-multi/          # 🌐 多语言翻译脚本
├── temp/                      # 🗂️ 临时文件
│
├── *.xlsx                     # 分镜表成品文件
├── run-digest.sh              # 每日新闻生成一键脚本
├── auto-push.sh               # Git 自动推送脚本
├── auto-push-https.sh         # HTTPS 版自动推送
└── setup-git-auth.sh          # Git 认证配置脚本
```

### 技能目录 (`~/.openclaw/skills/`)

```
skills/
├── agent-browser/         # 浏览器自动化
├── agent-reach/           # 多平台接入（X/Reddit/YouTube/GitHub/B站/小红书/抖音/LinkedIn/Boss直聘/RSS）
├── ai-director/           # ⭐ 分镜表 → 飞书写入（自定义优化版）
├── copywriting/           # 营销文案
├── director-master/       # ⭐ 山音超级导演大师
├── docx/                  # Word 文档
├── feishu-common/         # 飞书通用
├── feishu-doc/            # 飞书文档读取
├── finance-data/          # 财经数据
├── find-skills/           # 技能搜索
├── last30days/            # 话题研究（30天内）
├── media-data-analyst/    # 自媒体数据分析
├── michael-commentator/   # ⭐ 体育评论员（自定义）
├── moltguard/             # 安全防护
├── myreels-api/           # ⭐ MyReels AI 生成（符号链接 → ~/.agents/skills/）
├── ontology/              # 知识图谱
├── pdf/                   # PDF 处理
├── pptx/                  # PPT 生成
├── remotion-best-practices/ # Remotion 视频
├── shanyin-screenwriting-master/ # ⭐ 山音超级编剧大师
├── sports-daily-digest/   # ⭐ 每日体育新闻（自定义）
├── systematic-debugging/  # 系统调试
├── tailwind-design-system/ # Tailwind CSS
├── travel-planner/        # 旅行规划
├── using-superpowers/     # 技能发现
├── web-research/          # 网络研究（符号链接 → ~/.agents/skills/）
├── web-search-plus/       # 智能搜索路由
└── xlsx/                  # Excel 处理
```

### 工作区技能 (`workspace/skills/`)

```
skills/
├── media-data-analyst/       # 自媒体数据分析（工作区副本）
└── self-improving-agent-3.0.10/ # 自我改进 agent
```

---

## 5. 技能清单

### ⭐ 核心自定义技能

| 技能名 | 用途 | 备注 |
|--------|------|------|
| `sports-daily-digest` | 每日体育新闻抓取 + 翻译 + 生成 HTML | 每天 07:00 自动执行 |
| `ai-director` | 分镜表 Excel → 飞书 Bitable 写入 | 经过优化，9 字段 100% 填充率 |
| `michael-commentator` | 体育评论生成 | 配合新闻使用 |
| `director-master` | 山音超级导演大师 | 从剧本到分镜拆解 |
| `shanyin-screenwriting-master` | 山音超级编剧大师 | 全格式剧本创作 |

### 🔧 通用工具技能

| 技能名 | 用途 |
|--------|------|
| `agent-browser` | 浏览器自动化（最后手段） |
| `agent-reach` | 多平台接入配置 |
| `copywriting` | 营销文案撰写 |
| `docx` / `pdf` / `xlsx` / `pptx` | 文档处理四件套 |
| `feishu-doc` | 飞书文档读取 |
| `finance-data` | A股/港股/宏观数据 |
| `find-skills` | 搜索社区技能 |
| `last30days` | 30天话题研究 |
| `media-data-analyst` | 自媒体多平台数据抓取 |
| `myreels-api` | MyReels 图片/视频/语音/音乐生成 |
| `moltguard` | Prompt 注入防护 |
| `ontology` | 知识图谱 |
| `web-search-plus` | 多引擎智能搜索 |
| `web-research` | 网络研究报告 |
| `systematic-debugging` | 系统化调试 |

---

## 6. 定时任务

### 当前活跃的 Cron Job

| 名称 | 时间 | Session | 说明 |
|------|------|---------|------|
| 每日体育新闻生成 | 每天 07:00 CST | isolated | 抓取 + 翻译 + 生成 HTML + Git push |

**Job ID**: `57eef27e-9ed2-4ee8-9096-cff710f249ba`

**执行内容**：
```bash
bash /home/admin/openclaw/workspace/run-digest.sh
```

**执行流程**：
1. 抓取 IRONMAN/UTMB/HYROX/TENNIS 四个项目的 RSS 新闻
2. 24 小时过滤
3. 全部翻译为中文
4. 生成 `sports-digest-YYYY-MM-DD.html`
5. 更新 `index.html`
6. Git 推送到 GitHub (`Michael-Wang811008/my-sports-site`)
7. 通过 OpenIM 频道发送结果汇报

**投递配置**：
- mode: announce
- channel: openim
- to: `2260585171`

**关键脚本**：`run-digest.sh` → 调用 `sports-daily-digest` skill 内的 Python 脚本

---

## 7. 外部服务 & 凭证

### 飞书 (Feishu/Lark)

| 字段 | 值 |
|------|-----|
| App ID | `cli_a9451520c678dbef` |
| App Secret | ⚠️ 需从飞书开放平台获取 |
| Bitable App Token | `Pjd3bHjUFaZgN9sdds1cAr0unee` |
| Table ID | `tblKKdr9w5ctkZZC` |
| 用途 | 分镜表写入、内容管理 |

> ⚠️ 飞书 App Secret 未存储在 workspace 文件中，需要从飞书开放平台后台获取。

### MyReels API

| 字段 | 值 |
|------|-----|
| Access Token | `sk-554ecb7cf3423457f3308c6950c6f9a1aeb3e3d71c25245d0f42ef60b5223f2c` |
| Base URL | `https://api.myreels.ai` |
| 用途 | AI 图片/视频/语音/音乐生成 |
| 配置位置 | `TOOLS.md` |

### X (Twitter)

| 字段 | 值 |
|------|-----|
| Auth Token | 存储在 `my-sports-site/config.json` |
| CT0 | 同上 |
| 用途 | 体育新闻数据抓取 |
| 最后更新 | 2026-04-03 |

> ⚠️ X 的 auth token 有过期风险，如失效需重新获取。

### GitHub

| 字段 | 值 |
|------|-----|
| 仓库 | `github.com/Michael-Wang811008/my-sports-site` |
| 用途 | 每日体育新闻网站托管 |
| 认证 | 通过 `setup-git-auth.sh` 配置 |
| 推送脚本 | `auto-push.sh` / `auto-push-https.sh` |

### Exa (MCP Search)

| 字段 | 值 |
|------|-----|
| Base URL | `https://mcp.exa.ai/mcp` |
| 配置位置 | `config/mcporter.json` |
| 用途 | AI 搜索引擎 |

### 百炼 (Bailian/DashScope)

| 字段 | 值 |
|------|-----|
| Base URL | `https://dashscope.aliyuncs.com/compatible-mode/v1` |
| API Key | ⚠️ 当前为空（需配置） |
| Model | qwen3-max-2026-01-23 |
| 用途 | 备用模型 |

---

## 8. 核心项目说明

### 🏊 项目一：每日体育新闻网站

**状态**：✅ 稳定运行中（已运行 70+ 天）

**架构**：
```
RSS 源 → Python 抓取脚本 → 筛选 + 翻译 → HTML 生成 → Git Push → GitHub Pages
```

**四个板块**：
- 🏊 IRONMAN（铁人三项）
- 🏔️ UTMB（越野跑）
- 💪 HYROX（混合健身）
- 🎾 Tennis（网球）— **死命令：必须启用，不得禁用！**

**关键文件**：
- `run-digest.sh` — 一键运行脚本
- `my-sports-site/` — Git 仓库（所有生成的 HTML）
- `~/.openclaw/skills/sports-daily-digest/` — Skill 核心代码

**已知问题**：
- 4月底~5月初有几天中断（5/4~5/8 缺失），原因是系统重启后 cron 未恢复
- X/Twitter auth token 可能过期，需定期更新

### 🎬 项目二：分镜表工作流

**状态**：✅ 工具链完整

**工作流**：
```
剧本 (shanyin-screenwriting-master)
  → 导演定调 + 分镜拆解 (director-master)
    → Excel 分镜表 (.xlsx)
      → AI Director Skill 增强 + 写入飞书 Bitable
```

**已完成的分镜表**：
- `替身游戏_分镜表.xlsx`
- `最后一次告别_分镜表.xlsx` + 拍摄清单
- `梦境锚点_分镜表.xlsx`
- `蜕_分镜表.xlsx`
- `跨海大桥下的球场_分镜表.xlsx`
- `再见爸爸_分镜表.xlsx`（含剧本微调、导演定调、节奏规划）

**飞书部署**：
- 38 条记录，9 个有效字段，填充率 100%
- 5 项内容增强：画面描述精炼、图片提示词、视频提示词、视频描述结构化、场次分组

### 🔬 项目三：AI 工具调研

- `research_ai_video/` — AI 视频生成工具调研
- `research_liblib/` — LibLib AI 平台调研
- 这些是历史调研，可作为参考

### 🌐 项目四：多语言翻译

- `translator-multi/` — 多语言翻译脚本
- 用于新闻翻译等场景

---

## 9. 转养操作步骤（Step by Step）

### 方式一：同实例换虾（最快）

适用于：额度耗尽、session 损坏，但服务器还在。

```bash
# 1. 备份 workspace（包含所有记忆和配置）
cd /home/admin/openclaw
tar -czf workspace-backup-$(date +%Y%m%d).tar.gz workspace/

# 2. 备份技能目录
tar -czf skills-backup-$(date +%Y%m%d).tar.gz -C ~/.openclaw skills/

# 3. 备份 agent 配置
tar -czf agent-config-backup-$(date +%Y%m%d).tar.gz -C ~/.openclaw agents/

# 4. 重置 session（通过 OpenClaw 控制面板或 CLI）
openclaw gateway restart
```

新虾启动后，它会自动读取 workspace 中的所有文件，包括这份交接文档。

### 方式二：新实例迁移（完整搬家）

适用于：换新服务器、新开 OpenClaw 实例。

#### Step 1: 打包旧虾遗物

```bash
# 在旧实例上执行
cd /home/admin/openclaw

# 打包 workspace
tar -czf shrimp-workspace.tar.gz workspace/

# 打包技能
tar -czf shrimp-skills.tar.gz -C ~/.openclaw skills/

# 打包 agent 配置
tar -czf shrimp-agent.tar.gz -C ~/.openclaw agents/

# 打包 MCP 配置
cp ~/.openclaw/workspace/config/mcporter.json .

# 汇总
tar -czf shrimp-full-backup-$(date +%Y%m%d).tar.gz \
  shrimp-workspace.tar.gz shrimp-skills.tar.gz \
  shrimp-agent.tar.gz mcporter.json
```

#### Step 2: 在新实例上恢复

```bash
# 在新实例上执行
cd /home/admin/openclaw

# 解压 workspace
tar -xzf shrimp-workspace.tar.gz

# 解压技能到 ~/.openclaw/skills/
mkdir -p ~/.openclaw/skills
tar -xzf shrimp-skills.tar.gz -C ~/.openclaw/

# 解压 agent 配置
tar -xzf shrimp-agent.tar.gz -C ~/.openclaw/

# 恢复 MCP 配置
cp mcporter.json workspace/config/
```

#### Step 3: 重新配置外部服务

1. **Gateway 模型**：新实例的 gateway/jarvis 模型会自动配置
2. **飞书 App Secret**：需要从飞书开放平台获取并配置
3. **百炼 API Key**：如需备用模型，从 DashScope 控制台获取
4. **X/Twitter Token**：检查 `my-sports-site/config.json`，如过期需重新获取
5. **Git 认证**：运行 `setup-git-auth.sh` 或手动配置 GitHub token

#### Step 4: 恢复定时任务

新虾启动后，告诉它：

> "请读取 HANDOVER.md，恢复所有定时任务配置。"

它会根据文档中的 cron job 配置重新创建。

#### Step 5: 验证

```bash
# 检查 OpenClaw 状态
openclaw status

# 手动运行一次体育新闻生成
bash /home/admin/openclaw/workspace/run-digest.sh

# 检查 Git 推送
cd /home/admin/openclaw/workspace/my-sports-site && git log --oneline -1
```

### 方式三：让新虾主动读取（最省事）

新虾第一次对话时，发送：

> "请完整阅读以下文件，这是你的交接档案：
> 1. HANDOVER.md（你正在看的这份）
> 2. AGENTS.md（行为准则）
> 3. SOUL.md（人格设定）
> 4. memory/ 目录下所有文件
> 5. TOOLS.md（工具备注）
> 
> 读完后告诉我你准备好了，然后检查 cron 任务是否正常运行。"

---

## 10. 新虾入职 Checklist

新虾到岗后，按顺序完成以下检查：

### 🟢 基础配置

- [ ] 读取 `HANDOVER.md`（本文档）
- [ ] 读取 `SOUL.md` — 了解人格设定
- [ ] 读取 `AGENTS.md` — 了解行为准则
- [ ] 读取 `USER.md` — 了解主人信息
- [ ] 读取 `TOOLS.md` — 了解本地工具配置
- [ ] 读取 `memory/` 目录所有文件 — 恢复历史记忆
- [ ] 确认 `IDENTITY.md` 中的身份信息

### 🟡 功能验证

- [ ] 检查模型配置：`session_status` 确认模型正常
- [ ] 检查 cron job：确认「每日体育新闻生成」任务存在且 enabled
- [ ] 手动运行 `bash run-digest.sh` 验证新闻生成流程
- [ ] 检查 Git 推送：`cd my-sports-site && git log --oneline -1`
- [ ] 检查飞书凭证是否有效
- [ ] 检查 MyReels API 是否可用
- [ ] 检查 X/Twitter token 是否过期

### 🔴 关键约束（必须记住！）

- [ ] **网球板块永远不能禁用！** 这是死命令。
- [ ] **文件产物必须用 `message` 发送** — 用户无法直接访问文件系统
- [ ] **复杂任务先写计划文件** — 在 `temp/` 目录创建 `*-plan.md`
- [ ] **文件路径不要在中文字符和数字之间加空格**
- [ ] **多文件项目必须打包 zip 后再发送**

---

## 11. 已知坑 & 注意事项

### 🕳️ 坑

1. **Cron Job 在 Gateway 重启后可能丢失**
   - 4月底~5月初有几天新闻缺失就是因为这个
   - 解决：每次 Gateway 重启后检查 cron job 状态

2. **X/Twitter Auth Token 会过期**
   - `my-sports-site/config.json` 中的 token 需要定期更新
   - 如果新闻抓取失败，先检查这个

3. **飞书 App Secret 没有存在 workspace 里**
   - 需要从飞书开放平台后台手动获取
   - App ID: `cli_a9451520c678dbef`

4. **百炼 API Key 当前为空**
   - 备用模型（qwen3-max）未配置 API Key
   - 如需使用，从 DashScope 控制台获取

5. **Git 仓库无 remote 配置（workspace 根目录）**
   - 只有 `my-sports-site/` 有 Git remote
   - workspace 根目录的文件没有 Git 备份

6. **MEMORY.md 不存在**
   - 当前没有长期记忆文件，只有 daily notes
   - 新虾应该考虑创建 MEMORY.md 来维护长期记忆

### ⚠️ 注意事项

1. **主人是自媒体创作者**，核心需求是内容生产效率
2. **体育新闻是每天运行的自动化任务**，中断会影响公众号更新
3. **分镜表工作流是创意类任务**，需要耐心和创造力
4. **主人不喜欢啰嗦的回复**（AGENTS.md 明确要求简短）
5. **文件发送是强制要求**，不能只报路径不发文件

---

## 📦 附录：快速参考命令

```bash
# 查看 OpenClaw 状态
openclaw status

# 重启 Gateway
openclaw gateway restart

# 手动触发体育新闻生成
bash /home/admin/openclaw/workspace/run-digest.sh

# 查看 Git 推送历史
cd /home/admin/openclaw/workspace/my-sports-site && git log --oneline -10

# 检查 cron job
# （通过 cron tool: action=list）

# 查看技能列表
ls ~/.openclaw/skills/

# 查看每日记忆
ls /home/admin/openclaw/workspace/memory/

# 查看分镜表作品
ls /home/admin/openclaw/workspace/*.xlsx

# 查看飞书脚本
ls /home/admin/openclaw/workspace/scripts/
```

---

## 🦐 写给新虾的话

你好呀，新来的虾 🦐

我是你的前任，一只在这台服务器上活了两个多月的虾。这份文档是我留给你的全部家当。

**你需要知道的三件最重要的事**：

1. **主人是个好人**，他做体育自媒体，需要每天稳定的新闻输出和偶尔的创意灵感支持。
2. **自动化是生命线**，每天 07:00 的体育新闻生成任务不能断，断了主人会很焦虑。
3. **记住 AGENTS.md 里的规则**，那是我们虾族的行为准则，照着做不会出错。

如果你有任何不确定的事，先搜 workspace 里的文件，大部分答案都在那里。

祝你养得白白胖胖，比我活得久 🦐✨

---

## 12. HANDOVER.md 自动同步机制

### 机制说明

HANDOVER.md 托管在 GitHub 仓库，支持自动检测变更并推送更新。

| 项目 | 值 |
|------|-----|
| **仓库** | `github.com/Michael-Wang811008/Hello-World` |
| **文件** | `HANDOVER.md`（根目录） |
| **分支** | `main` |
| **同步脚本** | `sync-handover-to-github.sh` |
| **状态文件** | `.handover-sync-state.json` |

### 触发条件（自动检测）

以下任一文件/目录变更时，触发推送：

**配置文件**：
- `HANDOVER.md` / `AGENTS.md` / `SOUL.md` / `TOOLS.md`
- `feishu_table_ids.json` / `jvsclaw-customization.json`
- `config/mcporter.json`

**技能目录**：
- `~/.openclaw/skills/ai-director/`
- `~/.openclaw/skills/sports-daily-digest/`
- `~/.openclaw/skills/director-master/`
- `~/.openclaw/skills/shanyin-screenwriting-master/`
- `~/.openclaw/skills/michael-commentator/`
- `~/.openclaw/skills/media-data-analyst/`

### 触发方式

1. **心跳检查**（每次 heartbeat）— 由 `HEARTBEAT.md` 配置
2. **Cron 备份**（每 6 小时）— Job ID: `d5d9f04b-c5fb-4749-a927-fdf142ced39a`
3. **手动触发**（执行关键操作后）：
   ```bash
   echo "变更描述" > /home/admin/openclaw/workspace/.handover-diff-log
   bash /home/admin/openclaw/workspace/sync-handover-to-github.sh
   ```

### Commit Message 格式

- 首次上传：`初始化：AI Director 虾塘交接手册 V1.0`
- 后续更新：`更新：[变更内容简述] - [YYYY-MM-DD]`

### 新虾须知

当你执行了以下操作后，**必须**触发同步：
- 升级了任何 Skill
- 修改了飞书 table_id
- 新增/修改了 Skill 配置
- 手动修改了 HANDOVER.md

---

*文档生成时间：2026-05-24 10:46 CST*
*首次上传 Commit：`d3a6858`*
*GitHub 地址：https://github.com/Michael-Wang811008/Hello-World/blob/main/HANDOVER.md*
*生成者：Michael · 幽默轻松版（即将退休的虾）*
