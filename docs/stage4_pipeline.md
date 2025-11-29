## Stage 4 – Pipeline Walkthrough With Diagrams

### 0. BLUF

End-to-end RAG = **Ingest → Index → Retrieve → Prepare Context → Generate Answer**. Below is the full pipeline with ASCII diagrams and internal data flow.

---

### 1. Complete RAG Pipeline (Bird’s-Eye)

```
   ┌──────────────────────────────────────────┐
   │               INGEST PHASE               │
   │  (Prepare and store your knowledge base) │
   └──────────────────────────────────────────┘
                     │
                     ▼
         ┌───────────────────────────┐
         │  1. Chunking              │
         └───────────────────────────┘
                     │
                     ▼
         ┌───────────────────────────┐
         │  2. Embeddings            │
         └───────────────────────────┘
                     │
                     ▼
         ┌───────────────────────────┐
         │  3. Vector Index (DB)     │
         └───────────────────────────┘

   ┌──────────────────────────────────────────┐
   │               QUERY PHASE                │
   └──────────────────────────────────────────┘
                     │
                     ▼
         ┌───────────────────────────┐
         │  4. Query Embedding       │
         └───────────────────────────┘
                     │
                     ▼
         ┌───────────────────────────┐
         │  5. Retriever (ANN + BM25)│
         └───────────────────────────┘
                     │
                     ▼
         ┌───────────────────────────┐
         │  6. Context Builder       │
         └───────────────────────────┘
                     │
                     ▼
         ┌───────────────────────────┐
         │  7. LLM Generator         │
         └───────────────────────────┘
                     │
                     ▼
         ┌───────────────────────────┐
         │  8. Final Answer          │
         └───────────────────────────┘
```

---

### 2. Ingest Phase (Offline)

#### Purpose

Prepare your corpus so the system can retrieve from it at runtime.

---

#### Step 1. Chunking

```
Original Documents
    │
    ▼
[Split into chunks of ~300–800 tokens with overlap]
    │
    ▼
Chunked Corpus
```

Chunking rules:

* Maintain semantic coherence
* Include overlaps so meaning doesn’t break
* Preserve structure (Markdown headings, paragraphs)

---

#### Step 2. Embeddings

```
Each chunk ──► Embedding Model ──► Chunk Vector
```

Output = vector + metadata (doc_id, page_no, text).

---

#### Step 3. Vector Indexing

```
Chunk Vectors ─┐
               ▼
        [Vector Database]
               │
               └── stores:
                     - vectors
                     - raw text
                     - metadata
                     - ANN index
```

Vector DB builds internal ANN graphs for fast retrieval.

---

### 3. Query Phase (Online)

This is the real-time flow when a user asks a question.

---

#### Step 4. User Query → Embedding

```
User Query "What is X?"
        │
        ▼
Embedding Model
        │
        ▼
Query Vector
```

---

#### Step 5. Retrieval

```
 Query Vector
      │
      ▼
ANN Search in Vector DB
      │
      ▼
Top-K Candidate Chunks
      │
      ▼
(Optional) Reranker (cross-encoder)
      │
      ▼
Final Ranked Chunks
```

Hybrid RAG =
Dense ANN output + BM25 sparse output → merged → reranked.

---

#### Step 6. Context Builder

```
     Final Chunks
          │
          ▼
[Insert into prompt template]
          │
          ▼
LLM-ready input:
  "You must answer using ONLY the passages below.
   Question: ...
   Context:
   [chunk 1]
   [chunk 2]
   [chunk 3]"
```

Controls:

* Token limit
* Deduplication
* Formatting
* Citations
* Instruction tuning

---

#### Step 7. LLM Generator

```
(Question + Context)
          │
          ▼
       LLM
          │
          ▼
      Answer
```

LLM uses attention to align question tokens with relevant chunk tokens.

---

#### Step 8. Final Answer

Grounded, cite-able output based on your data.

---

### 4. Compact ASCII Diagram (Side-by-Side View)

```
          ┌──────────────┐
          │ User Query    │
          └───────┬──────┘
                  ▼
           Embed Query
                  │
                  ▼
        ┌───────────────────┐
        │   Retriever       │
        │ (ANN / Hybrid)    │
        └───────┬───────────┘
                ▼
        Retrieved Chunks
                │
                ▼
        Build Prompt (Context)
                │
                ▼
            LLM (G)
                │
                ▼
          Final Answer
```

Behind the scenes:

```
Documents → Chunking → Embeddings → Vector DB
```

---

### 5. Data Flow Summary Table

| Stage           | Input         | Processing            | Output           |
| --------------- | ------------- | --------------------- | ---------------- |
| Chunking        | Raw docs      | segment text          | chunks           |
| Embedding       | chunks        | vector encoding       | chunk vectors    |
| Indexing        | chunk vectors | ANN indexing          | searchable index |
| Query Embedding | query         | vector encoding       | query vector     |
| Retriever       | query vector  | similarity search     | top-k chunks     |
| Context Builder | chunks        | prompt assembly       | LLM prompt       |
| Generator       | prompt        | reasoning + synthesis | answer           |
