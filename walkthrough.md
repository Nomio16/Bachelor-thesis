# Web Application Build — Walkthrough

## What Was Built

A complete web application with **Hexagonal Architecture** for Mongolian social media NLP analysis.

### Architecture

```
webapp/
├── nlp_core/               # DOMAIN CORE (7 modules, pure Python)
│   ├── models.py            # Dataclasses: AnalysisResult, EntityResult, etc.
│   ├── preprocessing.py     # Mongolian text cleaning + stopwords
│   ├── ner_engine.py        # NER via Davlan/bert-base-multilingual-cased-ner-hrl
│   ├── sentiment.py         # Sentiment via cardiffnlp/twitter-xlm-roberta-base
│   ├── topic_modeler.py     # BERTopic with multilingual MPNet embeddings
│   ├── network_analyzer.py  # Co-occurrence graph builder (NetworkX)
│   └── knowledge_base.py    # SQLite admin knowledge tree (only DB usage)
│
├── adapters/api/            # FASTAPI ADAPTER
│   ├── main.py              # App entry + CORS config
│   ├── schemas.py           # Pydantic request/response models
│   └── routers/
│       ├── analysis.py      # POST /api/upload, /api/analyze, /api/network
│       ├── insights.py      # POST /api/insights
│       └── admin.py         # GET/POST /api/admin/knowledge, labels, stopwords
│
├── adapters/sumbee/         # Future integration docs
│
└── frontend/                # NEXT.JS (Dashboard + Admin)
    └── src/app/
        ├── page.tsx         # Dashboard: upload, stats, sentiment, entities, network
        ├── admin/page.tsx   # Admin: knowledge base, labels, stopwords CRUD
        ├── layout.tsx       # Root layout with navbar
        └── globals.css      # Premium dark-mode design system
```

## Build Verification

- ✅ Next.js build passes (`npm run build`) — both `/` and `/admin` routes
- ✅ All NLP core modules have no framework dependencies
- ✅ All FastAPI routers use domain services through clean interfaces

## How to Run

### Backend
```bash
cd webapp
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cd adapters/api
PYTHONPATH=../../ uvicorn main:app --reload --port 8000
```

### Frontend
```bash
cd webapp/frontend
npm install && npm run dev
```

Then open http://localhost:3000 and upload your [sumbee_thesis_dataset.csv](file:///home/nomio/Documents/%D0%91%D0%A1%D0%90/preprocessing/sumbee_thesis_dataset.csv).
