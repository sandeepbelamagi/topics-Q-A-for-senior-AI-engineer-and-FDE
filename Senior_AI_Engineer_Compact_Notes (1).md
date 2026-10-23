# Senior AI Engineer — Q&A, Priorities, and FDE Guide

Purpose: quick revision and low-token reuse. All three earlier user questions are included below; responses are condensed, not verbatim. Added topics fill gaps in the original coverage. No fixed chunk size, model, or architecture is universally best.

## Priority ranking — study this first

Scope: Senior AI Engineer building production LLM applications. This is a practical study recommendation, not a universal hiring rubric. Rank 1 is the highest priority. Importance is different from learning order: Python, HTTP, SQL, and ML basics are prerequisites even when studied alongside higher-ranked system topics.

Scale: **Critical** = design, implement, measure, debug, and defend tradeoffs independently; **High** = strong working knowledge and practical implementation; **Medium** = explain mechanisms and make informed choices; **Low for this scope** = specialize when the job requires it. Low does not mean technically unimportant.

| Rank | Topic | Importance | Target depth |
|---|---|---|---|
| 1 | AI system design and architecture | Critical | Own an end-to-end design under real constraints |
| 2 | Evaluation and experimentation | Critical | Prove quality, detect regressions, define release gates |
| 3 | Python and software engineering | Critical | Build maintainable, tested, concurrent services |
| 4 | RAG and data ingestion quality | Critical | Diagnose missing, stale, or unusable evidence |
| 5 | Production debugging and observability | Critical | Isolate latency, quality, cost, and reliability failures |
| 6 | Security, permissions, and privacy | Critical | Prevent unauthorized reads/actions across all layers |
| 7 | HTTP, APIs, and integrations | Critical | Handle retries, deadlines, streaming, and side effects |
| 8 | SQL, databases, and data modeling | High | Maintain correctness, freshness, and query performance |
| 9 | Embeddings and vector/hybrid search | High | Tune retrieval quality versus latency/resources |
| 10 | Prompting, context, and structured output | High | Produce reliable, validated task outputs |
| 11 | Agents and tool orchestration | High | Build bounded, recoverable action workflows |
| 12 | Deployment, testing, and LLMOps | High | Ship versioned changes with rollout and rollback |
| 13 | ML and statistical foundations | High | Recognize leakage, shift, uncertainty, and weak evidence |
| 14 | Latency, inference serving, and cost | High | Meet quality and service targets within budget |
| 15 | Transformers | High: concepts; Medium: internals | Explain architecture and inference behavior |
| 16 | Attention mechanism | High: concepts; Medium: kernel details | Explain Q/K/V, masking, scaling, and cache tradeoffs |
| 17 | Tokenization | High: operational use | Budget context and prevent truncation/mismatch |
| 18 | Memory and state | High when agents/personalization are central | Scope, update, retain, and recover state correctly |
| 19 | Fine-tuning, LoRA, and QLoRA | Medium; High for adaptation roles | Justify adaptation and evaluate regressions |
| 20 | Cloud infrastructure and GPU internals | Medium; Critical for platform roles | Understand capacity, networking, identity, and memory |
| 21 | Multimodal AI | Medium when relevant | Handle modality-specific extraction and evaluation |
| 22 | Distributed model training | Low for API application roles; Critical for training roles | Understand sharding, synchronization, recovery |
| 23 | Custom CUDA kernels and architecture research | Low for this scope; High/Critical for specialist roles | Optimize or invent model internals when required |

**Your original 11, ranked:** System Design → Evaluation → Python → HTTP & APIs → Databases → Vector Search → Embeddings → Transformers → Attention → Tokenization → Memory. These adjacent ranks are close; agent-heavy roles move Memory much higher. Practical tokenization knowledge is required even without deep tokenizer-algorithm expertise.

**Study sequence:** Python/HTTP/SQL/ML basics → Transformer/attention/tokenization/embedding concepts → RAG/vector search → evaluation/system design/security → agents/state → deployment/debugging/cost → role-specific specialization.

**Suggested preparation allocation (planning guide):** 25% coding/APIs/data; 25% RAG/evaluation; 20% design/security; 20% deployment/debugging/agents; 10% model internals/specialization. Adjust to the job description and your weakest areas.

## 1. Original question: Core concepts

> why do embedding work?
> why does chunk size affect retrieval?
> why does an attention mechanism matter?
> what actually happens before an LLM generates the first token?
> explain me these in quick and understandable way

**Q: Why do embeddings work?**
A: Training organizes numerical representations so useful relationships become measurable. Retrieval models learn to score relevant queries/documents highly. “Reset password” can match “forgot login credentials” without identical words. Similarity is learned, imperfect, and not a truth/confidence score.

