# Senior AI Engineer — Compact Q&A

Purpose: quick revision and low-token reuse. All three earlier user questions are included below; responses are condensed, not verbatim. Added topics fill gaps in the original coverage. No fixed chunk size, model, or architecture is universally best.

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
