==============================
FILE: MLflow & MLOps Pipelines
==============================

### HIGH PRIORITY

---

Q1. What is MLflow, and what are its four main components?

A1.
MLflow is an open-source platform for managing the ML lifecycle — from experimentation to deployment.

**Four components**:
1. **MLflow Tracking**: Logs experiments — parameters, metrics, artifacts (model files, plots), code version. "I ran logistic regression with C=0.1, got AUC=0.87." Each run is recorded, comparable, and reproducible.

2. **MLflow Projects**: Packages ML code in a reusable, reproducible format. A project is a directory with a `MLproject` file specifying the entry point, parameters, and environment (conda, Docker). Anyone can run `mlflow run git://repo` and get the same result.

3. **MLflow Models**: A standard format for packaging models from any framework (sklearn, PyTorch, TensorFlow). A model includes the artifact and a `MLmodel` file specifying flavors (ways to serve). Deploy to REST API, batch inference, or edge.

4. **MLflow Model Registry**: A centralized store for model versioning and lifecycle management. Models go through stages: None → Staging → Production → Archived. Supports model approval workflows and lineage tracking.

---

Q2. How does MLflow Tracking work? What do you log and why?

A2.
MLflow Tracking records every experiment run with:

**Parameters**: Hyperparameters — learning_rate, max_depth, batch_size. What configuration produced this result?

**Metrics**: Performance numbers — accuracy, AUC, loss, training time. Loggable as single values or over steps (loss per epoch).

**Artifacts**: Output files — trained model, confusion matrix plots, feature importance charts, data samples.

**Tags and metadata**: Run name, data version, git hash, team member.

```python
with mlflow.start_run():
    mlflow.log_param("learning_rate", 0.01)
    mlflow.log_param("max_depth", 5)
    mlflow.log_metric("auc", 0.92)
    mlflow.log_metric("f1", 0.88)
    mlflow.sklearn.log_model(model, "model")
```

**Why it matters**: Without tracking, you forget what you tried, can't reproduce results, and waste time repeating experiments. Two months later: "Which model had 0.93 AUC? What parameters? What data version?" MLflow answers all of these instantly.

**Backend**: Tracking server stores data in a database (SQLite for local, PostgreSQL for teams). Artifacts go to a store (local filesystem, S3, GCS).

---

Q3. What is the MLflow Model Registry, and how does it support model lifecycle management?

A3.
The Model Registry is a centralized hub where models are registered, versioned, and transitioned through lifecycle stages.

**Workflow**:
1. Train a model, log it to MLflow Tracking.
2. Register it: `mlflow.register_model(model_uri, "FraudDetector")`. This creates version 1.
3. Retrain with better data → register as version 2.
4. Transition version 2 to "Staging" → test it.
5. After validation, transition to "Production" → serving system picks it up.
6. Version 1 transitions to "Archived."

**Key features**: Version history, stage transitions (None/Staging/Production/Archived), annotations/descriptions per version, lineage tracking (which run produced this model), webhooks for automation (trigger CI/CD on stage transition).

**Team workflow**: A data scientist registers a model. An ML engineer reviews it in staging — checks performance metrics, runs integration tests, validates latency. If approved, transitions to production. If not, leaves notes on what needs to change.

---

Q4. What is MLOps? How does it differ from DevOps?

A4.
MLOps applies DevOps principles (CI/CD, automation, monitoring) to ML systems. But ML adds complexity that pure DevOps doesn't handle:

**DevOps**: Code changes → test → deploy → monitor.
**MLOps**: Code changes + data changes + model changes → test (code tests + model validation + data validation) → deploy (model + feature pipeline + serving infra) → monitor (system health + model performance + data drift).

**Key differences**:
- **Data is a first-class citizen**: Code doesn't change, but data does — and that changes model behavior. You need data versioning and validation.
- **Continuous training**: Models degrade over time even without code changes. You need automated retraining triggers.
- **Experiment tracking**: DevOps doesn't track "which hyperparameters produced which results." MLOps does.
- **Model validation**: Beyond unit tests — does the model meet accuracy thresholds? Is it fair? Is it faster than the current production model?