**Q: Why does chunk size affect retrieval?**
A: Small chunks can omit context (“It expires in 30 days”—what expires?). Large chunks mix topics, dilute relevance, and consume prompt space. Preserve meaningful sections; test size, overlap, and top-k together. Overlap helps continuity but adds duplication and storage.

**Q: Why does attention matter?**
A: It lets each token combine information from relevant available tokens. In “The animal stopped because it was tired,” it helps connect “it” to “animal.” Multiple heads learn different relationships. Attention weights alone do not explain all model reasoning.

**Q: What happens before the first generated token?**
A: Assemble instructions/history/retrieved evidence → tokenize → embed tokens and incorporate position → run Transformer layers (prefill) → calculate vocabulary scores → select a token. Subsequent decoding reuses cached keys/values where supported. Ordinary inference does not update weights.

## 2. Original question: Production failures

> also explain me if retrieval quality drops? if latency suddenly doubles? if hallucinations increase? if AI Agent gets stuck in a loop?

| Problem | Likely causes | First check | Response |
|---|---|---|---|
| Retrieval worsens | Ingestion gaps, stale index, bad chunks/filters, incompatible embeddings | Inspect results for known questions; verify answer exists in corpus | Repair failing stage; compare against baseline |
| Latency doubles | Queueing, longer context/output, slow tools, retries, provider load | Trace stage durations, input/output tokens, concurrency, cache hits | Optimize measured bottleneck; enforce deadlines |
| Hallucinations increase | Missing/conflicting evidence, ignored context, pressure to answer | Compare each factual claim with supplied evidence | Improve retrieval/instructions; verify citations; allow abstention |
| Agent loops | Unclear success, tool errors, lost state, repeated unproductive actions | Inspect action/input/result history and progress | Define completion; persist state; cap steps/retries/time/cost; escalate |

Debug pattern: reproduce → compare with last good version → isolate stage → change one variable → rerun evaluation → monitor/rollback.

## 3. Original question: Senior AI Engineer topics

> and also give me the notes or Question & answers on below topics for Senior AI Engineer:
>
> 1. Transformers
> 2. Attention mechanism
> 3. Embeddings
> 4. Tokenization
> 5. Vector Search
> 6. HTTP & APIs
> 7. System Design
> 8. Memory
> 9. Databases
> 10. Evaluation
> 11. Python fundamentals

### 3.1 Transformers

- **What?** Repeated attention and feed-forward layers, with normalization, residual connections, and positional information. Attention exchanges token information; feed-forward layers transform representations.
- **Architectures?** Encoder-only: bidirectional representations/classification. Decoder-only: causal generation. Encoder–decoder: input encoding plus conditioned generation.
- **Training vs inference?** Training computes loss/backpropagation and updates weights. Ordinary inference keeps weights fixed. Next-token training handles many positions together; ordinary output generation is sequential.
- **Prefill vs decode?** Prefill processes the prompt; decode generates subsequent tokens. Measure time to first token (TTFT), time per output token, and total latency separately.
- **KV cache?** Reuses previous attention keys/values; reduces repeated work but consumes memory as context/concurrency grow.

### 3.2 Attention

- **Q/K/V?** Query = information sought; key = matching signal; value = information contributed. All are learned projections.
- **Formula?** Attention(Q,K,V) = softmax(QKᵀ / √d_k + mask)V. Compare → scale/mask → normalize → combine values.
- **Causal mask?** Blocks future tokens during autoregressive modeling, including next-token training.
- **Multi-head?** Parallel learned relationships, combined afterward; heads have no guaranteed human-readable roles.
- **Complexity?** Full attention has O(n²d) attention computation. FlashAttention reduces memory traffic/materialization, not full attention's quadratic arithmetic dependence on sequence length.

### 3.3 Embeddings

- **Types?** Token representations and retrieval-oriented sentence/document embeddings serve different purposes; do not substitute them blindly.
- **Training?** Contrastive objectives can bring relevant pairs closer and separate negatives. Geometry depends on training data/objective.
- **Cosine?** cos(a,b) = (a·b)/(||a||||b||). For unit vectors, cosine equals dot product. Score ≠ probability of relevance.
- **Migration?** Version model, preprocessing, dimensions, metric, and index together. Different models usually require re-embedding even at equal dimensions.
- **Evaluation?** Test actual domain vocabulary, languages, identifiers, paraphrases, and hard negatives. More dimensions do not guarantee better retrieval.

### 3.4 Tokenization

