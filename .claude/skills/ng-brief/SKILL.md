---
name: ng-brief
description: NG简报 — 预测市场情报简报。从 Polymarket、Kalshi、Manifold 三大预测市场采集 AI/科技、国际局势、Crypto 相关高价值预测信号，生成中文情报简报。Use when user mentions NG简报, 预测市场, prediction market, 情报简报, market intelligence brief, or Polymarket/Kalshi/Manifold data collection.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, WebSearch, WebFetch, Task, mcp__gemini__gemini-search, mcp__gemini__gemini-query
---

# 预测市场情报简报 (Prediction Market Intelligence Brief)

**Brand: 西雅图楠哥（Felix Cheng）**

从 Polymarket、Kalshi、Manifold 三大预测市场平台采集数据，筛选与 AI/科技/职业/国际局势/跨境贸易/Crypto 相关的高价值预测信号，生成中文情报简报。

## 产品定位

> 本简报从 Polymarket、Kalshi、Manifold 三大预测市场平台采集数据，筛选与 AI/科技/职业/国际局势/跨境贸易/Crypto 相关的高价值预测信号。数据即观点——这些是全球交易者用真金白银投票产生的概率，不是任何人的主观预测。

**输出格式**: Markdown — 可直接用于 newsletter / 社交媒体素材库
**输出目录**: `./issues/` (relative to repo root)
**文件命名**: `brief_{YYYY-MM-DD}.md`

## Input

无需参数，直接运行即可。自动使用当前日期生成简报。

## 三大话题定义

### Topic 1: 🤖 AI 科技 & 职业影响
- **角度**: AI发展到什么程度了？这对你的工作和创业意味着什么？
- **关键词**:
  - Polymarket: AI, AGI, OpenAI, Anthropic, Google AI, IPO, tech, layoffs, bubble
  - Manifold: AGI, AI bubble, OpenAI, Anthropic, Claude, GPT, remote work, tech layoffs, AI regulation, AI replaces jobs
  - Kalshi: artificial intelligence, technology, employment, jobs, layoffs

### Topic 2: 🌍 国际局势 & 跨境贸易
- **角度**: 全球发生了什么？这对跨境生意和你的钱包意味着什么？
- **关键词**:
  - Polymarket: recession, Fed, tariff, trade, Iran, China, oil, TikTok, sanctions, Ukraine
  - Manifold: US recession, trade war, China, TikTok ban, supply chain, dollar, oil price, cross-border
  - Kalshi: recession, interest rate, inflation, CPI, trade deal, tariff, gas price, unemployment, GDP

### Topic 3: 🪙 Web3 & Crypto
- **角度**: Crypto市场的真实共识是什么？社交媒体噪音之外的信号
- **关键词**:
  - Polymarket: Bitcoin, Ethereum, crypto, DeFi, stablecoin, SpaceX IPO, Coinbase
  - Manifold: Bitcoin, Ethereum, crypto regulation, DeFi, web3, stablecoin, NFT
  - Kalshi: BTC, ETH, SOL, crypto, Bitcoin

## 数据源 & API

| 平台 | 类型 | 认证 | 主要端点 |
|------|------|------|---------|
| Polymarket | 真金白银 (USDC) | 无需认证 | `https://gamma-api.polymarket.com/events?active=true&closed=false&limit=100&order=volume&ascending=false` |
| Manifold | 虚拟币 (Mana) | 无需认证 | `https://api.manifold.markets/v0/search-markets?term={keyword}&sort=liquidity&filter=open&limit=20` |
| Kalshi | 真金白银 (USD) | 需认证 → 用 Web Search fallback | `WebSearch: site:kalshi.com {keyword}` |

## 完整工作流 (5 Steps)

### Step 0: 环境准备

1. 获取当前日期
2. 扫描输出目录 `./issues/` 中已有的子目录数量，确定期号（#001, #002, ...）
3. 确定本期日期范围（上一期的日期 → 今天）

### Step 1: 数据采集

**使用 3 个 Task agent 并行采集三个话题的数据。** 每个 agent 的职责如下：

#### Agent 1: 🤖 AI 科技 话题数据

**1a. Polymarket 采集:**
```bash
curl -s "https://gamma-api.polymarket.com/events?active=true&closed=false&limit=100&order=volume&ascending=false" | python3 -c "
import json, sys
data = json.load(sys.stdin)
keywords = ['ai', 'agi', 'openai', 'anthropic', 'google ai', 'ipo', 'tech', 'layoff', 'bubble', 'gpt', 'claude']
results = []
for event in data:
    title = (event.get('title','') + ' ' + event.get('description','')).lower()
    if any(kw in title for kw in keywords):
        markets = event.get('markets', [])
        for m in markets:
            if m.get('active') and not m.get('closed'):
                prices = json.loads(m.get('outcomePrices', '[]'))
                yes_price = float(prices[0]) if prices else 0
                results.append({
                    'question': m.get('question'),
                    'probability': f'{yes_price*100:.1f}%',
                    'volume': m.get('volumeNum', 0),
                    'volume_1wk': m.get('volume1wk', 0),
                    'end_date': m.get('endDateIso'),
                    'slug': m.get('slug'),
                    'url': f'https://polymarket.com/event/{event.get(\"slug\")}'
                })
results.sort(key=lambda x: x['volume'], reverse=True)
for r in results[:15]:
    print(json.dumps(r, ensure_ascii=False))
"
```