**Maturity levels**: Level 0 (manual — Jupyter notebook), Level 1 (automated training pipeline), Level 2 (automated CI/CD/CT — continuous training with automated validation and deployment).

---

Q5. What are CI/CD/CT pipelines in the context of MLOps?

A5.
**CI (Continuous Integration)**: Automatically test code changes — unit tests for feature engineering functions, integration tests for the pipeline, data validation tests. Triggered on every PR/commit.

**CD (Continuous Delivery/Deployment)**: Automatically deploy validated models. When a model is promoted to "Production" in the registry, CD deploys it to the serving infrastructure — Kubernetes, SageMaker endpoint, etc.

**CT (Continuous Training)**: Unique to MLOps — automatically retrain models when triggered (scheduled, drift-detected, new data arrives). The CT pipeline retrains → evaluates → if better than current production model → registers new version → triggers CD.

**How they connect**:
- Data scientist pushes code → CI runs tests → if passing, pipeline runs.
- Pipeline trains model → CT evaluates → if passing, registers to Model Registry.
- Model promoted to Production → CD deploys to serving.

This automation is what separates "ML in Jupyter notebooks" from "ML in production." Most organizations are between Level 0 and Level 1.

---

Q6. How do you version datasets in MLOps? Why is data versioning critical?

A6.
Data versioning tracks which data was used to train each model version. Without it, you can't reproduce models or debug production issues.

**Why it's critical**: The same code with different data produces different models. "Why did model v3 perform differently from v2?" Maybe the data changed — a new data source was added, a filtering bug was introduced, or the distribution shifted.

**Approaches**:
- **DVC (Data Version Control)**: Tracks large files alongside Git. Stores metadata (hash, pointer) in Git, actual data in S3/GCS. `dvc push` / `dvc pull` syncs data.
- **Delta Lake / Iceberg**: Table formats with time travel — query data as it existed at a specific timestamp or version.
- **Snapshot + hash**: Store snapshots in versioned S3 paths (`s3://data/v1/`, `v2/`). Log the path/hash in MLflow as a parameter.
- **Feature store versioning**: Feature stores like Feast version feature definitions and enable point-in-time retrieval.

**Best practice**: Every MLflow run should log the data version (hash, path, or DVC commit). This makes any model reproducible.

---

Q7. How do you set up experiment tracking for a team of data scientists?

A7.
**Central MLflow Tracking Server**: Deploy a shared MLflow tracking server backed by a PostgreSQL database (for run metadata) and S3/GCS (for artifacts). All team members point their MLflow client to the same server.

**Organization**:
- **Experiments**: Group runs by project or model type. "fraud-detection-v2", "churn-prediction". Use `mlflow.set_experiment("fraud-detection")`.
- **Run naming**: Consistent naming conventions — include the model type and key parameter values.
- **Tags**: Tag runs with `team`, `data_version`, `purpose` (baseline, experiment, production-candidate).

**Collaboration workflow**: Scientists run experiments, log results. Weekly review of the tracking UI — compare metrics across runs. Identify the best-performing configuration. Promote to Model Registry for engineering review.

**Access control**: MLflow community edition lacks fine-grained permissions. Managed platforms (Databricks, AWS SageMaker) add team-level access control. For open-source, wrap with a reverse proxy and authentication.

---

### MEDIUM PRIORITY

---

Q8. What is model serving with MLflow? What deployment options does it support?

A8.
MLflow packages models in a standard format that can be deployed multiple ways:

**Local REST API**: `mlflow models serve -m runs:/<run_id>/model -p 5001`. Creates a REST endpoint for predictions. Good for testing.

**Docker container**: `mlflow models build-docker` creates a Docker image with the model and a REST API. Deploy to Kubernetes, ECS, or any container platform.