- **What?** Text → token IDs; tokens may be words, fragments, punctuation, or bytes. Common algorithms: BPE, WordPiece, Unigram.
- **Why important?** Controls context usage, truncation, cost, latency, and chunk limits. Equal character counts need not produce equal token counts.
- **Compatibility?** Tokenizer/vocabulary/chat format must match the model's expectations.
- **Budget?** Account for instructions + history + retrieved text + tool definitions/results + reserved output. Count with the relevant tokenizer; tokenizers differ across embedding/generation models.

### 3.5 Vector search

- **Exact vs ANN?** Exact finds true nearest neighbors under the chosen metric; approximate nearest neighbor search trades some recall for speed/resources. Neither guarantees semantic relevance.
- **HNSW vs IVF?** HNSW navigates a proximity graph; IVF searches selected clusters. Tune candidate exploration against latency, recall, build cost, and memory.
- **Hybrid?** Combine lexical search (identifiers/exact terms) with vector search (meaning); merge rankings and optionally rerank.
- **Reranking?** More carefully score a candidate subset, often using query+document together. Adds latency; cannot recover missing candidates.
- **Filters?** Apply permissions before evidence enters the prompt. Filter placement and selectivity affect ANN recall and result counts; test realistic combinations.

### 3.6 HTTP & APIs

- **Production basics?** Schemas, input validation, authentication, authorization, versioning, deadlines, rate limits, connection pooling, request IDs, and predictable errors.
- **Idempotency?** Repeating an operation has the same intended effect. PUT/DELETE are idempotent by HTTP semantics; POST needs an application contract for safe duplication. Responses may differ across repeats.
- **Retries?** Only transient failures and safe-to-repeat operations; bounded exponential backoff+jitter, total deadline, server retry guidance. Timeout does not prove no action occurred.
- **Status codes?** 400 malformed request; 401 missing/invalid authentication; 403 forbidden; 404 not found; 429 rate-limited; 5xx server failures. Choose retry behavior from the operation and failure, not status alone.
- **Streaming?** Earlier partial output, not necessarily shorter total generation. Handle disconnects, cancellation, partial failure, and backend work continuing after disconnect.

### 3.7 System design

- **Clarify first?** Users/QPS, corpus size/freshness, access boundaries, quality goals, p95 latency, availability, and cost.
- **Ingestion?** Parse → clean → chunk → embed → index; track source/version/permissions and failed jobs.
- **Answering?** Authenticate/authorize → retrieve → rerank → assemble evidence → generate → validate/return citations.
- **RAG vs fine-tuning?** RAG supplies external/current knowledge; fine-tuning changes learned behavior/task performance. They can coexist; neither guarantees factual accuracy.
- **Scaling?** Bound concurrency, queue work, batch where appropriate, cache safely, use load shedding/fallbacks, and avoid retry storms.
- **Agent or workflow?** Prefer deterministic steps when the path is known. Add model-chosen actions only where flexibility produces measurable value.

### 3.8 Memory

- **Kinds?** Context = current model input; conversation store = persisted history; application memory = selected retrievable facts/experiences; weights = learned patterns; KV cache = inference intermediates.
- **Does a conversation ID imply recall?** No. Inspect which history/memory the application actually supplies.
- **What to store?** Useful facts with provenance, timestamp, user/tenant scope, retention, correction, and deletion controls; not every message by default.
- **Failure modes?** Stale/conflicting facts, lossy summaries, irrelevant retrieval, cross-user leakage. New evidence needs a defined update policy.

### 3.9 Databases

- **Roles?** Relational DB: transactions/joins/constraints. Vector index: similarity. Cache: reusable fast access. Object store: large files. One product may cover multiple roles.
- **ACID?** Atomicity: all-or-nothing. Consistency: defined rules hold. Isolation: controlled concurrent interaction. Durability: committed data survives within stated guarantees. Consistency does not establish AI factual truth.
- **Isolation?** Controls concurrent anomalies; stronger levels can increase contention/aborts and require retries. Database-specific behavior matters.
- **Index synchronization?** Stable IDs, source/content/embedding versions, idempotent jobs, deletion propagation, lag monitoring, reconciliation.
- **Slow SQL?** Inspect execution plan, row scans, joins, indexes, locks, and pool waits. Indexes accelerate suitable reads but add write/storage cost.

### 3.10 Evaluation

- **Separate layers?** Retrieval, generation, agent behavior, operations, and end-to-end user outcomes.
- **Retrieval metrics?** Recall@k: fraction of known relevant items found. Precision@k: fraction of top-k relevant. MRR: reciprocal rank of first relevant result, averaged. nDCG: graded relevance with position discount.
- **Generation metrics?** Correctness, groundedness, citation support, completeness, appropriate abstention. Groundedness alone is insufficient if source evidence is wrong.
- **Agent/operational metrics?** Task success, valid tool use, repeated/unsafe actions, p50/p95 latency, error rate, cost per successful task.
- **Dataset?** Representative verified cases, edge/unanswerable/permission/adversarial cases; held-out tests and slice-level results. Avoid train/test leakage.
- **LLM judges?** Useful but biased/imperfect. Calibrate against humans; use rubrics and deterministic checks where possible.

