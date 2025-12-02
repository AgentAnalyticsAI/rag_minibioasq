# Real-World Benchmarking — RAG Mini BioASQ

Benchmarking framework to compare vector retrieval systems (WaveflowDB vs Pinecone) using the `enelpol/rag-mini-bioasq` dataset and a curated set of RAG queries.

This repository contains scripts to prepare the dataset, upload documents to vector stores, generate embeddings, and run a parallel evaluation pipeline that computes standard retrieval metrics and timing information.

## Key Features

- Multi-system comparison: WaveflowDB and Pinecone
- Metrics: Precision, Recall, F1, MRR, nDCG@k
- Multiprocessing support for scalable evaluation
- Optional hybrid filtering (VQL conversion) for WaveflowDB
- Timing metrics for embeddings and queries
- Configurable via a `.env` file

## Project Files

```
rag_minibioasq/
├── 1_get_data.py            # Download & prepare raw dataset from HF
├── 2_prepare_data_id.py     # Normalize filenames; map queries -> passages
├── 3_waveflow_upload.py     # Upload documents to WaveflowDB / VectorLake
├── 4_pinecone_upload.py     # Extract text, create embeddings, upsert to Pinecone
├── 5_run_pipeline.py        # Main evaluation pipeline (parallel)
├── utils.py                 # Helper utilities (VQL, filename cleaning, etc.)
├── Instructions.txt         # Short execution notes
├── requirements.txt         # Python deps
├── .env                     # Environment config (template)
└── results/                 # Output metrics and logs (generated)
```

## Quick Start

Requirements

- Python 3.8+
- (Optional) GPU for faster embedding generation

Install dependencies (PowerShell):

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -U pip
pip install -r requirements.txt
python -m spacy download en_core_web_sm
```

Prepare environment

1. Copy the included `.env` template and fill in required values (API keys, paths):

- `DATA_DIR_SOURCE`, `DATA_DIR_TARGET`, `DATA_DIR_FORMATTED`, `QUERY_MAP_SOURCE_FILE`
- `RESULTS_DIR`, `LOGS`, `DELIMITER`
- `MODEL_NAME`, `BATCH_SIZE`, `TOP_K`, `MAX_WORKERS_QUERY`
- `PINECONE_API_KEY`, `PINECONE_INDEX_NAME` (for Pinecone)
- `WAVEFLOWDB_API_KEY`, `BASE_URL`, `NAMESPACE`, `USER_ID`, `SESSION_ID` (for WaveflowDB)

## Obtaining API Keys

- **Pinecone:**

  - Visit the Pinecone console at `https://app.pinecone.io` and sign in or create an account.
  - Create a project (or select an existing one) and navigate to the "API Keys" section.
  - Create a new API key, copy the key value and add it to your `.env` as:

    ```env
    PINECONE_API_KEY=your_pinecone_api_key
    PINECONE_INDEX_NAME=your_index_name
    ```

  - Note: Pinecone may provide separate keys for different environments (dev/prod). Keep keys secret.

- **WaveflowDB:**

  - Visit the Pinecone console at `https://db.agentanalytics.ai`and sign in or create an account.
  - Create a database and then navigate to "API Endpoints"

- Create a new API key, copy the key value and add it to your `.env` as:

  ```env
  WAVEFLOWDB_API_KEY=your_waveflow_api_key
  BASE_URL=https://your-waveflow-host.example.com
  NAMESPACE=your_namespace(Database)
  USER_ID=your_user_id
  SESSION_ID=your_session_id
  ```

## Typical Workflow

1. Download and prepare dataset:

```powershell
python 1_get_data.py
```

2. Normalize filenames and prepare `processed_data`:

```powershell
python 2_prepare_data_id.py
```

3. (Optional) Upload processed documents to WaveflowDB:

```powershell
python 3_waveflow_upload.py
```

4. (Optional) Create embeddings and upload to Pinecone:

```powershell
python 4_pinecone_upload.py
```

5. Run the evaluation pipeline (parallel):

```powershell
python 5_run_pipeline.py
```

Outputs

- `results/{system}_results_top{k}_hybrid{filter}.csv` — per-system results
- `results/merged_results_top{k}_hybrid{filter}.csv` — merged per-setting
- `results/all_results.xlsx` — raw + aggregated sheets
- `logs/` — execution logs and `prepare_data_output.csv`

## Evaluation Metrics

- Precision, Recall, F1
- MRR (Mean Reciprocal Rank)
- nDCG@k (Normalized Discounted Cumulative Gain)
- Embedding / Query / Total timings