**Cloud endpoints**: Deploy directly to Azure ML, AWS SageMaker, or Databricks serving. MLflow handles the packaging; the cloud handles scaling and infrastructure.

**Batch inference**: `mlflow.pyfunc.spark_udf()` wraps the model as a Spark UDF for distributed batch scoring on millions of records.

**Model flavors**: MLflow stores models with flavors — `sklearn`, `pytorch`, `tensorflow`, `pyfunc` (generic Python function). The serving framework picks the appropriate flavor. A `pyfunc` wrapper lets you serve any custom Python model through the same API.

---

Q9. What are the differences between MLflow, Kubeflow, and Weights & Biases?

A9.
**MLflow**: Experiment tracking, model registry, model packaging. Lightweight, framework-agnostic, self-hosted or managed. Best for: teams that need tracking and registry without heavy infrastructure.

**Kubeflow**: End-to-end ML platform on Kubernetes. Pipeline orchestration (Kubeflow Pipelines), training (TFJob, PyTorchJob), serving (KFServing/KServe), AutoML (Katib). Best for: organizations already on Kubernetes that need a full ML platform.

**Weights & Biases (W&B)**: Experiment tracking (superior visualization), hyperparameter sweeps, model registry, dataset versioning. SaaS-first. Best for: research teams and deep learning projects where visualization and collaboration matter.

**Overlap and complementarity**: Many teams use MLflow for the model registry + W&B for experiment tracking. Or MLflow for tracking + Kubeflow Pipelines for orchestration. They're not fully competing — they complement each other depending on your needs.

---

Q10. What is a model artifact? What should be included when saving a model?

A10.
A model artifact is the complete package needed to load and run a model without the training code.

**What to include**:
- **Model weights/parameters**: The learned values — pickle file, HDF5, ONNX, SavedModel.
- **Preprocessing pipeline**: Scalers, encoders, tokenizers — anything applied to input before the model. If you StandardScaled features during training, the mean/std values must be saved.
- **Model configuration**: Architecture definition, hyperparameters.
- **Dependencies**: Python version, library versions (`requirements.txt` or `conda.yaml`). A model trained with scikit-learn 1.2 may not load in 1.4.
- **Signature**: Input/output schema — what features does the model expect, what does it output? MLflow infers this automatically.

**Common mistakes**: Not saving the preprocessor (features are scaled differently at serving time), not pinning dependency versions (model fails to load months later), not including the tokenizer for NLP models.

---

Q11. How do you implement automated model validation before deployment?

A11.
Automated validation sits between training and deployment — a quality gate that prevents bad models from reaching production.

**Checks to automate**:
1. **Performance threshold**: AUC > 0.85 on holdout test set. If below, block deployment.
2. **Performance comparison**: New model must be ≥ current production model on the same test set. Prevents regressions.
3. **Latency test**: Run inference on sample data, verify p95 latency < 100ms. A model that's accurate but slow is useless for real-time serving.
4. **Input/output validation**: Does the model accept the expected input schema? Does it return the expected output format?
5. **Fairness checks**: Run Fairlearn/AIF360 analysis across protected groups. Flag if demographic parity or equalized odds are violated beyond threshold.
6. **Data slice analysis**: Does the model perform well across all important segments (geographies, user types)? A great overall AUC might hide poor performance on a critical sub-segment.

**Implementation**: Python scripts triggered by CI/CD. On PR to model registry → run validation pipeline → if all checks pass → approve for staging → if staging metrics look good → promote to production.

---

Q12. What is a model's serving signature in MLflow?

A12.
The signature defines the model's expected input and output schema — column names, types, and shapes.

```python
from mlflow.models import infer_signature

signature = infer_signature(X_train, model.predict(X_train))
mlflow.sklearn.log_model(model, "model", signature=signature)
```

**Stored in the MLmodel file**:
```yaml
signature:
  inputs: '[{"name": "age", "type": "long"}, {"name": "income", "type": "double"}]'
  outputs: '[{"name": "prediction", "type": "long"}]'
```

