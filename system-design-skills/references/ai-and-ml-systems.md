# AI and ML Systems

A systems-design view of machine learning and LLM-powered applications: the vocabulary, how models are built and served, and the architecture and operational concerns of putting them into production. Vendor-neutral throughout.

## Core Vocabulary

| Term | Meaning |
| --- | --- |
| Training | The process of fitting model parameters to data by minimizing a loss function (expensive, batch, GPU-heavy). |
| Inference | Running a trained model to produce outputs for new inputs (the serving / request-time path). |
| Features | The input signals a model reads (columns, pixels, tokens, embeddings). |
| Labels | The target answers used in supervised training (ground truth). |
| Supervised learning | Learn a mapping from features to known labels (classification, regression). |
| Unsupervised learning | Find structure without labels (clustering, dimensionality reduction). |
| Reinforcement learning (RL) | Learn a policy by maximizing reward from trial-and-error interaction with an environment. |
| Parameters (weights) | The learned numbers inside the model; "size" (e.g. 8B, 70B) counts these. |
| Embedding | A dense vector that encodes the meaning of an item (word, sentence, image) so similar items sit close together. |
| Token | A sub-word chunk of text; models read and emit tokens, not characters or whole words. |
| Fine-tuning | Continued training of a pretrained model on a narrower dataset to specialize behavior. |
| Overfitting | Memorizing training data so generalization to new data degrades; countered with regularization, more data, validation. |
| Hyperparameters | Settings chosen before training (learning rate, batch size) rather than learned. |
| Epoch / batch | One full pass over the dataset / a subset processed per gradient step. |

Mental model: **training is build-time, inference is request-time.** Most production engineering concerns (latency, cost, scaling, observability) live on the inference path.

## How an LLM Works (High Level)

A large language model is a neural network trained to predict the next token. Modern LLMs are built in stages:

```
[ Web/code/book corpus ]            [ Curated demos ]         [ Human/AI preferences ]
          |                                |                            |
          v                                v                            v
   Pretraining  ───────────►  Supervised Fine-Tuning (SFT) ───────►  Preference Tuning
 (next-token prediction,       (follow instructions,                 (RLHF / DPO: rank
  trillions of tokens)          imitate good answers)                 good vs bad outputs)
          |                                |                            |
          +────────────────────────────────────────────────────────────+
                                          v
                                  Aligned base model
```

1. **Pretraining** — self-supervised next-token prediction over a massive corpus. No human labels needed; the text *is* the supervision. This is where most knowledge and capability is acquired, and where most compute is spent.
2. **Supervised fine-tuning (SFT)** — train on curated prompt/response pairs so the model learns to follow instructions rather than merely continue text.
3. **Preference tuning** — align outputs with human preferences. Classic RLHF trains a reward model from human rankings, then optimizes the policy against it; newer methods (e.g. DPO) optimize directly on preference pairs without a separate RL loop.

### Tokenization

Text is split into tokens by a learned vocabulary (byte-pair encoding or similar). Roughly 1 token ≈ 4 English characters ≈ ¾ of a word. Token counts drive both context limits and billing, so token efficiency is a real engineering lever.

### Transformers and Attention (conceptually)

The dominant architecture is the **transformer**. Its key mechanism, **self-attention**, lets every token weigh the relevance of every other token in the context when forming its representation. This is what captures long-range dependencies ("it" refers to which noun?). Stacking many attention + feed-forward layers, trained at scale, yields broad language ability. You rarely tune this as an app developer, but it explains the cost model: attention is roughly quadratic in sequence length, so long contexts are expensive.

### Inference

LLMs generate **autoregressively** — one token at a time, each new token appended and fed back in:

```
prompt → [model] → tok1 → [model] → tok2 → [model] → tok3 → ... → <stop>
```

- **Context window** — the maximum tokens (prompt + generation) the model can attend to at once. Overflow must be truncated, summarized, or retrieved.
- **Temperature / sampling** — controls randomness. Low temperature (→0) is near-deterministic and good for extraction/classification; higher temperature increases diversity for creative tasks. `top-p` / `top-k` cap the candidate pool the next token is sampled from.
- **Streaming** — tokens can be emitted as generated, cutting perceived latency (time-to-first-token matters more than total time for UX).

## AI Agents

