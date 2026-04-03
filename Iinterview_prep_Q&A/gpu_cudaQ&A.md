# GPU & CUDA Programming — Interview Q&A

---

## 1. What is a GPU and why is it used for deep learning?

A **GPU (Graphics Processing Unit)** has thousands of small cores optimized for **parallel computation**.

| Feature | CPU | GPU |
|---------|-----|-----|
| Cores | 8-64 (powerful) | 1000s-10000s (simpler) |
| Optimized for | Sequential, complex tasks | Parallel, repetitive tasks |
| Memory | Large shared RAM | High-bandwidth VRAM (HBM) |
| Use case | General computing | Matrix operations, graphics |

**Why DL:** Neural network training = massive matrix multiplications. A single forward pass of a 7B model involves billions of multiply-add operations → GPUs handle this in parallel (100x faster than CPUs).

---

## 2. What is CUDA?

**CUDA (Compute Unified Device Architecture)** is NVIDIA's parallel computing platform and API. It lets you write code that runs on NVIDIA GPUs.

**Programming model:**
```c
// CUDA kernel — runs on GPU
__global__ void add(float *a, float *b, float *c, int n) {
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    if (idx < n) c[idx] = a[idx] + b[idx];
}

// Launch kernel
add<<<numBlocks, threadsPerBlock>>>(a, b, c, n);
```

**Hierarchy:**
- **Thread:** Smallest execution unit
- **Warp:** 32 threads (execute in lockstep on SM)
- **Block:** Group of threads (share memory)
- **Grid:** Group of blocks (entire kernel launch)

---

## 3. What are CUDA cores, Tensor cores, and SMs?

| Component | Function |
|-----------|----------|
| **CUDA Core** | Basic floating-point arithmetic unit (1 multiply-add per cycle) |
| **Tensor Core** | Specialized for matrix multiply-accumulate (4x4 matrix ops per cycle) |
| **SM (Streaming Multiprocessor)** | Processing unit containing multiple CUDA cores + shared memory |

**Tensor Cores** are the key for deep learning:
- Operate on mixed-precision (FP16 input, FP32 accumulate)
- 8-16x faster than CUDA cores for matrix ops
- Used by cuBLAS, cuDNN automatically when enabled

**GPU hierarchy:** GPU → SMs → Warps → Threads

---

## 4. What is GPU memory hierarchy?

```
Global Memory (VRAM, HBM) — Large (16-80 GB), slow
    ↓
Shared Memory — Per SM, fast (~100x faster), programmer-managed
    ↓
Registers — Per thread, fastest, limited
    ↓
L1/L2 Cache — Hardware-managed
```

**Key insight:** Moving data between GPU and CPU (PCIe/NVLink) is expensive. Keep data on GPU. The biggest bottleneck in DL training is often **memory bandwidth**, not compute.

---

## 5. What is mixed-precision training? Why use it?

Mixed-precision uses **FP16 (half precision) for computation** and **FP32 for critical operations** (loss scaling, weight updates).

**Benefits:**
- **2x memory reduction:** FP16 weights use half the memory → larger batch sizes or models
- **2-8x faster compute:** Tensor Cores operate on FP16
- **Same accuracy:** With loss scaling, no quality degradation

**How (with PyTorch):**
```python
scaler = torch.cuda.amp.GradScaler()
with torch.cuda.amp.autocast():
    output = model(input)
    loss = criterion(output, target)
scaler.scale(loss).backward()
scaler.step(optimizer)
scaler.update()
```

**BF16 (bfloat16):** Same range as FP32, less precision. Preferred on A100/H100 (no loss scaling needed).

---

## 6. What are the common NVIDIA GPU generations for AI?

| GPU | Architecture | VRAM | Tensor Core Gen | Key Feature |
|-----|-------------|------|-----------------|-------------|
| V100 | Volta | 16/32 GB HBM2 | 1st gen | First Tensor Cores |
| A100 | Ampere | 40/80 GB HBM2e | 3rd gen | MIG, TF32, sparsity |
| H100 | Hopper | 80 GB HBM3 | 4th gen | Transformer Engine, FP8, 3x A100 |
| H200 | Hopper | 141 GB HBM3e | 4th gen | Larger memory for inference |
| B200 | Blackwell | 192 GB HBM3e | 5th gen | 2.5x H100 for LLMs |

---

## 7. What is model parallelism vs data parallelism?