**Why it matters**: When the model is served, MLflow validates incoming requests against the signature. Wrong column names, missing features, or wrong data types are caught at the serving layer instead of producing silent errors or crashes.

Without a signature, the model accepts any input and fails unpredictably when the input doesn't match expectations. Always log signatures — it takes one line and prevents hours of debugging.

---

### LOW PRIORITY

---

Q13. How does MLflow support multi-framework model comparison?

A13.
MLflow's tracking API is framework-agnostic. You can log runs from scikit-learn, PyTorch, TensorFlow, XGBoost, LightGBM — all to the same experiment.

```python
# Experiment: "churn-prediction"
# Run 1: sklearn LogisticRegression
mlflow.sklearn.log_model(lr_model, "model")
mlflow.log_metric("auc", 0.82)

# Run 2: XGBoost
mlflow.xgboost.log_model(xgb_model, "model")
mlflow.log_metric("auc", 0.89)

# Run 3: PyTorch
mlflow.pytorch.log_model(nn_model, "model")
mlflow.log_metric("auc", 0.91)
```

All three runs appear in the same experiment dashboard. You compare AUC, training time, model size side by side. The model with the best trade-off of accuracy, complexity, and latency can be registered.

The pyfunc flavor provides a common interface — `model.predict(input)` — regardless of the underlying framework. This means your serving infrastructure doesn't care whether the model is sklearn or PyTorch.

---

Q14. What is a feature pipeline, and how does it connect to MLOps?

A14.
A feature pipeline is the automated process that computes and stores features for model training and serving. It's the "data" side of MLOps.

**Batch feature pipeline**: Runs on a schedule (hourly, daily). Spark/dbt job computes aggregate features (user_avg_purchase_last_30_days) and writes to the offline feature store (data warehouse).

**Streaming feature pipeline**: Processes events in real-time (Kafka/Flink). Computes real-time features (user_clicks_last_5_minutes) and writes to the online feature store (Redis).

**Connection to MLOps**: The feature pipeline is a dependency of the training pipeline. Changes to feature computation can change model behavior even without code changes. Feature pipelines need their own CI/CD — unit tests for transformation logic, data validation, and monitoring.

**Feature pipeline + model pipeline = ML pipeline**. Most production ML failures trace back to feature pipeline issues, not model code.

---

Q15. What are MLOps maturity levels?

A15.
Google's MLOps maturity model defines three levels:

**Level 0 — Manual**: Data scientists work in Jupyter notebooks. Manual training, manual deployment (export model, hand it to engineering). No monitoring, no retraining pipeline. Most organizations start here.

**Level 1 — ML Pipeline Automation**: Training pipeline is automated — data ingestion, feature engineering, training, evaluation run automatically. Continuous training is triggered by schedule or events. Feature store exists. But deployment is still manual or semi-automated.

**Level 2 — CI/CD/CT Pipeline Automation**: Full automation. Code changes trigger CI (tests). Validated models are automatically deployed (CD). Models are automatically retrained (CT). Data and model monitoring trigger alerts and retraining. Feature pipelines and model pipelines are orchestrated end-to-end.

**Reality**: Most companies are at Level 0-1. Level 2 requires significant infrastructure investment and is justified for systems where model freshness directly impacts revenue (recommendations, ads, pricing).

---

Q16. What is experiment reproducibility, and how does MLflow enable it?

A16.
Reproducibility means anyone can rerun an experiment and get the same (or very similar) results. Without it, you can't verify claims, debug issues, or build on previous work.

**What MLflow captures for reproducibility**:
- **Parameters**: Exact hyperparameters used.
- **Metrics**: Results achieved.
- **Artifacts**: Model files, plots, data samples.
- **Source code**: Git commit hash, file path.
- **Environment**: Conda/pip dependencies, Python version.
- **Data**: Reference to the data version (logged as parameter or artifact).

