# Daily Step-by-Step Guide

Companion to `ai_architect.md`, `project-ideas.md`, and `roadmap.html`. Every day follows the same 5-step rhythm,
then a day-specific set of actions. Total ≈ 150 minutes.


## Before Day 1 — one-time setup (2–3 hrs)

1. Install Python 3.11+, `uv`, the Rust toolchain (`rustup`), Git, and an editor with Rust + Python support.
2. Create a GitHub account/profile you are willing to show publicly. Add a short bio and photo.
3. Get API access to **one** cheap provider plus a local model (Ollama or vLLM). Local-first is the default.
4. Set a monthly budget ceiling **under $50**; write it down and set a billing alert.
5. Create the repo `ai-model-selection-harness` (public, empty).
6. Create a `learning-log` repo or use GitHub Issues on your profile repo. You will open one issue per week.
7. Install `gh` (GitHub CLI) and authenticate, so commits and issues are fast.
8. Book 150 minutes per day on your calendar for 12 weeks. Protect it.
9. Pick the Rust networking or telecom-adjacent OSS codebase you will use from Week 3 onward; confirm it clones and builds.

> Anchor: telecom NOC copilot over a Rust networking codebase. Demo locally with `docker compose`; no hosting.
> Rule: never start a sprint with the previous repo unpolished.


## The daily rhythm (use every day)

1. **15 min — Set up.** Open today's learning-log entry. Write the day's question in one sentence.
2. **45 min — Learn.** Read the listed topics. Take notes as bullets, not prose. Draw one diagram if the topic is structural.
3. **30 min — Practice.** Do the small exercise. This is where learning sticks.
4. **45 min — Build.** Apply it in the sprint project. Commit at least once.
5. **15 min — Prove and log.** Answer the checkpoint out loud or in one paragraph. Update the log. Close the day.

> If the day slips: cut step 3, never step 5.


# Month 1 — Foundations and retrieval
## Week 1 — Foundations and model selection

### Day 1 — The AI architect role and the system landscape
1. **Setup:** open learning-log issue #1: "Day 1 — what an AI architect owns."
2. **Learn:** AI vs ML vs LLM vs agent; architect concerns (data, model, retrieval, tools, evals, ops, cost, safety); build vs buy; when not to use AI.
3. **Practice:** write three reference workloads and one "do not use AI here" case.
4. **Build:** scaffold `ai-model-selection-harness`; write the README problem statement, audience, and non-goals; create `ADR-001` skeleton.
5. **Prove:** answer "Why is 'use an LLM' an architecture decision?" Commit and close.

**Done when:** public repo has README + ADR skeleton, and the three workloads are written down.

### Day 2 — LLM fundamentals at architecture level
1. **Setup:** write today's question: "What bounds a model's usable capability?"
2. **Learn:** tokens/tokenization, context windows, embeddings (concept), sampling, structured output, tool calling, model families.
3. **Practice:** read three model spec sheets; tabulate context, price, modality, structured-output support.
4. **Build:** define a `Provider` interface and implement 2–3 adapters; load keys from `.env`; smoke-call each model once.
5. **Prove:** explain why two runs of the same prompt can differ, and what that means for tests.

**Done when:** one command reaches every candidate model.

### Day 3 — Evaluation design
1. **Setup:** question: "What makes an eval trustworthy?"
2. **Learn:** benchmarks vs task evals; eval-set design; scoring methods; quality/latency/cost tradeoff; failure taxonomy.
3. **Practice:** define five task categories and a rubric for each.
4. **Build:** create `tasks/tasks.jsonl` with 5 seed tasks (one per category); implement exact-match and schema scoring.
5. **Prove:** answer "How can an eval be gamed?" Commit.

**Done when:** 5 seed tasks exist and can be scored.

