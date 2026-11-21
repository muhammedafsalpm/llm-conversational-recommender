# 🚀 CRS Backend - FastAPI API

This is the backend for the LLM Conversational Recommender: a FastAPI service that recommends movies with few-shot prompting, RAG or an agent approach, using OpenAI or a local model through Ollama.

## 📁 Project Structure

```text
backend/crs_api/
├── app/
│   ├── main.py             # API routes and startup
│   ├── config.py           # Settings (read from .env)
│   ├── models.py           # Pydantic schemas
│   ├── middleware.py       # Rate limiting and request logging
│   ├── exceptions.py       # Error types
│   ├── services/           # few_shot.py, rag.py, agent.py, llm_client.py
│   ├── utils/              # Dataset loader and prompts
│   └── vector_store/       # FAISS index implementation
├── .env.example            # Copy to .env
├── requirements.txt        # Dependencies
├── run.py                  # Starts the API
└── test_concurrency.py     # Concurrent load test
```

## 🛠️ Setup & Running

1. **Install Dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

2. **Configuration**:
   ```bash
   cp .env.example .env
   ```
   Choose `LLM_PROVIDER` (`openai` or `ollama`). `DATA_PATH` is already set to the dataset at `../../other/LLM_Redial/Movie`.

3. **Launch Server**:
   ```bash
   python run.py
   ```

The API documentation will be available at `http://localhost:8000/docs`.

---
**For full system setup including the UI, please refer to the [Root README.md](../../README.md).**
