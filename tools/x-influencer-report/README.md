# X 博主推广合作评估报告

用来在商务合作前评估一个 X (Twitter) 账号：近 1 年流量、趋势、内容结构，以及近 3 个月推广广告的期数、流量和互动率。
只用 Python 3.9+ 标准库，不需要安装依赖。

## 1. 获取数据（任选一种）

**A. X 官方 API（推荐，数据最全，含浏览量和收藏数）**

```bash
export X_BEARER_TOKEN=xxxx      # developer.x.com → Keys and tokens → Bearer Token
python3 x_report.py fetch --user PhyrexNi --days 365 --out phyrexni.json
```

用户时间线接口最多返回最近约 3200 条（含回复和转推）。发帖量很大的账号可能不到一年，脚本会给出提示，
这时改用下面的第三方导出。

**B. 第三方导出（Apify "Tweet Scraper"、twitterapi.io 等）**

把导出的 JSON / JSONL / CSV 直接交给 `analyze` 即可。常见字段名都能识别：
`createdAt / viewCount / likeCount / retweetCount / replyCount / quoteCount / bookmarkCount / isReply / isRetweet`。

**C. 手工整理的 CSV**

至少需要 `id, created_at, text, views, likes, retweets, replies` 这几列（也支持中文列名：推文ID、发布时间、正文、浏览量、点赞、转推、评论）。

## 2. 生成报告

```bash
python3 x_report.py analyze --input phyrexni.json --user PhyrexNi --out-dir report \
  --followers 250000 --quote-usd 3000
```

`--followers` 用 X API 抓取时会自动读取，可省略；`--quote-usd` 是可选的对方单条报价，用于估算 CPM。

输出文件：

| 文件 | 内容 |
|---|---|
| `report.html` | 给老板看的汇报页（KPI 卡片、月度趋势图、内容结构、广告分析） |
| `report.md` | 同内容 Markdown，方便贴进飞书 / Notion |
| `posts.csv` | 每条推文的明细、分类、互动率、广告判定 |
| `monthly.csv` | 月度流量与互动 |
| `ad_candidates.csv` | 疑似推广帖清单，用于人工复核 |

## 3. 人工复核广告（建议）

自动识别基于关键词（#ad、赞助、邀请码、返佣…）、交易所/券商/外汇品牌词和带参注册链接打分。
报给老板前，打开 `ad_candidates.csv`，在 `is_ad` 列填 1 / 0（可修改 `brand`），然后重跑：

```bash
python3 x_report.py analyze --input phyrexni.json --user PhyrexNi --out-dir report \
  --ad-review report/ad_candidates.csv
```

## 指标口径

- **主贴** = 原创 + 引用 + 回复自己的长推串；回复他人和纯转推不计入流量均值。
- **互动** = 点赞 + 转推 + 评论 + 引用 + 收藏。
- **加权互动率** = 总互动 / 总浏览；**单帖互动率中位数** 用来排除爆款干扰；**粉丝互动率** = 平均互动 / 粉丝数。
- **合作期数**：同一品牌 3 天内连续发布的推广帖算 1 期。
- **趋势**：近 90 天 vs 前 90 天的主贴浏览中位数、均值、互动率变化，以及月度浏览中位数的线性斜率。
- 分类和广告词表可以用 `--config my.json` 覆盖，键为 `categories`、`ad_strong`、`ad_weak`、`ad_brands`。
