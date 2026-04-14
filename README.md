![Sidekick](image/sidekick.png)

# Sidekick — AI Personal Co-Worker

Sidekick is an autonomous AI agent that works on tasks on your behalf. You give it a request and a success criteria; it keeps working—browsing the web, reading and writing files, searching for information—until it meets the criteria or needs your input.

---

## How It Works

Sidekick is built on a **LangGraph state machine** with three nodes that loop until the task is done:

```
START → Worker → Tools → Worker → ... → Evaluator → END (or back to Worker)
```

### 1. Worker Agent
The worker is a GPT-4o-mini LLM with access to all tools. It receives:
- Your original request (conversation history)
- The success criteria you defined
- Feedback from the evaluator (if a previous attempt was rejected)

It either calls tools to gather information or do work, or produces a final answer (which is then sent to the evaluator).

### 2. Tools
When the worker needs to take action, it calls one of these tools:

| Tool | Purpose |
|------|---------|
| **Playwright browser** | Navigate, click, fill forms, extract text from web pages |
| **Web search** (Google Serper) | Search the internet for current information |
| **Wikipedia** | Look up encyclopedic knowledge |
| **File management** | Read, write, and list files inside the `sandbox/` directory |
| **Push notification** | Send an alert to your phone via Pushover |

### 3. Evaluator Agent
After the worker produces a response, a separate GPT-4o-mini LLM evaluates it against your success criteria. It returns a structured decision:

- **Success criteria met** → conversation ends, answer is shown
- **User input needed** → the worker has a question or is stuck, so control returns to you
- **Criteria not met** → the evaluator sends feedback to the worker and the loop continues

### Conversation State

Each session has a unique `thread_id`. LangGraph's `MemorySaver` checkpointer keeps the full conversation history across turns, so you can follow up or refine results without losing context.

---

## Setup

### Requirements
- Python 3.12+
- [`uv`](https://github.com/astral-sh/uv) package manager

### Install

```bash
uv sync
playwright install chromium
```

### Configure environment

Create a `.env` file in the project root:

```env
OPENAI_API_KEY=your_openai_key

# Web search
SERPER_API_KEY=your_serper_key

# Push notifications (optional)
PUSHOVER_TOKEN=your_pushover_token
PUSHOVER_USER=your_pushover_user_key

# LangSmith tracing (optional)
LANGCHAIN_API_KEY=your_langsmith_key
LANGCHAIN_ENDPOINT=https://eu.api.smith.langchain.com
LANGCHAIN_TRACING_V2=true
```

### Run

```bash
python app.py
```

This opens a Gradio web UI in your browser automatically.

---

## Using the UI

1. Type your **request** in the top text box.
2. Enter your **success criteria** — a plain-English description of what a good answer looks like (e.g. *"The answer should include a numbered list of at least 5 sources"*).
3. Click **Go!** or press Enter.
4. Sidekick works autonomously and shows both its answer and the evaluator's feedback in the chat.
5. If it has a question, reply in the same chat to continue.
6. Click **Reset** to start a fresh session.

---

## Project Structure

```
sidekick.py        # Core orchestration: Sidekick class and LangGraph graph
sidekick_tools.py  # Tool setup: Playwright, file management, search, notifications
app.py             # Gradio web UI
sandbox/           # Isolated directory for file operations
pyproject.toml     # Dependencies (managed with uv)
```