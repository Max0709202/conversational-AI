# Conversational Intelligence Engine

A Streamlit chat client that sends user messages to a **Langflow** flow hosted on **DataStax Astra** (`api.langflow.astra.datastax.com`). The Langflow side can orchestrate multi-agent routing, Groq LLM calls, Astra DB lookups, and RAG-style PDF-to-vector search; this repository contains the minimal UI and HTTP integration that drives that flow.

![Streamlit chat UI](https://raw.githubusercontent.com/Max0709202/conversational-AI/main/AI-Chatbot-Frontend.jpg)

**Streamlit chat UI** — Text area and “Run flow” action; the app posts the message to your deployed Langflow API and renders the returned assistant text.

## Agents in Langflow

![Agents in Langflow](https://raw.githubusercontent.com/Max0709202/conversational-AI/main/agents_langflow.jpg)

**Multi-agent layout** — Example of specialized agents (for example FAQ, order lookup, product lookup) coordinated inside Langflow.

## Full Langflow flow

![Full Langflow flow](https://raw.githubusercontent.com/Max0709202/conversational-AI/main/langflow.jpg)

**End-to-end workflow** — How inputs move through agents, tools, and models before a reply is sent back to the Streamlit app.

## PDF to vector (RAG)

![PDF to vector pipeline](https://raw.githubusercontent.com/Max0709202/conversational-AI/main/pdftovector.jpg)

**Document pipeline** — PDFs ingested into embeddings for semantic search, often used to ground answers in your own documents.

---

## What’s in this repo

| Item | Role |
|------|------|
| `main.py` | Streamlit UI + `requests.post` to the Langflow run API |
| `requirements.txt` | `streamlit`, `requests`, `python-dotenv` |

Flow behavior (agents, Groq, Astra DB, vector store) is defined in **Langflow on Astra**, not in Python files here.

## Requirements

- Python 3.8+
- A Langflow deployment on DataStax Astra with a runnable flow
- An application token with permission to call that flow’s API

## Configuration

1. **Environment variable** — Set `APP_TOKEN` to your Langflow application token (see `main.py`; it uses `os.environ.get("APP_TOKEN")`).

2. **IDs in `main.py`** — Set `LANGFLOW_ID` and `FLOW_ID` to match your Astra Langflow project and flow. The run URL is built as:

   `{BASE_API_URL}/lf/{LANGFLOW_ID}/api/v1/run/{ENDPOINT}`

3. **Endpoint name** — `ENDPOINT` defaults to `"customer"`; it must match the endpoint name configured for your flow in Langflow.

Create a `.env` file in the project root (optional but convenient with `load_dotenv()`):

```env
APP_TOKEN=your_application_token_here
```

## Install and run

```bash
pip install -r requirements.txt
streamlit run main.py
```

Open the URL Streamlit prints (usually `http://localhost:8501`), enter a message, and click **Run flow**.

## How the app calls Langflow

The client sends a JSON payload with `input_value`, `output_type: "chat"`, and `input_type: "chat"`, and an `Authorization: Bearer <APP_TOKEN>` header. The response is parsed to extract the first chat message text from the nested `outputs` structure (see `main.py`).

## Architecture (high level)

```
User → Streamlit (main.py) → Langflow API on Astra → agents / tools / LLM / DB / vectors
                              ↓
                    Assistant text → Streamlit markdown
```

## Stack (typical setup)

- **UI:** Streamlit, Python  
- **Orchestration:** Langflow (hosted on DataStax Astra)  
- **Often used in the flow:** Groq (e.g. LLaMA family), Astra DB, embeddings and vector search for PDFs/RAG  

## Troubleshooting

- **401 / auth errors** — Check `APP_TOKEN` and that the token is valid for the Langflow instance.  
- **404 / wrong URL** — Verify `LANGFLOW_ID`, `FLOW_ID` (if required by your setup), and `ENDPOINT` against the Langflow UI/API docs.  
- **Parse errors in the UI** — The API response shape may differ; adjust the path in `main.py` that reads `response["outputs"][0]...` to match your flow’s output format.