### Day 4 — Prompting and structured output as a contract
1. **Setup:** question: "Where should schema validation live?"
2. **Learn:** instruction hierarchy, few-shot, JSON schema, validation/repair, prompt versioning, tool schemas.
3. **Practice:** write prompts for the five task categories.
4. **Build:** wire prompts into the runner; validate outputs; add a repair/retry path.
5. **Prove:** explain the difference between prompt-level and client-level validation.

**Done when:** the runner produces schema-valid output or fails loudly.

### Day 5 — Cost, latency, and reliability economics
1. **Setup:** question: "Why can p50 look fine while users suffer?"
2. **Learn:** token accounting, pricing, p50/p95/tail, streaming, batching, caching, retries, rate limits, fallbacks.
3. **Practice:** build a price table for the shortlisted models.
4. **Build:** instrument latency (p50/p95) and cost per run; store them with results.
5. **Prove:** explain what you investigate when p95 is bad. Write the week-1 log; close the issue.

**Done when:** every result row has latency and cost, not just quality.

## Week 2 — Measurement, reliability, and governance

### Day 6 — Evaluation harness engineering
1. **Setup:** question: "What metadata makes a run trustworthy?"
2. **Learn:** reproducibility, seeds/versions, dataset versioning, experiment tracking, CI evals, small-sample variance.
3. **Practice:** run the same eval twice and compare variance.
4. **Build:** add run metadata and a clean-clone reproduction path; expand tasks toward 20–30.
5. **Prove:** list the metadata a stranger needs to trust your results.

**Done when:** a clean clone reproduces the same raw records.

### Day 7 — Failure analysis and robustness
1. **Setup:** question: "How do you tell a model failure from an eval failure?"
2. **Learn:** hallucination vs grounding vs refusal; over-refusal; format drift; prompt injection (direct/indirect); root-cause analysis.
3. **Practice:** categorize 20 baseline failures using the taxonomy.
4. **Build:** generate the decision matrix; add a failure-mode section to the report.
5. **Prove:** explain how retrieved or user content becomes an attack surface.

**Done when:** the report names failures with examples.

### Day 8 — Decision records and architecture communication
1. **Setup:** question: "What will an AI lead challenge in this decision?"
2. **Learn:** ADR structure; tradeoff writing; context/component/sequence diagrams; decision vs preference.
3. **Practice:** outline ADR-001 from your evidence.
4. **Build:** write ADR-001 and the README tradeoff section.
5. **Prove:** defend the model choice against "the leaderboard says otherwise."

**Done when:** ADR-001 states a decision with linked evidence and consequences.

### Day 9 — Security, privacy, and governance basics
1. **Setup:** question: "What data must never leave the boundary?"
2. **Learn:** PII/data minimization/retention/residency; provider data terms; OWASP LLM Top 10; NIST AI RMF; EU AI Act tiers.
3. **Practice:** list data flows and mark sensitive data.
4. **Build:** fix secrets handling; add privacy limitations to the README.
5. **Prove:** explain how you would enforce a data boundary.

**Done when:** no secrets in the repo and data flows are documented.

### Day 10 — Sprint review and teach-back
1. **Setup:** question: "What does the evidence actually say?"
2. **Learn:** synthesis, uncertainty, retrospection.
3. **Practice:** present the decision to a peer in 5 minutes.
4. **Build:** audit against the DoD; fix links; link the repo from the portfolio index.
5. **Prove:** explain decision + tradeoff + risk in under 3 minutes. Close week-2 log and retro.

**Done when:** Sprint 1 passes the DoD and is publicly linked.


## Week 3 — RAG foundations

> Apply every retrieval exercise to your chosen Rust networking or telecom-adjacent codebase.

### Day 11 — Why retrieval
1. **Setup:** open Sprint 2 log; question: "When is RAG the wrong answer?"
2. **Learn:** knowledge cutoff, private data, grounding; RAG vs fine-tuning vs long context; the retrieval pipeline.
3. **Practice:** write the retrieval problem statement and success criteria.
4. **Build:** scaffold `rust-codebase-rag`; start ADR-002.
5. **Prove:** name two cases where fine-tuning beats RAG, and two where it does not.

