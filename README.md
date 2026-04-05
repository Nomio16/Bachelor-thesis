# БСА — Bachelor Thesis: Mongolian NLP Social Monitoring

Content analysis, NER, sentiment analysis, and topic modeling for Mongolian social media text.

## Project Map

```
This repo (БСА)
├── NLP-intelligence/          # Web app + NLP backend + training data (submodule)
│   ├── nlp_core/              #   NER, sentiment, topic modeling, preprocessing
│   ├── adapters/api/          #   FastAPI REST API
│   ├── frontend/              #   Next.js dashboard
│   ├── Data/                  #   Training data & scripts
│   └── eval/                  #   Model evaluation
├── Thesis/                    # LaTeX thesis document (submodule)
├── colab/API.ipynb            # Colab notebook: run backend with GPU tunnel
├── NLP1.ipynb                 # Exploration: topic modeling + NER experiments
└── NLP2.ipynb                 # Exploration: sentiment + network analysis
```

### External platforms

| Platform | What lives there | Link |
|----------|-----------------|------|
| GitHub | This repo + submodules | [Nomio16/Bachelor-thesis](https://github.com/Nomio16/Bachelor-thesis) |
| GitHub | Web app repo | [Nomio16/NLP-intelligence](https://github.com/Nomio16/NLP-intelligence) |
| HuggingFace | Fine-tuned NER model | [Nomio4640/ner-mongolian](https://huggingface.co/Nomio4640/ner-mongolian) |
| HuggingFace | HF Space (deployed app) | [Nomio4640 NLP Intelligence](https://huggingface.co/spaces/Nomio4640/NLP-Intelligence) |
| Google Colab | GPU training + API tunnel | [API.ipynb on Drive](https://colab.research.google.com) (also versioned at `colab/API.ipynb`) |
| Kaggle | Social media dataset | [nomio211/socialdata](https://www.kaggle.com/datasets/nomio211/socialdata) |

---

## Running the App

### Option A: Fully local (no GPU needed for inference)

```bash
# Terminal 1 — Backend
cd NLP-intelligence
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cd adapters/api
PYTHONPATH=../../ uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

```bash
# Terminal 2 — Frontend
cd NLP-intelligence/frontend
npm install
npm run dev
```

Open http://localhost:3000. API docs at http://localhost:8000/docs.

### Option B: Backend on Colab (GPU) + Frontend local

Use this when you need GPU acceleration or don't want to install Python deps locally.

1. Open `colab/API.ipynb` in Google Colab (or the [Drive version](https://colab.research.google.com))
2. Run all cells — the last cell prints a **public tunnel URL** like `https://xxx.trycloudflare.com`
3. Copy that URL
4. Edit `NLP-intelligence/frontend/.env.local`:
   ```
   NEXT_PUBLIC_API_URL=https://xxx.trycloudflare.com
   ```
5. Start the frontend locally: `cd NLP-intelligence/frontend && npm run dev`
6. Open http://localhost:3000 — it now talks to Colab's GPU backend

**To switch back to local:** change `.env.local` back to `http://localhost:8000` and restart `npm run dev`.

### Option C: Docker (one command)

```bash
cd NLP-intelligence
docker build -t nlp-intelligence .
docker run -p 7860:7860 nlp-intelligence
```

Open http://localhost:7860. Everything runs inside the container.

---

## Workflows

### A. Changing the web app

```
1.  Edit code in NLP-intelligence/ (frontend/ or adapters/api/ or nlp_core/)
2.  Test locally (Option A above)
3.  Commit inside NLP-intelligence:
      cd NLP-intelligence
      git add -A && git commit -m "description" && git push
4.  Update parent repo's submodule pointer:
      cd ..  # back to БСА root
      git add NLP-intelligence
      git commit -m "update NLP-intelligence submodule"
      git push
```

### B. Retraining / improving the model

```
1.  Update training data in NLP-intelligence/Data/
      - v1 (CoNLL): edit Data/data/train_final.txt
      - v2 (JSONL): edit scripts in Data/datav2/, run generate_training_data.py
2.  Push data changes to GitHub
3.  Open Colab training notebook → it pulls latest code automatically
4.  Train → evaluate (use eval/evaluate.py for metrics)
5.  Push new model to HuggingFace:
      model.push_to_hub("Nomio4640/ner-mongolian")
      tokenizer.push_to_hub("Nomio4640/ner-mongolian")
6.  Tag the version locally:
      git tag model-v2.0 -m "F1: 0.XX, trained on train_v2_merged.jsonl"
      git push --tags
7.  Restart the web app backend — it auto-downloads the latest model from HF Hub
```

### C. Changing training data

```
1.  Edit files in NLP-intelligence/Data/
      - Reference data: Data/NER-dataset/ (locations.json, abbreviations, etc.)
      - Training scripts: Data/datav2/generate_training_data.py
      - Raw annotations: Data/data/train.txt
2.  Re-generate if needed:
      cd NLP-intelligence/Data/datav2
      python generate_training_data.py
3.  Commit & push
4.  If model needs retraining → follow Workflow B
```

### D. Updating the thesis

```
1.  Edit LaTeX files in Thesis/chapters/
2.  Compile: cd Thesis && pdflatex main && bibtex main && pdflatex main && pdflatex main
3.  Commit inside Thesis/:
      cd Thesis && git add -A && git commit -m "description" && git push
4.  Update parent submodule pointer (same as Workflow A step 4)
```

---

## Where things live (quick reference)

| I want to change... | Edit these files |
|---------------------|-----------------|
| NER model behavior | `NLP-intelligence/nlp_core/ner_engine.py` |
| Sentiment analysis | `NLP-intelligence/nlp_core/sentiment.py` |
| Topic modeling | `NLP-intelligence/nlp_core/topic_modeler.py` |
| Text preprocessing | `NLP-intelligence/nlp_core/preprocessing.py` |
| API endpoints | `NLP-intelligence/adapters/api/routers/*.py` |
| Frontend UI | `NLP-intelligence/frontend/src/app/` |
| Backend URL for frontend | `NLP-intelligence/frontend/.env.local` |
| Training data (v1) | `NLP-intelligence/Data/data/` |
| Training data (v2) | `NLP-intelligence/Data/datav2/` |
| Model evaluation | `NLP-intelligence/eval/` |
| Colab API notebook | `colab/API.ipynb` |
| Thesis text | `Thesis/chapters/*.tex` |

---

## Submodule Cheatsheet

```bash
# Clone this repo with all submodules
git clone --recurse-submodules https://github.com/Nomio16/Bachelor-thesis.git

# Pull latest changes (including submodules)
git pull && git submodule update --remote

# After editing a submodule, update the parent pointer
git add NLP-intelligence   # or Thesis
git commit -m "update submodule"
```
