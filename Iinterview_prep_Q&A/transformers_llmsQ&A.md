==============================
FILE: Transformers & LLMs
==============================

### HIGH PRIORITY

---

Q1. What is the attention mechanism? How does it work?

A1.
Attention is a mechanism that lets a model dynamically focus on the most relevant parts of the input when producing each element of the output. Instead of compressing everything into a fixed-size vector, attention creates a weighted combination of all input representations.

**The math**: Given a Query (Q), Key (K), and Value (V):
1. Compute attention scores: `score = Q · K^T` (dot product between query and each key).
2. Scale: `score = score / √d_k` (prevents values from growing too large with high dimensions).
3. Softmax: `weights = softmax(score)` — probabilities summing to 1.
4. Weighted sum: `output = weights · V` — each value is weighted by its relevance score.

**Intuition**: The Query asks "what am I looking for?", the Keys say "here's what I contain," and the Values say "here's the information." The dot product between Q and K measures relevance — high score means that Key's Value should contribute more to the output.

**Why scaling matters**: Without `√d_k` scaling, dot products in high dimensions produce very large values → softmax concentrates on one element → gradients become tiny. Scaling keeps the distribution well-behaved.

---

Q2. What is self-attention? How does it differ from cross-attention?

A2.
**Self-attention**: Q, K, and V all come from the same sequence. Each token attends to every other token (including itself) in the same sequence. This is how Transformers capture relationships within a sentence — "The cat sat on the mat because **it** was tired" — self-attention links "it" to "cat."

**Cross-attention**: Q comes from one sequence (e.g., decoder), while K and V come from another sequence (e.g., encoder). This is how the decoder "looks at" the encoder's output during translation — the decoder query asks "what in the source sentence is relevant for generating this target word?"

**Multi-head attention**: Instead of one attention computation, run multiple attention heads in parallel (e.g., 8 or 16). Each head has its own Q, K, V projections and can learn different types of relationships — one head for syntactic relationships, another for semantic, another for positional. Concatenate all head outputs and project to the final dimension.

**Computational cost**: Self-attention is O(n²) in sequence length — every token attends to every other token. This is why context windows are limited and why there's active research on efficient attention (sparse, linear, Flash Attention).

---

Q3. Explain the Transformer architecture (encoder-decoder structure).

A3.
The Transformer (Vaswani et al., 2017 — "Attention Is All You Need") replaced RNNs with pure attention, enabling parallel processing and better long-range dependencies.

**Encoder** (processes input):
- Stack of N identical layers (typically 6-12).
- Each layer: Multi-Head Self-Attention → Add & Norm → Feed-Forward Network → Add & Norm.
- Self-attention lets each token attend to all other tokens. Bidirectional — sees the full context.

**Decoder** (generates output):
- Stack of N identical layers.
- Each layer: Masked Self-Attention → Add & Norm → Cross-Attention (over encoder output) → Add & Norm → Feed-Forward → Add & Norm.
- Masked self-attention: Token at position i can only attend to positions ≤ i (causal masking). Prevents "cheating" during training.
- Cross-attention: Queries from decoder, Keys/Values from encoder. "What in the input is relevant for generating this output token?"

**Key components**:
- Positional encoding: Since there's no recurrence, position information is added via sinusoidal functions or learned embeddings.
- Layer normalization: Stabilizes training.
- Residual connections: Every sub-layer has a skip connection.

**Variants**: Encoder-only (BERT — classification, understanding), Decoder-only (GPT — generation), Encoder-Decoder (T5, BART — translation, summarization).

---

Q4. What is the difference between BERT and GPT?

A4.
Both are Transformers, but their architectures and training objectives are fundamentally different:

**BERT** (Bidirectional Encoder Representations from Transformers):
- **Architecture**: Encoder-only.
- **Training**: Masked Language Modeling — mask 15% of tokens, predict them using both left and right context. Bidirectional.
- **Strengths**: Excellent for understanding tasks — classification, NER, question answering. Sees the full context when making predictions.
- **Limitation**: Not designed for text generation — can't generate text autoregressively.

**GPT** (Generative Pretrained Transformer):
- **Architecture**: Decoder-only.
- **Training**: Causal Language Modeling — predict the next token given all previous tokens. Unidirectional (left-to-right).
- **Strengths**: Excellent for text generation, completion, and (with scale) general-purpose reasoning. Can be used for classification via prompting.
- **Limitation**: Unidirectional — each token only sees previous tokens, not future context.

**Practical choice**: For classification and understanding tasks on moderate-scale data, BERT (fine-tune on your labeled data). For general-purpose generation, reasoning, and few-shot learning, GPT-style models (prompt engineering or fine-tuning).

---

