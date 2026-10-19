# Archive

Projects in this repository.

| Directory | Summary |
| --- | --- |
| `01-telco-cust-agent` | Telco customer support ticket triage agent with a Streamlit evaluation lab (prompt A/B testing, experiment history, regression checks, cost analysis) and a CLI triage flow with schema validation, bounded retries, human approval, and SQLite audit logging. |
| `02-oxidetel-core` | OxideTel BSS/OSS: a modular-monolith telecom business support system written in Rust (Axum + SQLx + PostgreSQL), with DDD bounded contexts (iam, crm, catalog, ordering, rating, billing, dunning), JWT RBAC, CDR rating, invoicing, payment webhooks, automated dunning workers, Prometheus metrics, Docker Compose deployment, and E2E lifecycle tests. |
| `03-rag-raw` | Retrieval-Augmented Generation chat app (Streamlit + LangChain + OpenAI): upload PDF/TXT/DOCX files or index public URLs, then chat with history-aware, token-streaming conversations over a chunked vector store, with a chunking metrics dashboard and pytest/ruff tooling. |
| `04-resolve-agent` | Ecommerce customer-support agent in pure Python for returns, exchanges, and order tracking: deterministic policy engine (the LLM only explains decisions), typed/validated tool registry with transactional writes and audit trails, input/output guardrails, contextvar-based tracing and metrics, an Anthropic client with retries/streaming, and a golden-set eval suite. |
