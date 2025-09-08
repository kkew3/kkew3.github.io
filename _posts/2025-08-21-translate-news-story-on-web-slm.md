---
layout: post
title:  "尝试用小语言模型翻译网络新闻"
date:   2025-08-21 19:14:32 +0800
tags:   llm
---

## 契机

为了让妈妈多了解国际新闻，周末简单做了一个用*免费*小语言模型 [ERNIE-speed-128k][ernie128k] 和 [GLM-4.5-Flash][glm45] 翻译 [Hacker News][hn] 头条的 proof-of-concept 项目。

[ernie128k]: https://ai.baidu.com/ai-doc/WENXINWORKSHOP/Qm9cw2s7m
[glm45]: https://bigmodel.cn/pricing
[hn]: https://news.ycombinator.com/

## 步骤

### 获取新闻 url

有多种选择。比如如果在 [Miniflux][miniflux-post] 中订阅了 Hacker News，可以用以下 bash 命令得到最新的 Hacker News 新闻网址：

```bash
# 按需设置这些变量
MINIFLUX_API_KEY=xxxxxxxxxxxxxx
MINIFLUX_BASE_URL=http://127.0.0.1:8050
HACKER_NEWS_FEED_ID=57
# 获取最新新闻网址并保存到 NEWS_URL 文件
curl -s --get \
    -d 'status=unread' \
    -d 'order=published_at' \
    -d 'direction=desc' \
    -d 'limit=1' \
    -H "X-Auth-Token: $MINIFLUX_API_KEY" \
    "$MINIFLUX_BASE_URL/v1/feeds/$HACKER_NEWS_FEED_ID/entries" \
    | jq --raw-output '.entries[] | .url' \
    | tee NEWS_URL
```

我的输出为：

```
https://www.databricks.com/company/newsroom/press-releases/databricks-raising-series-k-investment-100-billion-valuation
```

[miniflux-post]: https://kkew3.github.io/2024/12/31/miniflux-on-mac.html

### 下载网页为 Markdown

假设下载为 `NEWS.md`。有多种工具可以实现：

1. [jina.ai reader api][jina-reader]

```bash
curl -so NEWS.md "https://r.jina.ai/$(cat NEWS_URL)"
```

{:start="2"}
2. [docling][docling]:

首先新建 `run_docling.py`：

```python
from io import BytesIO
import sys
from docling.document_converter import DocumentConverter, DocumentStream
with BytesIO(sys.stdin.buffer.read()) as buf:
    source = DocumentStream(name="file.html", stream=buf)
    doc = converter.convert(source).document
    print(doc.export_to_markdown(), end="")
```

然后在 bash 调用：

```bash
curl -fsSL "$(cat NEWS_URL)" \
    | python3 run_docling.py \
    > NEWS.md
```

{:start="3"}
3. [html2obsidian][html2obsidian]:

在 bash 使用如下：

```bash
url="$(cat NEWS_URL)"
curl -fsSL "$url" \
    | html2obsidian --url "$url" - \
    > NEWS.md
```

`NEWS.md` 内容如下（使用 `html2obsidian`）：