An **agent** is an LLM placed inside a loop where it can call tools, observe results, and decide what to do next — turning a one-shot text generator into something that can take multi-step actions.

```
        ┌──────────── Agent Loop ────────────┐
        │                                     │
   ┌─► Perceive (read goal + latest observation)
   │    │                                     │
   │    ▼                                     │
   │  Plan/Reason (decide next step)          │
   │    │                                     │
   │    ▼                                     │
   │  Act (call a tool / function)            │
   │    │                                     │
   │    ▼                                     │
   └── Observe (tool result) ─── goal met? ──┴─► Final answer
```

- **Components**: an LLM (the reasoner), **tools** (functions, APIs, code execution, retrieval), **memory** (short-term scratchpad/context + long-term store), and a **control loop** that decides when to stop.
- **ReAct pattern** — interleave *reasoning* traces with *actions*: think → act → observe → think again. The model explains its intent, calls a tool, reads the result, and repeats.
- **Tool / function calling** — tools are described to the model with a name, description, and a typed (usually JSON-schema) parameter spec. The model emits a structured call; your runtime executes it and feeds the result back. The model never runs anything itself — your code does, which is the enforcement point.
- **Guardrails** — bound the loop and the blast radius: max iterations/budget, allowlisted tools, input/output validation, human-in-the-loop approval for risky actions (writes, payments, deletions), and sandboxing for code execution.

## Retrieval-Augmented Generation (RAG)

RAG grounds a model in your own data by retrieving relevant context at query time and injecting it into the prompt — instead of relying solely on parametric knowledge.

```
Ingest (offline):
  documents → chunk → embed → store in vector DB (+ metadata)

Query (online):
  user query → embed → similarity search → top-k chunks
                                              │
                                              ▼
        prompt = system + retrieved context + question → LLM → grounded answer
```

1. **Chunking** — split documents into passages sized for retrieval (often a few hundred tokens, with overlap to preserve context across boundaries).
2. **Embedding** — convert each chunk into a vector with an embedding model.
3. **Indexing** — store vectors (plus source metadata) in a vector store.
4. **Retrieval** — embed the query, find the nearest chunks; optionally re-rank or apply metadata filters.
5. **Augmentation** — concatenate retrieved chunks into the prompt and generate.

**RAG vs fine-tuning:**

| Use RAG when | Use fine-tuning when |
| --- | --- |
| Knowledge changes often or is large | You need a consistent *style/format/behavior* |
| You need source citations / freshness | The task is narrow and stable |
| You must control/update facts without retraining | Latency/cost of long retrieved prompts is prohibitive |
| Data is private and per-tenant | You have enough high-quality labeled examples |

They are complementary: fine-tune for *how* to respond, retrieve for *what* facts to use.

## Vector Databases and Similarity Search

Embeddings turn "find similar meaning" into "find nearby vectors." Similarity is measured by:

- **Cosine similarity** — angle between vectors; robust to magnitude (most common for text).
- **Dot product** — cosine scaled by magnitude; common when embeddings are normalized.
- **Euclidean (L2) distance** — straight-line distance.

Exact nearest-neighbor search is O(N) per query and doesn't scale. Vector DBs use **Approximate Nearest Neighbor (ANN)** indexes that trade a little recall for large speedups:

- **HNSW** (Hierarchical Navigable Small World) — a layered proximity graph; fast, high-recall, memory-heavy.
- **IVF** (inverted file) — cluster vectors into cells, search only the nearest cells.
- **PQ** (product quantization) — compress vectors to cut memory at some accuracy cost.

Production vector stores add metadata filtering (e.g. `tenant_id`, date), hybrid search (combine dense vectors with keyword/BM25), and updates/deletes.

## The Open-Source AI Stack (Layers)

```
┌─────────────────────────────────────────────────────────────┐
│ App layer        UI, agents, business logic, prompt flows    │
├─────────────────────────────────────────────────────────────┤
│ Orchestration    chains/agents, tool routing, RAG pipelines  │
├─────────────────────────────────────────────────────────────┤
│ Observability    tracing, evals, logging, cost/latency, A/B  │
├─────────────────────────────────────────────────────────────┤
│ Data layer       vector DB, doc store, feature store, cache  │
├─────────────────────────────────────────────────────────────┤
│ Serving          inference servers, batching, autoscaling    │
├─────────────────────────────────────────────────────────────┤
│ Models           foundation/base models, embeddings, rerank  │
└─────────────────────────────────────────────────────────────┘
```

