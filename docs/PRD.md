# 预测市场情报简报 — Product Requirement Doc & Template v2

---

## 👋 关于西雅图楠哥 & 为什么做这份简报

我是西雅图楠哥，工科博士，AI 创业者。从大厂科学家、产品经理一路做到技术高管，研究过亚马逊的物流优化和搜广推算法，也带着团队蹲过货运站和生产车间。现在全职创业，用计算机视觉技术赋能传统行业。我最擅长的事情是在最前沿的 AI 技术和最泥泞的行业实践之间搭桥——让深奥的东西落地，让落地的东西有深度。

做这份简报的原因很简单：我们每天被海量信息淹没，但真正能帮你做决策的信号少之又少。我发现了一个被大多数人忽略的信号源——预测市场。在 Polymarket、Kalshi、Manifold 这三大平台上，全球交易者正在用真金白银对未来下注：AI 什么时候突破？美国会不会衰退？关税政策怎么走？这些不是谁的主观预测，而是成千上万人用钱投票产生的概率——准确率超过 94%，比民调准，比专家快，比新闻早。

但问题是：这些数据散落在三个不同平台上，原始信息是英文的、金融化的、对普通人不友好的。所以我决定每周把它们采集、筛选、翻译成对我们真正有用的情报——不管你是 AI 创业者、跨境电商卖家、求职者，还是想搞清楚世界正在往哪走的人，这份简报都能帮你在噪音中找到信号。

这不是一份千篇一律的模板报告。你在这里看到的，是经过筛选、跨平台对比、并附上解读的定制化情报。数据有出处，概率有来源，每一个判断都可以追溯到真金白银的市场共识。

---

## 📋 产品概述

**产品名称**: Prediction Market Intelligence Brief (预测市场情报简报)
**目标用户**: Felix 本人 → 后续可邀请朋友订阅
**输出频率**: 每周两次（周一 & 周四早上各生成一期）
**输出格式**: Markdown → 可直接用于 newsletter / 社交媒体素材库
**数据来源**: Polymarket (真金白银) + Kalshi (CFTC监管) + Manifold (虚拟币/长尾话题)

---

## 🏗️ Skill 功能需求

### 核心流程

1. **数据采集**: 通过各平台 API / Web Search，拉取三个话题下的活跃市场
2. **筛选逻辑**: 按交易量、概率变化幅度、距离结算时间排序，每个话题选 5-10 个最值得关注的 poll
3. **信号标注**: 标注本周概率变化方向和幅度（如 "↑ 从31%升至45%"）
4. **跨平台对比**: 同一事件在不同平台上的概率差异，标注出来（差异本身就是有价值的信号）
5. **资讯附加**: 围绕每个 poll 的话题，搜索 1-2 条最新相关新闻/分析（web search）
6. **输出生成**: 按下方模板格式输出 Markdown

### 筛选标准（每个话题选 poll 的优先级）

- **概率剧变**: 过去 7 天内概率变化 ≥ 10 个百分点的市场优先
- **高流动性**: 交易量 > $100K (Polymarket/Kalshi) 或 > 100 traders (Manifold)
- **时效性**: 距离结算日 7-90 天的市场优先（太远的信号弱，太近的已无悬念）
- **跨平台覆盖**: 同一事件在多个平台都有市场的优先（可做概率对比）
- **话题相关性**: 与 Felix 三大内容方向强相关

### 三大话题定义

```yaml
topics:
  - name: "🤖 AI 科技 & 职业影响"
    description: "AI技术前沿进展对创业决策和职场生存的影响"
    keywords_polymarket: ["AI", "AGI", "OpenAI", "Anthropic", "Google AI", "IPO", "tech", "layoffs"]
    keywords_kalshi: ["artificial intelligence", "technology", "employment", "jobs"]
    keywords_manifold: ["AGI", "AI bubble", "OpenAI", "Anthropic", "Claude", "GPT", "remote work", "tech layoffs", "AI replaces", "AI regulation"]
    polls_per_issue: 5-10
    content_angle: "AI发展到什么程度了？这对你的工作和创业意味着什么？"

  - name: "🌍 国际局势 & 跨境贸易"
    description: "宏观经济、地缘政治、关税贸易对跨境电商和全球商业的影响"
    keywords_polymarket: ["recession", "Fed", "tariff", "trade", "Iran", "China", "oil", "TikTok", "sanctions"]
    keywords_kalshi: ["recession", "interest rate", "inflation", "CPI", "trade deal", "tariff", "gas price", "unemployment"]
    keywords_manifold: ["US recession", "trade war", "China", "TikTok ban", "supply chain", "dollar", "oil price"]
    polls_per_issue: 5-10
    content_angle: "全球发生了什么？这对跨境生意和你的钱包意味着什么？"

  - name: "🪙 Web3 & Crypto"
    description: "加密货币市场走势、监管动态、AI与Crypto的交叉"
    keywords_polymarket: ["Bitcoin", "Ethereum", "crypto", "DeFi", "stablecoin", "NFT", "SpaceX IPO"]
    keywords_kalshi: ["BTC", "ETH", "SOL", "crypto", "Bitcoin"]
    keywords_manifold: ["Bitcoin", "Ethereum", "crypto regulation", "DeFi", "web3", "stablecoin"]
    polls_per_issue: 5-10
    content_angle: "Crypto市场的真实共识是什么？社交媒体噪音之外的信号"
```

