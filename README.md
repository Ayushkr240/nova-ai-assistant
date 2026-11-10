# 🤖 NOVA-AI : Voice Assistant & Productivity Copilot

A full-stack conversational AI assistant built with **React, FastAPI, LangGraph, LangChain, Groq, RAG, SQLite, and MCP**.

The assistant supports persistent multi-threaded conversations, PDF-based question answering, voice input, and productivity actions such as email and Google Calendar management. The React frontend communicates with the Python AI backend through FastAPI, while LangGraph orchestrates retrieval, reasoning, guardrails, and MCP tool execution.

---

## ✨ Features

### 💬 Conversational AI

- Natural multi-turn conversations
- Persistent chat threads
- Create, rename, delete, and resume conversations
- Automatic conversation titles
- SQLite-backed conversation persistence

### ⚛️ React Frontend

- Interactive chat interface built with React and Vite
- Communicates with the backend through FastAPI
- Supports text prompts, PDF uploads, and voice recordings
- Chat-thread management and conversation history

### 🎤 Voice Input

- Record voice directly from the React interface
- Speech-to-text using **Groq Whisper Large V3 Turbo**
- Convert voice recordings into text
- Route voice and text prompts through the same chatbot workflow
- Reset the recording component after processing each recording
- Support voice input across different chat threads

### 📄 RAG — Retrieval-Augmented Generation

- Upload PDF documents through the chat interface
- Extract and split document content into chunks
- Generate embeddings using Google Generative AI
- Store and search vectors using FAISS
- Retrieve relevant document context
- Generate responses grounded in retrieved document content

### 🧠 LangGraph Agent

The chatbot workflow is orchestrated using LangGraph and includes:

- Input guardrails
- Document retrieval
- LLM reasoning
- MCP tool execution
- Output guardrails
- Persistent checkpointing

### 🔧 MCP Tool Integration

The assistant connects to external services through the **Model Context Protocol (MCP)**.

**Gmail MCP**

- Read emails
- Search emails
- Send emails

**Google Calendar MCP**

- Read upcoming events
- Interact with Google Calendar
- Create calendar events where supported

### 🛡️ Guardrails

- Prompt-injection protection
- Unsafe-content filtering
- Input validation and length checks
- Output filtering
- PII protection
- Protection against exposing internal instructions
- Controlled MCP tool usage

---

## 🏗️ System Architecture

```text
┌──────────────────────────────────────────────────────┐
│                 React + Vite Frontend                │
│          Chat UI · Threads · PDF · Microphone         │
└──────────────────────────┬───────────────────────────┘
                           │ HTTP / REST API
                           ▼
┌──────────────────────────────────────────────────────┐
│                       FastAPI                         │
│               API endpoints · Request handling        │
└──────────────────────────┬───────────────────────────┘
                           │
                           ▼
┌──────────────────────────────────────────────────────┐
│                  Python AI Backend                    │
│                    LangGraph Agent                    │
│                                                      │
│ Text input ─────────────────────────────────────┐    │
│ Voice → Groq Whisper STT → Transcribed text ────┤    │
│ PDF → Document processing / FAISS retrieval ────┤    │
│                                                 ▼    │
│                                          Input Guardrail
│                                                 │    │
│                                                 ▼    │
│                                           RAG Retrieval
│                                                 │    │
│                                                 ▼    │
│                                             Agent / LLM
│                                                 │    │
│                                  ┌──────────────┴────┐
│                                  ▼                   ▼
│                              LLM Reply           MCP Tools
│                                                      │
│                                      ┌───────────────┴──────┐
│                                      ▼                      ▼
│                                  Gmail MCP             Calendar MCP
│                                      │                      │
│                                      ▼                      ▼
│                                  Gmail API            Calendar API
│                                      └──────────┬───────────┘
│                                                 ▼
│                                         Output Guardrail
└─────────────────────────────────────────────────┬────┘
                                                  ▼
                                            FastAPI Response
                                                  ▼
                                               React UI
```

**Core design principle:** Text and voice inputs converge into the same `user_input` and follow the same LangGraph workflow. The frontend handles the interface and recording experience; the backend handles transcription and AI processing.

---

## 🧩 LangGraph Architecture

```text
START
  ↓
guardrail_input
  ↓
retrieve_node
  ↓
agent_node
  ├── Tool requested → ToolNode → agent_node
  ↓
guardrail_output
  ↓
END
```

The graph uses asynchronous execution to support MCP tool calls and SQLite checkpointing.

## 🗄️ Persistence and Conversation Memory

The project uses SQLite for persistent conversation state.

```text
LangGraph
    ↓
AsyncSqliteSaver
    ↓
chatbot.db
```

Checkpointing allows conversations to continue across application reruns and supports separate conversation threads. Thread metadata, including thread IDs and chat titles, is managed separately through `database.py`, which also supports rename and delete operations.

## 📚 RAG Pipeline

```text
PDF Upload (React)
       ↓
FastAPI Upload Endpoint
       ↓
Document Loader
       ↓
Text Splitting
       ↓
Google Generative AI Embeddings
       ↓
FAISS Vector Store
       ↓
Similarity Retrieval
       ↓
Relevant Context
       ↓
LangGraph Agent
       ↓
Grounded Response
```

## 🛠️ Tech Stack

