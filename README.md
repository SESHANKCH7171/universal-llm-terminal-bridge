# 🔁 Universal LLM Terminal Bridge — FastAPI & LiteLLM Proxy

> An asynchronous middleware proxy that intercepts, translates, and dynamically routes standard Anthropic API requests to **Google Gemini** (or OpenAI) foundation models — without touching Anthropic's billing servers.

***

## 🧠 What Is This?

Advanced agentic CLI tools (like Claude Code) speak the **Anthropic API protocol** — they format every request using Anthropic's strict JSON schema (`MessagesRequest`, `ContentBlockText`, `ContentBlockToolUse`, etc.). Google Gemini and OpenAI **do not** understand this format natively.

This proxy acts as a **universal translation layer**:

```
Claude Code CLI  →  [server.py Proxy]  →  Google Gemini API
     ↑                    ↑
  Anthropic            FastAPI +
  Protocol             LiteLLM
```

You keep the Claude Code interface you love. The **underlying brain** is swapped to Gemini — giving you access to enterprise-grade models at a fraction of the cost, with full data compliance control.

***

## 🏗️ Systems Architecture

The proxy handles four core engineering challenges simultaneously:

### 1. 🔄 Schema Interception & Translation
The `convert_anthropic_to_litellm` function intercepts the outgoing Anthropic-formatted JSON and restructures it into a format LiteLLM (and Gemini/OpenAI) understands — on the fly, with zero downtime.

### 2. 🛠️ Strict Gemini Schema Sanitization
Gemini is notoriously strict about JSON validation. It rejects keys like `additionalProperties` or certain `format` values within string types.

The `clean_gemini_schema` function **recursively strips** all unsupported elements before forwarding to Google, preventing fatal schema validation crashes. This is a known pain point that most basic proxy implementations don't handle.

### 3. 🧠 Dynamic Intelligence Routing (Tiered Model Mapping)
Instead of hardcoding a single destination model, the proxy reads the incoming request's intelligence tier and **automatically maps** it:

| Incoming Request | Routed To |
|---|---|
| Claude Opus | `gemini/gemini-3.1-pro-preview` (heavy reasoning) |
| Claude Sonnet | `gemini/gemini-2.5-pro` (balanced) |
| Claude Haiku | `gemini/gemini-2.5-flash` (fast tasks) |

No manual terminal restarts. The proxy is an **intelligent switchboard**.

### 4. ⚡ Asynchronous Streaming (SSE Translation)
LLM responses are best served token-by-token via Server-Sent Events (SSE). The `handle_streaming` async generator:
- Captures `choices` and `delta` objects from LiteLLM/Gemini
- Repackages them into Anthropic's expected SSE format (`content_block_start`, `content_block_delta`, `message_stop`)
- Manages tool-call lifecycle to prevent client crashes

***

## 🚀 Key Features

- ✅ **Dynamic Intelligence Routing** — Maps Opus/Sonnet/Haiku tiers to appropriate Gemini models automatically
- ✅ **Strict Schema Sanitization** — Recursively cleans Anthropic tool schemas to meet Gemini's strict API requirements
- ✅ **Async SSE Streaming** — Full token-by-token streaming with correct Anthropic event format
- ✅ **Token Count Emulation** — Intercepts hidden token-counting requests to prevent client crashes
- ✅ **Vertex AI Ready** — Built-in support for `USE_VERTEX_AUTH` for enterprise, data-compliant deployments

***

## 🔒 Enterprise & Compliance Note

The standard Google AI Studio API key is designed for **development and testing**.

For enterprise B2B deployments (especially in regulated sectors like Aerospace & Defense), this proxy includes built-in support for **Google Cloud Platform Vertex AI** via:

```env
VERTEX_PROJECT=your-gcp-project-id
USE_VERTEX_AUTH=True
```