**Done when:** scope and success criteria are written.

### Day 12 — Ingestion and chunking
1. **Setup:** question: "What is the right unit of retrieval for code?"
2. **Learn:** parsing, structure-aware chunking, metadata, overlap, code chunking.
3. **Practice:** chunk one Rust file by item; record line ranges.
4. **Build:** implement the Rust chunker with metadata.
5. **Prove:** explain why 512-token chunks are a bad default for code.

**Done when:** every chunk traces to a file and line range.

### Day 13 — Embeddings and vector search
1. **Setup:** question: "What do you trade away for ANN speed?"
2. **Learn:** embedding models/dimensions, similarity, ANN indexes (HNSW, IVF), build vs query cost.
3. **Practice:** compare two embedding models on the same chunks.
4. **Build:** embed chunks and build the index; measure build time and cost.
5. **Prove:** explain the recall/latency/memory tradeoff.

**Done when:** the index rebuilds from scratch with one command.

### Day 14 — Hybrid retrieval and reranking
1. **Setup:** question: "Why can a reranker hurt?"
2. **Learn:** BM25 vs dense; fusion; cross-encoders vs bi-encoders; recall vs precision.
3. **Practice:** add BM25 + vector fusion.
4. **Build:** add a reranker; compare against the baseline.
5. **Prove:** show which change moved recall.

**Done when:** you have a measured before/after.

### Day 15 — Context assembly and citations
1. **Setup:** question: "How do you make 'I don't know' a first-class output?"
2. **Learn:** context budgeting, lost-in-the-middle, citation grounding, answerability.
3. **Practice:** test unsupported questions and refusals.
4. **Build:** force citations to source and add a refusal path.
5. **Prove:** explain how citations reduce hallucination — and where they do not.

**Done when:** answers cite real locations and refuse when unsupported.

## Week 4 — RAG evaluation and service design

### Day 16 — Retrieval evaluation
1. **Setup:** question: "Which metric do you optimize first?"
2. **Learn:** recall@k, precision@k, MRR, nDCG, groundedness, answer relevance, labeling.
3. **Practice:** compute recall@k and faithfulness on the frozen set.
4. **Build:** build the retrieval eval suite.
5. **Prove:** explain what recall@k hides.

**Done when:** the eval suite runs from one command.

### Day 17 — RAG failure modes and hardening
1. **Setup:** question: "How does retrieved content become an attack?"
2. **Learn:** vocabulary gap, multi-hop, stale index, embedding drift, ACL-aware retrieval, indirect injection.
3. **Practice:** root-cause 10 retrieval failures.
4. **Build:** fix the top two failure causes; add tests.
5. **Prove:** explain permission-aware retrieval.

**Done when:** the failure analysis is written with fixes.

### Day 18 — Service boundaries and the Rust/Python split
1. **Setup:** question: "Can you defend the split without 'I like Rust'?"
2. **Learn:** when Rust (ingestion, perf, safety, servers), when Python (ecosystem, orchestration), IPC/contracts, failure isolation.
3. **Practice:** draw the boundary and write the contract.
4. **Build:** implement the Python retrieval service against the Rust index.
5. **Prove:** defend ADR-002 in one paragraph.

**Done when:** the service boundary is documented with a contract.

### Day 19 — Retrieval cost, latency, and caching
1. **Setup:** question: "Where is the cheapest cache?"
2. **Learn:** embedding/index/query cost, caching layers, batching, latency budgets.
3. **Practice:** measure index build and query p95.
4. **Build:** add caching and context budgeting.
5. **Prove:** explain the cost/latency tradeoff of caching answers.

**Done when:** each query has a cost and latency number.

### Day 20 — Sprint review and ADR defense
1. **Setup:** question: "What would you do with half the latency budget?"
2. **Learn:** presenting retrieval quality honestly; connecting evals to decisions.
3. **Practice:** run a mock ADR-002 review.
4. **Build:** polish, reproduce, link the repo.
5. **Prove:** defend the retrieval design. Close week-2 log and retro.

