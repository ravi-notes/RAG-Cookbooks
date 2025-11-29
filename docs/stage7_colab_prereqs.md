# Stage 7 – Minimal Prerequisites for a Colab Prototype

## Minimal components

- Small corpus  
- Chunker  
- Embedding model (MiniLM, bge-small)  
- Vector DB (FAISS)  
- Retriever  
- LLM API  

---

## Install

```
pip install sentence-transformers faiss-cpu langchain openai
```

---

## Pipeline Steps

1. Load text  
2. Chunk  
3. Embed chunks  
4. Build FAISS index  
5. Query embed  
6. Retrieve  
7. Build prompt  
8. Generate answer  

---

## Mini architecture

```
Docs → Chunk → Embed → FAISS  
Query → Embed → Retrieve → LLM → Answer
```