**MLflow Projects**: Package the entire experiment as a reproducible unit. Anyone runs `mlflow run <project_uri> -P alpha=0.5` and gets the same environment and code.

**Gaps**: MLflow doesn't automatically version data (pair with DVC). Random seeds must be set explicitly. Hardware differences (GPU vs. CPU) can cause minor numerical differences. But MLflow covers 80% of the reproducibility problem with minimal effort.

---

Q17. How do you handle model rollback in production?

A17.
When a new model in production performs worse than expected (dropped conversion rate, increased errors), you need to quickly revert to the previous version.

**Using MLflow Model Registry**:
1. Current production model is v3.
2. Deploy v4. Performance degrades.
3. In the registry: transition v4 from "Production" back to "Staging" or "Archived."
4. Transition v3 back to "Production."
5. Serving infrastructure picks up v3 automatically (if configured to pull from the registry).

**Prerequisites for smooth rollback**:
- Model versioning with clear lineage.
- Serving infrastructure that references the registry stage, not a hardcoded model path.
- Feature compatibility — if v4 used new features that v3 doesn't expect, the feature pipeline must also roll back or degrade gracefully.
- Monitoring that catches degradation fast — the longer a bad model runs, the more damage.

**Blue-green deployment**: Run both models simultaneously. Traffic switches instantly between them. Rollback is just switching the traffic route — zero downtime.

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q18. What is canary deployment for ML models, and how does it differ from blue-green?

A18.
**Canary deployment**: Route a small percentage of traffic (1-5%) to the new model version while the majority stays on the current version. Gradually increase traffic as you gain confidence. If metrics degrade, roll back immediately.

**vs Blue-green**: Blue-green is all-or-nothing — 100% traffic switches instantly between two environments. Canary is gradual — you control the risk by limiting exposure.

**Why canary is preferred for ML models**:
- ML model behavior is harder to validate in staging than traditional software. Production data distributions differ from test data.
- A model might pass all offline tests but perform poorly on a specific user segment. Canary catches this with limited blast radius.
- Statistical significance takes time — you need enough traffic through the new model to detect regressions.

**Implementation**:
1. Deploy new model version alongside current (separate endpoint or container)
2. Load balancer/service mesh (Istio, AWS App Mesh) routes traffic split: 95/5
3. Monitor key metrics: latency, error rate, business metrics (CTR, conversion)
4. If metrics are stable after X hours/days, increase to 10%, 25%, 50%, 100%
5. If metrics degrade, route 100% back to the old model

**MLOps integration**: Tools like Seldon, BentoML, and KServe support canary deployment natively with traffic splitting and automatic rollback triggers.

---

Q19. How do you set up A/B testing for ML models?

A19.
A/B testing for ML models compares two model versions (A = control, B = treatment) with live traffic to determine which performs better on business metrics.

**Key differences from software A/B testing**:
- ML metrics (AUC, RMSE) don't directly map to business outcomes. You're testing the model's impact on users, not its offline accuracy.
- Effects can be subtle and slower to observe — a recommendation model change might take weeks to affect user retention.