**Done when:** Sprint 2 passes the DoD and is linked.


---

# Month 2 — Agents, MCP, and operations
## Week 5 — Agents and MCP

> Use telecom-flavored tools — simulated alarms, metrics, tickets — so the tool contract matches the capstone.

### Day 21 — Agent fundamentals
1. **Setup:** open Sprint 3 log; question: "How do you bound an agent loop?"
2. **Learn:** observe/plan/act loop; memory/state; termination; single vs multi-agent.
3. **Practice:** design one bounded agent task with an explicit stop condition.
4. **Build:** scaffold `rust-mcp-python-agent`; start ADR-003.
5. **Prove:** name three ways a loop fails to terminate and how you bound them.

**Done when:** the agent task and stop conditions are written.

### Day 22 — Tool design and contracts
1. **Setup:** question: "Why is a vague tool description an architecture bug?"
2. **Learn:** schemas, idempotency, side effects, error surfaces, least privilege.
3. **Practice:** write contracts for three tools.
4. **Build:** define the bounded tool surface (3–5 tools).
5. **Prove:** explain how a bad tool schema causes wrong calls.

**Done when:** every tool has a contract and permission assumption.

### Day 23 — MCP protocol
1. **Setup:** question: "What does MCP standardize?"
2. **Learn:** hosts/clients/servers; tools/resources/prompts; JSON-RPC; stdio vs HTTP; lifecycle.
3. **Practice:** trace a `tools/call` request and response.
4. **Build:** implement `initialize`, `tools/list`, `tools/call`.
5. **Prove:** explain the client/server responsibility split.

**Done when:** a client can list and call tools.

### Day 24 — Building an MCP server in Rust
1. **Setup:** question: "Where do you enforce a tool timeout?"
2. **Learn:** server lifecycle, capability declaration, dispatch, concurrency, structured errors.
3. **Practice:** implement one tool end to end.
4. **Build:** add remaining tools with validation and timeouts.
5. **Prove:** explain where errors should be surfaced to the model.

**Done when:** invalid input fails safely.

### Day 25 — Agent orchestration in Python
1. **Setup:** question: "How do you stop a loop on a failing tool?"
2. **Learn:** client loop, context management, tool-call parsing, iteration limits, observability hooks.
3. **Practice:** build the client loop against the server.
4. **Build:** complete one task end to end; record the transcript.
5. **Prove:** explain how you would debug a wrong tool choice.

**Done when:** the agent completes a task with a recorded trace.

## Week 6 — Trust, safety, and testing for agents

### Day 26 — Trust boundaries and permissions
1. **Setup:** question: "What is the blast radius of your most powerful tool?"
2. **Learn:** authn vs authz, service/user identity, least privilege, allowlists, audit logs.
3. **Practice:** write a permission matrix.
4. **Build:** enforce permission checks and audit logging.
5. **Prove:** explain the blast radius per tool.

**Done when:** a rogue call is blocked and logged.

### Day 27 — Agent security threats
1. **Setup:** question: "How does a document become a command?"
2. **Learn:** prompt injection, tool poisoning, confused deputy, excessive agency, exfiltration, sandboxing.
3. **Practice:** red-team your own agent with three attacks.
4. **Build:** add guardrails against the top attack.
5. **Prove:** explain indirect prompt injection with an example.

**Done when:** at least one attack is blocked by a guardrail.

### Day 28 — Testing agents
1. **Setup:** question: "How do you test a non-deterministic agent?"
2. **Learn:** unit/integration/trajectory evals; fixtures; recorded transcripts; failure injection.
3. **Practice:** write integration tests for server + agent.
4. **Build:** add failure-injection tests.
5. **Prove:** explain what to assert when output varies.

**Done when:** the test suite passes and covers failures.

