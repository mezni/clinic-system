# Project Ideas

Companion to `SPEC.md`, `ai_architect.md`, and `roadmap.html`.

Use this as a menu, not a mandate. The six core projects are the default track; the
alternatives let you anchor the same competencies in a domain you can get data for.

**Rule:** every project ships a public repo, at least one ADR, one measured eval, and a
short case study. If it has no eval and no decision record, it is a demo, not a portfolio
piece.

## Chosen anchor

These are already decided; the menu below is for supporting repos and alternatives.

- **Domain:** an open-source Rust networking or telecom-adjacent codebase, plus telecom
  runbooks and configuration documents.
- **Capstone:** a multi-tenant telecom NOC copilot — isolated knowledge, permissioned tools
  for alarms, metrics, and tickets, and a local reproducible demo.
- **Demo bar:** local reproducible via `docker compose`; no hosted demo.
- **Budget:** under $50/month; local models first, cheap API fallback.
- **Rust:** intermediate; scope Rust to the indexer, gateway, and MCP server.

## Legend

| Tag | Competency |
| --- | --- |
| RAG | Retrieval, ingestion, grounded answers |
| AG | Agents, tools, orchestration |
| MCP | Model Context Protocol servers and trust |
| OPS | LLMOps: evals, tracing, cost, CI, SLOs, incidents |
| SYS | System design, service boundaries, scale |
| SEC | Security, permissions, governance, tenant isolation |

## The core track (build these)

| # | Repo | CAP | Proves | Rust / Python split |
| --- | --- | --- | --- | --- |
| 1 | `ai-model-selection-harness` | CAP-2 | OPS, SYS | Python evals; Rust optional CLI |
| 2 | `rust-codebase-rag` | CAP-3 | RAG, SYS | Rust indexer; Python retrieval API |
| 3 | `rust-mcp-python-agent` | CAP-4 | MCP, AG, SEC | Rust MCP server; Python agent |
| 4 | `production-agent-ops` | CAP-5 | OPS, AG | Python service; shared eval harness |
| 5 | `telecom-noc-copilot` | CAP-6 | All six | Rust gateway/index; Python orchestration |
| 6 | profile + portfolio index | CAP-1, CAP-7 | Narrative | — |

## Developer tools

- **`codewiki`** — RAG over a large OSS Rust codebase; answers cite file and line, plus generated module and dependency summaries.
  *Proves RAG + SYS.* Rust parser/indexer, Python retrieval and answer service. Data: any active Rust repo.
  *Twist:* publish a reproducible "RAG over Rust repos" report on 3 repos.
- **`pr-review-agent`** — MCP server exposes git/GitHub tools; an agent reviews diffs against a written policy under a hard cost cap.
  *Proves MCP + AG + OPS.* Rust MCP server, Python agent and evals. Data: your own repos.
  *Twist:* every review must cite the policy rule it enforces.
- **`migration-copilot`** — RAG over docs and code to plan a library migration, scored against a labeled migration set.
  *Proves RAG + evals.* Rust indexing, Python planning. Data: a real migration you have done.
  *Twist:* measure how often the plan compiles and runs, not just how plausible it reads.
- **`sql-agent`** — natural language to SQL with guarded execution, row limits, and role-based access.
  *Proves AG + SEC + SYS.* Python agent, Rust gateway optional. Data: a public dataset or your own DB.
  *Twist:* publish the attack suite (prompt injection via row data, privilege escalation).

## Enterprise and operations

- **`support-triage`** — classify and route tickets, retrieve from the KB, draft replies, human-in-the-loop, report cost per ticket.
  *Proves RAG + AG + OPS.* Python orchestration; Rust optional ingestion. Data: public ticket corpora or synthetic.
  *Twist:* a queue-aware cost model — quality and spend per resolved ticket.
- **`runbook-agent`** — incident copilot that reads dashboards and logs through MCP tools and suggests runbook steps with an audit log.
  *Proves MCP + OPS + SEC.* Rust MCP tools, Python agent. Data: a local observability stack.
  *Twist:* replay a simulated incident and compare agent steps to the runbook.
- **`invoice-extraction`** — messy documents to validated JSON with a human review queue and field-level accuracy evals.
  *Proves structured output + evals.* Python pipeline; Rust optional parser. Data: public invoice samples.
  *Twist:* report accuracy per field and the cost of human correction.
- **`contract-risk-review`** — extract clauses, flag risks, cite the source, and report confidence.
  *Proves RAG + extraction + governance.* Python; Rust optional. Data: public contract templates.
  *Twist:* a refusal path for clauses outside the labeled taxonomy.

## Regulated and high-trust

Use public or synthetic data only. Never use real PHI or customer data.

- **`clinical-guideline-rag`** — strict citations, refusal when unsupported, PII redaction, full audit trail.
  *Proves governance + RAG + SEC.* Rust ingestion, Python answers. Data: public clinical guidelines.
  *Twist:* publish the red-team set and the audit schema.
