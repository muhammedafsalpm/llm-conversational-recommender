# 🎬 LLM Conversational Recommender

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.104.1-green.svg)](https://fastapi.tiangolo.com/)
[![React](https://img.shields.io/badge/React-Vite-61dafb.svg)](https://vite.dev/)
[![Ollama](https://img.shields.io/badge/Ollama-Supported-orange.svg)](https://ollama.com/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

An LLM-based conversational recommender system (CRS) for movies. A user chats about what they like, and the system recommends movies using one of three approaches: **few-shot prompting**, **RAG over similar past conversations**, or an **agent that picks from the user's personalised candidates**. Built on the [LLM-REDIAL](https://github.com/LitGreenhand/LLM-Redial) dataset (Findings of ACL 2024), with a FastAPI backend, streaming responses and a React chat UI.

## ✨ Features

- **🤖 Three Recommendation Approaches**
  - **Few-Shot**: puts example conversations from the dataset into the prompt (in-context learning)
  - **RAG**: embeds the conversation, retrieves the most similar dataset conversations with FAISS, and gives them to the LLM as context
  - **Agent**: gives the LLM the user's "might like" candidate list (or a slice of the catalogue for unknown users) as its tool/database, and keeps only recommendations that exist in that list

- **👤 Personalisation**: pass a dataset `user_id` to use that user's watch history and candidates

- **⚡ Async & Streaming**
  - Fully asynchronous endpoints
  - Server-Sent Events (SSE) streaming for real-time output
  - FAISS vector index (built once, then loaded from disk)

- **🔌 Multiple LLM Providers**
  - **OpenAI** (cloud)
  - **Ollama** (local, free, private)
  - **Mock LLM** for testing without any model (`USE_MOCK_LLM=true`)

- **🛡️ API Hardening**
  - Per-IP rate limiting (in memory, per minute)
  - Request validation with Pydantic
  - Error handling with graceful fallbacks
  - Health check and metrics endpoints

## 📋 Table of Contents

- [Prerequisites](#-prerequisites)
- [Project Structure](#-project-structure)
- [Installation & Setup](#️-installation--setup)
- [Configuration](#️-configuration)
- [Running the System](#-running-the-system)
- [API Endpoints](#-api-endpoints)
- [Testing](#-testing)
- [Demo Walkthrough](#-demo-walkthrough)
- [License](#️-license)

---

## 🚀 Prerequisites

- **Python** 3.10 or higher
- **Node.js** 20.19+ and npm (for the UI; required by Vite 8)
- **Memory**: 8 GB RAM minimum (16 GB recommended for local LLMs)
- **One LLM provider**:
  - **Ollama** (local, free): install from [ollama.com](https://ollama.com) and pull a model, e.g. `ollama pull mistral`
  - **OpenAI** (cloud, paid): an API key

### Dataset

The [LLM-REDIAL](https://github.com/LitGreenhand/LLM-Redial) dataset is included in `other/LLM_Redial/` (Movie, Books, Electronics and Sports domains). The app uses the **Movie** domain. Paper: *LLM-REDIAL: A Large-Scale Dataset for Conversational Recommender Systems Created from User Behaviors with LLMs*, Findings of ACL 2024 (`other/2024.findings-acl.529.pdf`). The dataset belongs to its authors; see their repository for its terms.

---

## 📁 Project Structure

```text
llm-conversational-recommender/
├── backend/
│   └── crs_api/                 # FastAPI backend
│       ├── app/
│       │   ├── main.py          # API routes and startup
│       │   ├── config.py        # Settings (read from .env)
│       │   ├── models.py        # Request/response schemas
│       │   ├── middleware.py    # Rate limiting and request logging
│       │   ├── services/        # few_shot.py, rag.py, agent.py, llm_client.py
│       │   ├── utils/           # Dataset loader and prompts
│       │   └── vector_store/    # FAISS index
│       ├── .env.example         # Copy to .env
│       ├── requirements.txt
│       ├── run.py               # Starts the API
│       └── test_concurrency.py  # Concurrent load test
├── frontend/                    # React (Vite) chat UI
│   ├── src/
│   └── package.json
├── other/
│   ├── LLM_Redial/              # Dataset (Movie domain + prebuilt FAISS index in Movie/index/)
│   └── 2024.findings-acl.529.pdf  # Dataset paper
└── LICENSE
```

---

## 🛠️ Installation & Setup

### 1. Clone
```bash
git clone https://github.com/muhammedafsalpm/llm-conversational-recommender.git
cd llm-conversational-recommender
```

### 2. Backend
```bash
cd backend/crs_api
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env      # On Windows: copy .env.example .env
```

Then edit `.env` and choose your LLM provider (see [Configuration](#️-configuration)).

### 3. Frontend
```bash
cd frontend
npm install
```

---

## ⚙️ Configuration

The backend reads `backend/crs_api/.env`. The most important settings:

| Variable | Default | Description |
|---|---|---|
| `LLM_PROVIDER` | `openai` | `openai` or `ollama` |
| `USE_MOCK_LLM` | `false` | `true` returns mock recommendations without calling any LLM |
| `OPENAI_API_KEY` | — | Required when `LLM_PROVIDER=openai` |
| `OPENAI_MODEL` | `gpt-3.5-turbo` | OpenAI model |
| `OLLAMA_MODEL` | `mistral` | Ollama model (e.g. `mistral`, `llama2`, `phi`) |
| `OLLAMA_URL` | `http://localhost:11434` | Ollama server address |
| `DATA_PATH` | `../../other/LLM_Redial/Movie` | Movie dataset folder, relative to `backend/crs_api` |
| `EMBEDDING_MODEL` | `all-MiniLM-L6-v2` | Sentence-Transformers model for RAG |
| `RETRIEVAL_K` | `5` | Conversations retrieved for RAG |
| `RATE_LIMIT_PER_MINUTE` | `60` | Requests per minute per IP |

`.env` is git-ignored, so your API key is never committed.

The frontend calls `http://localhost:8000` by default. To use another backend address, create `frontend/.env` with `VITE_API_URL=http://your-host:8000`.

---

## 🚀 Running the System

### 1. Start your LLM (Ollama only)
```bash
ollama run mistral
```

### 2. Start the backend
From `backend/crs_api`:
```bash
python run.py
```
The API runs at `http://localhost:8000`, with interactive docs at `http://localhost:8000/docs`. On first start the RAG index is built from the dataset; afterwards it is loaded from `other/LLM_Redial/Movie/index/`.

### 3. Start the frontend
From `frontend`:
```bash
npm run dev
```
The UI runs at `http://localhost:5173`.

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Health check |
| `POST` | `/api/v1/recommend` | Get recommendations |
| `POST` | `/api/v1/recommend/stream` | Same, streamed as Server-Sent Events |
| `GET` | `/api/v1/user/{user_id}` | A dataset user's history |
| `GET` | `/api/v1/movies/search?q=...&limit=10` | Search movie titles |
| `GET` | `/api/v1/metrics` | Dataset and system statistics |

Example request:

```bash
curl -X POST http://localhost:8000/api/v1/recommend \
  -H "Content-Type: application/json" \
  -d '{
    "conversation_history": [
      {"role": "user", "content": "I loved Inception. Any similar mind-bending movies?"}
    ],
    "recommender_type": "rag",
    "num_recommendations": 5,
    "user_id": "A30Q8X8B1S3GGT"
  }'
```

- `recommender_type`: `few_shot`, `rag` (default) or `agent`
- `num_recommendations`: 1–20
- `user_id`: optional, a user ID from the dataset for personalised results

---

## 🧪 Testing

Try the API without any LLM by setting `USE_MOCK_LLM=true` in `.env`.

Load-test concurrent requests while the backend is running:

```bash
cd backend/crs_api
python test_concurrency.py
```

---

## 🎯 Demo Walkthrough

1. **Health Check**: view the status indicator in the header.
2. **Metrics**: see dataset statistics in the sidebar.
3. **Personalisation**: enter User ID `A30Q8X8B1S3GGT` in the sidebar to load that user's movie history.
4. **Recommendation Modes**: switch between **Few-Shot**, **RAG** and **Agent**.
5. **Streaming Output**: watch the response appear in real time in the chat window.

---

## 🛡️ License

The code is released under the [MIT License](LICENSE). The LLM-REDIAL dataset in `other/LLM_Redial/` and the paper PDF belong to their authors.