### Day 29 — Agent cost, latency, and guardrails
1. **Setup:** question: "What happens when the cost ceiling is hit?"
2. **Learn:** loop budgets, model routing, timeouts, circuit breakers, fallbacks, guardrails.
3. **Practice:** add a loop budget and measure cost per task.
4. **Build:** add guardrails and cost evidence.
5. **Prove:** explain your cost ceiling and degradation behavior.

**Done when:** each task has a cost and a hard budget.

### Day 30 — Sprint review and trust-model defense
1. **Setup:** question: "What residual risk remains, and who owns it?"
2. **Learn:** explaining trust boundaries and residual risk.
3. **Practice:** present the permission matrix and blast radius.
4. **Build:** polish, reproduce, link the repo.
5. **Prove:** defend ADR-003. Close week-2 log and retro.

**Done when:** Sprint 3 passes the DoD and is linked.


## Week 7 — Production and LLMOps

### Day 31 — Production readiness
1. **Setup:** open Sprint 4 log; question: "Which SLO would you page on?"
2. **Learn:** SLI/SLO/SLA, error budgets, capacity, deployment strategies, config/secrets.
3. **Practice:** define three SLOs for the agent service.
4. **Build:** scaffold CI; start ADR-004; write the release contract.
5. **Prove:** explain why you page on that SLO.

**Done when:** a release has explicit pass/fail gates.

### Day 32 — Observability for LLM systems
1. **Setup:** question: "What must a trace contain to debug a wrong answer later?"
2. **Learn:** traces/spans, OpenTelemetry concepts, structured logs, metrics, privacy-preserving capture.
3. **Practice:** add a trace to one run.
4. **Build:** add run IDs, structured logs, and cost/latency metrics.
5. **Prove:** explain what you would look at first in a trace.

**Done when:** any run can be reconstructed from traces.

### Day 33 — Evals in CI
1. **Setup:** question: "What is the smallest eval that catches your last regression?"
2. **Learn:** golden sets, regression gates, artifact storage, release criteria, flaky evals.
3. **Practice:** put the eval suite in CI.
4. **Build:** fail the build on regression.
5. **Prove:** explain how you handle subjective evals in CI.

**Done when:** a bad change cannot merge silently.

### Day 34 — Prompt and model lifecycle
1. **Setup:** question: "How do you roll back a prompt without redeploying?"
2. **Learn:** prompt versioning, model upgrades, canary/A-B/rollback, change management.
3. **Practice:** version a prompt and run a canary comparison.
4. **Build:** add prompt/model versioning to the service.
5. **Prove:** explain the rollback path.

**Done when:** prompt and model versions are traceable.

### Day 35 — Incident response and postmortems
1. **Setup:** question: "What is the first mitigation for a provider outage?"
2. **Learn:** incident lifecycle, runbooks, on-call, blameless postmortems, follow-ups.
3. **Practice:** write a runbook for the agent service.
4. **Build:** simulate an incident and observe behavior.
5. **Prove:** explain your triage order.

**Done when:** the runbook exists and was walked through.

## Week 8 — Governance and operations at scale

### Day 36 — Guardrails and moderation
1. **Setup:** question: "What should happen when a guardrail trips?"
2. **Learn:** input/output/tool guardrails, PII redaction, jailbreak detection, policy vs model behavior.
3. **Practice:** add one input and one output guardrail.
4. **Build:** test the guardrails with failure cases.
5. **Prove:** explain fail-open vs fail-closed.

**Done when:** guardrails are enforced and tested.

### Day 37 — Cost governance
1. **Setup:** question: "How do you stop a runaway loop before it becomes a bill?"
2. **Learn:** budgets, quotas, chargeback, cost per request/tenant, caching economics, anomaly alerts.
3. **Practice:** build a cost dashboard from traces.
4. **Build:** enforce rate limits and cost caps.
5. **Prove:** explain your alert thresholds.

**Done when:** the service stops before it overspends.