### 3.11 Python fundamentals

- **Mutability?** Lists/dicts/sets mutate; strings/numbers/tuples do not. Tuples may contain mutable objects. Assignment binds references; shallow copies still share nested objects.
- **Defaults?** Mutable defaults persist across calls. Use None, then create a new container inside the function.
- **Equality?** == compares values; is compares identity. Use `x is None`.
- **Concurrency?** Async: concurrent nonblocking I/O. Threads: blocking I/O/native code releasing GIL. Processes: CPU-heavy Python with serialization/memory overhead. GIL-enabled CPython generally cannot run Python bytecode in parallel threads; free-threaded builds differ.
- **Async pitfall?** `async def` does not make blocking code nonblocking. Bound concurrency, propagate cancellation, use timeouts, and avoid blocking the event loop.
- **Generators?** Lazy `yield`-based iteration saves memory; usually single-pass, with errors potentially deferred until consumption.
- **Resource safety?** Context managers, deliberate exception handling, reusable clients/pools, logging without secrets. Type hints are not runtime validation.
- **Also revise?** Decorators/closures, iterables vs iterators, hashing, dict/set lookup average O(1), list membership O(n), reference cycles/GC, testing/mocking, profiling, packaging/dependency pinning.

## 4. Missing topics — added Q&A

### 4.1 ML and statistics foundations

**Q: What should a senior engineer know beyond LLMs?**
A: Train/validation/test separation, leakage, overfitting, regularization, distribution shift, calibration, precision/recall, and uncertainty. Accuracy can hide minority-class failures. Use repeated measurements/confidence intervals before claiming small improvements.

### 4.2 Prompting, decoding, structured output

**Q: What makes a prompt reliable?**
A: Clear task, trusted instruction hierarchy, bounded evidence, explicit output schema, relevant examples, and a failure/abstention policy. Version and evaluate prompts like code.

**Q: Does temperature=0 guarantee truth or determinism?**
A: No. Temperature changes sampling, not factual knowledge. Greedy decoding can still be wrong; backend/numerical differences may affect reproducibility.

**Q: Does valid JSON mean a valid answer?**
A: No. Schema validation checks shape/types; validate business rules, references, permissions, and factual content separately.

### 4.3 RAG ingestion and data quality

**Q: What can fail before retrieval even starts?**
A: OCR, table extraction, document structure, deduplication, version handling, or metadata. A better embedding cannot recover facts never ingested correctly. Preserve source/page links and test extraction separately.

### 4.4 Agents and tool execution

**Q: What is tool calling?**
A: The model proposes a structured action; application code validates, authorizes, executes, and returns the result. A model proposal is not permission.

**Q: How do you make agents recoverable?**
A: Explicit state transitions, checkpoints, bounded budgets, idempotent side effects, action deduplication, and clear success/failure states. Resume from known state rather than blindly replaying actions.

### 4.5 Security and privacy

**Q: What is prompt injection?**
A: Untrusted content attempts to redirect model behavior. Treat retrieved pages/tool results as data, enforce authorization outside the model, use least-privilege tools, validate actions/outputs, and protect secrets. Prompt instructions alone are not a security boundary.

**Q: How do you avoid tenant leakage?**
A: Enforce identity/scope in databases, retrieval, tools, caches, and logs. Include tenant/permission context in cache keys; invalidate when access changes. Test denied-access scenarios.

### 4.6 Fine-tuning and adaptation

**Q: Full fine-tuning, LoRA, QLoRA?**
A: Full tuning updates all model weights. LoRA trains low-rank adapter updates while freezing base weights. QLoRA combines a quantized base with trainable adapters to reduce training memory. Validate held-out performance, regressions, and serving complexity; lower memory does not guarantee better quality.

### 4.7 Serving, GPUs, and cost

**Q: What consumes inference memory?**
A: Model weights, KV cache, activations, and runtime buffers. Concurrency and context length can exhaust memory even when weights fit.

**Q: How do you reduce cost/latency?**
A: Smaller suitable models, less redundant context/output, caching, batching, quantization, and request routing. Quantization lowers numeric precision; speedups depend on hardware/kernels and quality needs testing. Optimize cost per successful task, not token price alone.

**Q: Does batching always help?**
A: It can improve throughput/utilization but increase waiting time and tail latency. Benchmark realistic load and output lengths.

### 4.8 LLMOps and observability

