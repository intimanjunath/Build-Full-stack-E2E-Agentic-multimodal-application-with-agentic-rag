# Personal Expense Assistant — Full-Stack Multimodal Agent (ADK + Gemini 2.5)

This repository hosts a full-stack, multimodal AI agent that ingests receipt images and text, extracts structured expense data, stores it with vector search, and answers grounded questions. The implementation follows the **Google Codelab: Personal Expense Assistant (Multimodal ADK)** end-to-end with a Gradio frontend, a FastAPI backend, Firestore (Native + Vector Search) for storage and retrieval, and Cloud Storage for artifacts.

---

## Overview

**What it does**
- Accepts images and text in a single chat session.
- Extracts structured receipt data (store, timestamp, items, totals, currency).
- Persists receipts to Firestore and original images to Cloud Storage.
- Enables Retrieval-Augmented Generation (RAG) via Firestore Vector Search.
- Answers natural-language queries using a blend of metadata filtering and semantic search.

**Key technologies**
- Agent Development Kit (ADK) + Gemini 2.5
- Gradio (frontend UI)
- FastAPI (backend API)
- Firestore (Native) + Vector Search (embeddings)
- Google Cloud Storage (image artifacts)
- Containerized for Cloud Run deployment

---

## Repository Contents

- `expense_manager_agent/`
  - `agent.py` — Agent wiring (model, tools, planner, callback)
  - `tools.py` — Tools for storing receipts, metadata filtering, vector search, fetch-by-image-id
  - `callbacks.py` — “Before model” callback to keep the newest image inline for vision and collapse older ones into lightweight markers
  - `task_prompt.md` — Agent instruction template and response format
- `backend.py` — FastAPI service exposing a single chat endpoint; integrates ADK Runner, artifact service, and session handling
- `frontend.py` — Gradio ChatInterface (multimodal) that uploads images and relays messages to the backend
- `schema.py` — Request/response data models used between frontend and backend
- `utils.py` — Utility functions for artifact management and response post-processing
- `logger.py` — Minimal logging helper
- `settings.py` / `settings.yaml` — Configuration loader and environment settings (project, region, bucket, collection, backend URL)
- `Dockerfile` — Container definition
- `supervisord.conf` — Process supervisor to run frontend (port 8080) and backend (port 8081) in the same container
- `README.md` / `CREATE_ME.md` — Project documentation and demo outline

> This README intentionally contains **no application code**; it documents structure, behavior, and expectations.

---

## How It Works (Conceptual Flow)

1. The user uploads receipt images and sends a message from the Gradio UI.
2. The frontend serializes images and sends the chat payload to the backend.
3. The backend stores images as artifacts, maintains a session, and invokes the ADK agent.
4. The agent reads the latest image inline, extracts structured fields, embeds a descriptive representation, and writes a Firestore document (including the vector).
5. For queries, the agent uses metadata filters (time/amount) or vector search (semantic similarity) to retrieve receipts, applies relevance checks, and returns grounded answers with optional attachments.
6. The frontend displays the response and any returned images.

---

## Data & RAG Model (High-Level)

- Each stored receipt document includes a unique identifier, store name, transaction time (ISO), total amount, currency, purchased items, and an embedding vector.
- Embeddings are generated for a descriptive text representation of the receipt to enable nearest-neighbor search.
- The agent selects between exact metadata filtering and vector retrieval based on the user query.

---

## Configuration (What You Provide)

- Google Cloud project and region.
- Firestore (Native) database and a collection for receipts with a configured vector field/index.
- A unique Cloud Storage bucket for artifact storage.
- A valid, current Gemini 2.5 model name available in the chosen region.
- Runtime IAM permissions for Firestore, Storage, and Vertex AI (for both local and deployed environments).

---

## Frontend & Backend (Roles)

- **Frontend (Gradio):** Chat UI with multimodal input; sends text and images; renders assistant messages, thinking section (if enabled), and attachments.
- **Backend (FastAPI):** Receives chat requests; stores artifacts; runs the ADK agent; post-processes the response to include sanitized text, optional “thinking” content, and embedded attachments.

---

## Deployment (Summary)

- The container runs both the frontend and backend via a process supervisor.
- The service is designed for Cloud Run with environment variables supplied via a YAML file.
- Post-deploy, the public URL serves the Gradio UI; the backend runs internally in the same container.

---

## Demo Walkthrough (Suggested Outline)

1. **Concept & Architecture** — Explain the agent, tools, callback, backend, frontend, database, and storage at a high level.
2. **Code Tour** — Briefly navigate the agent package, tools, callback, backend API, and frontend UI files (no deep dives required).
3. **Local Demo** — Show uploading receipts, storing them, and running a couple of grounded queries; verify Firestore and Storage entries.
4. **Deployed Demo** — Open the Cloud Run URL; repeat the flow to confirm production behavior.
5. **Close** — Summarize how the repository meets the assignment (multimodal agent, RAG, proper DB integration, end-to-end demo).

---

## Acceptance Criteria (Checklist)

- Multimodality: image + text ingestion with accurate field extraction.
- RAG: embeddings created; vector search returns relevant receipts.
- Database integration: Firestore documents with consistent schema and vector field.
- Storage integration: uploaded artifacts persisted in Cloud Storage.
- Frontend/Backend correctness: responsive UI and working API contract.
- Deployability: containerized service that runs on Cloud Run.
- Documentation: repository structure, configuration expectations, and a demo outline.

---

## References

- Google Codelab: Personal Expense Assistant (Multimodal ADK)  
- Companion articles: “Going Multimodal with Agent Development Kit (ADK) & Gemini 2.5” (Parts 1 & 2)

---
$$ Youtube : https://youtu.be/CNd9Zf2jAjU 