### Day 38 — Data governance and compliance
1. **Setup:** question: "What does 'delete my data' mean for embeddings and logs?"
2. **Learn:** GDPR basics, EU AI Act tiers, SOC 2 overview, auditability, data lineage.
3. **Practice:** map data flows to retention/deletion rules.
4. **Build:** document governance notes for the capstone.
5. **Prove:** explain deletion across indexes, caches, and logs.

**Done when:** data flows have retention and deletion notes.

### Day 39 — Reliability patterns
1. **Setup:** question: "When should you fail fast instead of retrying?"
2. **Learn:** retries with jitter/backoff, circuit breakers, bulkheads, multi-provider fallback, degradation.
3. **Practice:** test a provider-outage fallback.
4. **Build:** add fallback behavior and tests.
5. **Prove:** explain the retry budget.

**Done when:** fallback is tested and documented.

### Day 40 — Sprint review and runbook walkthrough
1. **Setup:** question: "Could someone else run this from the runbook?"
2. **Learn:** operating a system you did not build; on-call readiness.
3. **Practice:** have a peer follow the runbook and note gaps.
4. **Build:** write the postmortem; polish, reproduce, link.
5. **Prove:** defend ADR-004. Close week-2 log and retro.

**Done when:** Sprint 4 passes the DoD and is linked.


---

# Month 3 — Capstone and the switch
## Week 9 — Capstone architecture

> Capstone: `telecom-noc-copilot` — multi-tenant RAG over the Rust codebase and telecom runbooks, plus MCP tools for simulated alarms, metrics, and tickets. Local reproducible demo only.

### Day 41 — Multi-tenant architecture
1. **Setup:** open Sprint 5a log; question: "Which isolation model for 10 vs 10,000 tenants?"
2. **Learn:** isolation models, noisy neighbor, per-tenant config/keys/quotas, tenant lifecycle.
3. **Practice:** choose an isolation model and write the tradeoffs.
4. **Build:** start the capstone design doc and ADR index.
5. **Prove:** defend your isolation choice.

**Done when:** the tenancy ADR is written.

### Day 42 — Identity and access architecture
1. **Setup:** question: "Where does tenant identity enter and where is it enforced?"
2. **Learn:** OIDC/OAuth2 flows, RBAC vs ABAC, service auth, secrets.
3. **Practice:** draw the auth flow end to end.
4. **Build:** write the auth ADR.
5. **Prove:** explain the difference between authentication and authorization in your design.

**Done when:** the auth flow diagram exists.

### Day 43 — Architecture documentation
1. **Setup:** question: "Could a new engineer implement a component from this diagram?"
2. **Learn:** C4 model, sequence diagrams, contracts, data ownership.
3. **Practice:** write context and component diagrams.
4. **Build:** complete the architecture doc.
5. **Prove:** explain the system in 3 minutes using only the diagrams.

**Done when:** the design is reviewable without you present.

### Day 44 — SLOs, capacity, and cost modeling
1. **Setup:** question: "What breaks first at 10x load?"
2. **Learn:** workload estimation, unit economics, headroom, cost drivers.
3. **Practice:** build a cost model with three scenarios.
4. **Build:** write the SLO and cost docs.
5. **Prove:** state cost per tenant under load.

**Done when:** the cost model is defensible.

### Day 45 — Threat modeling
1. **Setup:** question: "Which threat is most likely, and which is most damaging?"
2. **Learn:** STRIDE, trust boundaries, attack trees, AI-specific threats, residual risk.
3. **Practice:** run STRIDE across components.
4. **Build:** write the threat model.
5. **Prove:** explain your top risk and mitigation.

**Done when:** threats and mitigations are documented.


## Week 10 — Capstone design decisions

### Day 46 — Retrieval at multi-tenant scale
1. **Setup:** question: "How do you prove tenant A cannot retrieve tenant B's chunk?"
2. **Learn:** per-tenant vs shared indexes, ACL-aware retrieval, leakage risks, re-indexing.
3. **Practice:** design ACL-aware retrieval.
4. **Build:** write the retrieval ADR.
5. **Prove:** explain where leakage could occur.