**Q: What must be versioned?**
A: Model, tokenizer, prompt, tools, embedding/chunking/index configuration, datasets, and code. Record enough metadata to reproduce failures without leaking sensitive content.

**Q: What belongs in a trace?**
A: Request/version IDs, stage timings, token counts, retrieved source IDs/scores, tool attempts/results, cache hits, errors, and cost. Use redaction/access controls.

**Q: How should changes ship?**
A: Offline regression/safety evaluation → limited canary or A/B rollout → quality/latency/cost monitoring → rollback when thresholds fail. Monitor freshness and user outcomes, not only infrastructure health.

### 4.9 Broader role-dependent depth

**Q: What else may an interview cover?**
A: Model-training roles: optimization, gradient accumulation, mixed precision, distributed training, checkpointing. Product roles: experimentation, UX, human review, business metrics. Multimodal roles: OCR/vision/audio encoders, modality alignment, modality-specific evaluation. Prioritize the job description.

## 5. Interview answer template

Definition → mechanism → concrete example → tradeoff → failure modes → measurement → recovery/rollback.

Avoid: “bigger model/chunks/context always helps,” “RAG removes hallucinations,” “temperature zero guarantees truth,” “async makes CPU work parallel,” “a vector score is confidence,” or “the model handles authorization.”

## References

- Transformers: https://arxiv.org/abs/1706.03762
- Attention API: https://docs.pytorch.org/docs/stable/generated/torch.nn.functional.scaled_dot_product_attention
- Sentence embeddings: https://aclanthology.org/D19-1410/
- Tokenization: https://github.com/huggingface/transformers/blob/main/docs/source/en/tokenizer_summary.md
- Vector indexing/filtering: https://github.com/pgvector/pgvector
- HTTP semantics: https://www.rfc-editor.org/rfc/rfc9110.html
- Database isolation: https://www.postgresql.org/docs/14/transaction-iso.html
- Python concurrency: https://docs.python.org/3/library/asyncio-task.html
- LoRA: https://huggingface.co/docs/peft/main/conceptual_guides/lora
- Agent security: https://genai.owasp.org/llmrisk/llm062025-excessive-agency/

References support mechanisms; architecture choices and debugging steps are engineering guidance to validate against your workload.

## 6. Detailed study targets and interview readiness

Use the earlier Q&A for definitions. Use this section to decide what you must be able to do. An interview-ready answer includes a concrete implementation or investigation, measurable outcomes, and a tradeoff.

### 6.1 AI system design — rank 1, Critical

**Study:** Functional requirements; QPS/concurrency; data volume/freshness; quality/latency/availability targets; ingestion versus serving; synchronous versus queued work; caches; tenancy; fallbacks; capacity and cost estimates.

**Why:** Senior engineers choose how components work together and own the consequences.

**Practice:** Design an enterprise assistant with document permissions, citations, source updates, and traffic spikes. Explain where you enforce authorization, how deletions propagate, and what happens if the model provider fails.

**Interview question:** “When would you choose SQL, search, RAG, an agent, or a deterministic workflow?” Start from the task and failure cost, then select components.

### 6.2 Evaluation — rank 2, Critical

**Study:** Test-set construction; reference labels; retrieval versus generation metrics; answerability; task success; judge calibration; slice analysis; paired comparisons; repeated runs; release thresholds; online experiments.

**Practice:** Compare two retrieval pipelines on the same held-out queries. Track quality, p95 latency, cost, and permission failures. Investigate examples where the aggregate score improves but a critical user group gets worse.

**Interview question:** “How do you prove a new prompt or model is better?” Define success before testing, compare against a baseline, and assess uncertainty and regressions.

### 6.3 Python and software engineering — rank 3, Critical

**Study:** Data structures/complexity; mutability/copying; iterators/generators; exceptions; typing/validation; async/thread/process choices; bounded concurrency; cancellation; context managers; dependency management; unit/integration testing.

**Practice:** Implement a service that calls several tools with timeouts and concurrency limits. Test partial failure, malformed responses, cancellation, resource cleanup, and duplicate requests.

**Interview question:** “Why did an async service stop responding under load?” Check blocking calls, exhausted pools, unbounded tasks, CPU work, and downstream saturation.

### 6.4 RAG and ingestion — rank 4, Critical

**Study:** Parsing/OCR; tables; section-aware chunking; metadata/provenance; deduplication; incremental indexing; deletions; query rewriting; candidate retrieval; reranking; context assembly; citation mapping.

**Practice:** Build an evaluation set containing tables, cross-section answers, contradictory versions, exact IDs, and unanswerable questions. Inspect failure at each stage rather than changing chunk size blindly.

**Interview question:** “The correct document exists, but answers are wrong—why?” Distinguish extraction failure, missing candidates, ranking failure, context omission, and generation misuse.