**1b. Manifold 采集:**
对以下关键词分别搜索: `AGI`, `OpenAI`, `AI bubble`, `AI regulation`, `tech layoffs`
```bash
curl -s "https://api.manifold.markets/v0/search-markets?term=AGI&sort=liquidity&filter=open&limit=10" | python3 -c "
import json, sys
data = json.load(sys.stdin)
for m in data:
    if m.get('outcomeType') == 'BINARY' and not m.get('isResolved'):
        print(json.dumps({
            'question': m.get('question'),
            'probability': f'{m.get(\"probability\",0)*100:.1f}%',
            'traders': m.get('uniqueBettorCount', 0),
            'volume': m.get('volume', 0),
            'url': m.get('url'),
            'close_time': m.get('closeTime')
        }, ensure_ascii=False))
"
```
重复以上命令，替换 `term=` 参数为其他关键词。**每次调用间隔至少 1 秒。**

**1c. Kalshi 采集 (Web Search fallback):**
- WebSearch: `site:kalshi.com artificial intelligence 2026`
- WebSearch: `site:kalshi.com technology jobs layoffs`
- 从搜索结果中提取市场名称、概率、链接

#### Agent 2: 🌍 国际局势 话题数据

同上结构，使用国际局势相关关键词:
- Polymarket: 同一 events 数据，keywords 改为 `['recession', 'fed', 'tariff', 'trade', 'iran', 'china', 'oil', 'tiktok', 'sanctions', 'ukraine', 'war', 'inflation', 'gdp']`
- Manifold: term 改为 `recession`, `tariff`, `trade war`, `inflation`, `oil price`
- Kalshi: WebSearch `site:kalshi.com recession`, `site:kalshi.com tariff trade`

#### Agent 3: 🪙 Web3 & Crypto 话题数据

同上结构，使用 Crypto 相关关键词:
- Polymarket: keywords 改为 `['bitcoin', 'btc', 'ethereum', 'eth', 'crypto', 'defi', 'stablecoin', 'coinbase', 'spacex ipo', 'solana', 'sol']`
- Manifold: term 改为 `Bitcoin`, `Ethereum`, `crypto regulation`, `DeFi`
- Kalshi: WebSearch `site:kalshi.com bitcoin crypto`, `site:kalshi.com BTC ETH`

### Step 2: 筛选排序

收到三个 agent 的数据后，对每个话题执行以下筛选：

1. **去重**: 同一事件在不同平台的数据合并为一条，标注各平台概率
2. **排序权重**:
   - 交易量/流动性 (40%): Polymarket 以美元计，Manifold 以 trader 数量计
   - 时效性 (30%): 距结算日 7-90 天的优先，太远的降权，太近的已无悬念也降权
   - 跨平台覆盖 (20%): 同一事件多平台都有的加分
   - 话题相关性 (10%): 与目标读者（AI创业者/跨境电商/Crypto投资者）越相关越好
3. **每个话题取 top 5-8 个 poll**
4. **标注跨平台概率差异**: 如同一事件各平台差异 > 5个百分点，特别标注

### Step 3: 新闻充实

对每个选中的 poll:
- WebSearch 搜索 `"{poll关键词}" latest news {当前月份}`
- 提取 1-2 条最相关新闻的标题和一句话摘要
- **Throttling**: 每次搜索间隔 2 秒
- 此步可选，如果上下文窗口紧张可以跳过

### Step 4: 组装 Markdown 输出

按以下模板格式生成完整简报，保存到 `./issues/{YYYY-MM-DD}/brief_{YYYY-MM-DD}.md`

---

## 输出模板