### API 端点参考

| 平台 | 类型 | 数据获取方式 | 关键端点 |
|------|------|-------------|---------|
| Polymarket | 真金白银 (USDC) | Gamma API (公开，无需认证) | `https://gamma-api.polymarket.com/markets?active=true` |
| Polymarket | 真金白银 (USDC) | 价格历史 | `https://clob.polymarket.com/prices-history?market={id}` |
| Kalshi | 真金白银 (USD，CFTC监管) | REST API (需认证) | `https://api.elections.kalshi.com/trade-api/v2/markets` |
| Manifold | 虚拟币 (Mana) | REST API (公开，无需认证) | `https://api.manifold.markets/v0/markets` |
| Manifold | 虚拟币 (Mana) | 搜索 | `https://api.manifold.markets/v0/search-markets?term={query}` |

### 注意事项
- Polymarket Gamma API 免费，无需认证，rate limit 约 100 req/min
- Manifold API 完全公开，500 req/min，最易集成
- Kalshi API 需要注册账号获取 API key，有 tiered rate limits
- 如 API 不可用，可 fallback 到 web search 采集数据

---

## 📝 Newsletter 模板

以下是每期简报的标准格式。每个 poll 用统一的卡片结构呈现。

---

# 🔮 预测市场情报简报

**Period**: [上期发送日] - [本期日期]（周一刊 / 周四刊）
**Generated**: [生成日期]
**Issue**: #[期号]

> 本简报从 Polymarket、Kalshi、Manifold 三大预测市场平台采集数据，筛选与 AI/科技/职业/国际局势/跨境贸易/Crypto 相关的高价值预测信号。数据即观点——这些是全球交易者用真金白银投票产生的概率，不是任何人的主观预测。

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

### Poll 2/N: [Poll 标题]
[同上格式重复]

---

[...Poll 3-10 继续...]

---

## 🌍 话题二：国际局势 & 跨境贸易

> 本周关键词：[如"衰退概率飙升、油价破百、关税不确定性加剧"]

### Poll 1/N: [Poll 标题]
[同上卡片格式]

---

[...继续 5-10 个 polls...]

---

## 🪙 话题三：Web3 & Crypto

> 本周关键词：[如"BTC横盘、监管趋严、SpaceX IPO确定性高"]

### Poll 1/N: [Poll 标题]
[同上卡片格式]

---

[...继续 5-10 个 polls...]

---

## 📊 本周信号仪表盘

> 一张表速览本周最重要的概率变化

| # | 信号 | 当前概率 | 7日变化 | 来源 | 一句话影响 |
|---|------|---------|--------|------|-----------|
| 1 | [事件] | [X%] | [↑/↓ Npp] | [平台] | [对读者意味着什么] |
| 2 | ... | ... | ... | ... | ... |
| ... | ... | ... | ... | ... | ... |

**最大变动 🔥**: [本周概率变化最大的单一事件，一句话解读]
**值得持续关注 👀**: [概率还没大变，但交易量在急剧上升的事件——可能是变化的前兆]

---

## 💡 内容灵感（可选模块）

> 基于本周数据，以下角度可能适合发社交媒体：

- **小红书/WeChat 角度**: [中文受众感兴趣的切入点，如"全球交易者用5亿美元在赌什么？"]
- **LinkedIn/X 角度**: [英文受众的切入点，如"What $529M in prediction market bets tell us about..."]
- **深度文章角度**: [适合长文的话题，如"三个平台对AI泡沫的定价差异说明了什么"]

---

## 🔧 Skill 实现规格（给 Claude Code）

### 执行步骤

```
Step 1: 数据采集
  - 对每个话题的 keywords，分别查询三个平台
  - Polymarket: GET gamma-api, filter by active=true, sort by volume
  - Manifold: GET /v0/search-markets?term={keyword}&sort=liquidity
  - Kalshi: GET /v2/markets?status=open (需 auth)
  - Fallback: 如 API 失败，用 web_search "site:polymarket.com {keyword}" 等

Step 2: 筛选排序
  - 合并三个平台的结果
  - 计算7日概率变化（如有历史数据）
  - 按以下权重排序:
    - 概率变化幅度 (40%)
    - 交易量/流动性 (30%)
    - 时效性（距结算日 7-90 天加分）(20%)
    - 跨平台覆盖（多平台都有同一事件加分）(10%)
  - 每个话题取 top 5-10

Step 3: 资讯附加
  - 对每个选中的 poll，执行 web_search "{poll主题} latest news"
  - 提取 1-2 条最相关的新闻标题和摘要
  - 注意：这一步是可选的，可以配置开关

Step 4: 组装输出
  - 按模板格式生成 Markdown
  - 生成信号仪表盘汇总表
  - 生成内容灵感模块
  - 保存为 .md 文件
```

