==============================
FILE: ML Systems Design
==============================

### HIGH PRIORITY

---

Q1. How would you design an end-to-end ML system for a recommendation engine?

A1.
**Problem framing**: Define the objective — maximize click-through rate? Watch time? Purchases? This shapes the entire system.

**Data pipeline**: Collect user events (views, clicks, purchases, dwell time), item metadata, and user profiles. Build a feature store with real-time features (last 5 items viewed) and batch features (user purchase history last 90 days). Store in a feature store like Feast.

**Model architecture**: Two-stage approach. **Candidate generation** (fast, wide net) → retrieve 100-500 candidates from millions using collaborative filtering, embedding similarity (ANN with FAISS), or simple rules. **Ranking** (accurate, narrow) → a gradient boosted tree or neural network scores and ranks the candidates. Features: user-item interaction history, item popularity, contextual signals (time of day, device).

**Serving**: Pre-compute recommendations for active users (batch), with real-time adjustments for in-session behavior. Cache heavily. Latency budget: < 100ms end-to-end.

**Feedback loop**: Log impressions and outcomes. Retrain periodically. Watch for popularity bias (recommending only popular items) and filter bubbles. Add exploration (random items in 5-10% of slots).

---

Q2. What is a feature store? Why is it important in ML systems?

A2.
A feature store is a centralized system for storing, managing, and serving ML features. It's the bridge between data engineering (feature computation) and ML model serving.

**Why it matters**:
- **Consistency**: Training and serving use the same feature definitions. Without it, a feature computed differently during training vs. serving causes training-serving skew — the #1 silent killer of ML systems.
- **Reusability**: Feature "user_avg_purchase_last_30_days" computed once, used by 10 different models. No duplicate logic.
- **Time-travel**: For training, you need features as they were at the time of each training example. The feature store stores feature values with timestamps, enabling point-in-time-correct feature retrieval.
- **Low-latency serving**: Online feature store (Redis, DynamoDB) serves precomputed features for real-time inference.

**Architecture**: Offline store (data warehouse — batch features computed by Spark/dbt), online store (key-value store — low-latency lookups), feature definitions (code that defines how features are computed), and a registry (metadata, versioning, lineage).

Tools: Feast (open-source), Tecton, Hopsworks, SageMaker Feature Store.

---

Q3. What is training-serving skew, and how do you prevent it?

A3.
Training-serving skew is when the data or feature distribution seen during training differs from what the model sees during inference. The model's performance degrades silently — metrics look good offline but poor in production.

**Common causes**:
- **Feature computation differences**: Training computes `avg_price` using a Spark job; serving computes it with a different SQL query that handles NULLs differently.
- **Data leakage**: Training accidentally uses future information (target leakage) or information not available at prediction time.
- **Preprocessing differences**: Tokenizer version differs between training and serving. Normalization parameters (mean, std) differ.
- **Data distribution shift**: The real-world distribution changes over time (concept drift).

**Prevention**:
- Use a feature store — single source of truth for feature computation.
- Share preprocessing code between training and serving (use the same model pipeline).
- Log serving predictions and features, then compare distributions against training data.
- Monitor input feature distributions and model outputs continuously.

---

Q4. How do you monitor a deployed ML model? What metrics and signals do you track?

A4.
ML monitoring goes beyond application monitoring — you're tracking model behavior, not just server health.

**What to monitor**:
- **Prediction distribution**: Is the model's output distribution shifting? If a fraud model suddenly predicts 0.01 for everything, something is wrong even if latency is fine.
- **Input feature distributions**: Feature values drifting from training distribution. Statistical tests (KS test, PSI — Population Stability Index) detect this.
- **Business metrics**: Click-through rate, conversion rate, revenue. The ultimate ground truth.
- **Data quality**: Missing values spiking, new categories appearing, schema changes in upstream data.
- **Latency and throughput**: Model serving performance — p50, p95, p99 latency.
- **Ground truth feedback loop**: When labels eventually arrive (was the prediction correct?), compute online accuracy/precision/recall.

