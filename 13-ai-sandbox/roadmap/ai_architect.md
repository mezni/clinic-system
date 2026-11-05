# AI Architect — Day-by-Day Learning Plan

A 12-week, 64-day curriculum for a Python/Rust developer switching into an AI architect role.
Each day pairs **concepts to learn** with a small **practice** and the **build** it feeds.

> Companion to `roadmap.html`, `project-ideas.md`, and `SPEC-ai-architect-portfolio`. The goal is not to
> cover everything; it is to be able to defend the architecture decisions you make.

---

## How to use this

- **Time:** ~2.5 hrs/day. Suggested split: 45 min read, 60 min practice, 30 min build, 15 min notes.
- **Method:** concept → small practice → apply in the sprint project → teach it back in one paragraph.
- **Spacing:** every Friday, re-answer 5 checkpoints from earlier days without notes.
- **Rule:** if you cannot explain a topic to an AI lead in 60 seconds, you have not learned it yet.
- **Resources:** use official docs and primary sources by name (model provider docs, the MCP specification,
  OpenTelemetry docs, Google SRE book, NIST AI RMF, OWASP Top 10 for LLM Applications). Prefer specs over blog posts.
- **Depth control:** learn enough to make and defend a decision. You are not doing ML research.
- **Anchor:** Rust networking/telecom-adjacent codebase for retrieval; telecom NOC copilot for the capstone.
- **Constraints:** local reproducible demos, under $50/month, intermediate Rust (indexer, gateway, MCP server only).

---

## Topic coverage map

| Competency | Days |
|---|---|
| Architecture & decision records | 8, 18, 41–50, 60 |
| System design | 14, 18, 23, 31, 38, 41–50 |
| LLMOps | 3, 5, 6, 32–37, 53 |
| Production-ready agents | 21–30, 35, 39, 53–55 |
| RAG | 11–20, 46, 53 |
| MCP | 23–28, 47 |
| Security & governance | 9, 26–27, 36, 38, 42, 45 |

---

# Month 1 — Foundations and retrieval

## Week 1 — Foundations and model selection

### Day 1 — The AI architect role and the system landscape
- **Learn**
  - What an AI architect actually owns: boundaries, tradeoffs, risk, operability — not model training.
  - AI vs ML vs LLM vs agent; where each fits.
  - The architecture concerns: data, model, retrieval, tools, evaluation, operations, cost, safety.
  - Build vs buy; API model vs self-hosted; when *not* to use AI.
  - Reference workloads: what the portfolio must be representative of.
- **Practice:** write three reference workloads and one "we should not use AI here" case.
- **Build tie-in:** Sprint 1 problem statement and workload definition.
- **Checkpoint:** Explain in 60 seconds why "use an LLM" is an architecture decision, not a feature.

### Day 2 — LLM fundamentals at architecture level
- **Learn**
  - Tokens and tokenization; why token counts drive cost and limits.
  - Context windows; input vs output budgets; truncation behavior.
  - Embeddings conceptually; similarity and semantic search.
  - Sampling: temperature, top-p; determinism and reproducibility limits.
  - Structured output and function/tool calling as an interface.
  - Model families and capability envelopes; reasoning vs fast models.
- **Practice:** read three model spec sheets; tabulate context, price, modality, structured-output support.
- **Build tie-in:** provider adapters and model shortlist.
- **Checkpoint:** Why can two runs of the same prompt differ, and what does that mean for tests?

### Day 3 — Evaluation design
- **Learn**
  - Benchmarks vs task-specific evals; why leaderboards mislead.
  - Eval sets: size, coverage, contamination, representativeness.
  - Scoring: exact match, schema validity, rubric, LLM-as-judge, human spot-check.
  - Quality vs latency vs cost as a Pareto tradeoff.
  - Failure taxonomy: over-refusal, format drift, hallucination, truncation, wrong label, timeout.
