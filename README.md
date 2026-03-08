# 💰 AI Finance Assistant — Agentic RAG + Real-Time Data

> A production-grade AI Finance Assistant built entirely in **n8n** — no code required. Combines a personal knowledge base (RAG) with live market data APIs to answer complex financial queries intelligently.

---

## 🎯 What It Does

Ask any finance question and the agent decides the best way to answer:

| Query Type | How Agent Responds |
|---|---|
| *"What is a P/E ratio?"* | Searches your uploaded documents (RAG) |
| *"What is Bitcoin's price in PKR?"* | Fetches live data from CoinGecko API |
| *"Tell me about OGDC and today's news"* | Uses BOTH documents + live news together |
| *"What is the USD to PKR rate?"* | Hits live forex API instantly |

---

## 🏗️ Architecture

```
User Message (Chat)
       ↓
  AI Finance Agent (GPT-4o)
       ↓
  ┌─────────────────────────────────────────┐
  │  Tool 1: RAG — Search Knowledge Base    │  ← PSX Annual Reports
  │  Tool 2: Live Stock Price (Alpha Vantage)│  ← Real-time
  │  Tool 3: Finance News (NewsAPI)          │  ← Real-time
  │  Tool 4: Crypto Price (CoinGecko)        │  ← Real-time
  │  Tool 5: Forex Rate (ExchangeRate API)   │  ← Real-time
  └─────────────────────────────────────────┘
       ↓
  Synthesized Answer → Chat Response
```

---

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Workflow Automation | [n8n](https://n8n.io) |
| LLM Brain | OpenAI GPT-4o |
| Vector Database | Supabase pgvector |
| Embeddings | OpenAI text-embedding-3-small |
| Stock Data | Alpha Vantage API |
| Crypto Data | CoinGecko API (free) |
| News Data | NewsAPI |
| Forex Data | ExchangeRate API (free) |

---

## 📁 Project Structure

```
ai-finance-assistant/
│
├── workflow1_document_ingestion.json   # Ingests PDFs into Supabase
├── workflow2_ai_finance_agent.json     # Main AI agent workflow
└── README.md
```

---

## 🚀 Quick Start

### Prerequisites
- n8n account (cloud or self-hosted: `npx n8n`)
- Supabase account (free)
- OpenAI API key
- Alpha Vantage API key (free)
- NewsAPI key (free)

### Step 1 — Set Up Supabase

Run this in your Supabase SQL Editor:

```sql
-- Enable vector extension
create extension if not exists vector;

-- Create documents table
create table documents (
  id bigserial primary key,
  content text,
  metadata jsonb,
  embedding vector(1536)
);

-- Create vector search index
create index on documents
  using ivfflat (embedding vector_cosine_ops)
  with (lists = 100);

-- Create search function
create or replace function public.embedding(
  query_embedding vector(1536),
  match_count int default 5,
  filter jsonb default '{}'
)
returns table (
  id bigint,
  content text,
  metadata jsonb,
  similarity float
)
language plpgsql
as $$
begin
  return query
  select
    documents.id,
    documents.content,
    documents.metadata,
    1 - (documents.embedding <=> query_embedding) as similarity
  from documents
  where documents.metadata @> filter
  order by documents.embedding <=> query_embedding
  limit match_count;
end;
$$;
```

### Step 2 — Import Workflows into n8n

1. Open n8n → **New Workflow** → **Import from file**
2. Import `workflow1_document_ingestion.json`
3. Import `workflow2_ai_finance_agent.json`

### Step 3 — Add Credentials

In n8n Settings → Credentials, add:
- **OpenAI API** — your OpenAI key
- **Supabase** — your project URL + service role key

### Step 4 — Add Your Finance Documents

Collect PDF annual reports from [financials.psx.com.pk](https://financials.psx.com.pk) and run **Workflow 1** to ingest them.

Recommended starting documents:
- OGDC Annual Report
- HBL Annual Report
- Engro Annual Report
- PSO Annual Report
- Lucky Cement Annual Report

### Step 5 — Activate & Chat

Activate **Workflow 2** → click **Open Chat** → start asking questions!

---

## 💬 Example Queries

```
What is OGDC's main business activity?
What is Bitcoin's price in PKR today?
Get me the latest news about Pakistan economy
What is the USD to PKR exchange rate right now?
Tell me about HBL from my documents and today's news about it
Compare OGDC and ENGRO based on my uploaded reports
```

---

## ⚙️ Configuration

### Agent System Prompt
The agent is configured to:
- Search documents first for concept questions
- Use live APIs for price/news questions
- Combine multiple tools for complex queries
- Never give direct investment advice
- Always cite whether answer is from documents or live data

### Chunking Settings
Documents are split into **500 character chunks** with **50 character overlap** for optimal retrieval accuracy.

---

## 📊 Knowledge Base Sources

| Source | URL | Content |
|---|---|---|
| PSX Financials | financials.psx.com.pk | Annual & quarterly reports |
| PSX Data Portal | dps.psx.com.pk | Market summaries |
| SBP Publications | sbp.org.pk | Monetary policy, banking data |

---

## 🔧 Troubleshooting

**RAG returns empty results:**
Make sure the embedding model in both workflows is identical (`text-embedding-3-small`)

**Supabase search error (PGRST202):**
Run the `create or replace function public.embedding(...)` SQL above

**n8n runs out of memory:**
Use smaller PDFs (under 5MB) or increase chunk size to 1000 characters

**API 401 errors:**
Check API keys are pasted correctly with no extra spaces

---

## 🗺️ Roadmap

- [ ] Add Telegram bot integration
- [ ] PSX real-time price feed
- [ ] Portfolio tracking feature
- [ ] Multi-language support (Urdu)
- [ ] WhatsApp integration
- [ ] Automated daily market summary

---

##  Acknowledgements

- [n8n](https://n8n.io) — workflow automation platform
- [Supabase](https://supabase.com) — open source Firebase alternative
- [CoinGecko](https://coingecko.com) — free crypto API
- [PSX](https://psx.com.pk) — Pakistan Stock Exchange

---

## 📄 License

MIT License — feel free to use, modify, and share.

---