**Setup**:
1. **Randomization**: Hash user ID to deterministically assign users to A or B. Consistent assignment — same user always sees the same model. Avoid session-level splitting (user sees different models across sessions).
2. **Traffic split**: Typically 50/50 for maximum statistical power. Use 90/10 if you're risk-averse about the new model.
3. **Metrics**: Define primary metric (what you're optimizing — e.g., conversion rate) and guardrail metrics (latency, error rate — must not degrade).
4. **Duration**: Run long enough for statistical significance. Calculate sample size upfront based on expected effect size and desired power (typically 80%).
5. **Analysis**: Use proper statistical tests (t-test, Mann-Whitney, or Bayesian methods). Check for novelty effects (users initially engaging more because it's different).

**Common mistakes**: Peeking at results too early and stopping (inflates false positive rate), not accounting for network effects (users influence each other), and not controlling for time-of-day/seasonal effects.

---

### IMPORTANT

---

Q20. How do you set up alerting for model degradation?

A20.
Model monitoring without alerting is just logging. You need automated alerts when model performance degrades.

**What to alert on**:
- **Prediction distribution shift**: Mean prediction drifts significantly (model suddenly predicts higher prices). Use Population Stability Index (PSI) or KL-divergence.
- **Feature distribution shift**: Input features change unexpectedly. A feature that was normally 0-100 suddenly has values of 10000 — data pipeline issue.
- **Performance metrics drop**: If you have delayed labels, track precision/recall when labels arrive. Alert when they drop below thresholds.
- **Operational metrics**: Latency spikes, error rate increase, memory usage growth.
- **Data quality**: Null rate spikes, schema changes, volume drops (upstream pipeline stopped sending data).

**Alert design**:
- Set thresholds with historical baselines — alert on deviations from normal, not absolute values
- Use multiple severity levels: WARNING (investigate when convenient) vs CRITICAL (page someone now)
- Avoid alert fatigue — too many false alarms and people ignore everything
- Include context in alerts: which model, which metric, what the expected vs actual value is, and a link to the dashboard

**Tools**: Prometheus + Grafana for operational metrics, Evidently AI / NannyML for ML-specific monitoring, PagerDuty/Opsgenie for routing alerts, custom dashboards for business metrics.

---

Q21. What is shadow mode deployment, and when would you use it?

A21.
Shadow mode (dark launch) runs the new model on production traffic but doesn't serve its predictions to users. Both the current model (serving responses) and the shadow model (logging predictions) process the same requests.

**How it works**:
1. Production request arrives
2. Current model generates the response → sent to user
3. Same request is also sent to the shadow model → prediction is logged but NOT returned
4. Compare the two models' predictions offline

**When to use**:
- **High-stakes domains**: Healthcare, finance, autonomous systems — you can't risk serving bad predictions. Shadow mode validates the model on real production data without risk.
- **New model architecture**: Switching from tree model to deep learning — verify it handles edge cases production data throws at it.
- **Quality validation**: Ensure the new model doesn't produce unreasonable outputs (negative prices, impossible classifications).

**What you can validate**:
- Prediction distribution comparison
- Latency profile on real production load
- Feature engineering correctness (does the feature pipeline work in production?)
- Edge case handling

**What you can't validate**: Actual business impact (need A/B test for that), user behavior changes, feedback loops.

**Limitation**: Shadow mode adds compute cost (running two models) and doesn't test the model's impact — only that it produces reasonable outputs. After shadow validation, move to canary, then full deployment.

---

### GOOD-TO-HAVE

---

Q22. What is model governance and why do you need audit trails?

A22.
Model governance ensures ML models are developed, deployed, and monitored responsibly — with accountability, compliance, and traceability.

**Why it matters**: Regulations (EU AI Act, GDPR's right to explanation, ECOA for lending), risk management (a bad model in production can cost millions), and organizational trust (stakeholders need to trust ML systems).

**Audit trail components**:
- **Training lineage**: Which dataset version, which code commit, which hyperparameters produced this model? MLflow experiment tracking handles this.
- **Approval workflow**: Who approved this model for production? Model Registry stages (Staging → Production) with required sign-offs.
- **Prediction logging**: What did the model predict and why? Log inputs, outputs, and explanation (SHAP values) for auditing.
- **Change history**: When was the model deployed, changed, or rolled back? Who made the decision and why?
- **Data lineage**: Where did the training data come from? Were there any data quality issues?

**Model cards**: Standardized documentation for each model — its purpose, performance characteristics, limitations, fairness evaluation, and intended use. Google pioneered this; it's becoming standard practice.

**Implementation**: MLflow + Model Registry handles most of the training lineage. Combine with a deployment platform that logs deployments, and prediction logging service for runtime audit. For regulated industries, integrate with your organization's GRC (Governance, Risk, and Compliance) tools.