```markdown
# 🔮 预测市场情报简报

**Period**: [上期日期] - [本期日期]（周一刊 / 周四刊）
**Generated**: [生成日期]
**Issue**: #[期号]

> 本简报从 Polymarket、Kalshi、Manifold 三大预测市场平台采集数据，筛选与 AI/科技/职业/国际局势/跨境贸易/Crypto 相关的高价值预测信号。数据即观点——这些是全球交易者用真金白银投票产生的概率，不是任何人的主观预测。

---

## 👋 关于西雅图楠哥 & 为什么做这份简报

我是西雅图楠哥，工科博士，AI 创业者。从大厂科学家、产品经理一路做到技术高管，研究过亚马逊的物流优化和搜广推算法，也带着团队蹲过货运站和生产车间。现在全职创业，用计算机视觉技术赋能传统行业。我最擅长的事情是在最前沿的 AI 技术和最泥泞的行业实践之间搭桥——让深奥的东西落地，让落地的东西有深度。

做这份简报的原因很简单：我们每天被海量信息淹没，但真正能帮你做决策的信号少之又少。我发现了一个被大多数人忽略的信号源——预测市场。在 Polymarket、Kalshi、Manifold 这三大平台上，全球交易者正在用真金白银对未来下注：AI 什么时候突破？美国会不会衰退？关税政策怎么走？这些不是谁的主观预测，而是成千上万人用钱投票产生的概率——准确率超过 94%，比民调准，比专家快，比新闻早。

但问题是：这些数据散落在三个不同平台上，原始信息是英文的、金融化的、对普通人不友好的。所以我决定每周把它们采集、筛选、翻译成对我们真正有用的情报——不管你是 AI 创业者、跨境电商卖家、求职者，还是想搞清楚世界正在往哪走的人，这份简报都能帮你在噪音中找到信号。

这不是一份千篇一律的模板报告。你在这里看到的，是经过筛选、跨平台对比、并附上解读的定制化情报。数据有出处，概率有来源，每一个判断都可以追溯到真金白银的市场共识。

---

## 🤖 话题一：AI 科技 & 职业影响

> 本周关键词：[根据本周数据动态生成，如"Anthropic领跑、AGI概率微升、AI泡沫担忧加剧"]

### Poll 1/N: [Poll 标题，用问题形式]

| 维度 | 数据 |
|------|------|
| **Polymarket** | [概率] [7日变化↑↓] · 交易量 $[X] · [链接] |
| **Kalshi** | [概率] [7日变化↑↓] · 交易量 $[X] · [链接] |
| **Manifold** | [概率] [7日变化↑↓] · [N] traders · [链接] |
| **跨平台差异** | [如有显著差异，标注：如"Polymarket 14% vs Kalshi 65%，时间窗口不同"] |
| **结算日期** | [日期] |

**🔍 为什么值得关注 (1-2句话)**:
[从创业者/职场人/内容创作者角度解读这个概率意味着什么]

**📰 相关资讯 (可选)**:
- [新闻标题1 + 一句话摘要 + 来源]
- [新闻标题2 + 一句话摘要 + 来源]

---

[...重复 Poll 2-N...]

---

## 🌍 话题二：国际局势 & 跨境贸易

> 本周关键词：[如"衰退概率飙升、油价破百、关税不确定性加剧"]

### Poll 1/N: [Poll 标题]
[同上卡片格式]

---

[...继续 5-8 个 polls...]

---

## 🪙 话题三：Web3 & Crypto

> 本周关键词：[如"BTC横盘、监管趋严、SpaceX IPO确定性高"]

### Poll 1/N: [Poll 标题]
[同上卡片格式]

---

[...继续 5-8 个 polls...]

---

## 📊 本周信号仪表盘

> 一张表速览本周最重要的概率变化

| # | 信号 | 当前概率 | 7日变化 | 来源 | 一句话影响 |
|---|------|---------|--------|------|-----------|
| 1 | [事件] | [X%] | [↑/↓ Npp] | [平台] | [对读者意味着什么] |
| 2 | ... | ... | ... | ... | ... |

**最大变动 🔥**: [本周概率变化最大的单一事件，一句话解读]
**值得持续关注 👀**: [概率还没大变，但交易量在急剧上升的事件——可能是变化的前兆]

---

## 💡 内容灵感

> 基于本周数据，以下角度可能适合发社交媒体：

- **小红书/WeChat 角度**: [中文受众感兴趣的切入点]
- **LinkedIn/X 角度**: [英文受众的切入点]
- **深度文章角度**: [适合长文的话题]
```

## 重要注意事项

### 数据采集规范
- **Polymarket**: 优先使用 events 端点（`/events`），它返回带嵌套 markets 的完整数据
- **Manifold**: 使用 search-markets 端点，按 liquidity 排序
- **Kalshi**: 因为需要认证，一律用 WebSearch `site:kalshi.com {keyword}` fallback
- **Rate Limiting**: Polymarket 100/min, Manifold 500/min, 每次请求间隔至少 1 秒
- **Fallback**: 如果 API 调用失败，用 WebSearch `site:polymarket.com {keyword}` 或 Gemini Search 替代

### 概率表示
- Polymarket: outcomePrices[0] 是 Yes 概率（0-1），乘以 100 显示
- Manifold: probability 字段直接是概率值（0-1），乘以 100 显示
- Kalshi: 从网页抓取的概率直接使用

### 交易量表示
- Polymarket: volumeNum 字段，单位 USDC（≈USD），显示格式 `$X.XM` 或 `$XXK`
- Manifold: uniqueBettorCount（trader数量）更有意义，显示格式 `N traders`
- Kalshi: 如有数据则显示，无则标注 "N/A"

### 跨平台事件匹配
- 通过关键词相似度判断是否为同一事件
- 注意不同平台可能用不同时间窗口（如 Polymarket 问 "by 2026" vs Kalshi 问 "by Q2 2026"）
- 时间窗口不同时在跨平台差异中标注

### 语言
- 简报正文全部使用中文
- Poll 标题保留英文原文（因为是外部市场的原始数据）
- 解读和分析用中文撰写

### 并行执行
- Step 1 的三个话题数据采集**必须用 3 个 Task agent 并行**，大幅缩短总时间
- Step 3 的新闻搜索也可以并行执行
- Step 4 的 Markdown 生成必须在所有数据采集完成后执行
