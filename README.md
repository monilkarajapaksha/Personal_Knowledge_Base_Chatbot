# Personal Knowledge Base RAG Chatbot

A Retrieval-Augmented Generation (RAG) system that answers questions grounded in your own documents — notes, CVs, study material — instead of relying on an LLM's memorized training data. Every answer includes the source file it was pulled from, and the system explicitly says "I don't know" rather than hallucinating when the answer isn't in the provided documents.

## Why this project

Standard LLMs hallucinate confidently when they don't actually know something, and have no knowledge of private/personal documents. This project solves both problems by retrieving relevant text *before* generation, so answers are grounded in real, citable source material.

## Architecture
```

flowchart TD
    A[Your Documents<br/>.txt / .md / .pdf] --> B[Chunking<br/>RecursiveCharacterTextSplitter]
    B --> C[Embedding<br/>sentence-transformers - local, free]
    C --> D[(ChromaDB<br/>Vector Store)]

    E[User Query] --> F[Retriever<br/>top-k similarity search]
    D --> F
    F --> G[Prompt Construction<br/>query + retrieved context]
    G --> H[LLM Generation<br/>Groq - Llama 3.1]
    H --> I[Answer + Cited Sources]

    style A fill:#EAF0FB,stroke:#2E5EAA,color:#1B2A4A
    style B fill:#EAF0FB,stroke:#2E5EAA,color:#1B2A4A
    style C fill:#EAF0FB,stroke:#2E5EAA,color:#1B2A4A
    style D fill:#FDEBD0,stroke:#B9770E,color:#1B2A4A
    style E fill:#EAF0FB,stroke:#2E5EAA,color:#1B2A4A
    style F fill:#EAF0FB,stroke:#2E5EAA,color:#1B2A4A
    style G fill:#EAF0FB,stroke:#2E5EAA,color:#1B2A4A
    style H fill:#D5F5E3,stroke:#1E8449,color:#1B2A4A
    style I fill:#D5F5E3,stroke:#1E8449,color:#1B2A4A
```

## Tech Stack

| Component | Tool |
|---|---|
| Orchestration | LangChain |
| Embeddings | sentence-transformers (`all-MiniLM-L6-v2`) — local, no API cost |
| Vector database | ChromaDB (local, persistent) |
| LLM | Groq API (Llama 3.1 8B) — free tier |
| Runtime | Google Colab / local Python |

## Setup

### 1. Clone and install
```bash
git clone https://github.com/<your-username>/personal-kb-rag-chatbot.git
cd personal-kb-rag-chatbot
pip install -r requirements.txt
```

### 2. Add your API key
Get a free key at [console.groq.com](https://console.groq.com), then copy the example env file:
```bash
cp .env.example .env
```
Edit `.env` and paste your key:
```
GROQ_API_KEY=your_key_here
```
> **Never commit your real `.env` file** — it's already excluded via `.gitignore`.

### 3. Add your documents
Place your `.txt`, `.md`, or `.pdf` files in the `data/` folder.

### 4. Run
```bash
python main.py
```
Or open `notebook.ipynb` in Google Colab / Jupyter and run the cells top to bottom.

## Example

```
> Ask: What is my educational background?

Answer: You hold a B.Sc. (Hons) in Computer Engineering from the University
of Ruhuna (2021-2025), and are ISTQB certified (CTFL 2024, CT-PT 2025).

Sources: cv_monilka.txt
```

```
> Ask: What is the capital of France?

Answer: I don't know — this isn't covered in the provided documents.

Sources: (none — correctly refused rather than hallucinating)
```

## Evaluation

A small manually-checked test set to track retrieval and answer quality as the system changes:

| # | Question | Expected | Result |
|---|---|---|---|
| 1 | *(answerable question from your docs)* | *(expected answer)* | ✅ / ❌ |
| 2 | *(answerable question from your docs)* | *(expected answer)* | ✅ / ❌ |
| 3 | *(answerable question from your docs)* | *(expected answer)* | ✅ / ❌ |
| 4 | *(out-of-scope question)* | Should say "I don't know" | ✅ / ❌ |
| 5 | *(out-of-scope question)* | Should say "I don't know" | ✅ / ❌ |

*(Fill this in with your own real test cases and results — it's the difference between "a RAG demo" and "a RAG system I tested".)*

## What I'd improve next

- Add hybrid search (BM25 + dense retrieval) for better exact-match handling (product names, dates, acronyms)
- Add a cross-encoder reranker to improve precision on the top-k results
- Automate the evaluation table above using [RAGAS](https://github.com/explodinggpt/ragas) instead of manual checking

## Tools Used

Built with LangChain, ChromaDB, sentence-transformers, and the Groq API.