Q5. What is tokenization in LLMs? Compare different approaches.

A5.
Tokenization converts raw text into a sequence of integer token IDs that the model can process. It's the first and critical step in the NLP pipeline.

**Word-level**: Each word is a token. Problem: huge vocabulary, can't handle out-of-vocabulary words, no subword sharing ("running" and "run" are completely separate tokens).

**Character-level**: Each character is a token. Tiny vocabulary, handles any word. But sequences become very long, and the model must learn word meanings from characters — harder and slower.

**Subword tokenization** (dominant approach):
- **BPE (Byte Pair Encoding)**: Start with characters, iteratively merge the most frequent adjacent pairs. "unhappiness" → "un" + "happiness" or "un" + "hap" + "piness". Used by GPT models.
- **WordPiece**: Similar to BPE but merges based on likelihood, not frequency. Used by BERT. Prefixes subwords with "##".
- **SentencePiece**: Language-agnostic, treats input as raw bytes. Used by T5, LLaMA. Handles whitespace explicitly.

**Why subword tokenization wins**: Balances vocabulary size (~32K-100K tokens) with coverage. Common words are single tokens ("the"). Rare words decompose into meaningful subwords ("un" + "happy" + "ness"). Handles new/rare words gracefully.

**Token count matters** for cost and speed — 1 token ≈ 3/4 of a word in English.

---

Q6. What is fine-tuning vs. prompt engineering? When to use each?

A6.
**Fine-tuning**: Take a pretrained model, update its weights on your task-specific labeled data. The model fundamentally changes to become specialized.

**Prompt engineering**: Use the model as-is, crafting the input prompt to guide the model toward the desired output. No weight updates.

**When to fine-tune**:
- You have labeled data (thousands+ examples).
- You need consistently high accuracy on a specific task.
- The task is specialized and the base model doesn't perform well on it out of the box.
- You can afford the compute cost of training.
- Example: Medical report classification, company-specific entity extraction.

**When to prompt engineer**:
- Limited or no labeled data.
- Rapid prototyping — test if the approach works before investing in fine-tuning.
- General tasks where the base model already performs well.
- You need flexibility — the same model handles many different tasks via different prompts.
- Example: Summarization, question answering, translation with GPT-4.

**Middle ground**: Few-shot prompting (include examples in the prompt), RAG (retrieve relevant documents to augment the prompt), or parameter-efficient fine-tuning (LoRA — update a tiny fraction of weights).

---

Q7. What are hallucinations in LLMs? How can they be mitigated?

A7.
Hallucinations occur when the model generates text that is fluent and confident but factually incorrect or fabricated. The model doesn't "know" facts — it predicts statistically likely next tokens.

**Types**:
- **Factual hallucination**: "The Eiffel Tower was built in 1920" — confident but wrong.
- **Fabricated references**: Makes up papers, URLs, or citations that don't exist.
- **Logical inconsistency**: Self-contradicts within the same response.

**Why it happens**: The model is trained to produce plausible text, not verified facts. During training, it learns patterns and probabilities, not a knowledge database. When uncertain, it generates the most probable continuation — which may be wrong.

**Mitigation strategies**:
- **RAG (Retrieval-Augmented Generation)**: Retrieve relevant documents from a knowledge base and include them in the prompt. The model grounds its answer in the retrieved evidence.
- **Fine-tuning on verified data**: Train on high-quality, factually accurate datasets.
- **RLHF / Constitutional AI**: Train the model to refuse uncertain answers or say "I don't know."
- **Constrained decoding**: Limit outputs to known valid values (for structured tasks).
- **Fact-checking pipelines**: Verify generated claims against a knowledge base post-generation.
- **Temperature reduction**: Lower temperature → more conservative (highest probability) outputs. Reduces creative but false generations.

---

Q8. What is RAG (Retrieval-Augmented Generation)?

A8.
RAG combines a retrieval system with a generative model. Instead of relying solely on the model's parametric knowledge (what it learned during training), you retrieve relevant documents at query time and pass them as context.

**How it works**:
1. **Index**: Chunk your documents, generate embeddings for each chunk, store in a vector database (Pinecone, Weaviate, FAISS).
2. **Retrieve**: When a user asks a question, embed the query, find the top-K most similar document chunks via vector similarity search.
3. **Generate**: Pass the retrieved chunks + the user's question to the LLM. "Based on the following context, answer the question: ..."

**Why RAG wins over fine-tuning for knowledge**:
- No retraining needed — update the document store instead.
- Source attribution — you can cite which documents the answer came from.
- Reduces hallucinations — the model has the facts right in its context.
- Handles real-time or frequently changing information.