### 6.5 Debugging and observability — rank 5, Critical

**Study:** Distributed traces; stage timings; p50/p95/p99; token counts; retries; queue depth; cache behavior; error budgets; version comparisons; incident mitigation and rollback.

**Practice:** Diagnose a latency increase using traces and a last-good baseline. Separate longer inputs/outputs from slower execution, queueing, or retries. Mitigate before conducting a full root-cause analysis.

**Interview question:** “Which dashboard would you build?” Include user success and quality alongside latency, failures, cost, dependency health, and freshness.

### 6.6 Security and privacy — rank 6, Critical

**Study:** Authentication versus authorization; tenant isolation; document/action permissions; prompt injection; output validation; secrets; data minimization; retention/deletion; auditability; least-privilege tools.

**Practice:** Test a malicious instruction inside a retrieved document, a cross-tenant cache hit, and a tool call beyond the user's permissions. Ensure application controls reject unauthorized behavior.

**Interview question:** “Can a system prompt enforce security?” No: access and action controls must exist outside model behavior.

### 6.7 HTTP and integrations — rank 7, Critical

**Study:** HTTP semantics; contracts; OAuth/OIDC concepts; TLS; pooling; pagination; webhooks; idempotency; backoff/jitter; deadlines; rate limits; streaming and disconnects.

**Practice:** Integrate an API where writes can time out after succeeding. Use idempotency or status reconciliation to avoid duplicate actions. Test rate limits and repeated webhook delivery.

**Interview question:** “Should we retry a timed-out POST?” Only with a safe operation contract or a reliable way to establish/reconcile the first action's outcome.

### 6.8 Databases — rank 8, High

**Study:** Schema design; constraints; joins; indexes; execution plans; transactions/isolation; migrations; pools; cache invalidation; job queues; eventual consistency between source DB and retrieval index.

**Practice:** Keep source records and searchable chunks synchronized using versioned jobs, retries, and reconciliation. Verify updates and deletes while traffic continues.

**Interview question:** “Why is the vector index showing deleted content?” Trace source deletion, queued events, index mutation, caches, and replicas.

### 6.9 Embeddings and search — rank 9, High

**Study:** Contrastive learning; hard negatives; cosine/dot/L2; normalization; embedding compatibility; ANN recall; HNSW/IVF concepts; lexical retrieval; rank fusion; reranking; filtered search.

**Practice:** Benchmark exact vector search, ANN, lexical, and hybrid pipelines on domain queries. Measure semantic relevance separately from ANN neighbor recall. Include restrictive permission filters.

**Interview question:** “Why does perfect ANN recall still give bad answers?” The embedding metric may not capture task relevance, or the corpus/chunks may be inadequate.

### 6.10 Prompting and context — rank 10, High

**Study:** Instruction structure; examples; evidence ordering; context limits; truncation; sampling; structured outputs; schema/business validation; versioned prompts; abstention behavior.

**Practice:** Extract records into a schema, then validate numeric ranges, required evidence, and cross-field rules. Test missing information and contradictory input.

**Interview question:** “Why does valid JSON still break the workflow?” Syntax/schema success does not establish semantic correctness or authorization.

### 6.11 Agents and tools — rank 11, High

**Study:** Tool schemas; validation; planning versus fixed workflows; explicit state; checkpoints; action budgets; retries; idempotency; progress detection; user review for consequential actions; recovery.

**Practice:** Build a support agent that can search and draft a resolution, with bounded tool use and a defined path when evidence or authorization is missing.

**Interview question:** “How do you recover after a tool executes but the agent crashes?” Persist action identity/outcome and reconcile before replay.

### 6.12 Deployment and LLMOps — rank 12, High

**Study:** Containers; configuration/secrets; CI/CD; offline gates; model/prompt/index versioning; canary/A/B rollout; feature flags; rollback; runbooks; backups and recovery.

**Practice:** Release a retrieval configuration change to limited traffic with observable quality, latency, and cost thresholds. Explain how you switch back without mixing incompatible indexes or embeddings.

**Interview question:** “What must be reproducible?” Code plus model/tokenizer, prompt, tool definitions, retrieval configuration, and relevant data versions.

### 6.13 ML and statistics — rank 13, High

**Study:** Loss functions; optimization basics; train/validation/test separation; leakage; bias/variance; regularization; class imbalance; calibration; distribution shift; confidence intervals and experiment design.

**Practice:** Explain why a small benchmark gain may be noise, contamination, or a changed test mix. Analyze performance by language, domain, task difficulty, and answerability.

**Interview question:** “Offline quality improved but production quality fell—why?” Compare distributions, labels, user behavior, environment, and evaluation validity.