### 配置文件

```yaml
# prediction_market_brief_config.yaml

meta:
  name: "预测市场情报简报"
  version: "2.0"
  author: "Felix / 西雅图楠哥"
  schedule: "twice_weekly"  # Mon & Thu
  language: "zh-CN"   # 主语言
  output_format: "markdown"

topics:
  - id: "ai_career"
    name: "🤖 AI 科技 & 职业影响"
    description: "AI技术前沿进展对创业决策和职场生存的影响"
    polls_per_issue: [5, 10]  # min, max
    keywords:
      polymarket: ["AI", "AGI", "OpenAI", "Anthropic", "Google AI", "IPO", "tech", "layoffs", "bubble"]
      kalshi: ["artificial intelligence", "technology", "employment", "jobs", "layoffs"]
      manifold: ["AGI", "AI bubble", "OpenAI", "Anthropic", "Claude", "GPT", "remote work", "tech layoffs", "AI regulation", "AI replaces jobs"]
    content_angle: "AI发展到什么程度了？这对你的工作和创业意味着什么？"

  - id: "global_trade"
    name: "🌍 国际局势 & 跨境贸易"
    description: "宏观经济、地缘政治、关税贸易对跨境电商和全球商业的影响"
    polls_per_issue: [5, 10]
    keywords:
      polymarket: ["recession", "Fed", "tariff", "trade", "Iran", "China", "oil", "TikTok", "sanctions", "Ukraine"]
      kalshi: ["recession", "interest rate", "inflation", "CPI", "trade deal", "tariff", "gas price", "unemployment", "GDP"]
      manifold: ["US recession", "trade war", "China", "TikTok ban", "supply chain", "dollar", "oil price", "cross-border"]
    content_angle: "全球发生了什么？这对跨境生意和你的钱包意味着什么？"

  - id: "web3_crypto"
    name: "🪙 Web3 & Crypto"
    description: "加密货币市场走势、监管动态、AI与Crypto的交叉"
    polls_per_issue: [5, 10]
    keywords:
      polymarket: ["Bitcoin", "Ethereum", "crypto", "DeFi", "stablecoin", "SpaceX IPO", "Coinbase"]
      kalshi: ["BTC", "ETH", "SOL", "crypto", "Bitcoin"]
      manifold: ["Bitcoin", "Ethereum", "crypto regulation", "DeFi", "web3", "stablecoin", "NFT"]
    content_angle: "Crypto市场的真实共识是什么？社交媒体噪音之外的信号"

platforms:
  polymarket:
    base_url: "https://gamma-api.polymarket.com"
    auth_required: false
    min_volume_usd: 50000
    rate_limit: "100/min"
  kalshi:
    base_url: "https://api.elections.kalshi.com/trade-api/v2"
    auth_required: true
    min_volume_usd: 30000
    rate_limit: "tiered"
  manifold:
    base_url: "https://api.manifold.markets/v0"
    auth_required: false
    min_traders: 30
    rate_limit: "500/min"

filtering:
  min_probability_change_7d: 5      # 最小7日概率变化（百分点），低于此值降低优先级
  highlight_threshold_7d: 10         # 高于此值标记为"重大变动🔥"
  settlement_window_days: [7, 90]    # 只选距结算日在此范围内的市场
  cross_platform_bonus: true         # 多平台覆盖同一事件加分

news_enrichment:
  enabled: true           # 是否为每个poll搜索相关新闻
  max_news_per_poll: 2    # 每个poll最多附加几条新闻
  search_language: "en"   # 新闻搜索语言（可改为 "en,zh" 双语）

output:
  include_dashboard: true          # 是否生成信号仪表盘汇总表
  include_content_ideas: true      # 是否生成内容灵感模块
  include_cross_platform_diff: true # 是否标注跨平台概率差异
  file_naming: "brief_{date}.md"   # 输出文件命名规则
```

---

## 📌 后续扩展路线图

### Phase 1: 自用 (当前)
- Claude Code 实现 skill，每周手动触发生成
- Felix 自己阅读，挑选素材写帖子

### Phase 2: 分享 (1-2个月后)
- 邀请 3-5 个朋友加入阅读
- 收集反馈调整话题和格式
- 加入概率趋势追踪（记录每周快照，生成趋势图）

### Phase 3: 产品化 (如果验证了价值)
- 自动生成社交媒体 draft（小红书/LinkedIn/X 各一版）
- 双语版本：中文版发小红书/WeChat，英文版发 LinkedIn/X
- 在 Manifold 上创建与自己行业相关的预测市场
- 考虑做成付费 newsletter 或 SaaS dashboard
