## Stage 3 – Components and Their Internals

### 0. BLUF

RAG has four core components: **Embeddings → Vector DB → Retriever → Generator**. Each has internal mechanics that determine overall quality.

---

### 1. Embeddings (Representation Layer)

#### Purpose

Convert text into dense numeric vectors that capture meaning.

#### Internals

* **Model type:** Sentence Transformers, OpenAI, Cohere, bge
* **Output:** A vector (e.g., 768 or 1024 dimensions)
* **Training objective:**

  * Contrastive learning (pull similar texts together, push dissimilar apart)
* **Tokenization:** Break text into tokens; model pools token-level representations → a single vector
* **Pooling strategies:**

  * Mean pooling (common, stable)
  * CLS token pooling (model-dependent)

#### What embeddings care about

* Semantic similarity
* Context around a passage
* Paraphrases
* Topic affinity

#### What embeddings fail at

* Exact numeric matches
* Very long documents (truncate)
* Tables, code, formulas (unless specialized)

---

### 2. Chunking (Preprocessing Layer)

#### Purpose

Split documents into manageable, semantically coherent pieces.

#### Key internals

* **Chunk size:** 200–1000 tokens depending on domain
* **Chunk overlap:** 10–25% to preserve context between segments
* **Segmentation strategy:**

  * Sentence-based
  * Paragraph-based
  * Markdown/HTML structure
  * Sliding window for technical docs

#### Failure points

* Too small → lost narrative
* Too big → irrelevant filler reduces retrieval accuracy
* No overlap → boundary information lost

Chunking = the “granularity dial” of RAG.

---

### 3. Vector Database (Index Layer)

#### Purpose

Store embeddings and support fast similarity search.

#### Main components

* **Storage:** Vectors + metadata + chunk text
* **Indexing algorithm:**

  * HNSW (most used)
  * IVF/PQ (quantized)
  * Flat (exact search, slow)
* **Distance metric:** cosine / dot product
* **Filtering:** metadata filters (“only from these docs”)

#### How ANN algorithms work (simple)

* Build a navigable graph where similar vectors are near each other
* During search, walk the graph to find neighbors quickly
* Trade small accuracy loss for huge speed gain

#### Popular vector DBs

* FAISS
* Chroma
* Milvus
* Weaviate
* Pinecone
* LanceDB

---

### 4. Retriever (Selection Layer)

#### Purpose

Given a question → identify top-K relevant chunks.

#### Types

1. **Dense retriever**

   * Embed query → compare with chunk embeddings
   * Pros: semantic, handles rephrasing
   * Cons: may miss names/numbers

2. **Sparse retriever**

   * BM25, TF-IDF
   * Pros: exact token matching
   * Cons: weak semantics

3. **Hybrid retriever**

   * Weighted mix of dense + sparse
   * Best of both worlds

4. **Rerankers (optional)**

   * Cross-encoder rerank
   * Reads (query, chunk) together
   * Much more accurate but slower
   * Usually final ranking step after ANN search

#### What retrievers optimize

* Recall@K (ensure correct chunk is in the top-K)
* Relevance score
* Domain specificity

---

### 5. Context Builder (Prompt Construction Layer)

#### Purpose

Assemble the final input to the LLM.

#### Steps

* Merge top-K chunks
* Add instructions (“use only retrieved info”)
* Insert citations
* Control formatting
* Optimize for context length

#### Common patterns

* **Stuffing:** Put all chunks inside one prompt
* **Map-Reduce:** Summaries of each chunk → reduced summary
* **ReAct / Chain-of-Thought:** Additional reasoning steps

---

### 6. Generator (LLM Layer)

#### Purpose

Produce final answer grounded in retrieved evidence.

#### Internals

* **Self-attention**: correlates question tokens ↔ chunk tokens
* **Reasoning**: chain-of-thought (hidden or explicit)
* **Grounding**: salience weighting of retrieved tokens
* **Biases**:

  * hallucination tendency
  * preference for narrative
  * token-level fluency over factuality

#### Model choices

* Llama
* Mistral
* GPT
* Gemma
* Qwen

#### What affects accuracy

* Prompt instructions
* Size of model
* Cleanliness of retrieval
* Chunking quality
* Reranking effectiveness

---

### 7. Putting components together (conceptual stack)

```
   User Query
        │
        ▼
   Embed Query
        │
        ▼
   Retriever
        │
   (ANN search)
        │
        ▼
   Retrieved Chunks
        │
   (build prompt)
        │
        ▼
      LLM
        │
        ▼
   Grounded Answer
```

Each box has independent hyperparameters that change system behavior.
