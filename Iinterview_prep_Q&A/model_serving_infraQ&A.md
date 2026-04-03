# Model Serving & MLOps Infrastructure — Interview Q&A

---

## 1. What is model serving and why is it different from model training?

| Aspect | Training | Serving (Inference) |
|--------|----------|-------------------|
| Goal | Learn parameters from data | Produce predictions for users |
| Latency | Hours/days acceptable | Milliseconds matter |
| Throughput | Batch processing | Concurrent real-time requests |
| Hardware | Max GPU utilization | Cost-efficient, right-sized |
| Optimization | Data parallelism, large batches | Quantization, batching, caching |
| Failure | Restart from checkpoint | Must be highly available |

---

## 2. What is NVIDIA Triton Inference Server?

Triton is an **open-source inference server** that supports multiple frameworks and optimizations.

**Key features:**
- **Multi-framework:** TensorRT, PyTorch, TensorFlow, ONNX, Python backend
- **Dynamic batching:** Automatically groups incoming requests into batches
- **Concurrent model execution:** Run multiple models on one GPU
- **Model ensembles:** Chain models (preprocessing → model → postprocessing)
- **Metrics:** Prometheus metrics built-in

**Model repository structure:**
```
model_repository/
├── model_a/
│   ├── config.pbtxt
│   └── 1/
│       └── model.onnx
├── model_b/
│   ├── config.pbtxt
│   └── 1/
│       └── model.pt
```

**When to use:** Enterprise production, multi-model serving, need for GPU optimization.

---

## 3. What is vLLM and how is it different from other serving solutions?

**vLLM** is an LLM-specific serving engine optimized for high throughput.

**Key innovation — PagedAttention:**
- Manages KV cache like OS virtual memory (paging)
- Non-contiguous memory allocation → eliminates fragmentation
- Memory sharing across sequences (beam search, parallel sampling)
- **2-4x higher throughput** than HuggingFace TGI

**Features:**
- Continuous batching (no waiting for batch to complete)
- Tensor parallelism across GPUs
- Quantization support (AWQ, GPTQ)
- OpenAI-compatible API

```bash
python -m vllm.entrypoints.openai.api_server \
    --model meta-llama/Llama-2-7b-chat-hf \
    --tensor-parallel-size 2
```

---

## 4. What is TensorRT and why use it?

**TensorRT** is NVIDIA's SDK for **high-performance inference optimization**.

**Optimizations:**
- **Layer fusion:** Combine multiple operations into one kernel
- **Precision calibration:** FP32 → FP16/INT8 with minimal accuracy loss
- **Kernel auto-tuning:** Select best kernel for target GPU
- **Dynamic tensor memory:** Minimize memory footprint

**Workflow:**
```
PyTorch Model → ONNX Export → TensorRT Engine → Deploy
```

**Speedup:** 2-6x faster than native PyTorch inference typically.

**Limitation:** GPU-specific (need to rebuild for different GPU architectures).

---

## 5. What is model versioning and A/B testing for ML?

**Model versioning:** Track model artifacts, hyperparameters, metrics, and data versions.
- **MLflow Model Registry:** Staging → Production transitions
- **DVC:** Version control for data + models
- **Weights & Biases:** Experiment tracking + artifact storage

**A/B testing for ML:**
```
Traffic → Router (90% → Model v1, 10% → Model v2)
                    ↓                      ↓
              Predictions              Predictions
                    ↓                      ↓
              Metrics logged          Metrics logged
                    ↓
              Compare (latency, accuracy, business KPI)
```

**Canary deployment:** Roll out new model to small % of traffic, monitor, gradually increase.
**Shadow mode:** Run new model in parallel, log predictions, but serve old model's results. Compare offline.

---

## 6. What is feature serving / feature store?

A **feature store** is a centralized repository for storing, versioning, and serving ML features.

**Components:**
- **Offline store:** Historical features for training (data warehouse — BigQuery, Redshift)
- **Online store:** Real-time features for inference (Redis, DynamoDB)
- **Feature transformation:** Compute features from raw data

**Tools:** Feast (open source), Tecton, Amazon SageMaker Feature Store.

**Why:**
- **Consistency:** Same feature logic for training and serving (prevents training-serving skew)
- **Reusability:** Features shared across models
- **Point-in-time correctness:** Avoid data leakage (use features available at prediction time)

