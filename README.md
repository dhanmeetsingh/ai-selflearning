# Self-Learning AI — Mem0 + Qdrant

A simple AI memory demo using:

* **OpenAI** — generates AI responses
* **Mem0** — extracts and manages memories from conversations
* **Qdrant** — stores and retrieves vector memories
* **Python** — application runtime
* **Docker** — runs Qdrant locally

The main example is:

```text
oss/memory_demo.py
```

---

## Prerequisites

Before starting, make sure you have installed:

* Python **3.11+**
* Docker Desktop
* Git
* VS Code

Check Python:

```bash
python3 --version
```

Check Docker:

```bash
docker --version
```

Check Docker Compose:

```bash
docker compose version
```

---

# 1. Clone the repository

Clone the repository and enter the project:

```bash
git clone <YOUR_REPOSITORY_URL>
cd selflearning-ai
```

If you already have the repository, simply open the project folder in VS Code.

---

# 2. Create a Python virtual environment

From the project root:

```bash
python3 -m venv .venv
```

Activate it.

### macOS / Linux

```bash
source .venv/bin/activate
```

### Windows PowerShell

```powershell
.venv\Scripts\Activate.ps1
```

### Windows CMD

```cmd
.venv\Scripts\activate
```

After activation, you should see:

```text
(.venv)
```

at the beginning of your terminal prompt.

---

# 3. Install Python dependencies

Install the project requirements:

```bash
pip install -r requirements.txt
```

If the project does not contain a `requirements.txt`, install the required packages manually:

```bash
pip install openai mem0ai python-dotenv
```

---

# 4. Configure environment variables

Create a `.env` file in the **project root**.

The project should look like:

```text
selflearning-ai/
├── .env
├── .gitignore
├── docker/
│   └── docker-compose.yml
├── oss/
│   ├── config.py
│   ├── memory_demo.py
│   └── support_agent.py
└── ...
```

If the repository provides `.env.example`, copy it:

```bash
cp .env.example .env
```

Then open it:

```bash
code .env
```

Add your API keys:

```env
MEM0_API_KEY=your_mem0_api_key
OPENAI_API_KEY=your_openai_api_key
```

### Important

Never commit `.env` to Git.

Your `.gitignore` should contain:

```gitignore
.env
.venv/
__pycache__/
```

Never share your API keys publicly. If a key is accidentally exposed, revoke it and create a new one.

---

# 5. Start Docker Desktop

Make sure **Docker Desktop is running**.

On macOS, open Docker Desktop and wait until Docker reports that it is running.

Verify:

```bash
docker --version
```

---

# 6. Start Qdrant

The Docker Compose file is located at:

```text
docker/docker-compose.yml
```

You can start it from the project root with:

```bash
docker compose -f docker/docker-compose.yml up -d
```

You should see output similar to:

```text
✔ Image qdrant/qdrant:latest Pulled
✔ Network docker_default Created
✔ Container qdrant Created
```

Check that Qdrant is running:

```bash
docker compose -f docker/docker-compose.yml ps
```

The Qdrant container should show a running status.

---

# 7. Verify the `.env` file

The demo loads `.env` from the project root.

At the beginning of `oss/memory_demo.py`, use:

```python
from openai import OpenAI
from mem0 import Memory
from dotenv import load_dotenv

load_dotenv(".env")
```

Because the command is executed from the project root:

```bash
python oss/memory_demo.py
```

`.env` correctly refers to:

```text
selflearning-ai/.env
```

If you want the code to work regardless of the directory from which the script is launched, use:

```python
from pathlib import Path
from dotenv import load_dotenv

load_dotenv(Path(__file__).resolve().parent.parent / ".env")
```

---

# 8. Mem0 configuration

The current version of Mem0 expects `user_id` to be passed through `filters` when searching memories.

Use:

```python
relevant_memories = memory.search(
    query=message,
    filters={"user_id": user_id},
    limit=3,
)
```

Do **not** use the older syntax:

```python
memory.search(
    query=message,
    user_id=user_id,
    limit=3,
)
```

---

# 9. Configure Mem0 with Qdrant and OpenAI

A working configuration is:

```python
config = {
    "vector_store": {
        "provider": "qdrant",
        "config": {
            "host": "localhost",
            "port": 6333,
        },
    },
    "llm": {
        "provider": "openai",
        "config": {
            "model": "gpt-4o-mini",
            "temperature": 1,
        },
    },
}
```

The explicit `temperature: 1` is important for compatibility with models that only support the default temperature value.

Initialize the clients:

```python
openai_client = OpenAI()
memory = Memory.from_config(config)
```