**Done when:** the retrieval isolation design is explicit.

### Day 47 — Tool trust in multi-tenant systems
1. **Setup:** question: "What stops a tenant from invoking another tenant's tool?"
2. **Learn:** per-tenant permissions, sandboxing, audit trails, delegation risks.
3. **Practice:** extend the permission matrix to tenants.
4. **Build:** write the tool-trust ADR.
5. **Prove:** explain the delegation risk.

**Done when:** the tenant permission model is documented.

### Day 48 — Gateway and orchestration design
1. **Setup:** question: "Where do you enforce quotas — gateway or orchestrator?"
2. **Learn:** gateway responsibilities, Rust/Python boundaries, backpressure, failure propagation.
3. **Practice:** write API contracts and sequence diagrams.
4. **Build:** write the gateway/orchestration ADRs.
5. **Prove:** defend the boundary.

**Done when:** contracts and sequences are written.

### Day 49 — Design review and tradeoff defense
1. **Setup:** question: "Which decision would you most regret getting wrong?"
2. **Learn:** presenting under challenge, reversible vs irreversible decisions, recording dissent.
3. **Practice:** run a mock design review.
4. **Build:** revise ADRs from feedback.
5. **Prove:** defend the full design.

**Done when:** feedback is reflected in the design.

### Day 50 — Build planning and risk
1. **Setup:** question: "What will you cut first if week 11 slips?"
2. **Learn:** milestones, critical path, cut lists, risk registers.
3. **Practice:** write the week-11 plan with an explicit cut list.
4. **Build:** finalize the design and publish it.
5. **Prove:** explain the riskiest milestone. Close week-2 log.

**Done when:** the build plan is written and the design is public.


## Week 11 — Build and harden

### Day 51 — Integration architecture
1. **Setup:** open Sprint 5b log; question: "Where is the most likely integration failure?"
2. **Learn:** wiring RAG/MCP/agent/gateway, contract testing, error propagation, config.
3. **Practice:** trace one full request path.
4. **Build:** scaffold and get a request through the full path.
5. **Prove:** explain how you would detect an integration failure.

**Done when:** one request traverses the whole system.

### Day 52 — Tenant isolation testing
1. **Setup:** question: "Name three non-obvious places tenant data can leak."
2. **Learn:** cross-tenant authorization, leakage via logs/caches/embeddings, quota enforcement.
3. **Practice:** write cross-tenant denial tests.
4. **Build:** enforce isolation and run the tests.
5. **Prove:** show the test that proves isolation.

**Done when:** cross-tenant denial is tested.

### Day 53 — Evals and observability in the capstone
1. **Setup:** question: "Which dashboard would you show an AI lead first?"
2. **Learn:** end-to-end evals, cross-service tracing, dashboards, per-tenant metrics.
3. **Practice:** run the full eval suite and capture traces.
4. **Build:** add dashboards and per-tenant metrics.
5. **Prove:** explain what the dashboard proves.

**Done when:** quality, cost, and latency are visible per tenant.

### Day 54 — Load and failure testing
1. **Setup:** question: "Is 2x capacity behavior acceptable?"
2. **Learn:** load profiles, failure injection, degradation modes, capacity limits.
3. **Practice:** run a load test and one failure simulation.
4. **Build:** fix the top bottleneck.
5. **Prove:** explain the degradation mode.

**Done when:** load and failure results are recorded.

### Day 55 — Postmortem and iteration
1. **Setup:** question: "What did the design get wrong?"
2. **Learn:** honest postmortems, updating ADRs, deciding what not to fix.
3. **Practice:** write the capstone postmortem.
4. **Build:** update ADRs and record the demo.
5. **Prove:** explain what you would change. Close the week log.

**Done when:** the capstone is hardened and documented.


## Week 12 — Portfolio and career

