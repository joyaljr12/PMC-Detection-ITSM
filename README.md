# AI-Powered Problem Management Candidate Detection for ITSM


> **Portfolio notice**
>
> This repository is a sanitized portfolio version of an industry master's thesis
> project completed at CARIAD / Volkswagen Group.
>
> Original datasets, credentials, confidential configuration, internal endpoints,
> and active company API integrations have been removed for confidentiality and
> security reasons.
>
> The repository is intended to demonstrate the pipeline architecture, processing
> logic, modular design, experimentation approach, and selected implementation.

## Overview

This project implements the core workflow of an AI-assisted
**Problem Management Candidate (PMC) detection pipeline** developed during my
master's thesis.

The system processes IT Service Management incident data to identify groups of
recurring technical incidents and generate structured Problem Management
Candidates for further review.

The pipeline follows the workflow:

**Incident data → preprocessing → error-message extraction → semantic embeddings
→ clustering → PMC business rules → structured JSON payloads → GenAI-based
summarisation**

The public repository contains a sanitized implementation of the core processing
pipeline. Company-specific datasets, credentials, endpoints, and active internal
API integrations are intentionally excluded.


> **Note:** This repository is a sanitized portfolio version of an industry master’s-thesis project completed at CARIAD / Volkswagen Group. Original datasets, credentials, confidential configuration, internal endpoints, and active GenAI API integrations have been removed for confidentiality and security reasons. The repository is intended to demonstrate the pipeline architecture, processing logic, modular design, and engineering approach.



## Key Features

- Selective translation of non-English incident text
- Data cleaning and preprocessing
- Error-message extraction with fallback logic
- Sentence-BERT embedding generation using `all-MiniLM-L6-v2`
- Density-based clustering using DBSCAN
- Rule-based Problem Management Candidate generation
- Configurable minimum-incident and recurrence-window logic
- Structured JSON payload generation for each PMC
- LLM-based structured PMC summarisation
- Configurable pipeline stages
- Logging and processing metrics
- Modular Python implementation

## Pipeline Architecture

```text
Raw ITSM Incident Data
          │
          ▼
Selective Translation
          │
          ▼
Data Cleaning & Preprocessing
          │
          ▼
Error-Message Extraction
          │
          ▼
Semantic Embeddings
          │
          ▼
Density-Based Clustering
          │
          ▼
PMC Business Rules
          │
          ▼
Structured JSON Payload
          │
          ▼
LLM-Based PMC Summary
```

```markdown
## Experimentation & Model Selection

The thesis did not begin with a fixed embedding or clustering approach.
Multiple text representations and density-based clustering methods were evaluated
before selecting the configuration used in the portfolio implementation.

| Component | Approaches evaluated |
|---|---|
| Text representation | TF-IDF, Word2Vec, Sentence-BERT MiniLM, Sentence-BERT MPNet |
| Clustering | DBSCAN, HDBSCAN |
| Evaluation | Quantitative clustering metrics and manual cluster review |
| GenAI summarisation | Four GPT models |
| Selected LLM | GPT-4.1-mini |

Model selection was based not only on quantitative performance but also on
whether the resulting clusters were semantically meaningful and useful within
the Problem Management workflow.

The public implementation focuses on the selected pipeline configuration rather
than reproducing every experiment performed during the thesis.
```
## 📁 Project Structure 

├── config/
│   └── config.yaml                  # Safe configuration (no API keys)
│
├── prompts/
│   └── pmc_prompt.txt               # Prompt template for summarisation
│
├── src/
│   ├── clustering/
│   │   └── clusterer.py             # DBSCAN clustering
│   │
│   ├── embeddings/
│   │   └── text_embeddings.py       # SBERT embeddings
│   │
│   ├── pmc/
│   │   ├── pmc_creation.py          # PMC candidate logic
│   │   ├── pmc_payload_builder.py   # PMC JSON payload generator
│   │   └── pmc_summarizer.py        # LLM-driven summarisation
│   │
│   ├── preprocessing/
│   │   ├── cleaning.py              # Column drop, NA handling, date unification
│   │   ├── error_extraction.py      # A#3 extraction + fallback error logic
│   │   └── llm_description_cleaner.py
│   │
│   └── translation/
│       └── translator.py            # Optional translation module
│
├── main.py                          # Runs full pipeline end-to-end
├── pipeline.py                      # Pipeline controller / orchestrator
├── preprocess_pipeline.py           # Preprocessing-only pipeline
├── run_summarisation_only.py        # Summarise PMCs without recomputing embeddings
├── requirements.txt                 # List of Python dependencies
└── README.md                        # Project documentation
```


## ⚙️ Configuration (config.yaml)

This file controls all pipeline behaviour. It defines which dataset to load, whether translation is enabled, embedding model, clustering thresholds, PMC creation logic, JSON payload output paths, and summarisation settings.

```yaml
data:
  input_path: "<your_dataset.xlsx>"

translation:
  enabled: false
  api_key: "......"
  region: "northeurope"
  endpoint: "https://api.cognitive.microsofttranslator.com"

embeddings:
  model: "all-MiniLM-L6-v2"
  batch_size: 32

clustering:
  eps: 0.14
  min_samples: 5

pmc:
  min_tickets: 5
  window_days: 21
  export_path: "pmc_clusters.xlsx"

payload:
  output_dir: "outputs/payloads"

summarisation:
  enabled: false
  prompt_file: "prompts/pmc_prompt.txt"
  client_id: "....."
  client_secret: "....."
  virtual_key: "...."
  model: "gpt-4.1-mini"
  limit: 2
  output_dir: "outputs/summaries"
```

---

## ▶️ Installation & Setup

**1. Create a virtual environment**

```bash
python -m venv venv
venv\Scripts\activate
```

**2. Install required packages**

```bash
pip install -r requirements.txt
```

**3. Prepare your dataset**

Place your Excel dataset in the project root (next to `main.py`) and update `config.yaml`:

```yaml
data:
  input_path: "<your_dataset.xlsx>"
```

**4. Run the full pipeline**

```bash
python main.py
```

This will:
1. Load your dataset
2. Clean titles and descriptions
3. Extract raw and cleaned error messages
4. Generate SBERT embeddings
5. Cluster using DBSCAN
6. Create PMC candidates
7. Generate JSON payloads in `outputs/payloads/`
9. Export PMC supervisor Excel file: `pmc_clusters.xlsx`
10. Run PMC summarisation (if enabled)