- **Practice:** define five task categories and a rubric for each.
- **Build tie-in:** `tasks.jsonl` and scoring design.
- **Checkpoint:** What makes an eval set trustworthy, and what makes it gameable?

### Day 4 — Prompting and structured output as a contract
- **Learn**
  - System vs developer vs user messages; instruction hierarchy.
  - Few-shot examples; when they help and when they bias.
  - JSON schema, validation, repair, and retry strategies.
  - Prompt versioning and prompt-as-code.
  - Tool schemas and argument design.
- **Practice:** write and validate prompts for the five task categories.
- **Build tie-in:** runner prompts and schema validation.
- **Checkpoint:** Where should schema validation live — prompt, model, or client — and why?

### Day 5 — Cost, latency, and reliability economics
- **Learn**
  - Token accounting: prompt, completion, cached tokens.
  - Price per token; cost per request and per 1k tasks.
  - Latency: p50 vs p95 vs tail; first-token vs total.
  - Streaming, batching, caching, retries, timeouts, rate limits.
  - Fallback models and graceful degradation.
- **Practice:** build a price table and instrument p50/p95.
- **Build tie-in:** cost/latency columns in the decision matrix.
- **Checkpoint:** Your p50 is fine but p95 is terrible — what do you investigate first?

## Week 2 — Measurement, reliability, and governance

### Day 6 — Evaluation harness engineering
- **Learn**
  - Reproducibility: seeds, temperature, versions, run metadata.
  - Dataset versioning and experiment tracking.
  - CI for evals; golden sets; regression gates.
  - Statistical caution: small samples, variance, confidence.
- **Practice:** run the same eval twice and compare variance.
- **Build tie-in:** reproducible runs and the decision matrix generator.
- **Checkpoint:** What metadata must a run record for someone else to trust the result?

### Day 7 — Failure analysis and robustness
- **Learn**
  - Hallucination vs grounding vs refusal; they are different failures.
  - Over-refusal and safety overcorrection.
  - Format drift and downstream breakage.
  - Prompt injection: direct and indirect; data exfiltration.
  - Failure taxonomies and root-cause analysis.
- **Practice:** categorize 20 real failures from your baseline.
- **Build tie-in:** failure taxonomy and limitations section.
- **Checkpoint:** How do you distinguish "model is wrong" from "our eval is wrong"?

### Day 8 — Decision records and architecture communication
- **Learn**
  - ADR structure: context, drivers, options, evidence, decision, consequences, revisit triggers.
  - Writing tradeoffs for an AI lead: what they will challenge.
  - Diagrams: context, component, sequence; when each earns its place.
  - The difference between a decision and a preference.
- **Practice:** write ADR-001 from the model-selection evidence.
- **Build tie-in:** ADR-001 and README.
- **Checkpoint:** Defend your model choice against "but the leaderboard says otherwise."

### Day 9 — Security, privacy, and governance basics
- **Learn**
  - PII handling; data minimization; retention; data residency.
  - Provider data-use terms and enterprise controls.
  - OWASP Top 10 for LLM Applications (overview).
  - NIST AI Risk Management Framework (overview: govern, map, measure, manage).
  - EU AI Act risk tiers (overview).
- **Practice:** list the data flows in Sprint 1 and mark sensitive data.
- **Build tie-in:** secrets handling and README limitations.
- **Checkpoint:** What data should never leave your boundary, and how would you enforce it?

### Day 10 — Sprint review and teach-back
- **Learn**
  - Synthesis: what the evidence actually says.
  - Explaining uncertainty and limitations honestly.
  - Retrospective: what to change in the next sprint's process.
- **Practice:** present the model decision to a peer in 5 minutes; take objections.
- **Build tie-in:** sprint retro and portfolio index entry.
- **Checkpoint:** Can you explain the decision, the tradeoff, and the risk in under 3 minutes?

---

## Week 3 — RAG foundations

> Anchor every retrieval exercise to the chosen Rust networking or telecom-adjacent codebase.