### 6.14 Inference performance and cost — rank 14, High

**Study:** Prefill/decode; TTFT; tokens/second; queueing; batching; KV cache; token budgets; prefix/result caching; model routing; quantization; provider capacity and rate limits.

**Practice:** Explain whether a workload needs faster first output, higher throughput, or lower total completion time. Benchmark changes under realistic concurrency, context sizes, and output lengths.

**Interview question:** “Why did throughput improve while user latency worsened?” Batch waiting and queueing can increase even as device utilization improves.

### 6.15 Model fundamentals — ranks 15–17, High practical knowledge

**Study:** Transformer blocks; residuals/normalization; attention Q/K/V; causal masking; positional information; token embeddings versus retrieval embeddings; tokenization; vocabulary logits; sampling; KV cache.

**Practice:** Explain the entire path from text to first token, calculate rough context usage, and explain why longer context increases cost and memory. Understand the attention equation without needing to implement a custom GPU kernel for an application role.

**Interview question:** “Why doesn't a longer context guarantee better answers?” Relevant evidence may be absent, diluted, conflicting, truncated, or poorly used.

### 6.16 Memory and state — rank 18, High when central to the product

**Study:** Context versus persistent memory; conversation summaries; retrieval-based memory; state machines; retention; provenance; user scope; conflict resolution; correction/deletion.

**Practice:** Update a stored preference, remove the older conflicting version, and verify that it stops influencing later answers. Recover a multi-step workflow after a restart.

**Interview question:** “What should survive a request or restart?” Separate durable task/user state from disposable caches and inference intermediates.

### 6.17 Fine-tuning — rank 19, Medium by default

**Study:** Supervised fine-tuning data; full tuning versus adapters; LoRA rank/targets; quantization; training/serving formats; held-out evaluation; catastrophic forgetting; dataset governance.

**Practice:** Write a proposal explaining why prompting/RAG is insufficient, which examples teach the desired behavior, how improvement will be measured, and how adapters will be served/versioned.

**Interview question:** “Would you fine-tune to add a frequently changing policy?” Usually retrieve current policy; adapt behavior only when evidence supports that need.

### 6.18 Infrastructure and specialization — ranks 20–23, role-dependent

**Infrastructure:** Learn cloud IAM, private networking, containers, DNS/TLS, autoscaling, storage, resource limits, and GPU memory. Go deeper for self-hosted serving/platform jobs.

**Multimodal:** Learn extraction quality, modality alignment, image/audio context budgets, and modality-specific evaluation when the product needs them.

**Distributed training:** Learn data/model parallelism, sharding, synchronization, checkpoint/restart, and communication bottlenecks for model-training jobs; move this to Critical for those roles.

**Kernels/research:** Learn profiling, memory bandwidth, numerical precision, GPU programming, or architecture experiments for specialist roles. Do not spend most application-role preparation time here before mastering production fundamentals.

## 7. FDE — Forward Deployed Engineer topics

Assumption: FDE means Forward Deployed Engineer, focused here on AI deployments. FDE roles vary; some are platform/data-heavy, others focus on LLM applications. Keep the technical foundation above and add customer discovery, integration, adoption, and delivery ownership.

Role grounding: employer descriptions from Anthropic and Palantir emphasize working directly with customers and delivering production solutions. The priorities, exercises, and interview guidance below are recommendations, not employer scoring rubrics.

- Anthropic role: https://job-boards.greenhouse.io/anthropic/jobs/5391016008
- Palantir role: https://jobs.lever.co/palantir/5168e8fd-fec1-4fea-b7a1-81bdaea65850

### 7.1 FDE priorities, highest to lowest

| Rank | Topic | Importance | What to demonstrate |
|---|---|---|---|
| 1 | Customer discovery and problem decomposition | Critical | Turn an ambiguous request into a valuable, bounded problem |
| 2 | Hands-on coding and enterprise integration | Critical | Connect real systems and handle failures correctly |
| 3 | End-to-end delivery and ownership | Critical | Take a scoped use case from prototype through adoption |
| 4 | Communication and stakeholder alignment | Critical | Explain decisions to engineers, users, security, and executives |
| 5 | Enterprise data, identity, and permissions | Critical | Use customer data safely and enforce access boundaries |
| 6 | AI solution design and evaluation | Critical | Choose an appropriate approach and prove task quality |
| 7 | Deployment, debugging, and operations | Critical | Make the solution work in the customer's environment |
| 8 | Business metrics and adoption | High | Connect system quality to actual workflow outcomes |
| 9 | Full-stack prototyping and workflow UX | High | Give users a usable, testable workflow quickly |
| 10 | Scope, prioritization, and delivery planning | High | Manage constraints, dependencies, risks, and changing needs |
| 11 | Reusable patterns and product feedback | High | Turn repeated deployments into maintainable capabilities |
| 12 | Domain knowledge and governance | High when customer-relevant | Understand operational rules and error consequences |
| 13 | Advanced model tuning and GPU internals | Medium/conditional | Go deep when the customer deployment needs it |