- **`compliance-evidence`** — map controls to policy documents and generate evidence with citations.
  *Proves RAG + governance.* Python; Rust optional. Data: public frameworks and your own policies.
  *Twist:* show a control with no evidence found, and how the system says so.

## Platform and infrastructure

- **`llm-gateway`** — Rust gateway with routing, rate limits, caching, cost accounting, and provider fallback.
  *Proves SYS + OPS.* Rust core, Python admin and evals. Data: two model providers or one plus local.
  *Twist:* a published cost and p95 comparison across routing policies.
- **`eval-platform`** — run evals across models and datasets with regression gates and a dashboard.
  *Proves OPS + evals.* Python core, Rust optional runner. Data: your own task sets.
  *Twist:* one command reproduces every prior release's scores.
- **`mcp-toolkit`** — production-grade MCP servers (filesystem, git, HTTP, Postgres) with permissions and audit.
  *Proves MCP + SEC.* Rust servers, Python client examples. Data: local services.
  *Twist:* a published tool permission matrix and blast-radius analysis.
- **`agent-observability`** — tracing, cost, and eval SDK for agents with a small UI.
  *Proves OPS + AG.* Python SDK, Rust collector optional. Data: your own agents.
  *Twist:* replay a failed run from the trace alone.

## Telecom track

- **`telecom-noc-copilot`** — multi-tenant copilot for NOC engineers: RAG over a Rust network codebase and telecom runbooks, MCP tools for simulated alarms/metrics/tickets, cited diagnoses, and audit.
  *Proves all six.* Rust gateway/index/MCP tools, Python orchestration and evals. Data: OSS Rust networking code, public telecom glossary/runbook patterns, synthetic telemetry.
  *Twist:* replay a simulated outage and score the agent's diagnosis against the runbook.
- **`alarm-correlation-agent`** — group related alarms into incidents and propose a probable cause with evidence.
  *Proves AG + RAG + OPS.* Python agent; Rust optional stream processor. Data: synthetic or public alarm datasets.
  *Twist:* publish precision/recall for incident grouping, not just a plausible narrative.
- **`config-drift-detective`** — compare network configuration intent against observed state and explain drift with citations.
  *Proves RAG + SYS + SEC.* Rust parser/differ, Python explanation service. Data: sanitized configs you create.
  *Twist:* a refusal path when the intent document does not cover the drift.
- **`telecom-runbook-rag`** — answer operational questions from runbooks and vendor docs with strict citations and escalation rules.
  *Proves RAG + governance.* Rust ingestion, Python retrieval. Data: public vendor docs and your own runbooks.
  *Twist:* measure answerability — when the corpus cannot answer, the system must say so.

## Domain-specific (pick what you know)

- **`meeting-memory`** — transcribe, summarize, extract actions, retrieve across meetings, with per-user isolation.
  *Proves RAG + AG + privacy.* Python; Rust optional ingestion. Data: your own recordings with consent.
- **`learning-tutor`** — adaptive tutor with a graded eval set and guardrails against confident wrong answers.
  *Proves AG + OPS.* Python. Data: public question banks.
- **`trading-research-agent`** — research summaries over public filings with citations and strict disclaimers.
  *Proves RAG + governance.* Python; Rust optional ingestion. Data: public filings.
- **`doc-intelligence`** — parse PDFs and tables, chunk mixed formats, answer with citations.
  *Proves RAG + ingestion.* Rust parser, Python retrieval. Data: public reports.

## Capstone options

1. **Telecom NOC copilot (chosen)** — multi-tenant copilot over a Rust network codebase and telecom runbooks, with MCP tools for simulated alarms, metrics, and tickets. Combines the chosen domain, your Rust strength, and all six competencies.
2. **Multi-tenant code intelligence** — RAG over many repos + PR review agent + per-tenant indexes. Best if you later target developer-tooling or platform roles.
3. **Regulated document assistant** — guideline or contract RAG + agent tools + audit and redaction. Best only if you later target healthcare, finance, or legal.

Each capstone must publish the design first: architecture doc, ADR set, SLOs, threat model, cost model. Then build, harden, and postmortem. Demo locally with `docker compose`; do not add hosting you cannot afford to maintain.

## How to choose supporting repos

The anchor is fixed; use this to pick the 3–5 repos that surround it. Score each candidate 1–5 on:

| Criterion | Question |
| --- | --- |
| Data access | Can I get real or credible data this week? |
| Audience fit | Does my target AI lead or client care about this problem? |
| Competency coverage | Does it exercise RAG, tools, ops, and security? |
| Demo clarity | Can a stranger see the value in five minutes with a local run? |
| Cost | Can I run the evals and demo under $50/month? |

Pick the highest total that also gives you the **telecom NOC capstone** plus **3–5 supporting repos**.
Do not build more than six projects; depth beats breadth.

## Anti-patterns

- A chatbot over a toy corpus with no eval.
- Six unrelated demos with no shared ADR story.
- A "production-ready" claim with no SLO, trace, or postmortem.
- A capstone that starts with code before the design is reviewed.
- A domain you cannot get data for, chosen because it sounds impressive.
