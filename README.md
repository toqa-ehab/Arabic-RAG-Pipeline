# مُجيب | Mujeeb — Arabic RAG Question Answering

An Arabic Retrieval-Augmented Generation (RAG) system. Ask a question in Arabic, and the system retrieves the most relevant passages from a PDF document and uses an LLM to generate an answer grounded in them. The model runs in a Kaggle notebook, is served through a FastAPI backend, and is exposed to a web UI through an ngrok tunnel.

## Features

- Arabic-first retrieval using an Arabic embedding model (GATE-AraBERT)
- Fast semantic search with FAISS
- Answer generation with Mistral-Nemo-Instruct
- Shows the retrieved chunks next to each answer, so you can verify the source
- Responsive RTL web interface with dark/light themes
- Question history, copy answer, and text-to-speech (read aloud)

## How it works

```
PDF ──► text extraction ──► chunking ──► embeddings ──► FAISS index
                                                            │
User question ──► embed query ──► top-k similar chunks ◄────┘
                                        │
                        prompt (question + chunks)
                                        │
                              Mistral-Nemo-Instruct
                                        │
                           answer + source chunks ──► Web UI
```

1. **Extract**: text is read from the PDF with PyPDF2.
2. **Chunk**: the text is split into overlapping word windows (50 words, 5 overlap in the current notebook).
3. **Embed**: each chunk is encoded with `Omartificial-Intelligence-Space/GATE-AraBert-v0` (normalized vectors).
4. **Index**: vectors are stored in a FAISS `IndexFlatIP` (inner product = cosine similarity on normalized vectors).
5. **Retrieve**: the question is embedded and the top 3 chunks are returned.
6. **Generate**: the chunks and the question are placed in an Arabic prompt and sent to `mistralai/Mistral-Nemo-Instruct-2407`.
7. **Serve**: a FastAPI `/ask` endpoint returns the answer and the chunks to the UI.

## Tech stack

| Layer | Tools |
|---|---|
| LLM | Mistral-Nemo-Instruct-2407 (Hugging Face Transformers, PyTorch) |
| Embeddings | GATE-AraBERT-v0 (Sentence-Transformers) |
| Vector search | FAISS (CPU) |
| PDF parsing | PyPDF2 |
| Backend | FastAPI, Uvicorn |
| Tunnel | ngrok (pyngrok) |
| Frontend | HTML, Tailwind CSS, vanilla JavaScript |
| Environment | Kaggle Notebook (GPU) |

## Repository structure

```
.
├── notebook.ipynb   # RAG pipeline + FastAPI server (run on Kaggle)
├── index.html       # Web UI
└── README.md
```

## Getting started

### Requirements

- A Kaggle account with GPU enabled and **Internet turned on** (Notebook settings)
- A free [ngrok](https://ngrok.com) account and authtoken
- A PDF to query (the example uses an Arabic machine learning document)

### 1. Prepare the notebook

1. Upload `notebook.ipynb` to Kaggle.
2. Upload your PDF as a Kaggle dataset and set `path` in the notebook to its location.
3. In Kaggle, go to **Add-ons → Secrets** and add a secret named `NGROK_TOKEN` with your ngrok authtoken.

### 2. Run the cells in order

Run the whole pipeline **in the same session and in order**:

1. Install dependencies and load the LLM
2. Define `generate_text`
3. Load the embedding model and define the helper functions (`text_from_pdf`, `chunk_text`, `embed_chunks`, `faiss_index`, `search_index`)
4. Build the chunks, embeddings and FAISS index
5. Define the FastAPI app and start the server in a background thread
6. Open the ngrok tunnel and copy the printed public URL

> The API reads `embedding_model`, `index`, `chunks`, `search_index` and `generate_text` from the notebook's memory, so all earlier cells must have been run first.

### 3. Connect the UI

Open `index.html` and set your ngrok URL in the fetch call:

```javascript
const API_URL = "https://xxxx.ngrok-free.app"; // your ngrok URL

const response = await fetch(`${API_URL}/ask`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "ngrok-skip-browser-warning": "true"
  },
  body: JSON.stringify({ question }),
});
```

Then open `index.html` in your browser and ask a question.

## API

### `POST /ask`

**Request**

```json
{ "question": "ما هو التعلم الآلي؟" }
```

**Response**

```json
{
  "answer": "…generated answer…",
  "chunks": ["retrieved chunk 1", "retrieved chunk 2", "retrieved chunk 3"]
}
```

Interactive docs are available at `/docs` while the server is running.

## Known limitations

- The ngrok URL changes every time the notebook session restarts, so `API_URL` must be updated.
- The API only runs while the Kaggle session is active.
- Generation with a 12B model is slow; expect several seconds per answer.
- Some UI elements are placeholders: the voice input button is a simulation, and the "response time" and "RAG v2.4" labels in the UI are static text.
- Chunks are split by word count, not by sentence or paragraph, so retrieved passages may begin or end mid-sentence.

## Roadmap

- Support uploading PDFs from the UI
- Sentence-aware chunking
- Use `max_new_tokens` and streaming responses for faster perceived speed
- Add source page numbers to the retrieved chunks
- Deploy the backend to a persistent host instead of a notebook
- Evaluate retrieval quality (e.g., hit rate on a labelled question set)

## Acknowledgements

- [Mistral AI](https://mistral.ai) for Mistral-Nemo-Instruct
- [Omartificial Intelligence Space](https://huggingface.co/Omartificial-Intelligence-Space) for GATE-AraBERT
- [FAISS](https://github.com/facebookresearch/faiss), [FastAPI](https://fastapi.tiangolo.com), and [Sentence-Transformers](https://www.sbert.net)

## Author

Your Name — [GitHub](https://github.com/your-username)

## License

Add a license of your choice (for example MIT). Note that the models used have their own licenses; check them before commercial use.