### 7.2 Customer discovery and scoping

**Learn:** Map the current workflow, users, decision owners, data owners, bottlenecks, and costs of errors. Ask what success looks like, how it is measured today, and what must remain under human control.

**Deliver:** A problem statement, current baseline, scoped first workflow, measurable acceptance criteria, dependencies, and out-of-scope items.

**Q: A customer says “we need an AI agent”—what next?**
A: Ask which task, who performs it, which systems/actions it needs, where mistakes hurt, and whether a simpler workflow solves the problem. Agree on an outcome before selecting technology.

### 7.3 Integrations and enterprise constraints

**Learn:** Customer APIs/SDKs, SQL, ETL, pagination, schemas, file ingestion, webhooks, SSO/OIDC/OAuth concepts, service identities, network restrictions, and least privilege. Learn connector protocols, including MCP when relevant, after mastering API/auth fundamentals.

**Deliver:** A minimal integration tested against realistic permissions, missing fields, expired credentials, rate limits, schema changes, duplicate events, and outages.

**Q: A demo works locally but fails at the customer—why?**
A: Compare identity scopes, proxy/DNS/TLS configuration, egress, secrets, permissions, versions, payloads, and data assumptions. Reproduce within the actual deployment boundary.

### 7.4 Prototype to production

**Learn:** Build a small working vertical slice, collect feedback, evaluate quality, and then add operational controls. For user-facing work, practice basic frontend state, loading/error flows, streaming, evidence display, and human review.

**Deliver:** Working workflow plus tests, deployment configuration, monitoring, rollback, support owner, and runbook. Define who approves production readiness and how incidents are handled.

**Q: How do you avoid a pilot that never ships?**
A: Agree early on data access, sponsor, end users, acceptance criteria, operational owner, rollout path, and the next decision date. Test the hardest integration assumption before polishing the demo.

### 7.5 Communication, ambiguity, and prioritization

**Learn:** Separate facts from assumptions; explain tradeoffs in business terms; maintain a decision/risk log; distinguish blockers from preferences; give concrete options with consequences.

**Deliver:** Short updates covering outcome, evidence, unresolved risk, next action, owner, and any decision required. Explain the same architecture at an executive and engineering level.

**Q: The customer changes requirements mid-project—what do you do?**
A: Reconfirm the desired outcome, assess impact, propose a smaller first release or explicit scope tradeoff, and agree on revised acceptance criteria.

### 7.6 Business value, adoption, and handoff

**Learn:** Measure baseline time/cost, successful completion, correction rate, adoption, and downstream impact. Account for review time, integration/support costs, and failure consequences.

**Deliver:** A pilot scorecard comparing the existing workflow to the deployed solution; user training, ownership, and support instructions. Time saved creates value only if it translates into useful capacity or outcomes.

**Q: Model accuracy is good but users do not adopt it—why?**
A: Investigate workflow fit, trust, UX, evidence, latency, integration friction, incentives, and training. Observe real usage before assuming a larger model will help.

### 7.7 Product feedback and reusable engineering

**Learn:** Identify which customer needs are specific and which recur across deployments. Prefer configurable connectors and shared evaluation/operations patterns over copying entire projects.

**Deliver:** A documented recurring problem with evidence, a minimal reusable component, migration/maintenance considerations, and a clear product recommendation.

**Q: How do you balance customization with maintainability?**
A: Keep customer-specific policy/configuration at clear boundaries, reuse stable capabilities, and make ongoing ownership explicit.

### 7.8 Portfolio exercise for both roles

Build a permission-aware SLA/policy assistant as a practice project:

1. Connect a sample SQL dataset and policy documents; define which users may see which records/fields.
2. Route exact counts and numeric filtering to validated SQL; retrieve narrative clauses from documents.
3. Return answers with evidence; allow “insufficient information”; require controlled authorization for any write.
4. Evaluate answer correctness, retrieval, denied-access cases, tool failures, latency, and cost.
5. Demonstrate update/deletion propagation, rate limits, duplicate actions, outage handling, and rollback.
6. Present the architecture, working UI, evaluation results, business baseline, rollout plan, and handoff runbook.

**Senior AI Engineer emphasis:** Correct architecture, measurable quality, reliability, security, and maintainable engineering.

**FDE emphasis:** Discover the right workflow, integrate with customer constraints, ship a usable solution, communicate clearly, and demonstrate adoption/value.
