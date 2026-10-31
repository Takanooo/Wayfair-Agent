# Wayfair Agent

A set of [n8n](https://n8n.io) workflows that use AI agents to research the rug market for Wayfair and turn the findings into downloadable HTML reports — market trends, competitor analysis, a content strategy, and a summary dashboard.

Supported categories: **Area Rug**, **Outdoor Rug**, **Hallway Runner**, **Shag Rug**.

## How it fits together

```
 Trend Analyzer Agent  ──► P2 Market Trend Report (.html) ─┐
                                                           ├──► Content Strategy (P4) ──► Strategy Report (.html)
 Competitor Analyzer   ──► P3 Competitor Report  (.html) ──┤
                                                           └──► Dashboard ──► Dashboard (.html)
```

The Trend and Competitor workflows run first. Their HTML reports are then uploaded into the Content Strategy and Dashboard workflows.

## Workflows

| File | Trigger | What it does |
|---|---|---|
| `Trend Analyzer Agent.json` | Chat | **P2 – Market Trends.** Pulls Amazon products plus Instagram, Pinterest and blog trend data, filters out off-category products, standardizes product attributes, identifies style micro-segments, and generates an AI image for each trend (FLUX via Hugging Face/fal.ai). Outputs a report with executive summary, scope, market research, category, attribute, visual-trend, risk and recommendation sections. |
| `Competitor Analyzer.json` | Chat | **P3 – Competitor Analysis.** Fetches products from Wayfair, Amazon and Walmart for the chosen category and builds a report covering scope, executive summary, Amazon analysis, retailer comparison, pricing & whitespace, supplier identification and recommendations. |
| `Project 4 - Combined Content Strategy Report (v4).json` | Form upload | **P4 – Content Strategy.** Takes the P2 and P3 reports plus a category and optional focus area, and writes a content strategy in Wayfair's brand voice: foundation, content & social, campaigns & email, competitive positioning and priorities. |
| `dashboard.json` | Form upload | **Dashboard.** Parses the P2 and P3 reports and fills an HTML dashboard template (product counts, key metrics, risks) into a single summary page. |

## Requirements

- An n8n instance (self-hosted or n8n Cloud) with the LangChain AI nodes
- A **Mistral AI** API key (`mistral-large-latest` / `mistral-small-latest`)
- An **OpenRouter** API key (used with `openrouter/free` models)
- A **Hugging Face** token with access to fal.ai inference (Trend Analyzer images only)
- Access to the project data server, which serves product, trend and dashboard-template data:
  - `GET /api/products/{wayfair|amazon|walmart}?category=<key>&limit=10`
  - `GET /api/trends/{instagram|pinterest|blogs}?category=<key>`
  - `GET /api/dashboard/template`

## Setup

1. In n8n, go to **Workflows → Import from File** and import each `.json` file.
2. Create credentials in n8n for **Mistral Cloud** and **OpenRouter**, then select them on every chat-model node.
3. In `Trend Analyzer Agent`, open the **Hugging Face** node and set the `Authorization` header to `Bearer <your HF token>`.
4. Point the workflows at your data server: in every **Fetch …** HTTP node, replace `YOUR_DATA_SERVER` in the URL with your server's address.
5. Activate the workflows.

## Usage

**Trend Analyzer / Competitor Analyzer** — open the workflow's chat and name a category. You can add an optional focus keyword:

```
area rug
outdoor rug focus: washable
```

The final node returns a downloadable `.html` report.

**Content Strategy / Dashboard** — open the form URL, enter the product category, and upload the P2 and P3 `.html` reports. Download the generated `.html` file.

> Tip: n8n's HTML preview node crops long pages — open the downloaded file in a browser to see the full report.

## Notes

- The workflows include **Wait** nodes between AI calls to stay under free-tier rate limits. If you hit rate-limit errors, increase the wait times; with paid keys you can shorten them.
- AI output is cleaned and parsed defensively (code fences removed, truncated JSON repaired), but LLM results can still vary between runs. Review reports before sharing them.
- Never commit API keys. Credentials live in n8n, not in these JSON files.
