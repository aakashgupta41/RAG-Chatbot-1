# Resume RAG Chatbot

Chat with your resume using a fully local RAG pipeline. No OpenAI, no paid APIs, no cloud services.

<img width="800" height="450" alt="aakash - rag - gif" src="https://github.com/user-attachments/assets/9b8bba5a-c6ba-446f-835d-299d9ed6af5c" />

## Description

This project lets you ask natural-language questions about a resume PDF, such as "What internships has he done?" or "Which AI frameworks does he know?", and get answers grounded only in the resume's content.

It is a Retrieval-Augmented Generation (RAG) pipeline. The resume is split into chunks and converted to vector embeddings, which are stored in ChromaDB. A question is embedded the same way and compared with the stored vectors using cosine similarity. The 3 most relevant chunks are passed to a local Mistral model, which writes the answer using only that context. Everything runs on your own machine.

## Features

- Runs 100% locally: no API keys and no internet needed after setup
- Works with any text-based PDF
- Cosine-similarity semantic search with Top-K = 3
- Answers are limited to the resume context, so the model says "I couldn't find that in the resume" for missing information
- Shows source chunks and page numbers for every answer
- Notebook-based, so each pipeline step can be run, inspected and fixed on its own
- Includes matplotlib visualisations of the query and the top 3 chunks

## How it works

```
INGESTION (ingest.ipynb, run once)
PDF -> PyPDFLoader -> Documents -> RecursiveCharacterTextSplitter (500 / 80)
    -> Nomic Embed Text (via Ollama) -> ChromaDB (cosine, saved on disk)

QUERY + GENERATION (chat.ipynb, every question)
Question -> Nomic embedding -> cosine search in ChromaDB -> top 3 chunks
         -> prompt (context + question) -> Mistral (via Ollama) -> answer + sources
```

## Tech stack

| Component | Tool |
|---|---|
| Language | Python 3.12 |
| Package manager | uv |
| Framework | LangChain |
| LLM | Mistral (via Ollama) |
| Embeddings | Nomic Embed Text (via Ollama) |
| Vector database | ChromaDB (cosine similarity) |
| PDF parsing | PyPDF |
| Text splitting | RecursiveCharacterTextSplitter (chunk_size=500, chunk_overlap=80) |
| Visualisation | matplotlib |

## Prerequisites

- Python 3.12
- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- [Ollama](https://ollama.com)
- About 8 GB of RAM (Mistral 7B) and about 5 GB of free disk space for the models
- VS Code with the Python and Jupyter extensions (or JupyterLab)

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/resume-rag-chatbot.git
cd resume-rag-chatbot
```

### 2. Install the Ollama models

Make sure Ollama is running, then:

```bash
ollama pull mistral
ollama pull nomic-embed-text
ollama list
```

Both models should appear in the list.

### 3. Create the virtual environment and install packages

```bash
uv venv
```

Activate it:

```bash
# Windows (PowerShell)
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate
```

Install the packages:

```bash
uv pip install -r requirements.txt
```

### 4. Add your resume

Copy your PDF into the project folder (next to the notebooks) and name it `resume.pdf`. It must contain selectable text, not a scanned image.

### 5. Run the notebooks

Open the folder in VS Code (`code .`). In each notebook, click **Select Kernel**, then **Python Environments**, and choose the `.venv` of this project.

1. **`ingest.ipynb`**: run all cells from top to bottom. This loads the PDF, splits it, embeds the chunks and stores them in `chroma_db/`. Run it once, and again whenever the PDF changes.
2. **`chat.ipynb`**: restart the kernel, then run all cells from top to bottom. Change the `question` variable and re-run to ask something else, or use the `ask()` function in the last cells.

## Using your own PDF

1. Put your PDF in the project folder.
2. In `ingest.ipynb`, Cell 2, set the file name:
   ```python
   PDF_PATH = BASE_DIR / "your-file-name.pdf"
   ```
3. In `chat.ipynb`, edit `SYSTEM_PROMPT` and replace "Aakash's resume" with the right name or document type.
4. Re-run `ingest.ipynb` completely, then restart and re-run `chat.ipynb`.

## Configuration

| Setting | Where | Default | Meaning |
|---|---|---|---|
| `PDF_PATH` | ingest | `resume.pdf` | The document to index |
| `CHUNK_SIZE` | ingest | 500 | Maximum characters per chunk |
| `CHUNK_OVERLAP` | ingest | 80 | Characters repeated between neighbouring chunks |
| `COLLECTION_NAME` | both | `resume` | ChromaDB collection (must match in both notebooks) |
| `EMBED_MODEL` | both | `nomic-embed-text` | Embedding model (must match in both notebooks) |
| `LLM_MODEL` | chat | `mistral` | The chat model |
| `TOP_K` | chat | 3 | Number of chunks retrieved per question |
| `temperature` | chat | 0 | 0 gives factual, repeatable answers |

If you change the chunk settings, the PDF or the embedding model, re-run `ingest.ipynb`.

## Project structure

```
resume-rag-chatbot/
├── ingest.ipynb        # load -> clean -> split -> embed -> store
├── chat.ipynb          # query -> embed -> retrieve -> prompt -> Mistral
├── requirements.txt
├── pyproject.toml
├── assets/             # demo GIF and screenshots
└── chroma_db/          # generated by ingest.ipynb (not in Git)
```

## Troubleshooting

| Problem | Fix |
|---|---|
| `Connection refused` on port 11434 | Ollama isn't running. Start the Ollama app or run `ollama serve` |
| `model "mistral" not found` | Run `ollama pull mistral` |
| `PDF exists: False` | Check the file name in `PDF_PATH` and that the notebook runs from the project folder |
| No text extracted from the PDF | The PDF is a scanned image. Use a text-based PDF |
| `Chunks in database: 0` | Run `ingest.ipynb` first |
| `ModuleNotFoundError` | The notebook isn't using the project's `.venv`. Re-select the kernel |
| Very slow answers | Normal on CPU, especially the first question while Mistral loads |

## Possible improvements

- Wrap `ask()` in a FastAPI or Flask endpoint to use it in a website
- Add MMR retrieval or hybrid search (BM25 + dense)
- Add chat history for follow-up questions

## License

MIT
