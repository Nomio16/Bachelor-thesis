# Web Application & Thesis Integration Plan

## Project Review Summary

After reviewing the entire `БСА` folder, here is the current state of the project:

| Component | Status | Details |
|---|---|---|
| **Data Extraction** | ✅ Done | [data.py](file:///home/nomio/Documents/%D0%91%D0%A1%D0%90/preprocessing/data.py) and [dataExtract.py](file:///home/nomio/Documents/%D0%91%D0%A1%D0%90/preprocessing/dataExtract.py) pull from Sumbee's SQL Server (Twitter, Facebook, Instagram, Threads, Google, Website) |
| **Preprocessing (NER)** | ✅ Done | [preprocess.py](file:///home/nomio/Documents/%D0%91%D0%A1%D0%90/preprocessing/preprocess.py) — Mongolian filtering, URL cleanup → [sumbee_thesis_dataset.csv](file:///home/nomio/Documents/%D0%91%D0%A1%D0%90/preprocessing/sumbee_thesis_dataset.csv) (~14K posts) |
| **Preprocessing (Topic Modeling)** | ✅ Done | [ppForTopicModeling.py](file:///home/nomio/Documents/%D0%91%D0%A1%D0%90/preprocessing/ppForTopicModeling.py) — deep clean, stopwords, lowercase → [tm_sumbee_thesis_dataset.csv](file:///home/nomio/Documents/%D0%91%D0%A1%D0%90/preprocessing/tm_sumbee_thesis_dataset.csv) |
| **Topic Modeling** | ✅ Working | BERTopic with `paraphrase-multilingual-mpnet-base-v2` embeddings |
| **NER** | ✅ Working | Two models tested: `nomio4640/mongolian-bert-ner` (custom) and `Davlan/bert-base-multilingual-cased-ner-hrl` |
| **Sentiment Analysis** | ✅ Working | `cardiffnlp/twitter-xlm-roberta-base-sentiment` (XLM-RoBERTa) |
| **Network Analysis** | ✅ Basic | NetworkX co-occurrence graph from NER entities |
| **NER Labeling** | 🔄 In Progress | Label Studio running with PER/ORG/LOC/MISC labels on 14,955 tasks |
| **LaTeX Thesis** | ✅ Structure done | 5 chapters, compiles with XeLaTeX |
| **Web App** | ❌ Not started | This is the next step |

---

## Data Input Strategy

> [!IMPORTANT]
> **Recommendation: Start with CSV upload, add API integration later.**

For the thesis demo app, the best approach is:

1. **CSV File Upload** (Primary — Implement now): Users upload a CSV with at minimum a [text](file:///home/nomio/Documents/%D0%91%D0%A1%D0%90/preprocessing/ppForTopicModeling.py#80-92) column. This is the simplest, most portable method and perfect for a thesis demo. Your existing [sumbee_thesis_dataset.csv](file:///home/nomio/Documents/%D0%91%D0%A1%D0%90/preprocessing/sumbee_thesis_dataset.csv) is already in this format.

2. **Direct Text Input** (Secondary): A text box where users can paste individual posts for quick testing/demo purposes.

3. **Sumbee API Integration** (Future / Chapter 5): Document the architecture for connecting to Sumbee's database directly, but don't implement it for the thesis demo. This keeps the scope manageable and demonstrates the modular design.

**Why NOT social media links?** Scraping social media (Facebook, Twitter) requires OAuth API keys, is rate-limited, and breaks frequently. For a thesis defense, a CSV upload is far more reliable and reproducible.

---

## Software Architecture: Hexagonal (Ports & Adapters)

> [!IMPORTANT]
> The core NLP logic lives in a **framework-independent domain layer**. This means the same business logic can be reused by your demo web app, by Sumbee.mn, or by any future application — just by swapping the outer adapter.

```
┌─────────────────────────────────────────────────────────────────┐
│                        ADAPTERS (outer)                         │
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────────┐  │
│  │ FastAPI      │  │ CLI / Script │  │ Sumbee SDK adapter    │  │
│  │ (Demo Web)   │  │ (Colab/Test) │  │ (future integration)  │  │
│  └──────┬───────┘  └──────┬───────┘  └───────────┬───────────┘  │
│         │                 │                      │              │
│  ═══════╪═════════════════╪══════════════════════╪═══ PORTS     │
│         │                 │                      │              │
│  ┌──────▼─────────────────▼──────────────────────▼───────────┐  │
│  │                  DOMAIN CORE (pure Python)                │  │
│  │                                                           │  │
│  │  ┌─────────────┐ ┌──────────┐ ┌───────────┐ ┌──────────┐ │  │
│  │  │ Preprocessor│ │ NER      │ │ Topic     │ │Sentiment │ │  │
│  │  │             │ │ Engine   │ │ Modeler   │ │ Analyzer │ │  │
│  │  └─────────────┘ └──────────┘ └───────────┘ └──────────┘ │  │
│  │  ┌──────────────────┐  ┌───────────────────────────────┐ │  │
│  │  │ Network Analyzer │  │ Knowledge Base (SQLite file)  │ │  │
│  │  └──────────────────┘  └───────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ═══════════════════════════════════════════════════ PORTS       │
│         │                                                       │
│  ┌──────▼───────────────────────────────────────────────────┐   │
│  │ Frontend Adapter: Next.js (Dashboard + Admin)            │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Why Hexagonal?
- **Reusability**: The domain core has zero framework imports. Sumbee can import `nlp_core` as a Python package or call it via the FastAPI adapter.
- **Testability**: Each service (NER, Topic, Sentiment) can be unit-tested independently without spinning up a web server.
- **Pluggability**: Swapping from SQLite to PostgreSQL, or from FastAPI to gRPC, only changes the adapter — the core logic stays untouched.

### Technology Stack

| Layer | Technology | Reason |
|---|---|---|
| Frontend | **Next.js** (React) | Fast setup, SSR, modern UI |
| Charts | **D3.js** / **react-force-graph** | Interactive network visualization |
| Backend Adapter | **FastAPI** (Python) | Same language as NLP code, auto-docs, async |
| Database | **SQLite** (knowledge base only) | Zero config, stores admin-managed labels and knowledge tree. No DB for analysis data — that lives in-memory during processing |
| NLP Models | HuggingFace Transformers | Your existing pipeline |
| Styling | Vanilla CSS | Clean, professional look |

---

## Page 1: Analysis Dashboard

### Features

1. **Data Upload Panel**
   - CSV file upload (drag & drop)
   - Text input box for quick single-post analysis
   - Sample data button (loads your existing dataset)

2. **Network Analysis Graph** (like the reference images you shared)
   - Interactive force-directed graph showing entity co-occurrence
   - Nodes colored by entity type (PER=red, ORG=blue, LOC=green)
   - Node size = frequency, Edge thickness = co-occurrence count
   - Click a node to filter the data table

3. **Statistics Cards**
   - Total posts analyzed
   - Sentiment breakdown (Positive / Neutral / Negative) with pie chart
   - Top 10 entities (PER, ORG, LOC) with bar charts
   - Topic distribution (top topics from BERTopic)

4. **Search & Filter**
   - Filter by: Source (Twitter, Facebook, etc.), Sentiment, Topic, Date range
   - Full-text search across posts

5. **Insights Panel**
   - **Complaints / Negative trends**: Posts with negative sentiment + high entity frequency
   - **Hot issues**: Topics with rising volume
   - **Compliments**: Posts with positive sentiment

### Page 2: Admin / Knowledge Management

1. **Entity Labels Manager**
   - View all detected entities, edit their types, merge duplicates
   - Add custom entities to the knowledge base

2. **Topic Labels Manager**
   - View auto-detected topics, rename them, merge similar ones

3. **Stopwords Manager**
   - View/add/remove Mongolian stopwords

4. **Knowledge Base Editor**
   - Tree-structured view of categories (matches your DB approach from internship)
   - Admin can add word → category mappings

---

## Sumbee.mn Integration: NLP Intelligence Layer

> [!IMPORTANT]
> The integration direction is: **Your NLP layer → INTO Sumbee**, not Sumbee's DB into your demo.

The hexagonal design makes this straightforward. Your `nlp_core` domain layer can be deployed as a **microservice inside Sumbee's infrastructure**:

```
┌─────────────────────────────────────────────────────────────┐
│                     Sumbee.mn Platform                       │
│                                                              │
│  ┌──────────────┐    ┌──────────────────────────────────┐    │
│  │  Sumbee      │    │  YOUR NLP INTELLIGENCE LAYER     │    │
│  │  Backend     │───▶│  (deployed as microservice)      │    │
│  │  (existing)  │    │                                  │    │
│  │              │◀───│  POST /api/analyze               │    │
│  │  Sends text  │    │  POST /api/network               │    │
│  │  Receives    │    │  POST /api/insights              │    │
│  │  results     │    │                                  │    │
│  └──────────────┘    │  Uses the SAME domain core as    │    │
│                      │  your thesis demo web app        │    │
│  ┌──────────────┐    │                                  │    │
│  │  Sumbee      │    └──────────────────────────────────┘    │
│  │  Dashboard   │                                            │
│  │  (existing)  │    Shows NER, topics, sentiment,           │
│  │              │    network graphs from your API            │
│  └──────────────┘                                            │
└──────────────────────────────────────────────────────────────┘
```

**How it works:** Sumbee's existing backend sends raw social post text to your NLP API. Your API preprocesses, runs NER + Topic + Sentiment, and returns structured JSON results. Sumbee's dashboard then renders these results. The core Python logic is **identical** to what runs in your demo web app — only the adapter changes.

---

## Proposed Changes

### Backend (FastAPI)

#### [NEW] [webapp/](file:///home/nomio/Documents/БСА/webapp/)
New webapp directory at the project root with hexagonal architecture:

```
webapp/
├── nlp_core/                  # DOMAIN CORE (pure Python, no framework deps)
│   ├── __init__.py
│   ├── preprocessing.py       # Your existing cleaning pipeline (ported)
│   ├── topic_modeler.py       # BERTopic wrapper
│   ├── ner_engine.py          # NER pipeline (HuggingFace)
│   ├── sentiment.py           # Sentiment analysis pipeline
│   ├── network_analyzer.py    # NetworkX graph generation
│   ├── knowledge_base.py      # SQLite-backed knowledge tree (only DB usage)
│   └── models.py              # Plain Python dataclasses (no Pydantic here)
│
├── adapters/                  # ADAPTERS (framework-specific)
│   ├── api/                   # FastAPI adapter (REST API)
│   │   ├── main.py            # FastAPI app entry point
│   │   ├── routers/
│   │   │   ├── analysis.py    # /api/upload, /api/analyze, /api/network
│   │   │   ├── insights.py    # /api/insights
│   │   │   └── admin.py       # /api/admin/* CRUD for knowledge base
│   │   └── schemas.py         # Pydantic request/response models
│   └── sumbee/                # Future: Sumbee integration adapter
│       └── README.md          # Documents how to plug into Sumbee
│
├── frontend/                  # Next.js app (presentation adapter)
│   ├── app/
│   │   ├── page.tsx           # Dashboard page
│   │   └── admin/
│   │       └── page.tsx       # Admin page
│   ├── components/
│   │   ├── NetworkGraph.tsx
│   │   ├── SentimentChart.tsx
│   │   └── EntityTable.tsx
│   └── package.json
│
├── knowledge.db               # SQLite file (auto-created, for knowledge base only)
├── requirements.txt
└── README.md
```

> [!TIP]
> **No DB for analysis data.** Uploaded CSVs are processed in-memory and results are returned as JSON. Only the admin knowledge base (labels, categories, stopwords) uses SQLite.

---

## Verification Plan

### Browser Testing
1. Start the FastAPI backend: `cd webapp/backend && uvicorn main:app --reload`
2. Start the Next.js frontend: `cd webapp/frontend && npm run dev`
3. Open `http://localhost:3000` in the browser
4. Upload the existing [sumbee_thesis_dataset.csv](file:///home/nomio/Documents/%D0%91%D0%A1%D0%90/preprocessing/sumbee_thesis_dataset.csv) file
5. Verify the network graph, sentiment stats, and NER results display correctly
6. Navigate to the Admin page and verify label editing works

### Manual Verification
- The user will visually verify the network graph matches their reference images
- The user will confirm the sentiment/NER/topic results match their Colab notebook output
- The user will test the admin page by editing an entity label and verifying it persists

---

## Implementation Order

1. **Phase 1**: FastAPI backend with NLP pipeline (port existing Colab code)
2. **Phase 2**: Next.js frontend — Dashboard page with network graph and stats
3. **Phase 3**: Admin page with knowledge base management
4. **Phase 4**: Update thesis chapters (3 & 4) with implementation details and screenshots