### Day 56 — Personal positioning
1. **Setup:** open Sprint 6 log; question: "What should a stranger believe after 5 minutes?"
2. **Learn:** architect narrative, two doors, proof-of-judgment, what to leave out.
3. **Practice:** write the 30-second pitch.
4. **Build:** write the profile README and two-door paths.
5. **Prove:** deliver the pitch.

**Done when:** the profile front door is live.

### Day 57 — Technical writing
1. **Setup:** question: "Does each case study prove a decision?"
2. **Learn:** case-study structure, README structure, ADR storytelling, evidence over adjectives.
3. **Practice:** write case studies for sprints 1–3.
4. **Build:** publish the first three case studies.
5. **Prove:** get one reader to summarize a case study back to you.

**Done when:** three case studies are published and linked.

### Day 58 — Portfolio information architecture
1. **Setup:** question: "Can a reviewer find your best evidence without asking?"
2. **Learn:** navigation, evidence links, consistency, readability.
3. **Practice:** build the portfolio index.
4. **Build:** fix broken links and inconsistent READMEs.
5. **Prove:** navigate from profile to capstone evidence in two clicks.

**Done when:** the index links every repo, ADR, and evidence artifact.

### Day 59 — Interviewing as an AI architect
1. **Setup:** question: "Can you design a RAG/agent system on a whiteboard in 30 minutes?"
2. **Learn:** system-design interviews, tradeoff questions, whiteboard flow, career-switch framing.
3. **Practice:** do one whiteboard design under time pressure.
4. **Build:** write interview notes and a reusable design template.
5. **Prove:** present the whiteboard design.

**Done when:** you have a reusable architecture interview approach.

### Day 60 — Mock reviews and feedback
1. **Setup:** question: "What did reviewers consistently misunderstand?"
2. **Learn:** receiving/filtering critique, taste vs substance, stopping polish.
3. **Practice:** run two mock reviews and record feedback.
4. **Build:** apply the top fixes.
5. **Prove:** explain what you changed and why. Close week-2 log.

**Done when:** feedback is reflected in the repos.

### Day 61 — Outreach and networking
1. **Setup:** question: "What is your specific ask?"
2. **Learn:** targeted outreach, message structure, communities, follow-up.
3. **Practice:** build a 20-person outreach list.
4. **Build:** send the first five messages.
5. **Prove:** explain why each person is a fit.

**Done when:** outreach has started.

### Day 62 — Final audit
1. **Setup:** question: "Which repo would you least want an AI lead to open?"
2. **Learn:** auditing against a DoD, finding the weakest link, honest maturity labels.
3. **Practice:** run the full DoD checklist.
4. **Build:** fix the top three gaps.
5. **Prove:** explain the weakest link and your fix.

**Done when:** every checklist item is true or documented as a gap.

### Day 63 — Teach-back and publish
1. **Setup:** question: "Can you explain the capstone, tradeoffs, and limits in 5 minutes?"
2. **Learn:** teaching as a test, turning a sprint into a post, honest limitations.
3. **Practice:** explain the capstone to a non-expert.
4. **Build:** publish one post.
5. **Prove:** answer questions without notes.

**Done when:** the post is live and the explanation lands.

### Day 64 — Retro and the next 90 days
1. **Setup:** question: "What is the next thing you will learn, and why?"
2. **Learn:** what you now know, remaining gaps, portfolio maintenance, choosing depth.
3. **Practice:** write a one-page retro.
4. **Build:** write the next-90-days plan; final portfolio polish.
5. **Prove:** present the retro and plan. Close the session.

**Done when:** the retro is written and the portfolio is live.


## If you fall behind

1. **Never skip step 5** (prove + log). A skipped log means the week did not happen.
2. **Cut practice, not build.** The repo is the portfolio.
3. **Cut scope, not evidence.** Fewer models, fewer tasks, fewer tools — but keep the eval and the ADR.
4. **Roll one day, not the sprint.** If you miss a day, compress the practice step and protect the build.
5. **At the end of each sprint, stop.** Do not start the next topic with an unpolished repo.