**Alerting**: Set up alerts for: feature distribution drift beyond threshold, prediction distribution shift, data quality failures, latency breaches, and significant drops in business metrics.

Tools: Evidently AI, WhyLabs, Arize, custom dashboards with Grafana.

---

Q5. How do you handle model retraining? What triggers a retrain?

A5.
**Triggers**:
- **Scheduled**: Retrain daily/weekly/monthly regardless. Simple, predictable. Works when data distribution is stable.
- **Performance-based**: Monitor online metrics; retrain when accuracy drops below a threshold. Requires labeled data with low latency.
- **Drift-based**: Monitor input feature distributions; retrain when drift is detected (PSI > threshold). Doesn't need labels — catches distribution changes early.
- **Event-based**: A major change happens — new product category launched, holiday season, pandemic. Trigger an ad-hoc retrain.

**Retraining strategy**:
- **Full retrain**: Train on all historical data. Most accurate but expensive.
- **Incremental/fine-tune**: Update model with only new data. Faster but risks catastrophic forgetting (losing old patterns).
- **Sliding window**: Train on the last N months of data. Balances recency and history.

**Best practice**: Automate retraining pipelines. Validate the new model on a holdout/shadow set before promoting. A/B test the new model against the current one. Never auto-deploy without validation — a retrained model can be worse than the current one.

---

Q6. What is A/B testing in ML? How do you determine if a new model is better?

A6.
A/B testing compares the current model (control) against a new model (treatment) by splitting live traffic randomly.

**Setup**: Route 90% of traffic to the current model, 10% to the new model. Ensure users are consistently assigned to one group (hash user ID).

**Metrics**: Define a primary metric (conversion rate, revenue per user) and guardrail metrics (latency, user complaints, error rate). The new model must improve the primary metric without degrading guardrails.

**Statistical rigor**: Run the test long enough to reach statistical significance (typically p < 0.05). Calculate sample size needed upfront based on the minimum detectable effect (MDE) and baseline metric. Avoid peeking at results and stopping early — it inflates false positive rates.

**Alternatives when A/B isn't feasible**: Interleaving (mix recommendations from both models in one list — users implicitly compare), shadow mode (run the new model alongside without serving its results — compare offline), bandit methods (dynamically allocate more traffic to the better-performing variant).

---

Q7. What is an ML pipeline, and what are the key stages?

A7.
An ML pipeline automates the end-to-end workflow from raw data to deployed model.

**Key stages**:
1. **Data ingestion**: Pull data from sources — databases, APIs, event streams.
2. **Data validation**: Check schema, completeness, distributions. Reject bad batches early.
3. **Feature engineering**: Compute features — joins, aggregations, encodings. Store in feature store.
4. **Data splitting**: Train/validation/test split. Stratified for imbalanced data. Time-based for temporal data.
5. **Model training**: Train model(s) with hyperparameter tuning. Track experiments (MLflow).
6. **Model evaluation**: Evaluate on holdout set. Compare against baseline and current production model.
7. **Model validation**: Automated checks — does it pass fairness tests? Is performance above threshold? Is serving latency acceptable?
8. **Deployment**: Register model, deploy to serving infrastructure (Kubernetes, SageMaker endpoint).
9. **Monitoring**: Track predictions, feature drift, performance degradation.

**Orchestration**: Airflow, Kubeflow Pipelines, or Vertex AI Pipelines orchestrate these stages. Each stage should be idempotent and independently retriggerable.

---

### MEDIUM PRIORITY

---

Q8. What is shadow deployment, and how does it de-risk model launches?

A8.
Shadow deployment runs the new model on live traffic in parallel with the current model, but only the current model's predictions are served to users. The new model's predictions are logged for comparison.

**Benefits**: You see exactly how the new model would perform on real data — not just holdout data. You can compare prediction distributions, latency, error rates, and (once labels arrive) accuracy. Zero risk to users.

