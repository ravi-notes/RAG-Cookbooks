## Stage 7 – Minimal Prerequisites for a Colab Prototype

### BLUF

To build the **smallest possible working RAG** in Google Colab, you only need:

1. a few files, 2) one embedding model, 3) one vector DB, 4) one retriever, 5) one LLM call.
   Nothing else.

---

# 1. Minimal Conceptual Requirements

### A. One small corpus

Examples:

* A text file
* A PDF converted to text
* A folder of `.txt` notes
* A few paragraphs you paste manually

Keep it tiny to reduce noise.

---

### B. A chunker

Simplest possible:

* Fixed size: 300–500 tokens
* Overlap: 50–100 tokens
* Use recursive text splitter in LangChain or a simple Python function

---

### C. An embedding model

Pick one:

* **bge-small-en** (fast, free, excellent)
* **sentence-transformers/all-MiniLM-L6-v2**
* **text-embedding-3-small** (OpenAI; optional)

For Colab: sentence-transformers is easiest.

---

### D. A vector store

Simplest/local options:

* **FAISS** (best for Colab)
* Chroma (optional)

FAISS keeps everything in RAM → zero infra complexity.

---

### E. A retriever

FAISSStore.asRetriever(k=3) or Chroma.asRetriever().

---

### F. An LLM

At minimal:

* **GPT-4o-mini** or **GPT-4.1** via API
* Or a local model like **Qwen2.5 0.5B** via transformers (only for demonstration)

For Colab, simplest working pipeline uses OpenAI API.

---

# 2. Minimal Technical Requirements (Software)

Install only these:

```
pip install langchain sentence-transformers faiss-cpu openai
```

Optional:

* PyPDF2 for PDFs
* tiktoken for token counting

---

# 3. Minimum Pipeline (Executable Sequence)

### Step 1. Load documents

* Read `.txt` or extract text from PDF

### Step 2. Chunk the text

* Use LangChain’s RecursiveCharacterTextSplitter or your own

### Step 3. Embed chunks

* Using Sentence Transformers model
* Produce vectors

### Step 4. Build index

* Store vectors in FAISS
* Save chunk texts as metadata

### Step 5. Build retriever

* FAISS → retriever

### Step 6. Build RAG chain

* Query → embed → retrieve top-K chunks → format prompt → LLM → answer

### Step 7. Test

Ask a question and inspect:

* Retrieved chunks
* Generated answer

---

# 4. Minimal Working RAG Code Skeleton (Pseudocode)

**This is not full code; it’s the conceptual “shape” of the notebook.**

```
# 1. Load text
text = open('notes.txt').read()

# 2. Chunk
chunks = chunk(text, size=500, overlap=100)

# 3. Embed
embeddings = embed(chunks)

# 4. Index
faiss_index = FAISS.from_embeddings(embeddings, chunks)

# 5. Retriever
retriever = faiss_index.as_retriever(k=5)

# 6. Query
query = "What does the document say about X?"
query_vector = embed(query)

results = retriever.get_relevant_documents(query)

# 7. Prompt + LLM
answer = call_llm(query, results)

print(answer)
```

This is literally all needed for the smallest RAG demo.

---

# 5. Minimal Notebook Structure (8–10 cells)

1. Install packages
2. Import libraries
3. Load data
4. Chunk data
5. Load embedding model
6. Generate + store embeddings
7. Build FAISS index
8. Query via retriever
9. Build prompt
10. Call LLM

---

# 6. Practical Tips for the Beginner Prototype

* Use **very small datasets** (2–3 paragraphs) so you can inspect every chunk.
* Print **retrieved chunks** each time; this is essential for learning retrieval behavior.
* Start with **top_k = 3**.
* Keep your LLM prompt extremely simple in Stage 1.
* Do not add rerankers, hybrid search, or agent pipelines initially.

Goal: **functional correctness, not sophistication**.

---

# 7. Smallest Buildable RAG Architecture Diagram

```
Documents ─→ Chunker ─→ Embedder ─→ FAISS Index
                                            │
                                    Query ─→│
                                            ▼
                                        Retriever
                                            │
                                            ▼
                                   Prompt Constructor
                                            │
                                            ▼
                                           LLM
                                            ▼
                                          Answer
```

---

Stage 7 complete.
Say **“continue”** to proceed to **Stage 8 → Final ultra-short summary + learning checklist**.