### Day 11 — Why retrieval
- **Learn**
  - Knowledge cutoff, private data, and grounding.
  - RAG vs fine-tuning vs long context: cost, freshness, control.
  - Retrieval pipeline: ingest → index → retrieve → rank → assemble → generate.
  - When RAG is the wrong answer.
- **Practice:** write the retrieval problem statement and success criteria.
- **Build tie-in:** Sprint 2 scope and question set.
- **Checkpoint:** Name two cases where fine-tuning beats RAG, and two where it does not.

### Day 12 — Ingestion and chunking
- **Learn**
  - Parsing structured sources; code vs prose vs tables.
  - Chunking strategies: fixed, semantic, structure-aware, parent-child.
  - Metadata: source, location, permissions, timestamps.
  - Overlap, chunk size, and retrieval granularity tradeoffs.
- **Practice:** chunk a Rust codebase by item and record line ranges.
- **Build tie-in:** Rust indexer/chunker.
- **Checkpoint:** Why is "chunk by 512 tokens" usually a bad default for code?

### Day 13 — Embeddings and vector search
- **Learn**
  - Embedding models and dimensions; normalization.
  - Similarity metrics: cosine, dot product, Euclidean.
  - ANN indexes: HNSW, IVF; recall vs latency vs memory.
  - Index build vs query cost.
- **Practice:** compare two embedding models on the same chunks.
- **Build tie-in:** embedding choice and index build.
- **Checkpoint:** What are you trading away when you tune ANN for speed?

### Day 14 — Hybrid retrieval and reranking
- **Learn**
  - Sparse (BM25) vs dense retrieval; why hybrids win.
  - Fusion: reciprocal rank fusion, weighted scores.
  - Cross-encoder rerankers vs bi-encoders.
  - Recall vs precision and the two-stage funnel.
- **Practice:** add BM25 + vector fusion, then a reranker; measure recall.
- **Build tie-in:** retrieval configuration and eval comparison.
- **Checkpoint:** Why can adding a reranker *hurt* latency-sensitive systems?

### Day 15 — Context assembly and citations
- **Learn**
  - Context-window budgeting; truncation policy.
  - Lost-in-the-middle and ordering effects.
  - Citation grounding; answerability and refusal when unsupported.
  - Answer synthesis without inventing.
- **Practice:** force citations and test unsupported questions.
- **Build tie-in:** cited answer path.
- **Checkpoint:** How do you make "I don't know" a first-class output?

## Week 4 — RAG evaluation and service design

### Day 16 — Retrieval evaluation
- **Learn**
  - Recall@k, precision@k, MRR, nDCG.
  - Groundedness/faithfulness, answer relevance, citation accuracy.
  - Building labeled question sets; inter-annotator agreement.
  - Offline vs online eval.
- **Practice:** compute recall@k and faithfulness on the frozen set.
- **Build tie-in:** retrieval eval suite.
- **Checkpoint:** Which metric do you optimize first, and what does it hide?

### Day 17 — RAG failure modes and hardening
- **Learn**
  - Retrieval misses: vocabulary gap, multi-hop, stale index.
  - Embedding drift and re-indexing policy.
  - Permission-aware retrieval and document-level ACLs.
  - Prompt injection through retrieved content.
- **Practice:** root-cause 10 retrieval failures.
- **Build tie-in:** failure analysis and hardening backlog.
- **Checkpoint:** How does retrieved untrusted content become an attack surface?

### Day 18 — Service boundaries and the Rust/Python split
- **Learn**
  - When Rust earns its place: ingestion, performance, memory safety, protocol servers.
  - When Python earns its place: ecosystem, orchestration, evals.
  - IPC and contracts: HTTP, gRPC, FFI, message queues.
  - Failure isolation and independent deployability.
- **Practice:** draw the boundary and write the contract.
- **Build tie-in:** ADR-002.
- **Checkpoint:** Defend the split without saying "because I like Rust."

