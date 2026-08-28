# Long-Term Memory for AI Agents

Example code for giving AI agents persistent, long-term memory using the
[mem0](https://github.com/mem0ai/mem0) Python library.

## Quick Start

1. Create a venv with Python 3.11+
2. Install requirements: `pip install -r requirements.txt`
3. Copy `.env.example` to `.env` and fill in your API keys
4. Run `docker compose -f docker/docker-compose.yml up -d` before running the examples in the `oss` folder
5. Run the example scripts

## How memory works here

Instead of stuffing an entire conversation history into the context window,
these examples extract and store only the salient facts from a conversation,
then retrieve the relevant ones on demand. Roughly:

1. **Fact extraction** — an LLM pulls structured facts (preferences, details,
   plans, etc.) out of the conversation.
2. **Memory management** — new facts are compared against what's already
   stored, and the system decides to add, update, or delete memories rather
   than duplicating them.
3. **Storage** — memories are kept in a vector store for semantic search,
   with an optional graph store for relational queries and a history log for
   auditing changes.

## Core Components

### Configuration

Memory behavior is configured through a config dict covering the vector
store, LLM, embedder, and (optionally) graph store:

```python
config = {
    "vector_store": {
        "provider": "qdrant",
        "config": {"host": "localhost", "port": 6333},
    },
    "llm": {
        "provider": "openai",
        "config": {"api_key": "your-api-key", "model": "gpt-4"},
    },
    "embedder": {
        "provider": "openai",
        "config": {"api_key": "your-api-key", "model": "text-embedding-3-small"},
    },
    "graph_store": {
        "provider": "neo4j",
        "config": {
            "url": "neo4j+s://your-instance",
            "username": "neo4j",
            "password": "password",
        },
    },
    "history_db_path": "/path/to/history.db",
    "version": "v1.1",
    "custom_fact_extraction_prompt": "Optional custom prompt for fact extraction",
    "custom_update_memory_prompt": "Optional custom prompt for update memory",
}
```

```python
m = Memory.from_config(config)
```

### Adding memories

```python
memory.add(messages, user_id="user123")
```

### Retrieving memories

```python
# Get by ID
memory.get(memory_id)

# Search semantically
memory.search(query="What do you know about me?", user_id="user123")

# Get all memories
memory.get_all(user_id="user123")
```

## Files

- `01-mem0-cloud-quickstart.py` — minimal example using the hosted mem0 Cloud API
- `02-mem0-oss-quickstart.py` — minimal example using the self-hosted (OSS) library
- `cloud/email_example.py` — storing and searching parsed emails as memories
- `oss/config.py` — a full self-hosted config example (vector store, LLM, embedder, graph store)
- `oss/memory_demo.py` — a small chat loop that reads/writes memories around each turn
- `oss/support_agent.py` — a customer-support agent that remembers past queries per user
- `docker/docker-compose.yml` — local Qdrant vector store for the OSS examples

## Key Takeaways

1. **Don't just store raw conversations** — extract and store meaningful facts
2. **Use embeddings** for natural-language semantic search over memories
3. **Reconcile updates** — handle add/update/delete rather than blind appends
4. **Track history** for debugging and auditing changes over time
