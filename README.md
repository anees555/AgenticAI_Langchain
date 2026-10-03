
# Agentic AI, LangChain, LangGraph, MCP, and RAG

This repository is a hands-on Python learning workspace for building applications around large language models (LLMs). It is intentionally organized as a collection of related experiments rather than as one production application.

The material moves from basic LangChain model calls to tool-using agents, stateful LangGraph workflows, Model Context Protocol (MCP) servers, conventional vector-based retrieval-augmented generation (RAG), and a structure-aware, vectorless RAG experiment using PageIndex.

## What This Repository Covers

| Area | What is demonstrated | Where to look |
| --- | --- | --- |
| LangChain fundamentals | Agents, model providers, prompts, messages, tools, schemas, and middleware | `langchainLearn/` |
| LangGraph | Explicit state graphs, tool loops, ReAct-style routing, memory, streaming, and human-in-the-loop interrupts | `langgraphLearn/` |
| MCP | A stdio math server, an HTTP weather server, and a LangChain client that exposes both as agent tools | `MCP_langchain/` |
| Vector RAG | Document loading, chunking, Sentence Transformer embeddings, FAISS persistence, retrieval, and LLM summarization | `RAG/src/`, `RAG/app.py` |
| Vectorless RAG | PageIndex tree indexing, LLM-guided node selection, section retrieval, and page-cited answer generation | `RAG/Page_index_vectorless_RAG.ipynb` |
| Data ingestion | LangChain `Document` objects, text and PDF loaders, directory loading, and loader-to-vector-store workflows | `RAG/notebooks/` |

## Repository Layout

```text
.
├── main.py                         # Minimal root smoke example
├── pyproject.toml                  # uv project metadata and dependencies
├── requirements.txt                # pip-style dependency list
├── uv.lock                         # Locked uv dependency resolution
├── langchainLearn/                 # LangChain notebook lessons
│   ├── 1_langchain_intro.ipynb     # Basic agent with a weather tool
│   ├── 2_model_integration.ipynb   # Gemini, Groq, OpenAI-compatible models, streaming, batch
│   ├── 3_tools.ipynb               # Tool definitions and execution loops
│   ├── 4_messages.ipynb            # Text prompts and typed message objects
│   ├── 5_structured_output.ipynb   # Pydantic, TypedDict, and dataclass-style output
│   └── 6_middleware.ipynb          # Summarization and human approval middleware
├── langgraphLearn/
│   └── 1_BasicChatbot/
│       ├── 1_basic_chatbot.ipynb   # Graph API, tools, memory, ReAct, streaming
│       └── 2_human_in_the_loop.ipynb
├── MCP_langchain/
│   ├── client.py                   # Async client for both MCP servers
│   ├── mathserver.py               # add and multiply tools over stdio
│   └── weather.py                  # Mock weather tool over streamable HTTP
└── RAG/
	├── app.py                      # FAISS RAG example
	├── src/                        # Loaders, embeddings, vector store, search
	├── notebooks/                  # Data ingestion and classic RAG walkthroughs
	├── Page_index_vectorless_RAG.ipynb
	├── data/                       # Example text and PDF source documents
	├── faiss_store/                # Persisted FAISS index and metadata
	└── data/vector_store/           # Persisted Chroma database artifacts
```

## Getting Started

### Requirements

- Windows, macOS, or Linux
- Python 3.14, as specified by `.python-version` and `pyproject.toml`
- An LLM provider API key for notebook examples that call hosted models
- `uv` is recommended because the repository includes `uv.lock`; pip can also be used with `requirements.txt`

### Install with uv

From the repository root:

```powershell
uv sync
```

The activation command above assumes the virtual environment is named `.venv`. In PowerShell, the literal command is:

```powershell
.\.venv\Scripts\Activate.ps1
```

Alternatively, run commands through uv without activating the environment:

```powershell
uv run python main.py
```

### Install with pip

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