### Day 19 — Retrieval cost, latency, and caching
- **Learn**
  - Embedding cost vs index build vs query cost.
  - Caching layers: embeddings, retrieval, answers.
  - Batching and async ingestion.
  - Latency budgets for interactive vs batch.
- **Practice:** measure index build and query p95.
- **Build tie-in:** cost/latency evidence.
- **Checkpoint:** Where is the cheapest place to cache in a RAG pipeline?

### Day 20 — Sprint review and ADR defense
- **Learn**
  - Presenting retrieval quality honestly.
  - Connecting eval numbers to product decisions.
- **Practice:** defend ADR-002 in a mock review.
- **Build tie-in:** sprint retro and portfolio entry.
- **Checkpoint:** What would you do differently with twice the budget? Half the latency budget?

---

# Month 2 — Agents, MCP, and operations

## Week 5 — Agents and MCP

> Use telecom-flavored tools — simulated alarms, metrics, tickets — so the tool contract matches the capstone.

### Day 21 — Agent fundamentals
- **Learn**
  - The agent loop: observe → plan → act → observe.
  - Tool use, memory, state, and termination conditions.
  - Single-agent vs multi-agent; when multi-agent adds cost without benefit.
  - Loop budgets and failure containment.
- **Practice:** design one bounded agent task with an explicit stop condition.
- **Build tie-in:** Sprint 3 scope.
- **Checkpoint:** What are three ways an agent loop fails to terminate, and how do you bound them?

### Day 22 — Tool design and contracts
- **Learn**
  - Tool schema design: names, descriptions, arguments, outputs.
  - Idempotency, retries, and side effects.
  - Error surfaces the model can understand.
  - Validation and least privilege.
- **Practice:** write contracts for three tools.
- **Build tie-in:** Rust MCP tool surface.
- **Checkpoint:** Why is a vague tool description an architecture bug?

### Day 23 — MCP protocol
- **Learn**
  - Model Context Protocol: hosts, clients, servers.
  - Tools, resources, prompts; capability negotiation.
  - JSON-RPC; stdio vs HTTP transports.
  - Lifecycle and error handling.
- **Practice:** trace a `tools/call` request and response.
- **Build tie-in:** MCP server skeleton.
- **Checkpoint:** What does MCP standardize that ad-hoc tool wiring does not?

### Day 24 — Building an MCP server in Rust
- **Learn**
  - Server lifecycle and capability declaration.
  - Tool registration and dispatch.
  - Transport handling and concurrency.
  - Structured errors and timeouts.
- **Practice:** implement `initialize`, `tools/list`, `tools/call`.
- **Build tie-in:** working Rust MCP server.
- **Checkpoint:** Where do you enforce a tool timeout — inside the tool or in the server loop?

### Day 25 — Agent orchestration in Python
- **Learn**
  - Client loop and context management.
  - Tool-call parsing and result injection.
  - Iteration limits, retries, and observability hooks.
  - Model choice for tool-use reliability.
- **Practice:** build the client loop against the server.
- **Build tie-in:** Python agent.
- **Checkpoint:** How do you keep the agent from looping forever on a failing tool?

## Week 6 — Trust, safety, and testing for agents

### Day 26 — Trust boundaries and permissions
- **Learn**
  - Authn vs authz; service identity and user identity.
  - Least privilege, allowlists, scopes.
  - Blast radius: what one bad call can touch.
  - Audit logging and non-repudiation.
- **Practice:** write a permission matrix for the tool surface.
- **Build tie-in:** ADR-003 trust model.
- **Checkpoint:** What is the blast radius of the most powerful tool you expose?

### Day 27 — Agent security threats
- **Learn**
  - Prompt injection and indirect injection.
  - Tool poisoning and malicious tool descriptions.
  - Confused deputy, excessive agency, data exfiltration.
  - Sandboxing and output filtering.
- **Practice:** red-team your own agent with three attacks.
- **Build tie-in:** guardrails and failure tests.
- **Checkpoint:** How does a retrieved document become a command?

