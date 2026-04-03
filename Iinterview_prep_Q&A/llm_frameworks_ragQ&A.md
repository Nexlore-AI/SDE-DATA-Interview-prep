# LLM Frameworks (LangChain, LlamaIndex, RAG) — Interview Q&A

---

## 1. What is LangChain and what problems does it solve?

LangChain is a **framework for building applications powered by LLMs**. It provides abstractions for:

- **Chains:** Sequence of LLM calls + tools
- **Agents:** LLMs that decide which tools to use dynamically
- **Memory:** Conversation history management
- **Retrieval:** Connect LLMs to external data (RAG)
- **Callbacks:** Logging, streaming, monitoring

**Core problem it solves:** LLMs alone can't access real-time data, execute code, or interact with APIs. LangChain orchestrates these capabilities.

---

## 2. What is RAG (Retrieval-Augmented Generation)?

RAG combines **retrieval** (search relevant documents) with **generation** (LLM produces answer using those documents).

```
User Query → Embedding → Vector Search → Top-K Documents → LLM + Context → Answer
```

**Why RAG:**
- LLMs have knowledge cutoff dates
- LLMs hallucinate when they don't know
- RAG grounds responses in actual documents
- Cheaper than fine-tuning (no retraining needed)
- Data stays private (documents don't go into model weights)

**Pipeline:**
1. **Indexing:** Split documents → embed with model (OpenAI, Cohere) → store in vector DB
2. **Retrieval:** Embed user query → search vector DB for similar chunks → return top-K
3. **Generation:** Pass query + retrieved chunks to LLM → generate grounded answer

---

## 3. What are vector databases? Compare popular options.

Vector databases store and search **high-dimensional embeddings** using similarity metrics (cosine, L2, dot product).

| Database | Type | Key Feature |
|----------|------|------------|
| **Pinecone** | Managed cloud | Fully managed, easy setup, metadata filtering |
| **Weaviate** | Open source | GraphQL API, hybrid search (vector + keyword) |
| **ChromaDB** | Open source | Lightweight, great for prototyping |
| **Qdrant** | Open source | Rust-based, fast, filtering capabilities |
| **Milvus** | Open source | Scalable, GPU-accelerated |
| **pgvector** | Postgres extension | Use existing Postgres infra |
| **FAISS** | Library (Meta) | In-memory, fastest for research/small scale |

**Choosing:**
- Prototype → ChromaDB or FAISS
- Production (managed) → Pinecone
- Production (self-hosted) → Weaviate, Qdrant, or Milvus
- Already on Postgres → pgvector

---

## 4. What are embeddings? How are they used in LLM applications?

Embeddings are **dense vector representations** of text (or images, audio) in high-dimensional space where semantically similar content is close together.

**Models:**
- OpenAI `text-embedding-3-small` (1536 dims)
- Cohere `embed-v3` (1024 dims)
- Open source: `sentence-transformers` (e.g., `all-MiniLM-L6-v2`, 384 dims)

**Use cases:**
- Semantic search (RAG retrieval)
- Recommendation systems
- Clustering / classification
- Deduplication

**Key concept:** Cosine similarity between embeddings measures semantic relatedness. "King" is closer to "Queen" than to "Car."

---

## 5. What is chunking strategy and why does it matter for RAG?

Documents must be split into **chunks** before embedding. Chunk size directly impacts retrieval quality.

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Fixed-size** | Split every N characters/tokens | General purpose |
| **Recursive** | Split by paragraphs → sentences → words | Structured text |
| **Semantic** | Split at topic boundaries using embeddings | High-quality retrieval |
| **Document-specific** | Markdown headers, HTML tags, code blocks | Structured documents |

**Chunk size trade-offs:**
- **Too small (100 tokens):** Loses context, may miss relevant info
- **Too large (2000 tokens):** Dilutes relevance, wastes context window
- **Sweet spot:** 500-1000 tokens with 10-20% overlap

**Overlap:** Include some text from the previous chunk to avoid losing information at boundaries.

---

## 6. What is prompt engineering? Common techniques?

| Technique | Description | Example |
|-----------|-------------|---------|
| **Zero-shot** | No examples | "Classify this as positive/negative" |
| **Few-shot** | Provide examples | "Example 1: ... Example 2: ... Now classify:" |
| **Chain-of-Thought (CoT)** | Ask model to reason step by step | "Let's think step by step..." |
| **ReAct** | Reason + Act (think → use tool → observe → repeat) | Agent loops |
| **System prompts** | Set behavior/role | "You are a helpful coding assistant..." |

**RAG prompt template:**
```
Context: {retrieved_documents}
Question: {user_question}
Answer based ONLY on the provided context. If the answer is not in the context, say "I don't know."
```

---

## 7. What is LlamaIndex and how does it differ from LangChain?

| Feature | LangChain | LlamaIndex |
|---------|-----------|------------|
| Focus | General LLM app framework (agents, chains, tools) | Data-focused (indexing, retrieval, RAG) |
| Strength | Agent orchestration, tool use | Structured data querying, document processing |
| Indexing | Basic (text splitter + vector store) | Advanced (tree, keyword, knowledge graph indices) |
| Complexity | More flexible, steeper learning curve | More opinionated, simpler for RAG |

**Use LangChain when:** Building agents, tool use, complex chains, multi-step workflows.
**Use LlamaIndex when:** RAG is the primary use case, working with structured documents.
**Both together:** LlamaIndex for retrieval + LangChain for orchestration.

---

## 8. What are LLM Agents? How do they work?

An **agent** is an LLM that can **decide which actions to take** to accomplish a goal.

**Architecture:**
```
User Input → LLM (decides action) → Tool Call → Observation → LLM (decides next) → ... → Final Answer
```

**Components:**
- **LLM:** The brain that reasons and plans
- **Tools:** Functions the agent can call (search, calculator, API calls, code execution)
- **Memory:** Past conversation and actions
- **Planning:** ReAct, Plan-and-Execute, Tree of Thought

**Example (ReAct loop):**
```
Thought: I need to find the current weather in NYC.
Action: search("current weather NYC")
Observation: 72°F, sunny
Thought: I now have the answer.
Final Answer: It's currently 72°F and sunny in NYC.
```

---

## 9. What is fine-tuning vs RAG? When to use each?

| | Fine-tuning | RAG |
|---|---|---|
| **What it does** | Updates model weights on custom data | Retrieves context at inference time |
| **Data requirement** | Structured training data (instruction + response pairs) | Documents in any format |
| **Cost** | High (GPU compute for training) | Low (embedding + vector DB) |
| **Knowledge update** | Requires retraining | Just update the vector store |
| **Best for** | Changing model behavior/style/format | Adding domain knowledge |
| **Hallucination** | Can still hallucinate | Grounded in retrieved docs |

**Use fine-tuning when:** You need the model to learn a specific style, format, or domain behavior.
**Use RAG when:** You need the model to reference specific, updatable documents.
**Use both when:** Fine-tune for style + RAG for knowledge (common in production).

---

## 10. What are the common techniques for improving RAG quality?

| Technique | Description |
|-----------|-------------|
| **Hybrid search** | Combine vector (semantic) + keyword (BM25) search |
| **Reranking** | Use a cross-encoder to rerank retrieved results (Cohere Rerank, BGE reranker) |
| **Query expansion** | LLM rewrites the query for better retrieval |
| **HyDE** | Generate hypothetical answer → embed that → search (Hypothetical Document Embeddings) |
| **Metadata filtering** | Filter by date, source, category before vector search |
| **Parent-child chunks** | Retrieve small chunks for precision, return parent chunk for context |
| **Multi-query** | Generate multiple search queries from one user question |
| **Contextual compression** | LLM extracts only relevant portions from retrieved chunks |

---

## 11. What is function calling / tool use in LLMs?

Modern LLMs (GPT-4, Claude, Gemini) can generate **structured function calls** instead of plain text.

```json
// You define available functions:
{
  "name": "get_weather",
  "parameters": {"location": "string", "unit": "celsius|fahrenheit"}
}

// LLM outputs:
{
  "function_call": {
    "name": "get_weather",
    "arguments": {"location": "NYC", "unit": "fahrenheit"}
  }
}
```

**Your code** executes the function, returns the result, and the LLM generates the final response.

**Use cases:** API integration, database queries, code execution, multi-step tool use.

---

## 12. What is LoRA / QLoRA fine-tuning?

**LoRA (Low-Rank Adaptation):** Instead of updating all model weights, inject small trainable matrices into each layer.

- Original weight matrix W (dim: d × d)
- LoRA adds: W + A × B where A (d × r), B (r × d), r << d (rank 4-64)
- Only train A and B → **0.1-1% of parameters**
- Merge at inference → no latency overhead

**QLoRA:** LoRA + 4-bit quantization of base model weights.
- 65B model fits on single 48GB GPU for training
- Near full fine-tuning quality at fraction of compute

---

## 13. What is RLHF and how does it relate to LLM training?

**RLHF (Reinforcement Learning from Human Feedback)** aligns LLMs with human preferences.

**Three stages:**
1. **Supervised Fine-Tuning (SFT):** Train on high-quality instruction-response pairs
2. **Reward Model Training:** Humans rank model outputs → train a reward model
3. **RL (PPO):** Use reward model to update the LLM via reinforcement learning

**Alternatives:**
- **DPO (Direct Preference Optimization):** Skip the reward model, optimize directly on preference pairs. Simpler, cheaper.
- **RLAIF:** Use AI to generate feedback instead of humans.

---

## 14. What are common LLM evaluation metrics?

| Metric | Measures | Used For |
|--------|----------|----------|
| **Perplexity** | How well model predicts next token | Language model quality |
| **BLEU** | N-gram overlap with reference | Translation |
| **ROUGE** | Recall of n-grams from reference | Summarization |
| **BERTScore** | Semantic similarity using BERT embeddings | General text quality |
| **Human evaluation** | Relevance, helpfulness, harmlessness | Gold standard |
| **LLM-as-judge** | Another LLM rates the output | Scalable evaluation |
| **Faithfulness** | Does the answer match the source docs? | RAG evaluation |
| **Relevance** | Are retrieved docs relevant to query? | RAG retrieval quality |

---

## 15. How do you deploy LLMs in production?

**Serving options:**
| Option | Best For |
|--------|----------|
| **API providers** (OpenAI, Anthropic) | Simplest, pay-per-token |
| **vLLM** | Self-hosted, high-throughput serving with PagedAttention |
| **TGI** (HuggingFace) | Self-hosted, streaming, multi-model |
| **Triton** (NVIDIA) | Enterprise, multi-framework serving |
| **Ollama** | Local development, easy setup |

**Key considerations:**
- **Latency:** Time to first token, tokens per second
- **Throughput:** Concurrent requests handled
- **Cost:** GPU hours, API costs
- **Quantization:** GPTQ, AWQ, GGUF reduce model size 2-4x with minimal quality loss
- **KV cache optimization:** PagedAttention (vLLM) manages memory efficiently
- **Batching:** Continuous batching serves multiple requests simultaneously