---

# 10. Run the memory demo

Make sure you are in the project root:

```bash
pwd
```

You should see something similar to:

```text
/.../selflearning-ai
```

Then run:

```bash
python oss/memory_demo.py
```

You should see:

```text
Chat with AI (type 'exit' to quit)
You:
```

---

# 11. Test the AI

Enter:

```text
Hi my name is Dhanmeet
```

You should receive an AI response such as:

```text
Hi Dhanmeet! How can I assist you today?
```

Now provide some information that Mem0 can remember:

```text
I am learning AI and building projects with Python.
```

Then:

```text
I really enjoy working with AI agents.
```

Exit:

```text
exit
```

---

# 12. Test persistent memory

Start the application again:

```bash
python oss/memory_demo.py
```

Then ask:

```text
What is my name and what am I learning?
```

The application should retrieve relevant memories from Mem0/Qdrant and use them when generating the response.

This demonstrates the main purpose of the project:

```text
User message
      ↓
Memory search
      ↓
Qdrant
      ↓
Relevant memories
      ↓
OpenAI
      ↓
AI response
      ↓
Mem0 extracts new memories
      ↓
Qdrant
```

---

# 13. Optional: Install FastEmbed

You may see this warning:

```text
fastembed not installed - BM25 keyword search disabled.
```

This does **not** prevent the basic demo from working.

If you want the optional functionality, install:

```bash
pip install "mem0ai[extras]"
```

Then run the application again:

```bash
python oss/memory_demo.py
```

---

# 14. PostHog warning

You may also see:

```text
[PostHog] Multiple active PostHog clients detected...
```

This is a warning and is not responsible for the memory demo failure.

For the basic demo, it can be ignored.

---

# Troubleshooting

## `Missing credentials`

If you see:

```text
OpenAIError: Missing credentials
```

make sure:

1. `.env` exists in the project root.
2. `OPENAI_API_KEY` is present.
3. `load_dotenv(".env")` is executed before `OpenAI()`.
4. You are running the command from the project root.

You can safely check whether the key is loaded without printing it:

```bash
python -c "from dotenv import load_dotenv; import os; load_dotenv('.env'); print('OPENAI_API_KEY loaded:', bool(os.getenv('OPENAI_API_KEY')))"
```

Expected:

```text
OPENAI_API_KEY loaded: True
```

Never print your actual API key.

---

## `no configuration file provided`

If you run:

```bash
docker compose up -d
```

and receive:

```text
no configuration file provided: not found
```

Docker Compose cannot find the Compose file in your current directory.

In this project, the Compose file is:

```text
docker/docker-compose.yml
```

Therefore run:

```bash
docker compose -f docker/docker-compose.yml up -d
```

---

## `Cannot connect to the Docker daemon`

If you see:

```text
Cannot connect to the Docker daemon
```

start Docker Desktop and wait for it to finish starting.

Then run:

```bash
docker compose -f docker/docker-compose.yml up -d
```

---

## Mem0 `user_id` error

If you see:

```text
Top-level entity parameters {'user_id'} are not supported in search()
```

change:

```python
memory.search(
    query=message,
    user_id=user_id,
    limit=3,
)
```

to:

```python
memory.search(
    query=message,
    filters={"user_id": user_id},
    limit=3,
)
```

---

## OpenAI `temperature` error

If you see:

```text
Unsupported value: 'temperature' does not support 0.1
```

configure Mem0 explicitly:

```python
"llm": {
    "provider": "openai",
    "config": {
        "model": "gpt-4o-mini",
        "temperature": 1,
    },
},
```

---

# Stopping Qdrant

When you're finished, you can stop the Qdrant container:

```bash
docker compose -f docker/docker-compose.yml down
```

This stops and removes the container/network created by Compose.

To start it again later:

```bash
docker compose -f docker/docker-compose.yml up -d
```

---

# Project Structure

```text
selflearning-ai/
│
├── .env
├── .env.example
├── .gitignore
├── requirements.txt
│
├── docker/
│   └── docker-compose.yml
│
└── oss/
    ├── config.py
    ├── memory_demo.py
    └── support_agent.py
```

---

# Quick Start

Once everything is configured, the normal workflow is:

### Terminal 1 — Start Qdrant

```bash
docker compose -f docker/docker-compose.yml up -d
```

### Terminal 2 — Activate Python environment

```bash
source .venv/bin/activate
```

### Run the demo

```bash
python oss/memory_demo.py
```

That's it.

You now have a local AI memory system running with:

**Python → OpenAI → Mem0 → Qdrant**
