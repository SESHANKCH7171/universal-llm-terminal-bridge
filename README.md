# Universal LLM Terminal Bridge 🔄

**Run Anthropic Clients (like Claude Code) seamlessly with Gemini, Vertex AI, OpenAI, or direct Anthropic backends.** 🤝

A high-performance transparent proxy server that translates Anthropic API calls into LiteLLM format, allowing Claude Code CLI to run on Google Cloud Vertex AI (Gemini 2.5 Pro / Flash), Google AI Studio, or OpenAI. 🌉

![Anthropic API Proxy](pic.png)

---

## Features ⚡

- 🔄 **Drop-in Claude Code Compatibility:** Run `@anthropic-ai/claude-code` CLI with any backend model.
- ☁️ **Google Cloud Vertex AI Support:** Authenticate via Service Account JSON key (`GOOGLE_APPLICATION_CREDENTIALS`) or Application Default Credentials (ADC).
- 🔑 **Google AI Studio Support:** Simple single API key setup with `GEMINI_API_KEY`.
- 🧠 **Smart Model Mapping:** Transparently maps Claude Opus, Sonnet, and Haiku to active Gemini or OpenAI models.
- 🌊 **Real-Time Streaming:** Full SSE event-stream translation with robust Windows Unicode & error formatting.

---

## Quick Start 🛠️

### 1. Clone the Repository

```bash
git clone https://github.com/SESHANKCH7171/universal-llm-terminal-bridge.git
cd universal-llm-terminal-bridge
```

### 2. Set Up Python Environment

Using `uv` (recommended):
```bash
uv venv
.venv\Scripts\Activate.ps1   # On Windows
# source .venv/bin/activate  # On Linux/macOS
uv pip install -r requirements.txt
```

Or using standard `venv`:
```bash
python -m venv .venv
.venv\Scripts\Activate.ps1   # On Windows
pip install -r requirements.txt
```

---

### 3. Configure `.env`

Copy the template file:
```bash
cp .env.example .env
```

#### Option A: Google Cloud Vertex AI (Use your GCP credits)
```dotenv
USE_VERTEX_AUTH=true
VERTEX_PROJECT="your-gcp-project-id"
VERTEX_LOCATION="us-central1"
GOOGLE_APPLICATION_CREDENTIALS="google-creds.json"

# Model Mappings
OPUS_MODEL=gemini/gemini-2.5-pro
SONNET_MODEL=gemini/gemini-2.5-pro
HAIKU_MODEL=gemini/gemini-2.5-flash

PORT=8082
HOST=127.0.0.1
```

#### Option B: Google AI Studio (Free tier API Key)
```dotenv
USE_VERTEX_AUTH=false
GEMINI_API_KEY="your-google-ai-studio-key"

OPUS_MODEL=gemini/gemini-2.5-pro
SONNET_MODEL=gemini/gemini-2.5-pro
HAIKU_MODEL=gemini/gemini-2.5-flash

PORT=8082
HOST=127.0.0.1
```

#### Option C: OpenAI Backend
```dotenv
OPENAI_API_KEY="sk-..."
PREFERRED_PROVIDER="openai"

PORT=8082
HOST=127.0.0.1
```

---

### 4. Run the Proxy Server

```powershell
python -u server.py
```
*(Server will start on `http://127.0.0.1:8082`)*

---

### 5. Launch Claude Code 🎮

1. **Install Claude Code CLI**:
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```
   *(On Windows, if needed: `npm install -g @anthropic-ai/claude-code-win32-x64`)*

2. **Connect to your proxy**:
   
   **Windows (PowerShell):**
   ```powershell
   $env:ANTHROPIC_BASE_URL="http://127.0.0.1:8082"
   $env:ANTHROPIC_API_KEY="sk-ant-dummy"
   $env:ANTHROPIC_AUTH_TOKEN=""
   claude
   ```

   **Linux / macOS:**
   ```bash
   export ANTHROPIC_BASE_URL="http://localhost:8082"
   export ANTHROPIC_API_KEY="sk-ant-dummy"
   export ANTHROPIC_AUTH_TOKEN=""
   claude
   ```

---

## Model Mapping Reference 🗺️

The proxy routes requests based on your configuration:

| Claude Model | Target Model (Vertex AI) | Target Model (OpenAI) |
|---|---|---|
| **Claude Opus** | `gemini/gemini-2.5-pro` | `openai/gpt-4o` |
| **Claude Sonnet** | `gemini/gemini-2.5-pro` | `openai/gpt-4o` |
| **Claude Haiku** | `gemini/gemini-2.5-flash` | `openai/gpt-4o-mini` |

---

## Contributing 🤝

Contributions and feature requests are welcome! Feel free to open an issue or submit a Pull Request.