### Day 28 — Testing agents
- **Learn**
  - Unit vs integration vs trajectory evaluation.
  - Determinism, fixtures, and recorded transcripts.
  - Failure injection: timeout, bad args, denial.
  - Evaluating tool choice, not just final answer.
- **Practice:** write integration tests for server + agent.
- **Build tie-in:** test suite.
- **Checkpoint:** How do you test a non-deterministic agent without flaky CI?

### Day 29 — Agent cost, latency, and guardrails
- **Learn**
  - Loop budget: max iterations, max tokens, max cost.
  - Model routing and cheap-first strategies.
  - Timeouts, circuit breakers, fallbacks.
  - Input/output guardrails.
- **Practice:** add a loop budget and measure cost per task.
- **Build tie-in:** cost evidence and guardrails.
- **Checkpoint:** What is your cost ceiling per task, and what happens when it is hit?

### Day 30 — Sprint review and trust-model defense
- **Learn**
  - Explaining trust boundaries to an AI lead.
  - Residual risk and mitigations.
- **Practice:** present the permission matrix and blast radius.
- **Build tie-in:** sprint retro and portfolio entry.
- **Checkpoint:** What risk remains after your mitigations, and who owns it?

## Week 7 — Production and LLMOps

### Day 31 — Production readiness
- **Learn**
  - SLI/SLO/SLA; error budgets.
  - Capacity planning and headroom.
  - Deployment strategies: blue/green, canary, rollback.
  - Configuration and secrets management.
- **Practice:** define three SLOs for the agent service.
- **Build tie-in:** Sprint 4 release contract.
- **Checkpoint:** Which SLO would you page on, and why that one?

### Day 32 — Observability for LLM systems
- **Learn**
  - Traces, spans, attributes; OpenTelemetry concepts.
  - Structured logs and correlation IDs.
  - Metrics: tokens, cost, latency, errors, tool calls.
  - Privacy-preserving prompt/response capture.
- **Practice:** add a trace to one agent run.
- **Build tie-in:** tracing and structured logs.
- **Checkpoint:** What must a trace contain to debug a wrong answer three days later?

### Day 33 — Evals in CI
- **Learn**
  - Golden sets and regression gates.
  - Eval artifact storage and versioning.
  - Release criteria and promotion.
  - Handling flaky and subjective evaluations.
- **Practice:** put the eval suite in CI and fail on regression.
- **Build tie-in:** eval gate.
- **Checkpoint:** What is the smallest eval that would have caught your last regression?

### Day 34 — Prompt and model lifecycle
- **Learn**
  - Prompt versioning and ownership.
  - Model upgrades, deprecations, and compatibility.
  - Canary, A/B, and rollback for prompts and models.
  - Change management.
- **Practice:** version a prompt and run a canary comparison.
- **Build tie-in:** lifecycle notes in ADR-004.
- **Checkpoint:** How do you roll back a prompt change without redeploying the app?

### Day 35 — Incident response and postmortems
- **Learn**
  - Incident lifecycle: detect, triage, mitigate, resolve.
  - Runbooks and on-call.
  - Blameless postmortems and contributing factors.
  - Follow-up tracking.
- **Practice:** write a runbook for the agent service.
- **Build tie-in:** runbook and postmortem.
- **Checkpoint:** What is the first mitigation for a provider outage?

## Week 8 — Governance and operations at scale

### Day 36 — Guardrails and moderation
- **Learn**
  - Input, output, and tool guardrails.
  - PII redaction and secret detection.
  - Jailbreak and prompt-injection detection.
  - Policy vs model behavior.
- **Practice:** add one input and one output guardrail.
- **Build tie-in:** guardrails and tests.
- **Checkpoint:** What should the system do when a guardrail trips?

### Day 37 — Cost governance
- **Learn**
  - Budgets, quotas, and chargeback.
  - Cost per request, per tenant, per feature.
  - Token accounting and caching economics.
  - Alerting on anomalous spend.
