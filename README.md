# 📄 Document Summarizer

A command-line tool that summarizes long **PDF** and **TXT** documents using large language models (LLMs). It splits a document into token-based chunks and picks a summarization strategy based on the document's size — **Stuff**, **Map-Reduce**, or **Refine**.

You can choose from several LLM providers (Gemini, MiniMax, GLM, Qwen) through one OpenAI-compatible client.

---

## ✨ Features

- **PDF and TXT support**: extracts text page by page with `pypdf`
- **Token-aware chunking**: splits text with `tiktoken` (`cl100k_base`) into 1000-token chunks with 100-token overlap
- **Three summarization strategies**
  - **Stuff**: for small documents (a single chunk), summarized in one call
  - **Map-Reduce**: summarizes every chunk in parallel (async), then combines the summaries into one final summary
  - **Refine**: summarizes the first chunk, then updates a running summary chunk by chunk
- **Multiple LLM providers**: switch models at runtime from a simple menu
- **Config-driven models**: add or edit models in `llm_config.toml` without touching the code
- **Clean terminal UI** built with `rich`

---

## 🧠 How It Works

```
 PDF / TXT file
       │
       ▼
  loader.py        → extract text, count tokens, save a copy in files/
       │
       ▼
  chunker.py       → split into 1000-token chunks (100 overlap)
       │
       ▼
  summarizer.py    → choose model + strategy
       │
       ▼
  strategies.py    → Stuff | Map-Reduce | Refine  → final summary
```

### Strategy selection

| Document size | Strategy | What happens |
|---|---|---|
| 1 chunk | **Stuff** (automatic) | The whole text goes to the model in a single call |
| More than 1 chunk | **Map-Reduce** (option 1) | All chunks are summarized in parallel, then the summaries are merged into one |
| More than 1 chunk | **Refine** (option 2) | Chunks are processed one after another, extending a running summary |

**Which one to choose?** Map-Reduce is faster because chunks are processed in parallel. Refine is slower but keeps the flow of the document in order.

---

## 📁 Project Structure

```
Project_3_document_summarize/
├── summarizer.py      # Entry point: menus, model and strategy selection
├── loader.py          # Reads PDF/TXT, extracts text, counts tokens
├── chunker.py         # Token-based chunking with overlap
├── strategies.py      # Stuff, Map-Reduce and Refine implementations
├── llm_config.toml    # Model/provider configuration
├── files/             # Extracted text copies are saved here
├── .vscode/           # Editor settings
└── .gitignore
```

---

## 🚀 Getting Started

### Prerequisites

- **Python 3.11 or higher** (the project uses the built-in `tomllib` module)
- An API key for at least one supported provider

### 1. Clone the repository

```bash
git clone https://github.com/satyam-sharma7/Project_3_document_summarize.git
cd Project_3_document_summarize
```

### 2. Create a virtual environment (recommended)

```bash
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate
```

### 3. Install dependencies

```bash
pip install rich python-dotenv pypdf tiktoken openai
```

### 4. Add your API keys

Create a `.env` file in the project root. You only need the key for the model you plan to use:

```env
OPENROUTER_API_KEY=your_openrouter_key      # MiniMax
GOOGLE_AI_STUDIO=your_google_ai_studio_key  # Gemini
HF_TOKEN=your_huggingface_token             # GLM
HF_TOKEN_2=your_huggingface_token           # Qwen
```

> ⚠️ Never commit your `.env` file. Make sure it is listed in `.gitignore`.

### 5. Make sure the `files/` folder exists

```bash
mkdir -p files
```

---

## ▶️ Usage

```bash
python summarizer.py
```

You will then be guided step by step:

1. **Enter the file path** of your `.pdf` or `.txt` document
2. **Choose a model** from the menu
3. The tool chunks the document and shows how many chunks were created
4. For large documents, **choose a strategy** (Map-Reduce or Refine)
5. The final summary is printed in the terminal

### Example session

```text
╭──────────────────────────────────────╮
│ Welcome to our Document summarizer   │
╰──────────────────────────────────────╯
Provide the path of your file: report.pdf

Your file contains 5400 tokens.
File extraction successful.

Choose Model:
 1. Gemini
 2. Minmax
 3. Glm
 4. Qwen
Enter Number [1]: 1

Chunking...
6 Chunks Created.

Your document is large, choose one method to summarize:
1. Map Reduce
2. Refine
Enter Number [1]: 1

Bot: <final summary appears here>
```

---

## ⚙️ Configuration

Models are defined in `llm_config.toml`. Each model needs a provider name, an OpenAI-compatible base URL, the name of the environment variable that holds its API key, and the model ID.

| Menu option | Provider | Endpoint | Env variable |
|---|---|---|---|
| 1 | Gemini | Google AI Studio | `GOOGLE_AI_STUDIO` |
| 2 | MiniMax | OpenRouter | `OPENROUTER_API_KEY` |
| 3 | GLM | Hugging Face Router | `HF_TOKEN` |
| 4 | Qwen | Hugging Face Router | `HF_TOKEN_2` |

To add a new model, add a block like this:

```toml
[models.my_model]
provider = "MyProvider"
url = "https://example.com/v1"
api = "MY_API_KEY_ENV_NAME"
name = "model-id"
temperature = 0.3
max_tokens = 500
```

Then register it in the `choose_model()` menu in `summarizer.py`.

### Tunable settings

| Setting | Location | Default |
|---|---|---|
| Chunk size | `chunker.py` → `chunk_size` | 1000 tokens |
| Chunk overlap | `chunker.py` → `overlap` | 100 tokens |
| Summary length | `strategies.py` → `summarize_chunk` | ~800 tokens (`max_tokens=900`) |

---

## 🛠️ Tech Stack

| Purpose | Library |
|---|---|
| PDF text extraction | `pypdf` |
| Tokenization and chunking | `tiktoken` |
| LLM API client | `openai` (AsyncOpenAI) |
| Parallel requests | `asyncio` |
| Terminal interface | `rich` |
| Environment variables | `python-dotenv` |
| Config parsing | `tomllib` (standard library) |

---

## ⚠️ Known Limitations

- Only `.pdf` and `.txt` files are supported. Scanned PDFs (images without a text layer) will not extract any text.
- The tool is terminal-only; there is no web interface yet.
- Free-tier models may be rate-limited or slow on long documents.

---

## 🗺️ Roadmap

- [ ] Add a `requirements.txt`
- [ ] Support `.docx` files
- [ ] Let the user set the summary length
- [ ] Save the final summary to a file
- [ ] Add a simple web UI

---

## 🤝 Contributing

Contributions, issues and feature requests are welcome. Feel free to open an issue or submit a pull request.

---

## 👤 Author

**Satyam Sharma**
GitHub: [@satyam-sharma7](https://github.com/satyam-sharma7)