---

## 7. What is model monitoring and drift detection?

**Types of drift:**
| Type | What Changed | Detection |
|------|-------------|-----------|
| **Data drift** | Input distribution changed | KS test, PSI, Jensen-Shannon divergence |
| **Concept drift** | Relationship between inputs and outputs changed | Monitor prediction accuracy over time |
| **Feature drift** | Individual feature distributions change | Per-feature statistical tests |

**Monitoring setup:**
- Log all predictions + input features
- Compare current distributions against reference (training data)
- Alert when drift exceeds threshold
- Retrain on fresh data when drift detected

**Tools:** Evidently AI, Whylabs, NannyML, custom Prometheus + Grafana dashboards.

---

## 8. What is the difference between batch and real-time inference?

| Feature | Batch | Real-time |
|---------|-------|-----------|
| Latency | Minutes-hours | Milliseconds |
| Trigger | Scheduled (cron) | User request |
| Input | Large dataset | Single request |
| Throughput | High (process millions) | Lower (per-request) |
| Infra | Spark job, batch pipeline | REST/gRPC API, model server |
| Use case | Recommendations precomputation, reports | Fraud detection, search ranking |

**Hybrid:** Precompute batch features → store in feature store → combine with real-time features at serving time.

---

## 9. Explain the architecture of a production ML system.

```
Data Sources → Feature Pipeline → Feature Store → Training Pipeline
                                       ↓               ↓
                                  Online Store    Model Registry
                                       ↓               ↓
User Request → API Gateway → Model Server (Triton/vLLM) → Response
                                  ↓
                          Prediction Logger
                                  ↓
                          Monitoring (drift, latency, errors)
                                  ↓
                          Alert → Retrain trigger
```

**Components:**
1. **Data pipeline:** Ingest, clean, store raw data
2. **Feature pipeline:** Compute features (batch + streaming)
3. **Training pipeline:** Train, evaluate, register model
4. **Serving pipeline:** Model server + feature lookup
5. **Monitoring pipeline:** Log predictions, detect drift, alert

---

## 10. What are common model optimization techniques for inference?

| Technique | Speedup | Trade-off |
|-----------|---------|-----------|
| **Quantization** (INT8/INT4) | 2-4x | Slight accuracy loss |
| **Pruning** | 2-3x | Removes unimportant weights |
| **Distillation** | Variable | Train smaller model to mimic larger one |
| **ONNX Runtime** | 1.5-3x | Framework-agnostic optimization |
| **TensorRT** | 2-6x | NVIDIA GPU specific |
| **Operator fusion** | 1.5-2x | Combine ops into single kernel |
| **Batching** | 2-10x | Increased latency for individual requests |
| **Caching** | 10-100x | Only for repeated/similar inputs |
| **Speculative decoding** | 2-3x | For LLMs (draft model + verify) |

---

## 11. What is CI/CD for ML (MLOps pipeline)?

```
Code Change → Linting/Tests → Data Validation → Training → Evaluation
                                                              ↓
                                                     Meets threshold?
                                                    /              \
                                                  Yes              No
                                                   ↓                ↓
                                          Register Model        Alert team
                                                   ↓
                                          Deploy (canary)
                                                   ↓
                                          Monitor performance
                                                   ↓
                                          Promote or rollback
```

**Tools:**
- CI/CD: GitHub Actions, GitLab CI, Jenkins
- Orchestration: Kubeflow, MLflow Pipelines, Vertex AI Pipelines
- Registry: MLflow Model Registry, Weights & Biases
- Deploy: Seldon Core, KServe, Triton, SageMaker Endpoints

---

## 12. What is BentoML and when would you use it?

BentoML is a framework for **packaging and deploying ML models as production services**.

```python
import bentoml

@bentoml.service(resources={"gpu": 1})
class TextClassifier:
    def __init__(self):
        self.model = bentoml.pytorch.load_model("text_classifier:latest")
    
    @bentoml.api
    def predict(self, text: str) -> dict:
        return self.model.predict(text)
```

**Workflow:** Save model → Build Bento (container) → Deploy (Docker, K8s, cloud)

**When to use:** Quick model-to-API, multi-model serving, when you want batteries-included serving without Triton complexity.