- **Practice:** build a cost dashboard from traces.
- **Build tie-in:** cost budget evidence.
- **Checkpoint:** How do you stop a runaway loop before it becomes a bill?

### Day 38 — Data governance and compliance
- **Learn**
  - GDPR basics: lawful basis, retention, deletion.
  - EU AI Act risk tiers (overview).
  - SOC 2 and vendor risk (overview).
  - Auditability and data lineage.
- **Practice:** map data flows to retention and deletion rules.
- **Build tie-in:** capstone threat/governance notes.
- **Checkpoint:** What does "delete my data" mean for embeddings and logs?

### Day 39 — Reliability patterns
- **Learn**
  - Retries with jitter and backoff.
  - Circuit breakers and bulkheads.
  - Multi-provider fallback.
  - Graceful degradation and user-facing behavior.
- **Practice:** test a provider-outage fallback.
- **Build tie-in:** reliability section of ADR-004.
- **Checkpoint:** When should you fail fast instead of retrying?

### Day 40 — Sprint review and runbook walkthrough
- **Learn**
  - Operating a system you did not build.
  - On-call readiness.
- **Practice:** have a peer follow the runbook; note every gap.
- **Build tie-in:** sprint retro and portfolio entry.
- **Checkpoint:** Could someone else run this service from your runbook alone?

---

# Month 3 — Capstone and the switch

## Week 9 — Capstone architecture

> Capstone: `telecom-noc-copilot` — multi-tenant RAG over the Rust codebase and telecom runbooks, MCP tools for simulated alarms/metrics/tickets, local reproducible demo.

### Day 41 — Multi-tenant architecture
- **Learn**
  - Isolation models: shared table, schema per tenant, index per tenant.
  - Noisy neighbor and fairness.
  - Per-tenant config, keys, quotas.
  - Tenant lifecycle: onboarding, offboarding, deletion.
- **Practice:** choose an isolation model and write the tradeoffs.
- **Build tie-in:** tenancy ADR.
- **Checkpoint:** Which isolation model do you choose for 10 tenants vs 10,000, and why?

### Day 42 — Identity and access architecture
- **Learn**
  - OIDC and OAuth 2.0 flows at architecture level.
  - RBAC vs ABAC; tenant-scoped roles.
  - Service-to-service auth.
  - Secrets management.
- **Practice:** draw the auth flow end to end.
- **Build tie-in:** auth design and ADR.
- **Checkpoint:** Where does tenant identity enter the request path, and where is it enforced?

### Day 43 — Architecture documentation
- **Learn**
  - C4 model: context, container, component.
  - Sequence diagrams for critical paths.
  - Interface contracts and data ownership.
  - Documentation that stays true.
- **Practice:** write context and component diagrams.
- **Build tie-in:** capstone architecture doc.
- **Checkpoint:** Could a new engineer implement a component from your diagram alone?

### Day 44 — SLOs, capacity, and cost modeling
- **Learn**
  - Workload estimation: requests, tokens, tenants.
  - Unit economics per tenant/request.
  - Headroom and scaling assumptions.
  - Cost drivers and levers.
- **Practice:** build a cost model with three scenarios.
- **Build tie-in:** SLO and cost docs.
- **Checkpoint:** What breaks first under 10x load — and what does it cost?

### Day 45 — Threat modeling
- **Learn**
  - STRIDE: spoofing, tampering, repudiation, information disclosure, denial, elevation.
  - Trust boundaries and attack trees.
  - Mitigations and residual risk.
  - AI-specific threats: injection, exfiltration, model abuse.
- **Practice:** run STRIDE across the capstone components.
- **Build tie-in:** threat model.
- **Checkpoint:** Which threat is most likely, and which is most damaging?

## Week 10 — Capstone design decisions

