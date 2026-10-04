# Arabic-RAG-Pipeline 🚀

An end-to-end Arabic Retrieval-Augmented Generation (RAG) pipeline powered by **Mistral-Nemo**, **AraBert embeddings**, **FAISS**, and **FastAPI**. This system extracts information from Arabic documents (e.g., machine learning PDFs) to answer domain-specific questions accurately with context retrieval.

---

## 🌟 Key Features

- **Document Ingestion**: Extracts text from PDF files using `PyPDF2`.
- **Text Chunking & Vectorization**: Uses `GATE-AraBert-v0` for Arabic text embeddings.
- **Fast Similarity Search**: Vector retrieval powered by `FAISS` (Inner Product / Cosine Similarity).
- **Context-Aware Generation**: Text generation via `Mistral-Nemo-Instruct-2407`.
- **RESTful API**: Served via **FastAPI** with CORS enabled for frontend/UI integration.
- **Remote Tunneling**: Tunnel support via `ngrok` for external frontend integration during testing.

---

## 🏗️ Architecture Flow

1. **Upload PDF** ➡️ Extract text & split into overlapping chunks.
2. **Generate Embeddings** ➡️ Encode text chunks using `Omartificial-Intelligence-Space/GATE-AraBert-v0`.
3. **Index Vectors** ➡️ Store embeddings in a FAISS vector index.
4. **User Inquiry** ➡️ Search vector index for top-$k$ relevant text passages.
5. **Prompt Assembly** ➡️ Combine context chunks with user question.
6. **LLM Generation** ➡️ `Mistral-Nemo-Instruct` outputs exact Arabic answer.

---

## 🚀 Quick Start

### 1. Prerequisites

- Python 3.10+
- GPU with CUDA support recommended (due to model size)

### 2. Installation

Clone the repository and install required packages:

```bash
git clone [https://github.com/your-username/arabic-rag-pipeline.git](https://github.com/your-username/arabic-rag-pipeline.git)
cd arabic-rag-pipeline
pip install -r requirements.txt
```

### 3. Run the FastAPI Application

Start the backend server:

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

---

## 📡 API Usage

### Endpoint: `POST /ask`

#### Request Body
```json
{
  "question": "ما هو التعلم الآلي؟"
}
```

#### Response Example
```json
{
  "answer": "التعلم الآلي هو فرع من فروع الذكاء الاصطناعي يسمح للحاسوب بالتعلم من البيانات واكتشاف الأنماط...",
  "retrieved_chunks": [
    "الذكاء الاصطناعي هو مجال من علوم الحاسوب يهدف إلى...",
    "التعلم الآلي هو أحد فروع الذكاء الاصطناعي..."
  ]
}
```

---

## 🧪 Testing with cURL

```bash
curl -X POST "[http://127.0.0.1:8000/ask](http://127.0.0.1:8000/ask)" \
     -H "Content-Type: application/json" \
     -d '{"question": "ما هي Embeddings؟"}'
```

---

## 🛠️ Models & Tools Used

- **LLM**: [`mistralai/Mistral-Nemo-Instruct-2407`](https://huggingface.co/mistralai/Mistral-Nemo-Instruct-2407)
- **Embedding Model**: [`Omartificial-Intelligence-Space/GATE-AraBert-v0`](https://huggingface.co/Omartificial-Intelligence-Space/GATE-AraBert-v0)
- **Vector Search**: [FAISS](https://github.com/facebookresearch/faiss)
- **API Framework**: [FastAPI](https://fastapi.tiangolo.com/)