Vertex AI provides:
- **Zero Data Retention** — Customer data is never used to train Google's models
- **Data Residency Controls** — Lock processing to specific geographic regions
- **IAM Security** — Enterprise Identity and Access Management

***

## 💻 Setup Instructions

### Prerequisites
- Python 3.10+
- A Google AI Studio API Key (or GCP project for Vertex AI)

### 1. Clone the Repository

```bash
git clone https://github.com/SESHANKCH7171/UNIVERSAL-LLM-TERMINAL-BRIDGE.git
cd universal-llm-terminal-bridge
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure Environment Variables

Create a `.env` file in the project root (⚠️ **this file is in `.gitignore` — never commit it**):

```env
# Required: Your Google API Key
GEMINI_API_KEY=your_google_api_key_here

# Model Tier Mapping (customize as needed)
OPUS_MODEL=gemini/gemini-2.5-pro
SONNET_MODEL=gemini/gemini-2.5-pro
HAIKU_MODEL=gemini/gemini-2.5-flash

# Optional: For enterprise Vertex AI deployments
# VERTEX_PROJECT=your-gcp-project-id
# USE_VERTEX_AUTH=True
```

### 4. Start the Proxy Server

Open **Terminal 1** and run:

```bash
python -u server.py
```

You should see the server start on `http://127.0.0.1:8082` and log model routing activity.

### 5. Connect Your CLI Tool

Open **Terminal 2** and set the environment variables:

**PowerShell (Windows):**
```powershell
$env:ANTHROPIC_BASE_URL = "http://127.0.0.1:8082"
$env:ANTHROPIC_API_KEY  = "sk-ant-dummy"
```

**Bash/Zsh (macOS/Linux):**
```bash
export ANTHROPIC_BASE_URL="http://127.0.0.1:8082"
export ANTHROPIC_API_KEY="sk-ant-dummy"
```

Now launch your CLI tool from Terminal 2. All traffic will route through the proxy to Gemini.

### 6. Verify It's Working

Check Terminal 1 (proxy server logs). You should see routing logs like:

```
claude-sonnet-4-6 → gemini-2.5-pro
```

If you see this, the bridge is working. Your CLI interface is now running on Google Gemini.

***

## 🔑 Why a Dummy API Key Works

The `sk-ant-dummy` key never reaches Anthropic's servers. When your CLI sends a request:

1. It hits `http://127.0.0.1:8082` — **your local proxy**, not Anthropic
2. The proxy swaps the dummy key for your real `GEMINI_API_KEY`
3. The request is translated and forwarded to Google

No Anthropic billing is involved in the data path.

***

## ⚖️ Disclaimer

This is an **educational project** demonstrating API schema translation, local proxy routing, and LLM interoperability using FastAPI and LiteLLM.

- It is not affiliated with Anthropic, Google, or OpenAI
- The `ANTHROPIC_BASE_URL` environment variable is a **standard developer feature** provided by Anthropic for enterprise routing scenarios
- Users are responsible for adhering to the Terms of Service of their respective CLI tools and API providers
- Do **not** use this proxy to process proprietary or sensitive client data through standard API keys — use Vertex AI with enterprise contracts for such use cases

***

## 🛠️ Tech Stack

| Component | Technology |
|---|---|
| Proxy Server | FastAPI (Python) |
| LLM Translation | LiteLLM |
| Destination Model | Google Gemini 2.5 Pro / Flash |
| Async Streaming | Python `asyncio`, SSE |
| Schema Sanitization | Recursive JSON transformer (custom) |
| Enterprise Mode | GCP Vertex AI |

***

## 👤 Author

**Seshank** — AI Systems Architect | Agentic AI & LLM Interoperability  
🔗 [LinkedIn](https://www.linkedin.com/in/seshankch/) | 🐙 [GitHub](https://github.com/SESHANKCH7171)

***

## ⭐ If this helped you

Star the repo and share it with a developer who's tired of single-provider lock-in.