### Day 46 — Retrieval at multi-tenant scale
- **Learn**
  - Per-tenant indexes vs shared index with ACL filters.
  - ACL-aware retrieval and leakage risks.
  - Re-indexing and freshness per tenant.
  - Embedding cost at scale.
- **Practice:** design ACL-aware retrieval and test the failure case.
- **Build tie-in:** retrieval ADR.
- **Checkpoint:** How do you prove tenant A cannot retrieve tenant B's chunk?

### Day 47 — Tool trust in multi-tenant systems
- **Learn**
  - Per-tenant tool permissions.
  - Sandboxing and side-effect control.
  - Audit trails per tenant.
  - Delegation and impersonation risks.
- **Practice:** extend the permission matrix to tenants.
- **Build tie-in:** tool-trust ADR.
- **Checkpoint:** What stops a tenant from invoking another tenant's tool?

### Day 48 — Gateway and orchestration design
- **Learn**
  - API gateway responsibilities: auth, rate limits, routing, quotas.
  - Rust gateway vs Python orchestration boundaries.
  - Backpressure and queueing.
  - Failure propagation and timeouts.
- **Practice:** write API contracts and sequence diagrams.
- **Build tie-in:** gateway/orchestration ADRs.
- **Checkpoint:** Where do you enforce quotas — gateway or orchestrator — and why?

### Day 49 — Design review and tradeoff defense
- **Learn**
  - Presenting a design under challenge.
  - Separating reversible from irreversible decisions.
  - Recording dissenting options.
- **Practice:** run a mock design review; record objections and responses.
- **Build tie-in:** revised ADR set.
- **Checkpoint:** Which decision would you most regret getting wrong?

### Day 50 — Build planning and risk
- **Learn**
  - Milestones, critical path, and cut lists.
  - Risk registers and contingencies.
  - Definition of done for a capstone.
- **Practice:** write the week-11 plan with an explicit cut list.
- **Build tie-in:** build plan.
- **Checkpoint:** What will you cut first if week 11 slips?

## Week 11 — Build and harden

### Day 51 — Integration architecture
- **Learn**
  - Wiring RAG, MCP, agent, gateway, orchestration.
  - Contract testing across services.
  - Error propagation and retries.
  - Configuration across environments.
- **Practice:** get one request through the full path.
- **Build tie-in:** working skeleton.
- **Checkpoint:** Where is the most likely integration failure, and how will you see it?

### Day 52 — Tenant isolation testing
- **Learn**
  - Cross-tenant authorization tests.
  - Data leakage through logs, caches, embeddings.
  - Quota enforcement under load.
  - Offboarding and deletion.
- **Practice:** write and run cross-tenant denial tests.
- **Build tie-in:** isolation evidence.
- **Checkpoint:** Name three non-obvious places tenant data can leak.

### Day 53 — Evals and observability in the capstone
- **Learn**
  - End-to-end evals across retrieval, tools, and generation.
  - Tracing across service boundaries.
  - Dashboards for quality, cost, latency.
  - Per-tenant metrics.
- **Practice:** run the full eval suite and capture traces.
- **Build tie-in:** eval and observability evidence.
- **Checkpoint:** What is the one dashboard you would show an AI lead first?

### Day 54 — Load and failure testing
- **Learn**
  - Load profiles and realistic traffic.
  - Failure injection and chaos basics.
  - Degradation modes and user experience.
  - Capacity limits and backpressure.
- **Practice:** run a load test and one failure simulation.
- **Build tie-in:** load/failure results.
- **Checkpoint:** How does the system behave at 2x capacity — and is that acceptable?

### Day 55 — Postmortem and iteration
- **Learn**
  - Writing an honest postmortem.
  - Updating ADRs when reality diverges from design.
  - Deciding what not to fix.
- **Practice:** write the capstone postmortem and update the ADRs.
- **Build tie-in:** postmortem and final demo.
- **Checkpoint:** What did the design get wrong, and what did you learn?

## Week 12 — Portfolio and career