| Strategy | How It Works | Use Case |
|----------|-------------|----------|
| **Data Parallelism** | Same model on multiple GPUs, different data batches. Gradients averaged. | Model fits on one GPU |
| **Tensor Parallelism** | Split layers across GPUs (each GPU computes part of a matrix multiplication) | Large individual layers |
| **Pipeline Parallelism** | Split model layers across GPUs sequentially | Very deep models |
| **FSDP / ZeRO** | Shard model parameters, gradients, optimizer states across GPUs | Memory-efficient training |

**PyTorch DDP (Data Parallel):**
```python
model = torch.nn.parallel.DistributedDataParallel(model, device_ids=[local_rank])
```

**DeepSpeed ZeRO stages:**
- Stage 1: Shard optimizer states
- Stage 2: + shard gradients
- Stage 3: + shard parameters (full model sharding)

---

## 8. What is NVLink and why does it matter?

**NVLink** is NVIDIA's high-bandwidth GPU-to-GPU interconnect.

| Interconnect | Bandwidth |
|-------------|-----------|
| PCIe Gen 4 | ~32 GB/s |
| PCIe Gen 5 | ~64 GB/s |
| NVLink (A100) | 600 GB/s |
| NVLink (H100) | 900 GB/s |

**Why it matters:** Multi-GPU training requires frequent data exchange (gradient synchronization, tensor parallelism). NVLink is 10-30x faster than PCIe → critical for training performance.

---

## 9. What is GPU memory management in PyTorch?

```python
torch.cuda.memory_allocated()      # Currently used memory
torch.cuda.memory_reserved()       # Total memory reserved by PyTorch
torch.cuda.max_memory_allocated()  # Peak memory usage
torch.cuda.empty_cache()           # Release unused cached memory
```

**Common OOM solutions:**
1. Reduce batch size
2. Use gradient accumulation (simulate large batch)
3. Use mixed precision (FP16/BF16)
4. Use gradient checkpointing (recompute activations instead of storing)
5. Use FSDP / DeepSpeed ZeRO
6. Use model offloading (CPU offload)

**Gradient checkpointing:**
```python
from torch.utils.checkpoint import checkpoint
output = checkpoint(model.layer, input)  # Saves memory, costs compute
```

---

## 10. What is quantization for inference?

Quantization reduces model weight precision to reduce memory and speed up inference.

| Precision | Bits | Memory (7B model) | Use |
|-----------|------|-------------------|-----|
| FP32 | 32 | ~28 GB | Training |
| FP16/BF16 | 16 | ~14 GB | Training/Inference |
| INT8 | 8 | ~7 GB | Inference |
| INT4 | 4 | ~3.5 GB | Inference on consumer GPUs |

**Methods:**
- **GPTQ:** Post-training quantization using calibration data
- **AWQ:** Activation-aware quantization (better quality)
- **GGUF:** Quantization format for llama.cpp (CPU inference)
- **bitsandbytes:** Dynamic quantization in PyTorch (`load_in_4bit=True`)

---

## 11. What is KV cache and why does it matter for LLM inference?

During autoregressive generation, each new token needs to attend to **all previous tokens**. Without KV cache, you'd recompute attention for all previous tokens at each step.

**KV cache** stores the Key and Value matrices from previous tokens:
- Memory per token: 2 × num_layers × hidden_dim × precision_bytes
- For Llama 2 70B (FP16): ~2 MB per token per sequence

**Problem at scale:** Long contexts (32K+ tokens) × many concurrent users → KV cache eats all GPU memory.

**Solutions:**
- **PagedAttention (vLLM):** Manages KV cache like OS virtual memory — non-contiguous blocks, efficient sharing
- **Multi-Query Attention / Grouped-Query Attention:** Reduce KV cache size by sharing keys/values across heads
- **Sliding window attention:** Only cache recent tokens (Mistral)

---

## 12. What is MIG (Multi-Instance GPU)?

Available on A100/H100, **MIG** splits one physical GPU into up to 7 isolated instances.

**Each instance** has dedicated:
- Compute resources (SMs)
- Memory
- Memory bandwidth

**Use case:** Serve multiple small models on one GPU, multi-tenant inference, CI/CD GPU testing.

---

## 13. Common GPU performance profiling tools?

| Tool | Purpose |
|------|---------|
| `nvidia-smi` | GPU utilization, memory, temperature |
| `torch.profiler` | PyTorch operation-level profiling |
| NVIDIA Nsight Systems | System-wide CPU+GPU timeline |
| NVIDIA Nsight Compute | Kernel-level analysis |
| `nvtop` | Interactive GPU monitoring (like htop) |

**Key metrics to watch:**
- GPU utilization % (should be > 80%)
- Memory utilization
- SM occupancy
- Memory bandwidth utilization
- PCIe/NVLink throughput