| Layer                  | Technology                  |
| ---------------------- | --------------------------- |
| Frontend               | React, Vite                 |
| Backend API            | FastAPI, Uvicorn            |
| Languages              | Python 3.11, JavaScript     |
| Agent Orchestration    | LangGraph                   |
| LLM Framework          | LangChain                   |
| LLM                    | Groq                        |
| Speech-to-Text         | Groq Whisper Large V3 Turbo |
| RAG                    | LangChain, FAISS            |
| Embeddings             | Google Generative AI        |
| Vector Search          | FAISS                       |
| Conversation Memory    | SQLite                      |
| Async Checkpointing    | aiosqlite, AsyncSqliteSaver |
| External Tool Protocol | MCP                         |
| Email Integration      | Gmail API                   |
| Calendar Integration   | Google Calendar API         |
| Authentication         | Google OAuth                |
| Environment Management | python-dotenv               |

---

## 📁 Project Structure

```text
AI-voice-assistant/
├── backend/
│   └── api.py                  # FastAPI application and API endpoints
├── frontend/                   # React + Vite application
│   ├── src/                    # Chat, thread, PDF and voice UI
│   ├── package.json
│   └── ...
├── backend_sqlite.py           # LangGraph backend, LLM, STT and checkpointing
├── frontend_sqlite.py          # Legacy Streamlit frontend (if retained)
├── database.py                 # Chat thread metadata
├── guardrails.py               # Input/output safety and PII handling
├── mcp_client.py               # MCP client and tool loading
├── email_server.py             # Gmail MCP server
├── calender_server.py          # Google Calendar MCP server
├── rag/
│   ├── __init__.py
│   ├── embeddings.py           # Embedding and vector-store operations
│   ├── loader.py               # PDF loading and text splitting
│   ├── prompts.py              # RAG / agent prompts
│   └── retriever.py            # Document retrieval
├── requirements.txt            # Python dependencies
├── README.md
├── .gitignore
└── .env                        # Local environment variables (not committed)
```

> Adjust this tree if your actual repository uses different module paths or filenames. Never commit `.env`, OAuth credentials, tokens, databases, or other sensitive/generated files.

---

## 🚀 Getting Started

### Prerequisites

- Python 3.11
- Node.js and npm
- Git
- Groq and Google Generative AI API credentials
- Google OAuth credentials for Gmail and Google Calendar integrations

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd AI-voice-assistant
```

### 2. Set up the Python backend

```bash
python -m venv .venv
```

**Windows PowerShell**

```powershell
.venv\Scripts\activate
```

**macOS / Linux**

```bash
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

### 3. Set up the React frontend

Open a second terminal:

```bash
cd frontend
npm install
```

### 4. Configure environment variables

Create a `.env` file in the location expected by the backend configuration.

```env
GROQ_API_KEY="your_groq_api_key"
GOOGLE_API_KEY="your_google_api_key"
```

For Gmail and Google Calendar MCP integration, configure the required Google OAuth credentials and tokens according to the MCP server configuration.

**Never commit:**

```text
.env
credentials.json
gmail_token.json
API keys
OAuth credentials
access tokens
```

### 5. Run the application

Start the FastAPI backend from the project root:

```bash
uvicorn backend.api:app --reload --port 8000
```

Start the React development server in a second terminal:

```bash
cd frontend
npm run dev
```

Open the local URL printed by Vite in your browser.

> Ensure the frontend API base URL points to the FastAPI backend and that CORS is configured for the frontend development origin.

---

## 🧪 Example Prompts

**Normal chat**

```text
Explain what an API is.
```

**RAG**

Upload a PDF and ask:

```text
Summarize chapter 3.
What are the main conclusions of this document?
```

**Voice**

```text
What is the difference between TCP and UDP?
```

The recording is transcribed using Groq Whisper Large V3 Turbo and follows the normal chatbot workflow.

**Gmail MCP**

```text
List my recent emails.
```

**Calendar MCP**

```text
What are my upcoming calendar events?
```

---

## 🛡️ Safety Features

- Prompt-injection detection
- Unsafe-content filtering
- Input validation and length checks
- PII handling
- Output filtering
- Protection against exposing internal instructions
- Controlled MCP tool usage

These safeguards help reduce unsafe inputs, inappropriate outputs, and unintended external tool actions.

## ⚡ Asynchronous Execution

The backend uses asynchronous execution for LangGraph, MCP tools, and SQLite checkpointing.

```python
await chatbot.ainvoke(...)
```

The project uses `AsyncSqliteSaver` rather than the synchronous `SqliteSaver` implementation. Preserve the asynchronous execution flow when modifying the backend or MCP integrations.

## 🧠 Design Philosophy

Different input methods should converge into one chatbot workflow. Whether the user submits text or speaks into the microphone, the system produces a common `user_input` that passes through:

```text
Input
  ↓
Guardrails
  ↓
RAG Retrieval
  ↓
LangGraph Agent
  ↓
MCP Tools (when required)
  ↓
Output Guardrails
  ↓
FastAPI Response
  ↓
React UI
```

This separation keeps the React interface, API layer, and AI workflow modular and easier to maintain.

## 🔮 Future Improvements

- Token-by-token response streaming
- Improved RAG retrieval and reranking
- Automated unit and integration tests
- Additional MCP integrations
- Confirmation workflows for sensitive actions
- Improved MCP error handling
- Voice output / text-to-speech
- More advanced conversation memory
- Production-grade authentication and deployment

## 👨‍💻 Authors

**Ayush Kumar**

Built as an AI project combining conversational AI, retrieval-augmented generation, agentic workflows, MCP integrations, and voice interaction in a full-stack productivity assistant.