The `pyproject.toml` dependency list is more complete than `requirements.txt`; it also includes packages used by PageIndex, Chroma, and additional integrations.

### Configure environment variables

Create a local `.env` file in the repository root. Never commit real credentials.

```dotenv
GROQ_API_KEY=your_groq_key
GOOGLE_API_KEY=your_google_key
TAVILY_API_KEY=your_tavily_key
PAGEINDEX_API_KEY=your_pageindex_key
```

These keys are used as follows:

- `GROQ_API_KEY`: Groq-backed LangChain and LangGraph examples, including `openai/gpt-oss-20b`.
- `GOOGLE_API_KEY`: Gemini model and middleware examples.
- `TAVILY_API_KEY`: web-search tools in the LangGraph notebooks.
- `PAGEINDEX_API_KEY`: PDF upload and tree indexing in the PageIndex notebook.

Some notebooks load `.env` with `python-dotenv`, but notebook execution depends on the notebook working directory. Open the repository as a workspace and confirm the kernel can see the root `.env` before running provider-backed cells.

## Recommended Learning Path

### 1. LangChain notebooks

Start with `langchainLearn/1_langchain_intro.ipynb`, which creates a simple agent with a weather function. Continue through:

1. `2_model_integration.ipynb`: initialize Gemini, Groq, and OpenAI-compatible chat models, then compare normal invocation with streaming and batch calls.
2. `3_tools.ipynb`: define callable tools and follow the model-tool execution loop.
3. `4_messages.ipynb`: use text prompts and `SystemMessage`, `HumanMessage`, `AIMessage`, and `ToolMessage` objects.
4. `5_structured_output.ipynb`: request validated Pydantic objects and simpler TypedDict/dataclass-style structures, including nested models.
5. `6_middleware.ipynb`: apply conversation summarization when message or token thresholds are reached, and pause tool execution for human approval, editing, or rejection.

These notebooks are instructional experiments. Their stored outputs include model-generated prose and, in places, provider quota errors; outputs are not treated as deterministic tests.

### 2. LangGraph notebooks

`langgraphLearn/1_BasicChatbot/1_basic_chatbot.ipynb` builds a graph with a typed message state, an LLM node, and `START`/`END` edges. It then extends the graph with:

- `TavilySearch` and a custom multiplication tool
- `ToolNode` and conditional tool routing
- A ReAct-style loop that returns from tools to the LLM
- `MemorySaver` checkpointing with thread IDs
- `stream()` and `astream_events()` examples using `values` and `updates` modes

`2_human_in_the_loop.ipynb` adds a tool that calls `interrupt()`. The graph pauses, a human response is supplied through `Command(resume=...)`, and execution continues from the saved checkpoint.

### 3. MCP server and client

The MCP example separates tools from the agent:

- `mathserver.py` exposes `add` and `multiple` over stdio.
- `weather.py` exposes a deterministic mock `get_weather` tool over streamable HTTP.
- `client.py` loads both through `MultiServerMCPClient`, gives the tools to a LangGraph ReAct agent, and asks it a math question and a weather question.

Run the weather server in one terminal:

```powershell
python MCP_langchain\weather.py
```

The server listens at `http://localhost:8000/mcp`. In another terminal, run:

```powershell
python MCP_langchain\client.py
```

The client starts the math server itself through stdio. The current client contains a Windows absolute path to `mathserver.py`; update that path when running the repository from a different machine or directory.

### 4. Conventional vector RAG

The `RAG/` Python modules implement this pipeline:

```text
files -> LangChain Documents -> recursive chunks -> MiniLM embeddings
	  -> FAISS IndexFlatL2 -> nearest chunks -> Groq summary
```

`RAG/src/data_loader.py` supports PDF, TXT, CSV, XLSX, DOCX, and JSON files. `RAG/src/embedding.py` uses `RecursiveCharacterTextSplitter` with a default chunk size of 500 and overlap of 200, then embeds chunks with `all-MiniLM-L6-v2`. `RAG/src/vectorstore.py` persists the FAISS index and metadata to `RAG/faiss_store/`. `RAG/src/search.py` retrieves the nearest chunks and asks a Groq model to summarize them.