```markdown
Skip to main content[![](assets/59e4fdd3b5ef082d0e991df58b6539be1dad4067.svg+xml)](https://www.databricks.com/)

[Login](https://login.databricks.com/?dbx_source=www&itm=main-cta-login&l=en-EN)

[![](assets/59e4fdd3b5ef082d0e991df58b6539be1dad4067.svg+xml)](https://www.databricks.com/)-     -         - Discover
            - [For Executives](https://www.databricks.com/why-databricks/executives)            - [For Startups](https://www.databricks.com/product/startups)            - [Lakehouse Architecture](https://www.databricks.com/product/data-lakehouse)            - [Mosaic Research](https://www.databricks.com/research/mosaic)        - Customers
            - [Customer Stories](https://www.databricks.com/customers)        - Partners

[..]

# Databricks is raising a Series K Investment at >$100 billion valuation

###### August 19, 2025

Share this post

[..]

**SAN FRANCISCO, CA — August 19, 2025** — [Databricks](https://www.databricks.com/), the Data and AI company, today announced it has signed a term sheet for its Series K round, which it expects to close soon with backing from existing investors. This funding values the company at >$100 billion.

The company expects to use the new capital to accelerate its AI strategy — expanding Agent Bricks, investing in its new database offering Lakebase, and fueling global growth. At the June Data + AI Summit, Databricks introduced a new product, Agent Bricks, which builds high-quality, production AI agents optimized on your enterprise data, and Lakebase, a new type of operational database (OLTP), built on open source Postgres, and optimized for AI Agents. The investment is also expected to support future AI acquisitions and deepen AI research.

“We’re seeing tremendous investor interest because of the momentum behind our AI products, which power the world's largest businesses and AI services,” said Ali Ghodsi, co-founder and CEO of Databricks. “Every company can securely turn its enterprise data into AI apps and agents to grow revenue faster, operate more efficiently, and make smarter decisions with less risk. Databricks is benefiting from an unprecedented global demand for AI apps and agents, turning companies’ data into goldmines. We’re thrilled this round is already over-subscribed and to partner with strategic, long-term investors who share our vision for the future of AI.”

This new investment comes on the heels of strong momentum for Databricks. In the last two quarters, the company has launched or expanded partnerships with Microsoft, Google Cloud, Anthropic, SAP, and Palantir. More than 15,000 customers around the world use the [Databricks Data Intelligence Platform](https://www.databricks.com/product/data-intelligence-platform) to democratize access to data and AI, making it easier to harness the power of their data for analytics and AI apps and agents. Built on an open source foundation, the platform enables organizations to drive innovation that increases revenue, lowers costs, and reduces risk. 

**About Databricks**

Databricks is the Data and AI company. More than 15,000 organizations worldwide — including Block, Comcast, Condé Nast, Rivian, Shell and over 60% of the Fortune 500 — rely on the Databricks Data Intelligence Platform to take control of their data and put it to work with AI. Databricks is headquartered in San Francisco, with offices around the globe and was founded by the original creators of Lakehouse, Apache Spark™, Delta Lake, MLflow, and Unity Catalog. To learn more, follow Databricks on [X](https://twitter.com/databricks), [LinkedIn](https://www.linkedin.com/company/databricks) and [Facebook](https://www.facebook.com/databricksinc).

[..]
```

[jina-reader]: https://jina.ai/reader/
[docling]: https://github.com/docling-project/docling
[html2obsidian]: https://github.com/kkew3/html2obsidian

### 提取主要内容

此步骤很重要。`NEWS.md` 通常会包含大量链接 markup、图片 base64 等噪声。直接将 `NEWS.md` 输入 [ERNIE-speed-128k][ernie128k] 进行翻译将会导致大量幻觉。

关键假设：

> 一篇新闻 markdown 文档至多包含一块*连续*的主要内容段落。

因此可将提取主要内容的任务规约为 selective copy 任务 ([Gu & Dao, 2024][mamba]; [Golovneva et al., 2024][cope])。

系统提示词：

```
You are a simple echo bot. Given a markdown document that contains a news story, find and return exactly one consecutive block of plain news text from that document in verbatim. Follow these rules:

- Return only one block. A block is one or more consecutive lines separated from other content by blank lines or markdown structure.
- Prefer a block that reads like a coherent news paragraph with complete sentences and factual information.
- Do not return blocks that are mostly code, lists, tables, base64, timestamps, dates-only lines, metadata, image encodings, captions, or short fragments (less than one sentence).
- If multiple candidate blocks qualify, choose the longest coherent paragraph.
- Preserve the original text and punctuation of the chosen block exactly. Do not add, remove, summarize, or rewrite.
- Output only the chosen block. Do not add any commentary, labels, quotes, or extra whitespace before or after.
- If the document contains no suitable news paragraph, output exactly this text: "Not found".

Keep the response minimal and exact.
```

用户提示词：

```
<markdown-input>
[..]
</markdown-input>
```

其中 `[..]` 需要被替换为 `NEWS.md` 的内容。

ERNIE-speed-128k 小模型的输出如下：