### Day 56 — Personal positioning
- **Learn**
  - The architect narrative: judgment, not just skills.
  - Two doors: AI lead and client.
  - Proof-of-judgment framing.
  - What to leave out.
- **Practice:** write the profile README and the 30-second pitch.
- **Build tie-in:** profile README.
- **Checkpoint:** What should a stranger believe about you after 5 minutes?

### Day 57 — Technical writing
- **Learn**
  - Case-study structure: problem → options → tradeoff → outcome.
  - README structure for reviewers.
  - Writing ADRs people actually read.
  - Numbers and diagrams over adjectives.
- **Practice:** write case studies for sprints 1–3.
- **Build tie-in:** case studies.
- **Checkpoint:** Does each case study prove a decision, not just describe work?

### Day 58 — Portfolio information architecture
- **Learn**
  - Navigation and discoverability.
  - Evidence links in two clicks or fewer.
  - Consistency across repos.
  - Accessibility and readability.
- **Practice:** build the portfolio index and fix broken links.
- **Build tie-in:** portfolio index.
- **Checkpoint:** Can a reviewer find your best evidence without asking you?

### Day 59 — Interviewing as an AI architect
- **Learn**
  - System-design interviews for AI systems.
  - Tradeoff and failure-mode questions.
  - Whiteboard architecture: context → components → data flow → tradeoffs.
  - Behavioral framing of a career switch.
- **Practice:** do one whiteboard design under time pressure.
- **Build tie-in:** interview notes.
- **Checkpoint:** Can you design a RAG/agent system on a whiteboard in 30 minutes?

### Day 60 — Mock reviews and feedback
- **Learn**
  - Receiving and filtering critique.
  - Separating taste from substance.
  - Iterating without endless polishing.
- **Practice:** run two mock reviews; record feedback verbatim.
- **Build tie-in:** fixes across repos.
- **Checkpoint:** What did reviewers consistently misunderstand, and why?

### Day 61 — Outreach and networking
- **Learn**
  - Targeted outreach over spray-and-pray.
  - Message structure: context, evidence, ask.
  - Communities and where AI architects gather.
  - Follow-up discipline.
- **Practice:** build a 20-person outreach list and send the first five.
- **Build tie-in:** outreach list.
- **Checkpoint:** What is your specific ask, and why should they care?

### Day 62 — Final audit
- **Learn**
  - Auditing against a definition of done.
  - Finding the weakest link in the portfolio.
  - Honest maturity labeling (prototype vs production).
- **Practice:** run the full DoD checklist; fix the top three gaps.
- **Build tie-in:** final portfolio.
- **Checkpoint:** Which repo would you least want an AI lead to open, and why?

### Day 63 — Teach-back and publish
- **Learn**
  - Teaching as a test of understanding.
  - Turning a sprint into a post.
  - Explaining limitations without undermining confidence.
- **Practice:** publish one post and explain the capstone to a non-expert.
- **Build tie-in:** posts.
- **Checkpoint:** Can you explain the capstone, its tradeoffs, and its limits in 5 minutes?

### Day 64 — Retro and the next 90 days
- **Learn**
  - What you now know that you did not.
  - Gaps that remain and how to close them.
  - Maintaining the portfolio without over-polishing.
  - Choosing depth: evals, security, platform, or a domain.
- **Practice:** write a one-page retro and a next-90-days plan.
- **Build tie-in:** portfolio maintenance plan.
- **Checkpoint:** What is the next thing you will learn, and why that over everything else?

---

## Weekly review protocol

Every Friday, without notes:
1. Answer five checkpoints from earlier days.
2. Explain the week's hardest concept in one paragraph.
3. Update the learning-log issue with: learned, built, open questions.
4. Polish the previous repo before starting the next topic.

## What "learned" means

You can:
- explain the concept in plain language,
- name the tradeoff and when to choose the other side,
- point to evidence in your own repo,
- and state what would make you revisit the decision.

If any of those is missing, it is a review item, not a finished topic.
