# FinMarket News Assistant

A voice-enabled financial market news agent built with Google ADK and Elasticsearch semantic search.

**Live demo:** https://fin-news-agent-74205747062.us-central1.run.app/ *(news dataset covers 2023)*

## What it does

Ask questions about financial market news in natural language (voice mode only). The agent retrieves relevant articles from a semantic search index and synthesizes a grounded response combining a high-level overview with specific facts (dates, companies, events).
### Video Walkthrough
[![Watch the video](https://img.youtube.com/vi/onpptenpStw/hqdefault.jpg)](https://www.youtube.com/watch?v=onpptenpStw)

## Architecture
![Architecture](/News%20Agent%20Diagram.jpg)
## Stack

| Component | Technology |
|---|---|
| Agent framework | Google ADK |
| LLM | `gemini-2.5-flash-native-audio-preview-12-2025` |
| Tool integration | Elastic MCP Server / Agent Builder |
| Vector store | Elasticsearch (Elastic Cloud Serverless) |
| Embeddings | `jina-embeddings-v5-text-small` |
| Search | ES\|QL semantic search |
| Deployment | GCP Cloud Run |

## Prerequisites

- Python environment with `google-adk`, `elasticsearch`, `python-dotenv`
- Elasticsearch project (e.g. Elastic Cloud Serverless)
- Gemini API key
- Dataset: [`polygon_news_sample.json`](https://www.kaggle.com/datasets/rdolphin/financial-news-with-ticker-level-sentiment/data) placed in the project root

## Setup

**1. Index the dataset**

Run the first sections of [fin_news_agent.ipynb](fin_news_agent.ipynb) to:
- Create the `fin_market_news` Elasticsearch index with a `semantic_text` field
- Bulk-ingest 5,548 articles

**2. Create the Agent Builder tool**

The notebook creates a `find_news_by_topic` ES|QL tool in Elastic Agent Builder, exposed via the Elastic MCP Server.

**3. Set up and configure the ADK agent**

Scaffold the agent:

```bash
adk create --type=code fin_news_agent --model gemini-2.5-flash-native-audio-preview-12-2025 --api_key $GOOGLE_API_KEY
```

The `fin_news_agent/agent.py` provided in this repo replaces the generated scaffold with the MCP tool integration and system prompt.

Add the following variables to `fin_news_agent/.env`:
```
KIBANA_URL=https://<your-project>.kb.<region>.gcp.elastic.cloud:443
KIBANA_API_KEY=<your-api-key>
```

**4. Run locally**

```bash
adk web fin_news_agent          # browser UI
```

## Testing

Start with these sample queries to verify end-to-end functionality:

| Query | Expected behaviour |
|---|---|
| *"What happened with the Federal Reserve interest rates in 2023?"* | Retrieves FOMC news, synthesizes rate decision summary |
| *"What's the weather like?"* | Refuses: *"Sorry, I can only help with financial news."* |
| *"News about Tesla in Q3 2023"* | Date-filtered semantic search |

To verify semantic search directly, run the `semantic_search(...)` cell in the notebook before using the agent.

## Deployment (Cloud Run)

```bash
adk deploy cloud_run --project=$GC_PROJECT --region=us-central1 --service_name=fin-news-agent --with_ui fin_news_agent

gcloud run services update fin-news-agent --project=$GC_PROJECT --region=us-central1 --set-env-vars="GOOGLE_GENAI_USE_VERTEXAI=0,GOOGLE_API_KEY=...,KIBANA_URL=...,KIBANA_API_KEY=..."
```
