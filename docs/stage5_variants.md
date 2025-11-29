## Stage 5 – RAG Variants and Their Tradeoffs

### 0. BLUF

Variants differ in how they **retrieve**, **route**, and **reason**. They exist to fix typical failure modes (poor retrieval, multi-hop reasoning, context overload, niche domains).

---

### 1. **Basic RAG (Single-Hop)**

#### Definition

Retrieve top-K chunks once → pass to LLM → generate answer.

#### Diagram

```
Query → Retrieve → LLM → Answer
```

#### Strengths

* Simple
* Fast
* Good for short, direct Q&A
* Easy to implement (default in most libraries)

#### Weaknesses

* Fails when answer requires multiple dependent facts
* Retrieval errors propagate directly to the LLM
* Limited to top-K context window constraints

---

### 2. **Multi-Hop RAG**

#### Definition

LLM performs **iterative retrieval**:
Use the first retrieved context to refine the next query.

#### Diagram

```
Query → Retrieve → LLM → Subquery
                   ↑        │
                   └─ Retrieve Again ─→ (repeat)
```

#### Use Cases

* Questions requiring multi-step reasoning
* Linking information across documents
* “Explain relationships between A and B using corpus C”

#### Strengths

* Much higher recall for complex tasks
* Better for research, legal, medical, financial analyses

#### Weaknesses

* More latency
* Harder to control
* Requires strong query rewriting skills

---

### 3. **HyBRID RAG (Dense + Sparse)**

#### Definition

Combine semantic search (dense) with keyword search (BM25).

#### Diagram

```
      Dense ANN ──┐
                   ├── Merge → Rerank → Top-K
      BM25 ───────┘
```

#### Strengths

* Best retrieval accuracy for most domains
* Handles:

  * synonyms
  * exact keywords
  * numeric strings
  * rare entities
* Reduces embedding model blind spots

#### Weaknesses

* More components
* Heavier infra
* Requires scoring and weighting

---

### 4. **RAG with Reranking (Cross-Encoder)**

#### Definition

Retriever collects many candidates (k=50–200) → cross-encoder re-scores relevance.

#### Diagram

```
Top-100 candidates from ANN
       ↓
Cross-Encoder Rerank
       ↓
Final Top-K (3–8)
```

#### Strengths

* Dramatic improvement in precision
* Eliminates “garbage context”
* Especially helpful in:

  * legal
  * biomedical
  * finance
  * code search

#### Weaknesses

* High latency
* Higher cost
* Needs GPU for speed

---

### 5. **Graph RAG**

#### Definition

Build a graph of entities/relationships from corpus → retrieve subgraphs instead of raw chunks.

#### Diagram

```
Corpus → entity extraction → graph
Query → graph search → retrieve subgraph → LLM
```

#### Strengths

* Great for relationship-heavy corpora
* Multi-hop reasoning becomes explicit
* Excellent for highly structured domains

#### Weaknesses

* Heavy preprocessing
* Requires reliable entity extraction
* Complex to maintain

---

### 6. **Agentic / ReAct RAG**

#### Definition

LLM acts as an agent that can issue actions:

* Search again
* Reformulate question
* Read more chunks
* Validate evidence

#### Diagram

```
Query
  ↓
LLM → (action: search) → retriever → new context
  ↓
LLM → answer
```

#### Strengths

* Highly flexible
* LLM decides when/what to retrieve
* Long chains of reasoning possible

#### Weaknesses

* Unpredictable behavior
* Harder to guarantee privacy constraints
* Latency spikes

---

### 7. **Context-Compression RAG**

#### Definition

Compress chunks before adding to prompt:

* Summaries
* Extractive reduction
* Key sentences only

#### Strengths

* Fits more info into limited context windows
* Useful for small LLMs

#### Weaknesses

* Compression must preserve meaning
* Summaries may lose vital details

---

### 8. **Adaptive RAG (Router-Based)**

#### Definition

Different query types → different retrievers or knowledge sources.

#### Diagram

```
Router:
  factual → RAG
  reasoning → chain-of-thought
  code → code-index
```

#### Strengths

* Reduces irrelevant retrieval
* Improves accuracy and latency
* Useful in enterprise setups

#### Weaknesses

* Requires classifier or heuristics
* Complex pipeline

---

### 9. Comparison Table (Condensed)

| Variant     | Purpose                 | Strengths             | Weaknesses             |
| ----------- | ----------------------- | --------------------- | ---------------------- |
| Basic       | Simple QA               | Fast, easy            | Fails on multi-hop     |
| Multi-hop   | Complex reasoning       | Higher recall         | Slow, hard             |
| Hybrid      | Better accuracy         | Recovers dense+BM25   | Requires merging logic |
| Reranker    | Precision               | Best top-K relevance  | High latency           |
| Graph       | Relationship-heavy data | Structured, multi-hop | Heavy preprocessing    |
| Agentic     | Adaptive retrieval      | Max flexibility       | Unpredictable          |
| Compression | Fit more info           | Good for small LLMs   | Risk of info loss      |
| Adaptive    | Smart routing           | Lower noise           | Complex design         |