**When to use**: Before A/B testing — shadow first to catch obvious issues (crashes, latency spikes, bizarre predictions), then A/B test for business metric comparison.

**Limitations**: You can't measure the new model's impact on user behavior (if recommendations change, user behavior changes — shadow mode can't capture this). Resource cost — you're running two models simultaneously.

**Canary deployments** (related): Route 1-5% of live traffic to the new model and actually serve its predictions. Riskier than shadow but captures user behavior impact. Roll back immediately if metrics degrade.

---

Q9. How do you handle data drift and concept drift?

A9.
**Data drift** (covariate shift): The distribution of input features changes. Users' age distribution shifts, product prices change, new categories appear. The model was trained on different feature distributions.

**Concept drift**: The relationship between inputs and outputs changes. What predicted churn last year doesn't predict churn now — customer behavior has fundamentally changed.

**Detection**:
- Statistical tests on feature distributions: KS test, PSI (Population Stability Index), chi-squared test.
- Monitor model performance over time — declining accuracy signals drift.
- Evidently AI, NannyML, or custom monitoring dashboards.

**Response**:
- **Retrain**: Most common — retrain on recent data. Scheduled or triggered by drift detection.
- **Online learning**: Update model incrementally as new data arrives.
- **Ensemble with recency weighting**: Give more weight to models trained on recent data.
- **Feature engineering**: Design features that are less sensitive to drift (ratios instead of absolute values).

---

Q10. What are the considerations for model serving at scale (batch vs. real-time)?

A10.
**Batch inference**: Pre-compute predictions for all users/items periodically (hourly, daily). Store in a database or cache. Serving is a simple key-value lookup — fast and cheap.

Best for: Recommendations where predictions don't need to be real-time, scheduled reports, predictions for a known set of entities.

**Real-time inference**: Model runs on each request. Input features assembled at request time, prediction returned immediately.

Best for: Fraud detection (need to block the transaction NOW), search ranking, dynamic pricing, chatbots.

**Considerations**:
- **Latency**: Real-time models must be fast. Complex models may need distillation (train a simpler model to mimic the complex one), ONNX optimization, or GPU serving.
- **Throughput**: Can your serving infrastructure handle peak load? Auto-scaling with Kubernetes, or serverless (AWS Lambda, Cloud Functions for low-QPS models).
- **Feature availability**: Real-time features (user's last click) require a real-time feature pipeline. Batch features are easier.
- **Cost**: GPU serving is expensive. Batch inference is cheaper per prediction.

Most systems use a hybrid: batch for bulk predictions, real-time for latency-sensitive decisions.

---

Q11. What is model versioning, and why does it matter?

A11.
Model versioning tracks every model artifact (weights, hyperparameters, training data version, code version) with a unique identifier. It's version control for models.

**Why it matters**:
- **Reproducibility**: "What model was serving traffic on March 15th?" You can answer this and reproduce that exact model.
- **Rollback**: New model performs worse → instantly rollback to the previous version. Without versioning, you can't.
- **Comparison**: Compare performance across versions. "Did v3 actually improve over v2?"
- **Auditability**: Regulatory requirements may demand knowing which model made which prediction.

**What to version**: Model weights, hyperparameters, training code (git hash), training data (data version or hash), feature definitions, preprocessing pipeline, evaluation metrics.

**Tools**: MLflow Model Registry (standard), DVC (data version control), Weights & Biases, SageMaker Model Registry. Store artifacts in S3/GCS/Azure Blob with metadata in the registry.

---

Q12. What is model explainability? Why is it important, and what tools exist?

A12.
Model explainability tells you why a model made a specific prediction — which features mattered and how they influenced the output.

**Why it matters**:
- **Trust**: Stakeholders and users need to understand why a loan was denied or why a patient was flagged high-risk.
- **Debugging**: If the model uses "zip code" as the top feature for credit scoring, that might be a proxy for race — explainability catches this.
- **Regulatory compliance**: GDPR's "right to explanation" — some regulations require explaining automated decisions.

**Techniques**:
- **SHAP** (SHapley Additive exPlanations): Game-theory-based. Computes each feature's contribution to each prediction. Gold standard for tabular models.
- **LIME** (Local Interpretable Model-agnostic Explanations): Approximates the model locally with a simple, interpretable model. Works for any model type including images and text.
- **Feature importance**: Built into tree models — how much each feature reduces impurity across all trees. Quick but global, not per-prediction.
- **Partial Dependence Plots**: Show the relationship between a feature and the prediction, marginalizing over other features.

---

Q13. How do you detect and handle label leakage?

A13.
Label leakage occurs when information about the target variable "leaks" into the training features — giving the model an unfair advantage during training that doesn't exist at prediction time.

**Examples**:
- Using "treatment_outcome" to predict "diagnosis." The outcome happens after the diagnosis — it's not available at prediction time.
- Using "account_closure_date" to predict churn. If it's NULL for active users, the model learns NULL = not churned — trivially.
- Target encoding without proper cross-validation — the category's encoding contains information from the very samples being predicted.

**Detection**:
- Suspiciously high accuracy, especially on simple models.
- Feature importance shows unexpected features as top predictors.
- Performance drops dramatically in production vs. offline evaluation.

**Prevention**:
- Carefully audit each feature: "Would I have this information at the time I need to make the prediction?"
- Time-based splits for temporal data — never train on future data.
- Use proper cross-validation for target encoding and feature engineering.

---

Q14. What is the cold start problem in recommendation systems?

A14.
The cold start problem occurs when the system can't make good recommendations due to insufficient data — for new users (no interaction history) or new items (not seen by any users).

**New user solutions**:
- Ask for preferences during onboarding (explicit signals).
- Use demographic-based or context-based recommendations initially.
- Show popular/trending items until you have enough interaction data.
- Transfer learning from similar platforms.

**New item solutions**:
- Content-based recommendations using item metadata (title, category, description, images).
- Featured/editorial placements to gather initial interactions.
- Bandits — explore new items by showing them to a fraction of users and observing engagement.

**Hybrid approach**: Combine collaborative filtering (user-item interactions) with content-based filtering (item features). For new items with no interactions, content-based features still work. For new users, item popularity and content relevance can fill the gap.

---

### LOW PRIORITY

---

Q15. What is model distillation (knowledge distillation)?

A15.
Knowledge distillation trains a smaller "student" model to mimic a larger "teacher" model. The student learns not from the original labels but from the teacher's soft probability outputs (logits).

**Why soft outputs matter**: A teacher predicting "cat: 0.8, dog: 0.15, car: 0.05" contains more information than the hard label "cat." The student learns that cats and dogs are more similar than cats and cars — this "dark knowledge" improves generalization.

**Use cases**: Deploy a lightweight model in production (smaller, faster) that approaches the teacher's accuracy. Mobile deployment — a 500M parameter model distilled to 10M parameters. Serving at scale — replace a slow ensemble with a fast single model.

**Process**: Train the teacher normally. Then train the student on a mix of the original hard labels and the teacher's soft predictions (temperature-scaled softmax). The temperature parameter controls how "soft" the teacher's distribution is — higher temperature = softer = more information shared.

---

Q16. What is fairness in ML? What are common fairness metrics?

A16.
Fairness ensures the model doesn't discriminate against protected groups (race, gender, age) — either directly or through proxy features.

**Common metrics**:
- **Demographic parity**: Equal positive prediction rates across groups. "The model approves loans at the same rate for all races." Problem: ignores actual qualification differences.
- **Equalized odds**: Equal true positive and false positive rates across groups. "Among qualified applicants, approval rates are equal. Among unqualified applicants, rejection rates are equal."
- **Predictive parity**: Among those predicted positive, equal precision across groups. "Of those approved, the default rate is the same across groups."

**The impossibility theorem**: You can't satisfy all fairness criteria simultaneously (except in trivial cases). You must choose which definition of fairness matters most for your context.

**Mitigation techniques**: Pre-processing (reweight or resample data), in-processing (add fairness constraints to the loss function), post-processing (adjust thresholds per group). Tools: Fairlearn (Microsoft), AIF360 (IBM).

---

Q17. How do you handle serving ML models on edge devices?

A17.
Edge deployment means running models on devices with limited compute, memory, and power — phones, IoT sensors, embedded systems.

**Model optimization**:
- **Quantization**: Convert 32-bit floating point weights to 8-bit integers. 4x smaller model, faster inference, slight accuracy loss. Post-training quantization is quick; quantization-aware training is more accurate.
- **Pruning**: Remove weights or neurons that contribute least. Structured pruning (remove entire filters/layers) is better for hardware efficiency.
- **Distillation**: Train a small student model from the large teacher.
- **Architecture design**: MobileNet, EfficientNet — designed from scratch for mobile/edge. Depthwise separable convolutions reduce compute significantly.

**Frameworks**: TensorFlow Lite (mobile), ONNX Runtime (cross-platform), Core ML (Apple), TensorRT (NVIDIA — server-side GPU optimization).

**Trade-offs**: Accuracy vs. latency vs. model size. For real-time on-device inference (face detection, voice commands), latency matters most. For periodic tasks, you have more budget.

---

Q18. What is data augmentation, and when is it useful?

A18.
Data augmentation creates modified versions of existing training samples to increase dataset size and diversity, improving generalization.

**Image augmentation**: Rotation, flipping, cropping, brightness/contrast changes, color jitter, random erasing. Advanced: Mixup (blend two images and their labels), CutMix (paste a patch from one image onto another).

**Text augmentation**: Synonym replacement, random insertion/deletion, back-translation (English → French → English), paraphrasing with LLMs.

**Tabular data**: SMOTE for oversampling minority classes, noise injection, feature permutation.

**When useful**: Small datasets — augmentation is a form of regularization, exposing the model to more variation. Domain-specific invariances — a cat is still a cat when flipped horizontally.

**When to be careful**: Augmentations must be semantically valid. Flipping a "6" makes it look like a "9" — harmful for digit recognition. Rotating satellite images is fine; rotating medical images where orientation matters is not.

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q19. How do you handle real-time features vs pre-computed features in ML systems?

A19.
**Pre-computed (batch) features**: Calculated offline, stored in a feature store, looked up at inference time. Examples: user's average purchase amount (last 30 days), product popularity score, user embeddings. Updated on a schedule (hourly/daily).

**Real-time features**: Computed at request time from the live event stream. Examples: number of transactions in the last 5 minutes (fraud detection), current session click count, time since last action.

**Architecture**:
- **Batch features**: Spark/Flink batch job → Feature Store (Feast, Tecton, Vertex AI) → Online serving (Redis/DynamoDB for low-latency lookups)
- **Real-time features**: Event stream (Kafka) → Stream processor (Flink/Spark Streaming) → Aggregation → Feature Store or directly to model

**The challenge — training/serving skew**: If you compute features differently during training vs serving, the model sees different inputs and performs poorly. Solutions:
- Use the same feature computation code for training and serving (Feast's feature definitions)
- Log features at serving time and train on those logged features
- Feature stores with both offline (training) and online (serving) access to the same definitions

**Decision framework**: If the feature changes frequently and its freshness affects predictions (fraud signals, real-time bidding), make it real-time. If it changes slowly (user demographics, product categories), batch is simpler and cheaper.

---

Q20. What is embedding-based retrieval and the two-stage ranking pattern?

A20.
For large-scale recommendation and search, directly scoring all items with a complex model is infeasible — you can't run a BERT-based model on 100 million products. So you use two stages:

**Stage 1 — Retrieval (candidate generation)**: Use lightweight models to quickly narrow from millions to hundreds of candidates.
- **Embedding-based**: Encode users and items as vectors. Find nearest neighbors using ANN (HNSW, FAISS). Captures semantic similarity.
- **Collaborative filtering**: Users who liked similar items
- **Rules/heuristics**: Geographic filtering, availability checks
- Multiple retrieval sources → merge candidates (union of top-k from each)

**Stage 2 — Ranking**: Score the ~100-1000 candidates with a more complex model (gradient-boosted trees, deep ranking network). Consider user features, item features, cross-features, and context. Optimize for the business metric (purchase, click, engagement time).

**Optional Stage 3 — Re-ranking**: Business rules, diversity injection, and policy enforcement. Remove duplicate items, ensure fair exposure, apply freshness boosting.

**Why it works**: Retrieval is O(1) per query (ANN index lookup) regardless of catalog size. Ranking is O(k) for k candidates. The heavy model only sees a small fraction of items.

**Example — YouTube**: Candidate generation retrieves ~1000 videos from billions → ranking model scores and orders them → post-processing filters for diversity and freshness.

---

Q21. How do you design model fallback and graceful degradation in production?

A21.
In production, ML models fail — timeouts, bad inputs, model server crashes, corrupted model artifacts. Graceful degradation means the system still works, just with reduced quality.

**Fallback hierarchy**:
1. **Primary model** (complex, ML-based): Deep learning model, most accurate
2. **Secondary model** (simpler): Gradient-boosted tree, faster, still decent
3. **Heuristic fallback**: Hand-coded business rules. "Show most popular items" instead of personalized recommendations
4. **Default response**: Static fallback. "Show trending items" or "return average prediction"

**When to trigger fallback**:
- Model latency exceeds SLA (p99 > 200ms → fall back to simpler model)
- Model confidence below threshold (softmax prob < 0.3 → use heuristic)
- Model server unreachable (circuit breaker triggers)
- Input validation failure (missing features)

**Implementation patterns**:
- **Circuit breaker**: If model fails N times in M seconds, stop calling it for a cooldown period. Try again after timeout.
- **Timeout with fallback**: Set aggressive timeouts on model calls. If exceeded, return cached/default result.
- **Shadow scoring**: Run fallback model alongside primary; results are pre-computed and ready to serve.
- **Feature store defaults**: If a feature is missing, use a sensible default (population mean) rather than failing.

**Key principle**: Never let a model failure become a user-visible error. A slightly worse recommendation is infinitely better than a 500 error.

---

### IMPORTANT

---

Q22. What is online/continual learning, and when would you use it?

A22.
Online learning updates the model incrementally as new data arrives, rather than retraining from scratch on the full dataset.

**Batch retraining**: Collect data → retrain entire model periodically (daily/weekly) → redeploy. Simple but introduces latency — the model always lags behind recent trends.

**Online learning**: Model updates with each new example or small batch. The model adapts immediately to new patterns.

**When to use online learning**:
- **Fast-changing distributions**: Ad click prediction (new ads constantly), recommendations (trending content), fraud detection (evolving attack patterns)
- **Large-scale data**: Retraining on billions of examples periodically is expensive
- **Real-time adaptation**: Stock prices, demand forecasting

**Challenges**:
- **Catastrophic forgetting**: Model forgets old knowledge when learning new examples. Mitigated by replay buffers (periodically mix in old data).
- **Concept drift detection**: Need to monitor if the distribution is actually changing or if you're just chasing noise.
- **Feature engineering**: Online features must be computable incrementally
- **Model stability**: Online updates can cause performance oscillations. Use slower learning rates and limit update magnitude.

**In practice**: Many production systems use a hybrid — periodic full retraining (weekly) for stability + online updates (hourly/per-request) for freshness. The online model is validated against the batch model; if it degrades, fall back to the batch version.

---

Q23. How would you design a fraud detection system?

A23.
Fraud detection is a classic ML systems design question that covers real-time features, class imbalance, and high-stakes decision-making.

**Architecture**:
1. **Data ingestion**: Transaction events stream through Kafka
2. **Feature computation**: Mix of batch (user history, average transaction amount, account age) and real-time (transactions in last 5 min, velocity of card usage, geographic distance from last transaction)
3. **Model**: Usually gradient-boosted trees (XGBoost/LightGBM) — interpretable, fast, handle tabular data well. Deep learning for sequence patterns (RNN over transaction history).
4. **Scoring**: Sub-100ms inference. Return risk score (0-1).
5. **Decision engine**: Rules on top of the score. Score > 0.9 → block, 0.5-0.9 → 3D Secure challenge, < 0.5 → approve. Rules are tunable without model retraining.

**Key challenges**:
- **Class imbalance**: 0.1% fraud rate. Use precision-focused metrics (precision@k, PR-AUC, not accuracy). Oversampling (SMOTE) or focal loss.
- **Label delay**: You only know if a transaction was fraud days/weeks later (chargebacks). Train on delayed labels, but serve in real-time.
- **Adversarial adaptation**: Fraudsters change tactics. Model needs frequent retraining + rule updates.
- **False positive cost**: Blocking legitimate transactions loses revenue and annoys customers. Optimize for high precision—minimize false positives while catching the most fraud.
- **Explainability**: Regulators require explanations for declined transactions. SHAP values or rule-based explanations alongside ML scores.

---

Q24. How do you orchestrate multiple models in a single prediction pipeline?

A24.
Many production systems use multiple models that work together:

**Sequential pipeline** (cascading):
- Content moderation: Fast text classifier → if borderline → expensive image model → if still uncertain → human review
- Each stage is a gate — only uncertain cases proceed to the next (more expensive) model

**Parallel ensemble**:
- Run multiple models simultaneously, combine predictions (averaging, voting, stacking)
- Use for critical decisions where redundancy improves reliability
- Different model architectures catch different patterns

**Router/mixture-of-experts**:
- A lightweight router model directs each request to the appropriate specialist model
- E.g., a language detector routes to language-specific models
- Reduces compute by using expensive models only when needed

**Two-tower / multi-stage**:
- Recommendation: User tower + Item tower (embedding models) → retrieval → ranking model → business rules
- Each stage has different latency budgets and model complexity

**Orchestration challenges**:
- **Latency management**: Parallel calls with timeouts. If model B hasn't responded in 50ms, proceed without it.
- **Error handling**: One model failing shouldn't crash the pipeline. Use fallbacks per model.
- **Feature sharing**: Multiple models often need the same features. Compute once, share across models.
- **A/B testing**: When testing a new version of one model in the pipeline, hold others constant.

Tools: KServe (ML serving), Seldon Core, or custom orchestration behind an API gateway.

---

### GOOD-TO-HAVE

---

Q25. How do feature engineering pipelines differ from feature serving pipelines?

A25.
**Feature engineering pipeline** (offline/batch): Transform raw data into ML-ready features for training. Runs on historical data. Typically Spark, dbt, or Python batch jobs. Output: feature tables or Parquet files.

- Focus: Correctness, expressiveness, experimentation speed
- Scale: Process months/years of data
- Latency: Hours (batch jobs), minutes (incremental)
- Code: SQL, PySpark, Pandas

**Feature serving pipeline** (online): Compute or look up features at inference time with low latency. Must return features in milliseconds.

- Focus: Latency, availability, consistency with training features
- Scale: Per-request
- Latency: Single-digit milliseconds
- Code: Key-value lookups, stream processing

**The bridge — Feature Store**: Stores feature definitions once, computes them for both offline (training) and online (serving). Feast, Tecton, and Hopsworks provide this dual-compute capability.

**Common pattern**:
1. Define feature in a shared DSL/config
2. Batch pipeline computes features on historical data → training dataset
3. Same logic runs in streaming or on a schedule → materializes to an online store (Redis/DynamoDB)
4. At serving time, feature server looks up pre-computed features and computes real-time features → feeds to model

**The critical invariant**: Training and serving features MUST be computed the same way. If you use Pandas for training and Java for serving, subtle bugs (different timestamp handling, rounding, null handling) cause training-serving skew — the silent killer of ML systems.