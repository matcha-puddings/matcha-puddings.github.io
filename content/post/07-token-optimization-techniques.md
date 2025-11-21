+++
date = "2025-08-23T09:00:00-05:00"
title = "Token Optimization: 12 Techniques That Actually Save Money"
tags = ["Token Optimization", "LLM Costs", "Prompt Engineering", "Agents", "Evaluation"]
draft = false
description = "Twelve battle-tested tactics technical teams use to slash LLM token spend while keeping accuracy and UX intact."
+++

#### Introduction
Model vendors keep shipping cheaper SKUs, but real-world AI systems still burn cash because context windows bloat and agents loop uncontrollably. Teams that treat token spend like an SRE metric consistently keep budgets in line. The following twelve techniques are drawn from production deployments across support copilots, ETL agents, and compliance reviewers.

#### 1. Context Pruning & Compression
Start by analyzing traces to see which context blocks the model actually cites. Anything that never appears in outputs gets trimmed or summarized. For hot documents, create prebaked one-paragraph briefs and only send the full text if the user explicitly asks. Combine this with retrieval heuristics—e.g., “never include more than two documents from the same repo”—to avoid redundant passages.

**Implementation checklist:**
- Track token share per context source.
- Build summarization jobs that produce short/medium/long forms.
- Add guardrails (max chunks per source, deduplication by hash).

#### 2. Structured Prompts
Prompts written like JSON schemas instead of prose shrink instruction tokens and keep outputs tight. Define explicit fields, accepted values, and examples. Use template engines to keep prompts consistent, and run linters to enforce maximum length. Structured prompts also pair nicely with automatic validation before responses hit users.

#### 3. Rolling Memory & Session Summaries
Conversational apps waste tokens replaying entire histories. Move to rolling memory: after each exchange, summarize the conversation into a canonical state (facts known, decisions made, open questions). Persist only those summaries plus critical raw turns. You now feed a few hundred tokens instead of thousands on every request.

#### 4. Smaller Models as Filters
Insert an inexpensive model (Claude Haiku, GPT-4o mini, Llama 3 8B) in front of premium models. Use it to route intent, classify sentiment, detect duplicates, or answer FAQ-level questions. Measure deflection rate the same way support teams measure ticket deflection. Every percentage point routed away from the flagship model is immediate savings.

#### 5. Caching + Delta Generation
A shocking amount of traffic repeats identical or near-identical questions. Hash prompts + context to build a cache, and store not just responses but the underlying structured results (SQL output, API payloads). When requests only change incrementally, send the delta to the LLM along with the cached answer so it can edit instead of regenerate from scratch.

#### 6. Input Validation
Do not send malformed or incomplete requests to the LLM. Validate schemas, required parameters, and authentication before constructing the prompt. Bounce the request back to the user or UI layer if it is missing core signals. This prevents the classic “model asks three clarifying follow-ups” cycle that multiplies tokens.

#### 7. Tool-First Design
Let deterministic tools handle calculations, lookups, and formatting. The agent should plan with the LLM and execute with tools. For example, call a ledger API to fetch balances instead of asking the model to infer them from text. Tool outputs are short and structured, keeping the context window small while improving accuracy.

#### 8. Prompt Versioning & Testing
Token creep is real. Check prompts into Git, diff them, and run regression suites whenever they change. Pair this with telemetry: track average input/output tokens per prompt version. Rollback if a new prompt inflates cost without quality gains. Treat prompts like code, not copy.

#### 9. Adaptive Sampling Parameters
Batch workloads (report generation, contract summaries) do not all deserve the same budget. Assign profiles: “critical” requests get generous `max_tokens` and low temperature; “routine” requests get strict limits or even deterministic templates. Combine this with heuristics (document length, risk rating, SLA) to pick the cheapest acceptable configuration automatically.

#### 10. Multi-Resolution Retrieval
Store documents at multiple resolutions: snippets, abstracts, and full text. Retrieval first serves the smallest chunk that plausibly answers the question. If the model signals low confidence or the evaluation layer flags missing evidence, the system escalates to larger chunks. This adaptive context keeps token counts proportional to the question’s complexity.

#### 11. Budget-Aware Agents
Instrument every agent loop with a running token meter. Expose APIs like `get_remaining_budget()` so prompts can literally reason about cost: “If remaining budget < 2, switch to fallback plan.” Emit metrics per tool call and per user to enable chargebacks or alerts when spend spikes.

#### 12. Learn from Real Incidents
Open-source repos (LangChain, LlamaIndex, Goose AI) document token blowups caused by naive chunking or infinite loops. Build automated checks for those patterns—e.g., fail CI if a workflow’s theoretical worst-case token count exceeds a threshold. Make cost regressions visible alongside latency regressions.

#### Practical Savings Breakdown
Teams combining at least five of the techniques above often report:
- 15% from pruning/compression
- 10% from small-model routing
- 5% from caching/deltas
- 10% from budget-aware orchestration

The key is observability. Once tokens become a first-class metric, engineers can treat savings initiatives like any other performance effort.

#### Conclusion
Token optimization is not a plea for users to type shorter prompts—it is an engineering practice spanning retrieval, orchestration, and evaluation. Structure context, instrument agents, prefer tools, and make prompts contractual. Your finance team will notice, and your users will see faster, more consistent responses.
