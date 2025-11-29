## Stage 6 – Limitations and Debugging Patterns

### 0. BLUF

RAG almost always fails due to **retrieval**, **chunking**, **prompting**, or **LLM drift**. Fixing RAG = diagnosing which layer broke.

---

### 1. Core Limitations (Structural)

#### A. Retrieval quality bottleneck

If the correct chunk never reaches the LLM, the answer **cannot** be correct.

Symptoms:

* Confident hallucinations
* “The answer is not in the provided context.”
* Nonsense or off-topic responses

Root causes:

* Poor embeddings
* Chunking too small/large
* ANN index mis-tuned
* Domain-mismatch between embedding model and corpus

---

#### B. Context window limits

LLMs can only use token-level attention within their window (e.g., 8k, 32k, 128k).

Limits:

* Only a small slice of corpus is visible
* Adding more chunks ≠ better answer
* At scale (100k+ docs), retrieval precision becomes critical

---

#### C. Hallucination tendency

Even with context, LLMs may:

* invent missing links
* over-generalize
* “smooth” contradictions
* merge unrelated chunks into one narrative

Cause:
LLM is trained to be **fluent**, not **factual**.

---

#### D. Embedding model mismatch

General-purpose embeddings fail for:

* medical terminology
* legal norms
* code
* math
* tables
* finance documents
* multilingual corpora

---

#### E. Chunking failures

Chunk too small → no full meaning
Chunk too big → irrelevant background noise

Without good chunking, retrieval collapses.

---

#### F. Long-tail queries

LLMs fail when:

* Query uses rare terms
* Query is extremely specific
* Query requires reading entire document sets (RAG can’t load everything)

---

#### G. Latency/cost issues

Rerankers, hybrid retrieval, graph-RAG increase accuracy but also latency and cost.

---

### 2. Failure Modes (What You See and Why)

| Symptom                              | Likely Cause                                       |
| ------------------------------------ | -------------------------------------------------- |
| Answer is hallucinated               | Retrieval failed; chunks irrelevant                |
| Answer is generic                    | Prompt not instructing grounding; poor reranking   |
| Answer contradicts docs              | Bad chunk selection; multiple conflicting chunks   |
| Answer is incomplete                 | Only partial chunk retrieved; chunk size too small |
| “I cannot find information”          | Sparse-only retrieval failing on semantics         |
| Good chunks retrieved but answer bad | LLM not grounded; context overload                 |

---

### 3. Debugging Patterns (Practical Workflow)

#### Pattern 1 — Inspect retrieved chunks

Always check the actual top-K retrieved results.

If wrong → fix retrieval, not the LLM.

Checklist:

* Are chunks topically aligned?
* Any missing keywords?
* Too much filler text?
* Wrong document sections?

---

#### Pattern 2 — Tune chunking

Start with:

* size = 300–500 tokens
* overlap = 50–100 tokens

Adjust based on domain:

* **Legal/medical**: larger, structured chunks
* **Code**: small chunks with file structure preserved
* **Wiki/Markdown**: chunk by heading sections

---

#### Pattern 3 — Improve embeddings

Options:

* Switch to domain-specific embedding models
* Use larger embedding models
* Use multilingual embeddings if corpus spans languages

---

#### Pattern 4 — Use hybrid retrieval

Dense + BM25 fixes:

* rare tokens
* names
* numbers
* acronyms
* code identifiers

Often gives the biggest jump in accuracy.

---

#### Pattern 5 — Add a reranker

Cross-encoders dramatically improve precision.

Pipeline:

```
ANN top-50 → reranker → top-5 → LLM
```

---

#### Pattern 6 — Prompt the LLM to ground itself

Include constraints:

* “Answer **only** from provided context.”
* “If answer not present, say 'Not found'.”
* “Cite the exact context passages.”
* “Do not add external facts.”

These reduce hallucination rate.

---

#### Pattern 7 — Reduce context clutter

Too many chunks dilute attention.

Try:

* top-3 or top-4 instead of top-10
* reranking to get cleaner context
* context compression for long documents

---

#### Pattern 8 — Query rewriting

LLM or rule-based agent reformulates the user question before retrieval.

Example:
Original: “What did the report say about testing?”
Rewrite: “Summarize test procedures from the 2021 QA audit document.”

Improves recall.

---

### 4. Quick Debug Decision Tree

```
Bad answer?
  ↓
Check retrieved chunks:
    Bad? → Fix embedding/chunking/retrieval.
    Good? → Move on.
  ↓
Check prompt grounding:
    Not strict? → Strengthen instructions.
  ↓
Check chunk quantity:
    Too many? → Reduce K.
    Too few? → Increase K + rerank.
  ↓
Check domain mismatch:
    Use domain embedding model.
  ↓
Still bad?
    Add hybrid search / reranker / agentic search.
```

---

### 5. Hard Limits (What RAG Cannot Solve)

* Does not generate new knowledge beyond docs
* Cannot ensure 100% truth even with perfect retrieval
* Cannot aggregate massive corpora above context window without compression systems
* Cannot replace fine-tuning when task requires reasoning style changes
* Does not understand diagrams, tables, or images unless multimodal embeddings are used
* Cannot fix contradictory source documents
* Cannot guarantee privacy if agentic retrieval is uncontrolled