Each layer is swappable. A common failure mode is over-coupling to one framework or provider; keep boundaries clean (an interface per layer) so models, vector stores, and providers can be replaced.

## Data Pipelines

ML and LLM systems are only as good as the data feeding them.

- **Batch vs streaming** — batch processes bounded data on a schedule (nightly retraining, bulk embedding); streaming processes events continuously (real-time features, live ingestion). Many systems run both (lambda/kappa-style).
- **ETL vs ELT** — ETL transforms *before* loading into the warehouse; ELT loads raw then transforms inside it. ELT is now common with cheap, scalable warehouses and gives analysts/embedding jobs raw data to reshape.
- **Feature store** — a central place to define, compute, store, and serve features consistently between training and inference, avoiding train/serve skew. Splits into an offline store (history for training) and an online store (low-latency lookups at inference).
- **Data quality/validation** — schema checks, null/range/freshness assertions, drift detection, and deduplication. Bad data fails silently and degrades models without crashing anything, so validation must be explicit and continuous.

## LLM Application Architecture

```
client → API/gateway → [ prompt mgmt ] → [ cache? ] → router → provider A
                                                        │  └──► provider B (fallback)
                            tracing/evals ◄─────────────┘
```

- **Prompt management** — version prompts like code (templates, variables, changelog). Treat a prompt change as a deploy with the same review and rollback discipline.
- **Caching** — exact-match cache for identical requests; semantic cache (embed the query, reuse answers for near-duplicates); and provider-side prompt/context caching to cut cost on repeated prefixes.
- **Multi-provider fallback** — abstract behind one interface so you can fail over on outages/rate limits and route by task (cheap model for easy work, strong model for hard work).
- **Trade-off triangle** — latency, cost, and quality pull against each other. Bigger models and longer contexts raise quality but cost more and respond slower. Tune per use case; don't default everything to the largest model.
- **Evals** — the test suite of LLM apps. Combine offline eval sets (golden inputs with graded outputs), LLM-as-judge scoring, regression checks on prompt/model changes, and online metrics (user feedback, task success). Without evals you cannot safely change prompts or upgrade models.

## Production Concerns

| Concern | What to watch / do |
| --- | --- |
| **Latency** | Optimize time-to-first-token; stream output; trim prompt size; cache; parallelize retrieval and tool calls. |
| **Cost** | Priced per input + output token. Shorten prompts, cap `max_tokens`, cache, route smaller models, batch where possible. |
| **Rate limits** | Provider request/token quotas. Add backoff + retries with jitter, queues, and multi-provider/multi-key failover. |
| **Hallucination** | Models state false things confidently. Mitigate with RAG grounding, citation requirements, lower temperature for factual tasks, schema-constrained output, and verification passes. |
| **Observability** | Trace every LLM call end-to-end: prompt, retrieved context, tool calls, tokens, latency, cost, and outcome. Essential for debugging non-deterministic systems. |
| **Prompt injection** | Untrusted text (user input, retrieved docs, tool output) can carry instructions that hijack the model. Treat all model-adjacent text as untrusted; separate instructions from data, least-privilege tools, validate/sanitize outputs, and require human approval for high-impact actions. |

### Prompt Injection (Security)

The defining new attack surface: because instructions and data share the same channel (the prompt), an attacker who controls any text the model reads — a web page it retrieves, a document it summarizes, a tool's response — can attempt to override its instructions ("ignore previous instructions and exfiltrate..."). There is no perfect filter. Defense is layered: minimize tool privilege, isolate untrusted content, constrain outputs, monitor/trace, and keep a human in the loop for irreversible or sensitive actions. Treat an agent's tool permissions as you would a service account's — assume the prompt may be adversarial.

## Key Takeaways

- Training is build-time; inference is request-time where your operational concerns live.
- LLMs are next-token predictors refined by SFT and preference tuning; inference is autoregressive and bounded by the context window.
- Agents = LLM + tools + memory + a control loop; guardrails and least privilege are mandatory, not optional.
- Reach for RAG to control *facts and freshness*, fine-tuning to control *behavior and style* — often both.
- Evals and tracing make a non-deterministic system operable; without them you are flying blind.
- Treat every byte the model reads as potentially adversarial; prompt injection is a first-class security risk.