From the `RAG` directory, the intended example command is:

```powershell
cd RAG
python app.py
```

The first build can be run from Python by loading documents and calling `FaissVectorStore.build_from_documents(...)`; later runs can call `load()` and `query(...)`. The sample data includes `machine_learning.txt`, `python_intro.txt`, `anish_daha_data_science_resume.pdf`, and `nec-syllabus.pdf`.

The two ingestion notebooks provide smaller, progressively built examples:

- `RAG/notebooks/document.ipynb` introduces the LangChain `Document`, creates sample text files, and loads text and PDF directories.
- `RAG/notebooks/pdf_loader.ipynb` walks through splitting, embeddings, a vector store, retrieval, and generating an answer from retrieved context.

### 5. PageIndex vectorless RAG

`RAG/Page_index_vectorless_RAG.ipynb` explores an alternative to similarity-first retrieval. Its conceptual flow is:

```text
PDF -> PageIndex hierarchical tree -> LLM selects relevant node IDs
	-> retrieve complete sections -> Groq generates an answer with page citations
```

Unlike the FAISS pipeline, this notebook does not begin by cutting a document into fixed-size chunks and comparing embedding distances. PageIndex builds a tree that preserves document structure; an LLM reasons over section titles and summaries in a table-of-contents-like view, then the pipeline fetches the selected sections and asks the model to cite their titles and page numbers.

The notebook uploads and indexes a PDF through the PageIndex service, inspects and prints the resulting tree, counts nodes, runs an LLM tree search, and completes an end-to-end cited query. It is a cloud-backed experiment and requires both `PAGEINDEX_API_KEY` and `GROQ_API_KEY`.

## Root Smoke Check

The root `main.py` is intentionally minimal and only verifies that the Python project runs:

```powershell
python main.py
# Hello from agenticai-langchain!
```

It is not the entry point for the notebooks, MCP demo, or RAG workflows.

## Important Implementation Notes

- Provider calls incur external API usage, latency, and rate limits. A notebook cell may work once and fail later because of quota, model availability, or provider changes.
- The examples use several current LangChain and LangGraph APIs. Re-check imports if upgrading dependencies beyond the versions locked in `uv.lock`.
- `RAG/src/search.py` currently initializes `ChatGroq` with an empty local `groq_api_key` value rather than explicitly passing the loaded `GROQ_API_KEY`. If the provider does not pick up the environment automatically, this module needs that configuration corrected before the summarization step can run.
- RAG paths are written for execution from inside `RAG` (for example, `data` and `faiss_store`). Running modules from the repository root may require adjusting the working directory or import path.
- The checked-in FAISS and Chroma directories are generated artifacts. Rebuild them when source documents, embedding models, or chunking parameters change.
- The weather tool returns a fixed demonstration response; it is not a live weather integration.
- The notebooks contain exploratory cells, generated outputs, and some examples that depend on optional provider features. Treat them as guided lessons rather than a tested package API.

## Technology Stack

- Python 3.14
- LangChain and LangChain Community integrations
- LangGraph for explicit stateful workflows
- Groq, Google Gemini, and OpenAI-compatible model integrations
- Tavily web search tools
- Model Context Protocol (`mcp`) and `langchain-mcp-adapters`
- Sentence Transformers and FAISS for vector retrieval
- Chroma artifacts for notebook-based vector-store work
- PageIndex for structure-aware, vectorless document retrieval
- Jupyter/IPykernel for interactive exploration

## Next Steps

Natural extensions for this workspace would be to add automated tests around document loading and FAISS persistence, replace hard-coded paths with configuration, pass API keys consistently through environment variables, add citations and source metadata to the vector RAG pipeline, and separate experimental notebooks from production-oriented modules.