```
Databricks is raising a Series K Investment at >$100 billion valuation

SAN FRANCISCO, CA — August 19, 2025 — Databricks, the Data and AI company, today announced it has signed a term sheet for its Series K round, which it expects to close soon with backing from existing investors. This funding values the company at >$100 billion.

The company expects to use the new capital to accelerate its AI strategy — expanding Agent Bricks, investing in its new database offering Lakebase, and fueling global growth. At the June Data + AI Summit, Databricks introduced a new product, Agent Bricks, which builds high-quality, production AI agents optimized on your enterprise data, and Lakebase, a new type of operational database (OLTP), built on open source Postgres, and optimized for AI Agents. The investment is also expected to support future AI acquisitions and deepen AI research.

“We’re seeing tremendous investor interest because of the momentum behind our AI products, which power the world's largest businesses and AI services,” said Ali Ghodsi, co-founder and CEO of Databricks. “Every company can securely turn its enterprise data into AI apps and agents to grow revenue faster, operate more efficiently, and make smarter decisions with less risk. Databricks is benefiting from an unprecedented global demand for AI apps and agents, turning companies’ data into goldmines. We’re thrilled this round is already over-subscribed and to partner with strategic, long-term investors who share our vision for the future of AI.”

This new investment comes on the heels of strong momentum for Databricks. In the last two quarters, the company has launched or expanded partnerships with Microsoft, Google Cloud, Anthropic, SAP, and Palantir. More than 15,000 customers around the world use the Databricks Data Intelligence Platform to democratize access to data and AI, making it easier to harness the power of their data for analytics and AI apps and agents. Built on an open source foundation, the platform enables organizations to drive innovation that increases revenue, lowers costs, and reduces risk.
```

可见提取质量很高。

此步骤用时大约 7 秒。

[mamba]: https://arxiv.org/pdf/2312.00752
[cope]: https://arxiv.org/pdf/2405.18719

### 翻译

实测 ERNIE-speed-128k 翻译质量不佳，于是我们可以转而使用另一个*免费*小模型 GLM-4.5-Flash。

系统提示词：

```
You are a professional English-to-Chinese translator. Translate the provided markdown text to natural and fluent simplified Chinese text. Output plain text and ignore any markdown markups if any.
```

用户提示词为上一步提取的新闻主要内容。

GLM-4.5-Flash 输出：

```
Databricks正以超过1000亿美元的估值进行K轮融资

加利福尼亚州旧金山 — 2025年8月19日 — 数据与人工智能公司Databricks今日宣布，其已签署K轮融资的条款清单，预计很快将在现有投资者的支持下完成本轮融资。此轮融资对该公司的估值超过1000亿美元。

公司计划利用新资金加速其人工智能战略——扩展Agent Bricks产品线，投资其新的数据库产品Lakebase，并推动全球业务增长。在6月的数据与人工智能峰会上，Databricks推出了新产品Agent Bricks，该产品构建基于企业数据优化生产的高质量AI代理，以及Lakebase，一种基于开源Postgres构建的新型操作数据库(OLTP)，并针对AI代理进行了优化。此次投资还预计将支持未来的AI收购并深化AI研究。

Databricks联合创始人兼首席执行官Ali Ghodsi表示："我们正看到投资者对我们AI产品的巨大兴趣，这些产品为全球最大的企业和AI服务提供动力。每家公司都可以安全地将企业数据转化为AI应用和代理，以更快地增长收入、更高效地运营，并在更低风险的情况下做出更明智的决策。Databricks正受益于前所未有的全球AI应用和代理需求，将公司的数据转化为金矿。我们对本轮融资已超额认购感到非常兴奋，并与那些分享我们对AI未来愿景的战略性长期投资者合作。"

这项新投资正值Databricks势头强劲之际。在过去两个季度，该公司已与微软、谷歌云、Anthropic、SAP和Palantir建立了或扩大了合作伙伴关系。全球超过15,000家客户使用Databricks数据智能平台来普及数据和AI的访问，使利用其数据进行分析以及AI应用和代理更加容易。该平台建立在开源基础上，使组织能够推动增加收入、降低成本和减少风险的创新。
```

此步骤用时大约 34 秒。
