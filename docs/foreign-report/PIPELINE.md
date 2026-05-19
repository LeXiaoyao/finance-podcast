# 外资重磅 · 全流程手册

> 适用对象：人工操作 & Codex 自动化执行  
> 发布频率：每周一期（周日/周一完成，周一或周二发布）  
> 网站地址：https://lexiaoyao.github.io/finance-podcast/foreign-report/  
> GitHub 仓库：`git@github.com:LeXiaoyao/finance-podcast.git`  
> 本地仓库路径：`/Users/lexiaoyao/finance-podcast-public/`

---

## 目录

1. [命名规范](#1-命名规范)  
2. [Step 1 — 选题](#2-step-1--选题)  
3. [Step 2 — 出稿](#3-step-2--出稿)  
4. [Step 3 — 出封面](#4-step-3--出封面)  
5. [Step 4 — TTS 合成](#5-step-4--tts-合成)  
6. [Step 5 — 发布更新](#6-step-5--发布更新)  
7. [文件结构速查](#7-文件结构速查)  
8. [环境与密钥](#8-环境与密钥)  
9. [Codex 自动化执行指南](#9-codex-自动化执行指南)

---

## 1. 命名规范

每期有唯一 Episode ID，格式：

```
YYYYMMDD + F{N}
```

- `YYYYMMDD`：稿件对应的发布日期（非制作日期）  
- `F{N}`：全栏目累计期号（F1、F2、F3……）

**示例**

| 期号 | Episode ID   | 发布日期   |
|------|-------------|-----------|
| 第1期 | `20260426F1` | 2026-04-27 |
| 第2期 | `20260504F2` | 2026-05-04 |
| 第3期 | `20260512F3` | 2026-05-12 |
| 第4期 | `20260518F4` | 2026-05-18 |

---

## 2. Step 1 — 选题

**执行者：人工**

### 选题标准

本栏目聚焦**外资顶级投行对中国 A+H 股的最新研判**，每期围绕一个核心事件/报告展开：

- **目标来源**：高盛、摩根士丹利、摩根大通、花旗、美银、摩根士丹利、汇丰（中国股票策略报告）
- **触发条件**（满足任意一条即可）：
  - 顶级投行发布中国股票/宏观策略报告
  - 单周有 2+ 个重大宏观数据（PMI、CPI/PPI、外贸、GDP）
  - A/H 股出现 5% 以上单周涨跌幅
  - 外资资金流向出现明显拐点（北向大幅净买/净卖）

### 选题输出物

一段不超过 100 字的"选题摘要"，包含：
1. 本期核心事件 / 报告名称
2. 涉及机构（用于 `index.html` meta 字段）
3. 拟标题（中文，风格参考现有各期）

---

## 3. Step 2 — 出稿

**执行者：人工（或 LLM 辅助）**

### 稿件规格

| 维度      | 要求                                          |
|-----------|----------------------------------------------|
| 语言      | 简体中文，口播风格，无 Markdown 标记          |
| 时长      | 目标 7–10 分钟（对应约 2,500–4,000 字）        |
| 结构      | 开场引入 → 2–3 条主线 → 总结 + 思考题         |
| 禁忌      | 不含括号注释、英文缩写需展开（如"PMI"→"采购经理人指数"）|
| 数字      | 百分比用"加X点X%"表述（TTS 发音更自然）       |

### 开场模板

```
大家好，欢迎来到外资重磅第{N}期。
```

### 结尾模板

```
本节目仅作宏观观察和研报解读，不构成任何投资建议。
```

### 保存路径

```
/Users/lexiaoyao/财经研报播客/foreign_report_smoke/tts_input_{EpisodeID}.txt
```

---

## 4. Step 3 — 出封面

**执行者：Gemini API 自动生成（人工触发）**

### 生成方式

使用 Gemini `gemini-2.5-flash-image` 直接生图：

```bash
source /Users/lexiaoyao/小宇宙AI播客/shared/config/.env.listenhub

python3 /Users/lexiaoyao/小宇宙AI播客/gemini_genai_cover.py \
  "/Users/lexiaoyao/finance-podcast-public/docs/foreign-report/episodes/{EpisodeID}/covers/{EpisodeID}.png" \
  "为中国财经播客《外资重磅》生成第{N}期封面图。主标题：{标题}。
   风格：深色高端金融感背景（深蓝/暗金），中文大字标题醒目，配股市/数据可视化元素，
   正方形构图（1254×1254）。"
```

### 输出规格

- 格式：PNG，1254×1254 像素  
- 保存路径：`docs/foreign-report/episodes/{EpisodeID}/covers/{EpisodeID}.png`

### 依赖

- 环境变量：`GEMINI_API_KEY`（在 `.env.listenhub` 中）

---

## 5. Step 4 — TTS 合成

**执行者：MiniMax API 自动（人工触发）**

### 生成脚本（每期标准模板）

保存为：`/Users/lexiaoyao/财经研报播客/foreign_report_smoke/gen_tts_{EpisodeID}.py`

```python
#!/usr/bin/env python3
"""Generate TTS audio for 外资重磅 episode {EpisodeID} using MiniMax."""
import importlib.util
import os
import sys
from pathlib import Path

# Load env
ENV_FILE = Path("/Users/lexiaoyao/小宇宙AI播客/shared/config/.env.listenhub")
for raw_line in ENV_FILE.read_text(encoding="utf-8").splitlines():
    line = raw_line.strip()
    if not line or line.startswith("#") or "=" not in line:
        continue
    key, value = line.split("=", 1)
    os.environ.setdefault(key.strip(), value.strip().strip('"').strip("'"))

# Load minimax_tts module
MODULE_PATH = Path("/Users/lexiaoyao/小宇宙AI播客/minimax_tts.py")
spec = importlib.util.spec_from_file_location("minimax_tts", MODULE_PATH)
mod = importlib.util.module_from_spec(spec)
sys.modules[spec.name] = mod
spec.loader.exec_module(mod)

SCRIPT_PATH = Path(__file__).parent / "tts_input_{EpisodeID}.txt"
OUTPUT_PATH = Path("/Users/lexiaoyao/finance-podcast-public/docs/foreign-report/episodes/{EpisodeID}/audio/{EpisodeID}.mp3")
OUTPUT_PATH.parent.mkdir(parents=True, exist_ok=True)

text = SCRIPT_PATH.read_text(encoding="utf-8")
print(f"[gen] 文本长度: {len(text)} 字符")
print(f"[gen] 输出路径: {OUTPUT_PATH}")

result = mod.minimax_tts_with_alignment(
    text,
    OUTPUT_PATH,
    voice_id="Chinese (Mandarin)_News_Anchor",
    speed=1.10,
)

print(f"[gen] 完成: voice={result.voice_id}, audio_length_ms={result.audio_length_ms}")
duration_s = (result.audio_length_ms or 0) // 1000
print(f"[gen] 时长: {duration_s // 60:02d}:{duration_s % 60:02d}")
print(f"[gen] 文件大小: {OUTPUT_PATH.stat().st_size} bytes")
```

### TTS 参数说明

| 参数       | 值                                    |
|-----------|---------------------------------------|
| 模型       | `speech-2.8-hd`（在 minimax_tts.py 中定义）|
| 音色       | `Chinese (Mandarin)_News_Anchor`（普通话新闻女主播）|
| 语速       | `1.10`                                |
| 采样率     | 32000 Hz                              |
| 码率       | 128 kbps                              |
| 格式       | MP3，单声道                            |
| API 端点   | `https://api.minimax.chat/v1/t2a_v2`  |

### 输出路径

```
/Users/lexiaoyao/finance-podcast-public/docs/foreign-report/episodes/{EpisodeID}/audio/{EpisodeID}.mp3
```

---

## 6. Step 5 — 发布更新

**执行者：脚本自动 + git push**

每次发布需要更新 4 个位置：

### 6.1 新建 `episodes/{EpisodeID}/index.html`

```html
<!doctype html>
<html lang="zh-CN">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{标题}</title>
  <style>
    body { margin: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif; color: #111827; background: #f8fafc; }
    main { max-width: 760px; margin: 0 auto; padding: 40px 20px 56px; }
    h1 { font-size: 30px; line-height: 1.28; margin: 0 0 12px; }
    h2 { font-size: 20px; margin: 32px 0 12px; }
    p { font-size: 16px; line-height: 1.8; margin: 0 0 14px; }
    .meta { color: #64748b; margin-bottom: 24px; }
    audio { width: 100%; margin: 8px 0 24px; }
    .disclaimer { border-top: 1px solid #e2e8f0; margin-top: 28px; padding-top: 20px; color: #475569; }
  </style>
</head>
<body>
  <main>
    <h1>{标题}</h1>
    <p class="meta">发布时间：{YYYY-MM-DD} 12:00:00 +0800</p>
    <audio controls src="./audio/{EpisodeID}.mp3"></audio>
    <section>
      <h2>节目简介</h2>
      <p>{节目简介（1-2段）}</p>
    </section>
    <section class="disclaimer">
      <h2>免责声明</h2>
      <p>本节目仅作宏观观察和研报解读，不构成任何投资建议。</p>
    </section>
  </main>
</body>
</html>
```

### 6.2 新建 `episodes/{EpisodeID}/shownotes/{EpisodeID}.md`

```markdown
{节目简介正文，与 index.html 中相同}

本节目仅作宏观观察和研报解读，不构成任何投资建议。
```

### 6.3 在 `index.html`（首页）最前面插入新 `<li>` 卡片

插入位置：`<ul class="episode-list">` 标签内的**第一个 `<li>`** 之前。

```html
      <li>
        <div class="episode-card">
          <img src="./episodes/{EpisodeID}/covers/{EpisodeID}.png" alt="第{N}期封面">
          <div class="ep-body">
            <p class="ep-meta">第 {N} 期 &nbsp;·&nbsp; {YYYY-MM-DD} &nbsp;·&nbsp; {机构名}</p>
            <h3 class="ep-title"><a href="./episodes/{EpisodeID}/">{标题}</a></h3>
            <p class="ep-desc">{摘要（100字内）}</p>
            <span class="ep-duration">⏱ {MM:SS}</span>
          </div>
        </div>
      </li>
```

### 6.4 在 `feed.xml` 最前面插入新 `<item>`

插入位置：**第一个 `<item>` 标签之前**。

```xml
    <item>
      <title>{标题}</title>
      <link>https://lexiaoyao.github.io/finance-podcast/foreign-report/episodes/{EpisodeID}/</link>
      <guid isPermaLink="false">foreign-report-{EpisodeID}</guid>
      <pubDate>{RFC-2822 格式日期，如：Mon, 18 May 2026 12:00:00 +0800}</pubDate>
      <description>{摘要（与 index.html ep-desc 相同）}</description>
      <itunes:duration>{MM:SS}</itunes:duration>
      <itunes:image href="https://lexiaoyao.github.io/finance-podcast/foreign-report/episodes/{EpisodeID}/covers/{EpisodeID}.png" />
      <itunes:summary>{摘要}</itunes:summary>
      <enclosure url="https://lexiaoyao.github.io/finance-podcast/foreign-report/episodes/{EpisodeID}/audio/{EpisodeID}.mp3" length="{文件字节数}" type="audio/mpeg" />
    </item>
```

> `length` = MP3 文件的实际字节数（`os.path.getsize()`）  
> `itunes:duration` = `audio_length_ms // 1000` 换算为 `MM:SS`

### 6.5 更新 `feed.xml` 顶层 `<lastBuildDate>`

将 `<lastBuildDate>` 改为本期发布日期。

### 6.6 Git 提交推送

```bash
cd /Users/lexiaoyao/finance-podcast-public
git add docs/foreign-report/episodes/{EpisodeID}/ docs/foreign-report/feed.xml docs/foreign-report/index.html
git commit -m "feat: 外资重磅第{N}期上线 ({EpisodeID})"
git push
```

---

## 7. 文件结构速查

```
finance-podcast-public/
└── docs/
    └── foreign-report/
        ├── index.html                        ← 首页（新期放最前面）
        ├── feed.xml                          ← RSS Feed（新期放最前面）
        ├── covers/
        │   └── podcast-cover-v2.png          ← 播客频道封面（固定不变）
        └── episodes/
            └── {EpisodeID}/
                ├── index.html                ← 单集页面
                ├── audio/
                │   └── {EpisodeID}.mp3       ← TTS 生成音频
                ├── covers/
                │   └── {EpisodeID}.png       ← Gemini 生成封面（1254×1254）
                └── shownotes/
                    └── {EpisodeID}.md        ← 文字版节目简介

财经研报播客/foreign_report_smoke/
├── tts_input_{EpisodeID}.txt                 ← 口播稿（Step 2 产出）
└── gen_tts_{EpisodeID}.py                    ← TTS 生成脚本（每期一个）
```

---

## 8. 环境与密钥

**密钥文件**：`/Users/lexiaoyao/小宇宙AI播客/shared/config/.env.listenhub`

| 变量名             | 用途                     |
|-------------------|--------------------------|
| `MINIMAX_API_KEY` | MiniMax TTS 合成          |
| `GEMINI_API_KEY`  | Gemini 封面生成           |

**MiniMax 模块**：`/Users/lexiaoyao/小宇宙AI播客/minimax_tts.py`  
每个 gen_tts 脚本通过 `importlib` 动态加载，无需 pip install。

---

## 9. Codex 自动化执行指南

### 每期触发条件

每周五/六检查一次，如满足**选题标准**（见 Step 1），则执行以下自动化流程。

### Codex 执行的 Steps（按顺序）

#### Phase A：内容生成（需要网络 + LLM）

1. **搜集本周素材**  
   - 搜索高盛/摩根士丹利/摩根大通最新中国股票策略报告
   - 抓取本周 A/H 股关键数据：北向资金净流向、恒生科技涨跌幅、科创50涨跌幅
   - 抓取本周宏观数据更新（如有发布）

2. **生成口播稿**  
   - 按照 Step 2 规格写稿（口播风格，2500-4000 字）
   - 保存至：`/Users/lexiaoyao/财经研报播客/foreign_report_smoke/tts_input_{EpisodeID}.txt`

3. **确定本期元数据**（供后续步骤使用）：
   ```
   EpisodeID = YYYYMMDD + F{N}     # N = 当前最大期号 + 1
   标题 = "..."
   机构 = "高盛 / 摩根大通"
   摘要 = "..."（100字内）
   发布日期 = YYYY-MM-DD
   ```

#### Phase B：资产生成（调 API）

4. **生成 gen_tts 脚本**  
   - 按 Step 4 模板创建 `gen_tts_{EpisodeID}.py`，替换所有 `{EpisodeID}` 占位符

5. **生成封面**  
   ```bash
   source /Users/lexiaoyao/小宇宙AI播客/shared/config/.env.listenhub
   python3 /Users/lexiaoyao/小宇宙AI播客/gemini_genai_cover.py \
     "docs/foreign-report/episodes/{EpisodeID}/covers/{EpisodeID}.png" \
     "为播客《外资重磅》第{N}期生成封面。标题：{标题}。风格：深色金融感，深蓝暗金调，中文大字，正方形构图。"
   ```

6. **合成音频**  
   ```bash
   python3 /Users/lexiaoyao/财经研报播客/foreign_report_smoke/gen_tts_{EpisodeID}.py
   ```
   - 记录输出的 `audio_length_ms` 和文件大小（字节数）

#### Phase C：发布更新（写文件 + git push）

7. **创建单集目录结构**（`audio/`、`covers/`、`shownotes/` 子目录在步骤5、6中已自动创建）

8. **写 `episodes/{EpisodeID}/index.html`**（按 6.1 模板）

9. **写 `episodes/{EpisodeID}/shownotes/{EpisodeID}.md`**（按 6.2 模板）

10. **更新首页 `index.html`**：在 `<ul class="episode-list">` 后第一行插入新 `<li>` 卡片（按 6.3 模板）

11. **更新 `feed.xml`**：
    - 在第一个 `<item>` 前插入新条目（按 6.4 模板）
    - 更新 `<lastBuildDate>`

12. **Git 提交推送**（按 6.6）

### Codex 执行注意事项

- **密钥读取**：不要 hardcode，始终从 `.env.listenhub` 动态加载
- **期号计算**：解析 `feed.xml` 中现有 `<item>` 数量 + 1 得到 N
- **EpisodeID 日期**：取稿件对应的自然周一日期（如周日出稿则用下周一）
- **封面优先生成**：Gemini 图像生成可能需要 30–120s，先发起再做其他事
- **音频字节数**：从 `os.path.getsize()` 实时读取，不要估算
- **duration 格式**：`audio_length_ms // 1000` → `f"{s//60:02d}:{s%60:02d}"`
- **feed.xml 插入顺序**：新期永远排在最前面（RSS 阅读器按顺序读取）
- **index.html 插入顺序**：新期的 `<li>` 永远排在 `episode-list` 里第一位

### 检查清单（发布前验证）

每期发布前，必须同时遵守 `FOREIGN_REPORT_FACT_GUARDRAILS.md`。尤其是指数涨跌幅、盘中行情、外资机构观点和宏观数据，必须带清晰时间口径和来源口径，避免把盘中数据写成收盘结论，避免把旧机构观点写成最新动作。

- [ ] `tts_input_{EpisodeID}.txt` 已存在，字数 2500+
- [ ] `covers/{EpisodeID}.png` 已生成，文件大小 > 500KB
- [ ] `audio/{EpisodeID}.mp3` 已生成，文件大小 > 3MB
- [ ] `episodes/{EpisodeID}/index.html` 已创建
- [ ] `episodes/{EpisodeID}/shownotes/{EpisodeID}.md` 已创建
- [ ] `docs/foreign-report/index.html` 已更新（新卡片在最前）
- [ ] `feed.xml` 已更新（新 item 在最前，length 为实际字节数）
- [ ] Git commit 已推送，GitHub Actions 部署成功