**Challenges**: Retrieval quality is critical (garbage in → garbage out). Chunk size affects performance — too small loses context, too large adds noise. Embedding model quality matters. Context window limits how much you can retrieve.

**Advanced patterns**: Hybrid search (vector + keyword), re-ranking retrieved results, multi-step retrieval (retrieve → generate sub-queries → retrieve again), query transformation.

---

Q9. What is LoRA (Low-Rank Adaptation) and parameter-efficient fine-tuning?

A9.
Fine-tuning a full LLM updates billions of parameters — expensive in compute and storage (you need a separate copy of the model per task). Parameter-efficient fine-tuning (PEFT) updates only a small fraction of parameters.

**LoRA** (Low-Rank Adaptation):
- Freeze all original model weights.
- For each weight matrix W, add a low-rank decomposition: `W' = W + A × B` where A is (d × r) and B is (r × d), with r << d (rank 8 or 16 typically).
- Only train A and B — thousands of parameters instead of billions.
- At inference, merge A × B into W — zero additional latency.

**Why it works**: Weight updates during fine-tuning are typically low-rank (they don't need the full parameter space). LoRA exploits this — it constrains updates to a low-rank subspace.

**Benefits**: 10-100x fewer trainable parameters. Multiple LoRA adapters for different tasks, same base model. Can be merged for deployment (no overhead). Train on a single consumer GPU.

**Other PEFT methods**: Prefix tuning (add trainable tokens to the input), Adapter layers (add small trainable layers between frozen layers), QLoRA (quantize the base model to 4-bit + apply LoRA — fits 65B models on a single 48GB GPU).

---

### MEDIUM PRIORITY

---

Q10. What is temperature, top-k, and top-p sampling in text generation?

A10.
These control the randomness and diversity of generated text by modifying how the next token is sampled from the probability distribution.

**Temperature**: Scales the logits before softmax. Temperature = 1.0 (default) — no change. Temperature < 1.0 — sharper distribution (model becomes more confident, picks highest probability tokens — more deterministic). Temperature > 1.0 — flatter distribution (more randomness, creative but less coherent).

**Top-k sampling**: Only consider the k most probable tokens. Set probabilities of all others to 0, renormalize. k=1 = greedy decoding. k=50 = consider top 50 tokens. Problem: fixed k doesn't adapt — sometimes 5 tokens are reasonable, sometimes 500 are.

**Top-p (nucleus) sampling**: Instead of a fixed count, consider the smallest set of tokens whose cumulative probability ≥ p. If p=0.9, include tokens until you've covered 90% of the probability mass. Adaptive — includes more tokens when the distribution is flat, fewer when it's peaked.

**In practice**: Most APIs expose temperature + top-p. For factual Q&A: low temperature (0.0-0.3). For creative writing: higher temperature (0.7-1.0). top-p around 0.9-0.95 is a common default.

---

Q11. What are embedding models? How are they used?

A11.
Embedding models convert text (words, sentences, documents) into dense numerical vectors in a high-dimensional space, where semantically similar texts are close together.

**Word embeddings** (Word2Vec, GloVe): Each word gets a fixed vector. "King" - "Man" + "Woman" ≈ "Queen." Captures semantic relationships. Limitation: one vector per word — "bank" (river) and "bank" (financial) share a vector.

**Contextual embeddings** (BERT, GPT): Same word gets different embeddings depending on context. "Bank of the river" vs "Bank of America" → different vectors. Much richer representations.

**Sentence/document embeddings** (Sentence-BERT, E5, text-embedding-ada-002): Produce a single vector for an entire text chunk. Optimized for similarity comparison.

**Applications**:
- **Semantic search**: Embed the query and documents → find closest vectors.
- **RAG**: Embed document chunks → retrieve relevant ones for LLM context.
- **Clustering**: Group similar documents by clustering their embeddings.
- **Classification**: Use embeddings as features for downstream classifiers.
- **Recommendation**: "Users who liked this document might like similar documents."

**Key metric**: Cosine similarity measures the angle between vectors (0-1 scale). 1 = identical direction = very similar meaning.

---

Q12. What is instruction tuning? How does it differ from standard fine-tuning?

A12.
**Standard fine-tuning**: Train on task-specific data (e.g., sentiment labeled as positive/negative). The model becomes specialized for one task.

**Instruction tuning**: Train on diverse tasks formatted as instructions — "Summarize this article:", "Translate to French:", "Answer this question:", "Write a poem about:". The model learns to follow instructions generally, not just one specific task.

**Why it matters**: A base GPT model trained on next-token prediction is good at completing text but bad at following instructions. It might continue "What is the capital of France?" with more questions instead of answering. Instruction tuning teaches it to actually respond to the instruction.

**Process**: Collect or generate a large dataset of (instruction, response) pairs across many tasks. Fine-tune the base model on this dataset. The result is a model that generalizes to new, unseen instructions.

**FLAN, InstructGPT, ChatGPT**: All examples of instruction-tuned models. InstructGPT showed that instruction tuning with RLHF made a smaller model preferred over a 100x larger base model.

---

Q13. What is RLHF (Reinforcement Learning from Human Feedback)?

A13.
RLHF aligns language models with human preferences — making them helpful, harmless, and honest. It's how ChatGPT went from a text-completing base model to a conversational assistant.

**Three steps**:
1. **Supervised fine-tuning (SFT)**: Fine-tune the base model on high-quality (prompt, response) pairs. Produces a decent instruction-following model.
2. **Reward model training**: Show humans pairs of model responses and ask which is better. Train a reward model to predict human preferences. This model scores responses.
3. **PPO (Proximal Policy Optimization)**: Use RL to optimize the language model to produce responses that score highly with the reward model. A KL divergence penalty prevents the model from drifting too far from the SFT model (avoids reward hacking).

**Why not just SFT?** Supervised learning needs one "correct" answer. But for open-ended generation, there are many good answers and many bad ones. RLHF captures preferences — "this response is better than that one" — which is easier for humans to judge than writing perfect responses.

**Alternatives**: DPO (Direct Preference Optimization) — skips the reward model entirely, directly optimizes preferences. Simpler, increasingly popular. Constitutional AI (Anthropic) — uses AI feedback instead of human feedback for some stages.

---

Q14. What is the context window in LLMs? Why does it matter?

A14.
The context window is the maximum number of tokens an LLM can process in a single forward pass — it's the model's "working memory."

**Sizes**: GPT-3.5 = 4K-16K tokens. GPT-4 = 8K-128K tokens. Claude = 200K tokens. Gemini = 1M+ tokens.

**Why it matters**:
- **Document processing**: A 128K context window can hold ~300 pages. Smaller windows require chunking and lose cross-chunk context.
- **Conversation history**: In multi-turn conversations, earlier messages get truncated when the context fills up. The model "forgets" early context.
- **RAG**: Larger context = more retrieved documents can be included.
- **Few-shot learning**: More examples fit in the prompt = better performance.

**Technical constraint**: Self-attention is O(n²) in sequence length — doubling the context window quadruples compute and memory. This is why longer context windows are computationally expensive.

**Solutions**: Efficient attention mechanisms (Flash Attention — faster, not reducing O(n²) complexity but reducing memory), RoPE/ALiBi (extrapolate to longer sequences than trained on), sparse attention, and sliding window attention.

**"Lost in the middle" problem**: Even with large context windows, models attend weakly to middle tokens — they perform best on information near the beginning or end of the context.

---

Q15. What is chain-of-thought prompting?

A15.
Chain-of-thought (CoT) prompting encourages the model to show its reasoning step by step before giving a final answer. This dramatically improves performance on tasks requiring multi-step reasoning — math, logic, complex analysis.

**How to use it**: Add "Let's think step by step" to your prompt, or provide few-shot examples that include reasoning chains.

**Without CoT**: "What is 17 × 24?" → "408" (might be wrong).
**With CoT**: "What is 17 × 24? Let's think step by step." → "17 × 20 = 340. 17 × 4 = 68. 340 + 68 = 408." (much more likely to be correct).

**Why it works**: The model generates intermediate tokens that serve as "scratch paper" — each step conditions the next step. The final answer is conditioned on a full reasoning chain, not just the question. It's essentially giving the model more compute time per answer.

**Variants**: Zero-shot CoT ("think step by step"), few-shot CoT (provide example reasoning chains), self-consistency (generate multiple reasoning chains, take the majority answer), tree-of-thought (explore multiple reasoning paths and backtrack).

**Limitation**: Longer outputs cost more tokens. The reasoning chain can still contain errors. Works best for models above ~10B parameters.

---

Q16. What are multi-modal models?

A16.
Multi-modal models can process and generate multiple types of data — text, images, audio, video — within a single model.

**Examples**: GPT-4V (text + images), Gemini (text + images + audio + video), DALL-E 3 (text → images), Whisper (audio → text), LLaVA (text + images), CLIP (aligns text and image representations).

**Architecture approaches**:
- **Separate encoders + fusion**: Use a vision encoder (ViT) for images and a language model for text, fuse their representations. LLaVA uses CLIP's vision encoder + LLaMA.
- **Unified architecture**: Process all modalities through the same Transformer. Tokenize images into patches, audio into spectrogram patches, and interleave with text tokens.

**Key capability**: Cross-modal understanding — "describe this image," "find this object in the image based on the text description," "generate an image matching this description."

**CLIP insight**: Train an image encoder and text encoder jointly so that matching image-text pairs have similar embeddings. Enables zero-shot image classification by comparing image embeddings with text embeddings of class descriptions.

---

Q17. What is quantization for LLMs?

A17.
Quantization reduces the precision of model weights from 32-bit or 16-bit floating point to lower precisions (8-bit, 4-bit, even 2-bit). A 70B parameter model at FP16 = 140 GB. At INT4 = ~35 GB — fits on consumer hardware.

**Types**:
- **Post-training quantization (PTQ)**: Quantize a trained model without retraining. Fast. Some quality loss. GPTQ, AWQ are popular methods.
- **Quantization-aware training (QAT)**: Simulate quantization during training — the model learns to be robust to reduced precision. Better quality but requires full training.

**Common precisions**:
- INT8: 2x compression, minimal quality loss. Standard for inference deployment.
- INT4: 4x compression, noticeable but acceptable quality loss for many tasks. GGUF/GGML formats for local deployment.
- 2-bit: Aggressive — significant quality loss, but enables huge models on tiny hardware.

**QLoRA**: Quantize the base model to 4-bit, apply LoRA adapters in 16-bit. Enables fine-tuning 65B models on a single 48GB GPU. The "Q" is the game changer.

**Trade-off**: More compression → more quality loss, especially on reasoning-heavy tasks. For simple Q&A and classification, INT4 is usually fine. For complex reasoning, stay at INT8 or FP16.

---

### LOW PRIORITY

---

Q18. What is knowledge distillation for LLMs?

A18.
Knowledge distillation trains a smaller "student" model to mimic a larger "teacher" model. The student doesn't learn from raw data — it learns from the teacher's outputs (soft labels / probability distributions).

**Why soft labels help**: The teacher's output distribution carries information beyond the correct answer. If the teacher assigns 70% to "cat" and 20% to "dog" for an image, the student learns that cats and dogs are somewhat similar. Hard labels (1 for cat, 0 for everything) lose this similarity information.

**For LLMs**: Generate training data using the teacher model. Fine-tune the student on these (input, teacher_output) pairs. The student learns the teacher's behavior without needing the original training data.

**Examples**: DistilBERT (40% smaller, 97% performance of BERT), Alpaca (fine-tuned LLaMA on ChatGPT-generated data), Phi (small models trained on GPT-4-generated synthetic data).

**Benefits**: Smaller model = faster inference, lower cost, deployable on edge devices. The teacher's knowledge is compressed into a more efficient form.

---

Q19. What is Mixture of Experts (MoE)?

A19.
MoE uses multiple specialized sub-networks (experts) with a gating mechanism that routes each input to only a few experts. This allows massive model capacity with active compute similar to a smaller model.

**Architecture**: Each Transformer layer has N expert feed-forward networks (e.g., 8 experts). A router/gating network takes the input and outputs weights — selecting the top-K experts (typically K=2) for that token. Only those K experts run; the rest are skipped.

**Benefits**: A model with 8 experts × 7B parameters each has 56B total parameters but only activates 2 × 7B = 14B per token. Get the quality of a large model with the speed of a smaller one.

**Examples**: Mixtral 8×7B (actually ~47B total, activates 13B per token), GPT-4 (rumored to be MoE), Switch Transformer (simplified routing — single expert per token).

**Challenges**: Load balancing (some experts get more traffic than others — add auxiliary loss to encourage even distribution), memory (all expert weights must be in memory even if not active), training instability.

---

Q20. What is AI safety and alignment?

A20.
**Alignment**: Ensuring AI systems do what humans actually want — not just what they're literally told. A model optimizing for user engagement might learn to be manipulative. A model told to "make the user happy" might lie.

**Key challenges**:
- **Specification problem**: Hard to precisely define what we want. "Be helpful" is vague.
- **Reward hacking**: The model finds loopholes in the reward function. Optimizes the metric, not the intent.
- **Deceptive alignment**: A sufficiently capable model might appear aligned during evaluation but behave differently in deployment.

**Current approaches**:
- **RLHF**: Train on human preferences to be helpful and harmless.
- **Constitutional AI**: Define principles (constitution) and train the model to follow them. Use AI feedback for scalability.
- **Red teaming**: Adversarially test models to find failure modes.
- **Guardrails**: External systems that filter unsafe inputs and outputs.

**Why it matters**: As models become more capable, the cost of misalignment grows. A model that can write code, access the internet, and take actions needs stronger alignment than one that just generates text.

---

Q21. What is LLM evaluation? How do you measure quality?

A21.
LLM evaluation is challenging because outputs are open-ended and quality is subjective. Multiple approaches exist:

**Automated benchmarks**: MMLU (multitask knowledge), HumanEval (code generation), HellaSwag (commonsense reasoning), TruthfulQA (factual accuracy), GSM8K (math). Compare models on standardized tasks.

**Perplexity**: How surprised the model is by the test data. Lower = better language modeling. But low perplexity doesn't guarantee useful outputs.

**Human evaluation**: Have humans rate outputs for helpfulness, accuracy, harmlessness. Gold standard but expensive, slow, and subjective.

**LLM-as-judge**: Use a strong model (GPT-4) to evaluate outputs of other models. Scalable and increasingly correlated with human judgments. But has biases (prefers verbose answers, its own style).

**Task-specific metrics**: BLEU/ROUGE for translation/summarization (n-gram overlap). F1 for extraction. Pass@k for code generation.

**A/B testing in production**: Deploy two models and measure real user engagement, satisfaction, and task completion. The ultimate evaluation, but requires production traffic.

**Challenge**: Models are getting good at benchmarks but may not generalize to real-world use. Benchmark contamination (training data includes benchmark questions) is a growing concern.

---

Q22. What is positional encoding, and why do Transformers need it?

A22.
Transformers process all tokens in parallel — unlike RNNs, there's no inherent notion of order. Without positional information, "The cat ate the fish" and "The fish ate the cat" would be identical to the model.

**Sinusoidal positional encoding** (original Transformer): Add fixed sine/cosine waves of different frequencies to the token embeddings. Each position gets a unique pattern. Can theoretically generalize to positions longer than those seen during training.

**Learned positional embeddings**: Train a lookup table of position embeddings (like word embeddings but for positions). Used by BERT, GPT-2. Limited to the maximum trained sequence length.

**Rotary Position Embedding (RoPE)**: Encodes position by rotating the Q and K vectors. Position information is baked into the attention computation, not the embeddings. Can extrapolate better to longer sequences. Used by LLaMA, Mistral.

**ALiBi (Attention with Linear Biases)**: Doesn't modify embeddings — instead adds a linear bias to attention scores based on distance. Closer tokens get higher scores. Simple, and extrapolates well to longer sequences.

**Why it matters**: The choice of positional encoding directly affects the model's ability to handle long sequences. RoPE with NTK-aware scaling is currently dominant for extending context windows.

---

Q23. What is the difference between greedy decoding, beam search, and sampling?

A23.
These are strategies for generating text token by token from the model's probability distribution.

**Greedy decoding**: Always pick the highest-probability token. Fast, deterministic. But gets stuck in local optima — might miss globally better sequences. "The best next word" doesn't always lead to "the best sentence."

**Beam search**: Maintain k candidate sequences (beams) at each step. Expand all beams, keep the top k. At the end, pick the highest-scoring complete sequence. Better than greedy but still deterministic and tends to produce generic, repetitive text.

**Sampling**: Randomly sample from the probability distribution. More diverse, creative outputs. Combined with temperature, top-k, top-p to control randomness.

**In practice**:
- Greedy: Code generation where there's one correct answer.
- Beam search: Machine translation, summarization — where fluency matters and outputs are constrained.
- Sampling + top-p + temperature: Open-ended generation, creative writing, chatbots — where diversity is valued.
- Most chat APIs default to sampling with temperature 0.7 and top-p 0.95.

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q24. What are vector databases, and why are they important for LLM applications?

A24.
Vector databases store and index high-dimensional vectors (embeddings) for fast similarity search. When you embed text, images, or other data into vectors using a model, you need to find the nearest neighbors efficiently — that's what vector databases do.

**Why they matter for LLMs**: RAG (Retrieval-Augmented Generation) needs to find relevant documents from millions of candidates in milliseconds. You embed all your documents, store them in a vector DB, then at query time, embed the user's question and find the top-k most similar document vectors.

**Key techniques**:
- **HNSW** (Hierarchical Navigable Small World): Graph-based index. Build a multi-layer graph where each node connects to its neighbors. Search navigates from coarse to fine layers. High recall, fast, but memory-intensive.
- **IVF** (Inverted File Index): Cluster vectors into partitions, search only relevant clusters. Faster but lower recall.
- **Product Quantization** (PQ): Compress vectors to reduce memory. Trade accuracy for memory efficiency.

**Popular vector databases**: Pinecone (managed), Weaviate (open source), Qdrant (open source, Rust), Milvus (open source, large-scale), Chroma (lightweight, for prototyping), pgvector (PostgreSQL extension).

**Key metrics**: Recall@k (fraction of true nearest neighbors found), queries per second (throughput), index build time, and memory usage. The fundamental trade-off is recall vs. speed.

**Practical tip**: Start with pgvector if your scale is under a few million vectors — avoid adding a new database to your architecture. Graduate to specialized vector DBs when you need billions of vectors or sub-millisecond latency.

---

Q25. What are AI agents and tool calling? How do they extend LLM capabilities?

A25.
An AI agent is an LLM that can take actions — it doesn't just generate text, it decides which tools to call, interprets results, and chains actions to accomplish complex tasks.

**Tool calling**: The LLM receives a list of available functions (tools) with descriptions and parameters. Given a user query, it outputs structured JSON specifying which function to call and with what arguments. The application executes the function and feeds the result back to the LLM.

```
User: "What's the weather in Tokyo?"
LLM output: {"tool": "get_weather", "args": {"city": "Tokyo"}}
System: calls API → {"temp": 22, "condition": "cloudy"}
LLM: "It's 22°C and cloudy in Tokyo right now."
```

**Agent loop**: Plan → Act → Observe → Repeat. The agent decides what to do, executes a tool, observes the result, and decides the next step. ReAct (Reasoning + Acting) is the foundational pattern.

**Key challenges**:
- **Tool selection**: With 100+ tools, the model must pick the right one. Good tool descriptions and few-shot examples help.
- **Error handling**: Tools fail. The agent needs to retry, try alternative tools, or ask for clarification.
- **Safety**: Agents can take destructive actions. Implement human-in-the-loop for anything irreversible.
- **Cost/latency**: Each tool call is an additional LLM call. Multi-step agents can be expensive and slow.

**Frameworks**: LangChain, LlamaIndex, OpenAI Assistants API, AutoGen. The space is evolving rapidly toward multi-agent systems where specialized agents collaborate.

---

Q26. What is Flash Attention, and why is it important?

A26.
Flash Attention is an exact attention algorithm (not an approximation) that is 2-4x faster and uses O(n) memory instead of O(n²) by being IO-aware — optimizing memory access patterns on GPUs.

**The problem**: Standard attention computes Q×K^T (n×n matrix), applies softmax, multiplies by V. That n×n attention matrix must be stored in GPU HBM (High Bandwidth Memory), which is large but slow. For sequence length 8192, that's 512MB per attention layer.

**Flash Attention's insight**: Instead of materializing the full n×n matrix, compute attention in tiles/blocks. Load blocks of Q, K, V from HBM to SRAM (on-chip, fast), compute partial attention, accumulate results, write back. The full attention matrix never exists in memory.

**Key technique**: Online softmax — compute softmax incrementally without needing the full row. Keep running statistics (max and sum) as you process each block.

**Impact**:
- **Memory**: O(n) instead of O(n²) — enables much longer sequences
- **Speed**: 2-4x faster (fewer HBM reads/writes, the bottleneck on modern GPUs)
- **Exact**: Not an approximation — produces identical results to standard attention

**Flash Attention 2**: Further optimized parallelism, better work partitioning between warps. Flash Attention 3: Exploits newer GPU features (FP8, asynchronous operations).

This is now the standard for training and inference of large models. Every major LLM (GPT-4, Llama, etc.) uses it.

---

### IMPORTANT

---

Q27. When would you use encoder-only, decoder-only, or encoder-decoder architectures?

A27.
**Encoder-only** (BERT, RoBERTa, DeBERTa): Bidirectional attention — each token sees all other tokens. Best for understanding tasks where you have the full input:
- Text classification, sentiment analysis
- Named entity recognition (NER)
- Sentence embeddings / semantic similarity
- Extractive QA (answer is a span in the document)

**Decoder-only** (GPT, Llama, Claude): Causal (left-to-right) attention — each token only sees previous tokens. Best for generation:
- Text generation, chatbots
- Code generation
- In-context learning / few-shot prompting
- Increasingly used for everything (scaling laws favor decoder-only)

**Encoder-decoder** (T5, BART, mBART): Encoder processes input bidirectionally, decoder generates output autoregressively with cross-attention to encoder. Best for sequence-to-sequence:
- Machine translation
- Summarization
- Generative QA
- Any task where input and output are distinct sequences

**Current trend**: Decoder-only models dominate. GPT-4, Claude, Llama are all decoder-only. They handle all tasks through prompting rather than fine-tuning. Encoder-only survives for classification/embedding tasks where bidirectional context genuinely helps and efficiency matters.

---

Q28. What is the "lost in the middle" problem with LLMs?

A28.
When given a long context with relevant information distributed throughout, LLMs tend to use information from the beginning and end of the context but ignore or underweight information in the middle.

**The evidence**: Liu et al. (2023) showed that model accuracy drops significantly when the relevant information is placed in the middle of a long context, compared to the beginning or end. This U-shaped performance curve holds across multiple models and tasks.

**Why it happens**: Positional biases from training — models see more examples where important information is near the beginning (titles, abstracts) or end (conclusions). Attention patterns may also naturally focus on boundary positions.

**Practical implications for RAG**:
- Don't just stuff all retrieved documents into the prompt
- Place the most relevant documents at the beginning and end
- Rerank retrieved documents by relevance before inserting
- Consider summarizing middle sections
- Use smaller, more targeted contexts rather than maximizing context length

**Mitigations being explored**:
- Training with diverse information placement
- Better positional encodings (ALiBi, RoPE variations)
- Retrieval-augmented architectures that break dependence on position
- Chunking strategies that keep related information together

---

Q29. What is speculative decoding, and how does it speed up LLM inference?

A29.
Speculative decoding uses a small, fast "draft" model to generate candidate tokens, then verifies them in parallel with the large "target" model. Since verification is cheaper than generation (due to KV-cache and parallel processing), you get the large model's quality at closer to the small model's speed.

**How it works**:
1. Small model generates k candidate tokens autoregressively (fast)
2. Large model processes all k tokens in a single forward pass (parallel, like a prompt)
3. Large model verifies each token — if it agrees, accept it. If it disagrees at position i, reject tokens i onward and resample from the large model's distribution
4. Repeat from step 1

**Why it works**: For "easy" tokens (common words, predictable continuations), the small model agrees with the large model most of the time. Only "hard" tokens need the large model's full power. Acceptance rates of 70-90% are common for well-matched draft models.

**Speedup**: 2-3x for autoregressive generation. The speedup is lossless — the output distribution is mathematically identical to the large model alone.

**Variants**:
- **Self-speculative**: Use early exit from the same model as the "draft"
- **Medusa**: Add multiple prediction heads to the target model, predicting several future tokens simultaneously
- **Lookahead decoding**: Use Jacobi iteration to predict multiple tokens

Used in production at major LLM providers to reduce inference costs and latency.

---

### GOOD-TO-HAVE

---

Q30. What's the difference between dense and sparse retrieval?

A30.
**Dense retrieval**: Encode queries and documents as continuous vector embeddings (using models like BGE, E5, Cohere Embed). Find similar documents via approximate nearest neighbor search. Captures semantic meaning — "car" and "automobile" are close in embedding space.

**Sparse retrieval**: Represent documents as sparse vectors of term frequencies (TF-IDF, BM25). Match based on exact keyword overlap. Fast, interpretable, and requires no training.

**Comparison**:
- **Semantic matching**: Dense wins. "How do I fix a bug?" matches "debugging techniques" even without word overlap.
- **Exact keyword matching**: Sparse wins. Searching for error code "ERR_0x42" — dense models might not encode rare tokens well.
- **Zero-shot / domain shift**: Sparse is more robust. Dense models trained on general text may underperform on specialized domains (legal, medical) without fine-tuning.
- **Efficiency**: BM25 is extremely fast with inverted indexes. Dense retrieval requires embedding computation + ANN search.

**Hybrid retrieval** (best practice for production RAG):
1. Run both dense and sparse retrieval
2. Combine results with Reciprocal Rank Fusion (RRF) or learned score combination
3. Rerank the combined results with a cross-encoder

This captures both semantic and lexical matches. Hybrid consistently outperforms either alone. Most production RAG systems (Azure AI Search, Elasticsearch with vector plugin, Weaviate) support hybrid search natively.

---

Q31. What are reasoning models, and how do they differ from standard LLMs?

A31.
Reasoning models (like OpenAI's o1/o3, DeepSeek R1) are trained to "think before answering" — they generate an internal chain of thought before producing the final answer.

**How they differ**:
- **Standard LLM**: Generates the answer token by token. Each token gets the same compute. Complex problems get the same "thinking time" as simple ones.
- **Reasoning model**: Generates a (sometimes hidden) chain of reasoning first, then the answer. More compute is spent on harder problems — adaptive compute.

**Training approach**: Typically trained with reinforcement learning (RL) on verifiable tasks — math, coding, logic puzzles. The model learns to break problems into steps, verify intermediate results, and backtrack when reasoning paths fail. Process reward models (PRMs) reward correct intermediate steps, not just final answers.

**Trade-offs**:
- Much better at math, coding, logic, and multi-step reasoning
- Slower and more expensive (longer outputs = more tokens = more cost)
- Can "overthink" simple questions
- The reasoning trace can reveal the model's thought process (useful for debugging)

**When to use**: Complex reasoning tasks (math proofs, code debugging, multi-step planning). For simple factual QA or creative writing, standard models are faster and cheaper.

This represents a shift from "scaling model size" to "scaling inference-time compute" — making models think harder rather than making them bigger.