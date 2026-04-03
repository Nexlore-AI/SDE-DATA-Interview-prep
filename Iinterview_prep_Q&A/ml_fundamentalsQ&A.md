==============================
FILE: Machine Learning Fundamentals
==============================

### HIGH PRIORITY

---

Q1. What is the bias-variance tradeoff? How does it affect model performance?

A1.
**Bias** is the error from oversimplifying the model — it misses patterns in the data. A linear model trying to fit a quadratic relationship has high bias. This leads to underfitting — poor performance on both training and test data.

**Variance** is the error from being too sensitive to training data — the model captures noise as if it were signal. A very deep decision tree that perfectly fits training data but fails on new data has high variance. This is overfitting.

**The tradeoff**: As you increase model complexity, bias decreases (captures more patterns) but variance increases (captures more noise). The sweet spot is a model complex enough to capture real patterns but not so complex that it memorizes noise.

**How to diagnose**: If training accuracy is high but test accuracy is low → high variance (overfit). If both are low → high bias (underfit). Solutions for high variance: more training data, regularization, simpler model, dropout. Solutions for high bias: more features, more complex model, reduce regularization.

---

Q2. What is overfitting, and what techniques help prevent it?

A2.
Overfitting happens when a model learns the training data too well — including its noise and outliers — and fails to generalize to new data. It's like memorizing answers to practice questions instead of understanding the concepts.

**Prevention techniques**:
- **More training data**: The most effective defense — harder to memorize a larger dataset.
- **Regularization**: L1 (Lasso) adds penalty proportional to |weights| → drives some weights to zero (feature selection). L2 (Ridge) adds penalty proportional to weights² → shrinks all weights (prevents any single feature from dominating).
- **Cross-validation**: Train on K-1 folds, validate on the remaining fold. Gives a more reliable estimate of generalization performance.
- **Early stopping**: Monitor validation loss during training. Stop when it starts increasing even though training loss continues decreasing.
- **Dropout** (neural networks): Randomly disable neurons during training. Forces the network to learn redundant representations.
- **Ensemble methods**: Combine multiple models (bagging, boosting) to average out individual model errors.

---

Q3. What is the difference between supervised, unsupervised, and reinforcement learning?

A3.
**Supervised learning**: You have labeled data — input-output pairs. The model learns to map inputs to outputs. Classification (spam vs. not spam) and regression (predict house price). You know the right answer during training.

**Unsupervised learning**: No labels. The model finds patterns, structures, or groupings in the data on its own. Clustering (customer segmentation), dimensionality reduction (PCA), anomaly detection. You're asking "what's interesting in this data?"

**Reinforcement learning**: An agent learns by interacting with an environment and receiving rewards or penalties. No labeled data — it discovers good strategies through trial and error. Game playing (AlphaGo), robotics (walking), recommendation systems (maximize engagement). The feedback loop is: action → reward → learn → repeat.

**Semi-supervised**: Uses a small amount of labeled data with a large amount of unlabeled data. Common in practice because labeling is expensive. Self-training, knowledge distillation.

---

Q4. Explain precision, recall, F1-score, and when you would prioritize one over another.

A4.
For a binary classifier:
- **Precision**: Of all predicted positives, how many were actually positive? `TP / (TP + FP)`. "When the model says yes, how often is it right?"
- **Recall**: Of all actual positives, how many did the model catch? `TP / (TP + FN)`. "Of all the real positives, how many did the model find?"
- **F1-score**: Harmonic mean of precision and recall. `2 * (P * R) / (P + R)`. Balances both when neither is clearly more important.

**When to prioritize**:
- **Precision**: When false positives are expensive. Spam filter — marking a legitimate email as spam is worse than letting some spam through. Medical diagnosis — telling a healthy person they're sick causes unnecessary procedures.
- **Recall**: When false negatives are dangerous. Cancer screening — missing a real cancer case is worse than a false alarm. Fraud detection — missing fraud costs real money.
- **F1**: When you need to balance both and don't have a strong asymmetry in costs.

For imbalanced datasets, accuracy is misleading (99% accuracy by always predicting the majority class). Use precision, recall, F1, or AUC-ROC instead.

---

Q5. What is gradient descent? Explain the difference between batch, mini-batch, and stochastic.

A5.
Gradient descent is an optimization algorithm that minimizes the loss function by iteratively adjusting model parameters in the direction of steepest descent (negative gradient).

At each step: `weights = weights - learning_rate * gradient(loss)`

**Batch gradient descent**: Computes the gradient using the entire training dataset. Stable convergence but slow — for 10M samples, you need to process all 10M before updating weights once.

**Stochastic gradient descent (SGD)**: Computes the gradient using one sample at a time. Very fast updates but noisy — the loss zigzags because each sample pulls in a different direction. Can escape local minima due to noise.

**Mini-batch SGD**: Computes the gradient using a small batch (32, 64, 128, 256 samples). Best of both worlds — more stable than SGD, faster than batch. This is what everyone actually uses when they say "SGD."

**Learning rate** is the most critical hyperparameter. Too high → overshoot, diverge. Too low → slow convergence. Modern optimizers (Adam, AdaGrad, RMSProp) adapt the learning rate per parameter automatically.

---

Q6. What is cross-validation, and why is a simple train/test split sometimes insufficient?

A6.
A single train/test split gives you one estimate of model performance, which could be lucky or unlucky depending on how the data was split.

**K-Fold cross-validation**: Split data into K folds (usually 5 or 10). Train on K-1 folds, test on the remaining fold. Repeat K times, each time using a different fold as the test set. Average the K scores — this gives a more robust estimate of generalization performance.

**Stratified K-Fold**: Ensures each fold has the same class distribution as the full dataset. Critical for imbalanced datasets — a random split might put all rare-class samples in one fold.

**When simple split is insufficient**:
- Small datasets — you can't afford to "waste" 20% of data for testing.
- Class imbalanced data — a random split might not represent the minority class in the test set.
- High variance in results — different splits give wildly different scores.

**Time series exception**: Don't use random K-Fold for time series — it leaks future data into the training set. Use time-based splits: train on January-March, test on April. Walk-forward validation.

---

Q7. What is regularization, and how do L1 and L2 regularization differ?

A7.
Regularization adds a penalty to the loss function to discourage the model from learning overly complex solutions. It constrains the model's capacity to prevent overfitting.

**L1 (Lasso)**: Adds `λ * Σ|wᵢ|` to the loss. The absolute value penalty drives some weights exactly to zero — effectively performing feature selection. Useful when you suspect many features are irrelevant.

**L2 (Ridge)**: Adds `λ * Σwᵢ²` to the loss. The squared penalty shrinks all weights toward zero but rarely makes them exactly zero. Distributes the model's dependence across all features. Useful when you believe most features are relevant but don't want any single feature dominating.

**Elastic Net**: Combines L1 and L2: `α * L1 + (1-α) * L2`. Best of both worlds.

**λ** (regularization strength) controls the trade-off: higher λ = stronger penalty = simpler model (higher bias, lower variance). Tune it via cross-validation.

Regularization is essentially saying: "I'd rather have a slightly worse fit on training data if it means the model generalizes better."

---

Q8. What is a decision tree? What are its advantages and limitations?

A8.
A decision tree splits data recursively on features to make predictions. At each node, it chooses the feature and threshold that best separates the data (using Gini impurity or information gain for classification, MSE for regression).

**Advantages**: Highly interpretable — you can visualize and explain each decision. Handles numerical and categorical data. No feature scaling needed. Captures non-linear relationships.

**Limitations**: Prone to overfitting — a deep tree memorizes training data. High variance — small changes in data can produce very different trees. Biased toward features with more levels. Greedy splitting (not globally optimal). Poor at extrapolation.

**How to improve**: Pruning (limit depth, min samples per leaf). But more commonly — use ensemble methods. Random Forest (bagging many trees, each trained on a random subset of data and features) and Gradient Boosting (XGBoost, LightGBM — sequentially build trees that correct previous trees' errors). These ensembles sacrifice some interpretability for dramatically better performance.

---

Q9. What is the ROC curve, and what does AUC represent?

A9.
The **ROC curve** (Receiver Operating Characteristic) plots True Positive Rate (recall) vs. False Positive Rate at various classification thresholds. As you lower the threshold, you classify more samples as positive — catching more true positives but also more false positives.

**AUC** (Area Under the Curve): A single number summarizing the ROC curve. AUC = 1.0 is a perfect classifier. AUC = 0.5 is random guessing (the diagonal line). AUC tells you: "If I pick a random positive and a random negative, what's the probability the model scores the positive higher?"

**When to use**: AUC is threshold-independent — useful when you haven't chosen a decision threshold yet, or when comparing models across thresholds. Good for balanced and moderately imbalanced datasets.

**When NOT to use**: Highly imbalanced datasets — AUC can be misleadingly high. If 0.1% of transactions are fraud, a model with many false positives can still have great AUC. Use Precision-Recall AUC instead — it focuses on the positive class.

---

Q10. What is feature engineering? Give examples of commonly used techniques.

A10.
Feature engineering is creating, transforming, or selecting input features to improve model performance. It's often more impactful than choosing a fancier algorithm.

**Techniques**:
- **Encoding categoricals**: One-hot encoding (creates binary columns), label encoding (assigns integers — order matters for tree models), target encoding (replace category with mean of target — risk of leakage, use with cross-validation).
- **Scaling**: StandardScaler (zero mean, unit variance) for algorithms sensitive to scale (SVM, logistic regression, neural nets). MinMaxScaler for bounded ranges.
- **Binning**: Convert continuous variables to categories — age into age groups. Captures non-linear effects for linear models.
- **Interaction features**: Multiply features — `height * width = area`. Polynomial features for linear models.
- **Time-based**: Day of week, hour of day, is_weekend, days_since_last_event. Hugely important for time-series.
- **Domain-specific**: Text → TF-IDF, word embeddings. Images → pretrained CNN features. Geography → distance calculations, clustering.

"Applied machine learning is basically feature engineering." — Andrew Ng. The best models fail with bad features.

---

### MEDIUM PRIORITY

---

Q11. What is the difference between bagging and boosting?

A11.
Both are ensemble methods — combining multiple weak models to make a strong one.

**Bagging** (Bootstrap Aggregating): Train multiple models independently on random subsets of the data (sampled with replacement). Aggregate predictions by voting (classification) or averaging (regression). Reduces variance. Random Forest is the classic example — each tree sees a random subset of data AND features.

**Boosting**: Train models sequentially, where each new model focuses on the errors of the previous ones. Misclassified samples get higher weights. Reduces bias. XGBoost, LightGBM, AdaBoost. Typically produces higher accuracy but can overfit if not tuned carefully.

**Key differences**: Bagging models are independent (can be parallelized), boosting models are sequential. Bagging reduces variance (stabilizes unstable models), boosting reduces bias (improves underfitting models). Bagging rarely overfits; boosting can if you add too many rounds.

**Practical choice**: XGBoost/LightGBM (boosting) wins most Kaggle competitions and structured data tasks. Random Forest (bagging) is a great baseline — robust, hard to mess up, needs less tuning.

---

Q12. What is the curse of dimensionality?

A12.
As the number of features (dimensions) increases, data becomes increasingly sparse. In high-dimensional space, distances between points become meaningless — all points appear roughly equidistant.

**Practical consequences**:
- K-Nearest Neighbors fails because "nearest" loses meaning when all distances are similar.
- You need exponentially more data to maintain the same density — 10 features with 10 samples per feature need 10¹⁰ samples.
- Models overfit more easily — with many features, the model can find spurious correlations.
- Distance-based algorithms (KNN, SVM with RBF, clustering) degrade.

**Solutions**: Feature selection (remove irrelevant features), dimensionality reduction (PCA, t-SNE, UMAP), regularization (L1 to zero-out features), domain knowledge (only include meaningful features).

Tree-based models (Random Forest, XGBoost) handle high dimensionality better because they implicitly select features at each split.

---

Q13. What is Principal Component Analysis (PCA)? When is it useful?

A13.
PCA finds the directions (principal components) of maximum variance in the data and projects the data onto these directions, reducing dimensions while preserving as much information as possible.

The first principal component captures the most variance, the second captures the next most (orthogonal to the first), and so on. You keep the top K components that explain, say, 95% of the variance, and discard the rest.

**When useful**: Reducing features from 1000 to 50 for model training (speeds up computation, reduces overfitting). Visualization — projecting high-dimensional data to 2D/3D. Noise reduction — minor components often represent noise.

**Limitations**: Assumes linear relationships — can't capture non-linear structure (use t-SNE or UMAP for visualization in that case). Components are hard to interpret — they're linear combinations of original features, not individual features. Scale-sensitive — standardize features first.

**Example**: Image recognition — a 100x100 image has 10,000 features. PCA might reduce this to 100 components while retaining 95% of the information.

---

Q14. What is a confusion matrix, and what metrics can you derive from it?

A14.
A 2x2 matrix for binary classification:

| | Predicted Positive | Predicted Negative |
|---|---|---|
| **Actual Positive** | True Positive (TP) | False Negative (FN) |
| **Actual Negative** | False Positive (FP) | True Negative (TN) |

**Derived metrics**:
- Accuracy = (TP + TN) / Total — overall correctness. Misleading for imbalanced data.
- Precision = TP / (TP + FP) — reliability of positive predictions.
- Recall = TP / (TP + FN) — coverage of actual positives.
- F1 = 2 * Precision * Recall / (Precision + Recall) — harmonic mean.
- Specificity = TN / (TN + FP) — true negative rate.
- FPR (False Positive Rate) = FP / (FP + TN) = 1 - Specificity.

For multi-class, extend to NxN matrix. Each row is an actual class, each column is a predicted class. Diagonal = correct predictions. Off-diagonal cells show which classes are confused with each other — revealing systematic errors.

---

Q15. What is the difference between parametric and non-parametric models?

A15.
**Parametric**: Fixed number of parameters regardless of training data size. Linear regression (coefficients per feature), logistic regression, naive Bayes. You assume a functional form (linear, polynomial) and fit the parameters. Fast to train and predict, but if the assumption is wrong, the model underperforms.

**Non-parametric**: Number of parameters grows with data. KNN (stores all training points), decision trees (grows with data complexity), SVMs (support vectors depend on data). Makes fewer assumptions — more flexible but requires more data and can be slower.

**Practical implications**: Parametric models are simpler, faster, need less data, but can underfit complex relationships. Non-parametric models are more flexible but can overfit and are computationally expensive.

Note: "non-parametric" doesn't mean "no parameters" — it means the parameter space isn't fixed in advance. A deep neural network is technically parametric (fixed architecture) but has so many parameters it behaves like a non-parametric model in practice.

---

Q16. What is K-Means clustering? What are its limitations?

A16.
K-Means partitions data into K clusters by iteratively: (1) assigning each point to the nearest centroid, and (2) updating centroids to the mean of assigned points. Repeat until convergence.

**Choosing K**: The elbow method — plot inertia (sum of squared distances to centroids) vs. K. Look for the "elbow" where adding more clusters stops meaningfully reducing inertia. Silhouette score — measures how similar a point is to its own cluster vs. other clusters.

**Limitations**:
- Must specify K in advance — wrong K gives bad clustering.
- Assumes spherical, equally-sized clusters — fails on elongated, overlapping, or uneven clusters.
- Sensitive to initialization — different starting centroids → different results. Use K-Means++ for better initialization.
- Sensitive to outliers — outliers pull centroids away from the true cluster center.
- Only finds convex clusters — can't discover non-convex shapes.

**Alternatives**: DBSCAN (density-based, finds non-convex clusters, handles outliers), Gaussian Mixture Models (soft assignment, handles ellipsoidal clusters), hierarchical clustering (doesn't need K predetermined).

---

Q17. What is the difference between generative and discriminative models?

A17.
**Discriminative models** learn the boundary between classes — P(Y|X). "Given these features, what's the label?" Logistic regression, SVM, neural networks, decision trees. They directly model the decision boundary without understanding the underlying data distribution.

**Generative models** learn the data distribution — P(X|Y) and P(Y), then use Bayes' theorem to get P(Y|X). Naive Bayes, Gaussian Mixture Models, Hidden Markov Models, GANs, VAEs. They model how the data was generated.

**Practical difference**: Discriminative models usually achieve higher accuracy on classification tasks because they focus directly on the boundary. Generative models can generate new data samples, handle missing data, and work with fewer labeled samples.

Modern "generative AI" (GPT, Stable Diffusion) uses generative models at massive scale — they learn the distribution of text/images and can produce new samples from that distribution.

---

Q18. How do you handle imbalanced datasets?

A18.
When one class is much rarer (e.g., 1% fraud vs. 99% legitimate), standard models learn to predict the majority class and get 99% accuracy while catching zero fraud.

**Data-level techniques**:
- **Oversampling the minority**: SMOTE generates synthetic minority samples by interpolating between existing ones. Better than random duplication.
- **Undersampling the majority**: Randomly remove majority samples. Risks losing useful data.
- **Combination**: SMOTE + Tomek links (SMOTE oversamples minority, Tomek links removes borderline majority samples).

**Algorithm-level techniques**:
- **Class weights**: Most algorithms accept `class_weight='balanced'` — internally adjusts the loss to penalize minority class errors more.
- **Threshold tuning**: Instead of predicting class at 0.5, choose a threshold that optimizes precision-recall for your use case.
- **Anomaly detection framing**: If the minority is very rare, treat it as anomaly detection (one-class SVM, isolation forest).

**Evaluation**: Use precision, recall, F1, PR-AUC instead of accuracy. Accuracy is meaningless for imbalanced data.

---

### LOW PRIORITY

---

Q19. What is the difference between a generative and a discriminative classifier? Give examples.

A19.
This overlaps with the generative vs. discriminative model question but focused on classifiers:

**Naive Bayes** (generative): Models P(features|class) for each class, then uses Bayes' theorem. Assumes features are independent given the class — obviously false in practice, but works surprisingly well for text classification (spam filtering). Fast training, works with small data.

**Logistic Regression** (discriminative): Directly models P(class|features) using a logistic function. No assumption about how features are distributed — only about the decision boundary (linear). Generally more accurate than Naive Bayes when you have sufficient data.

**When generative wins**: Very small training sets, missing features (generative can marginalize), when you need to generate new samples or detect out-of-distribution inputs.

**When discriminative wins**: Sufficient training data, high-dimensional features, complex decision boundaries. In practice, discriminative models dominate classification tasks.

---

Q20. What is the kernel trick, and how does it relate to SVMs?

A20.
Support Vector Machines find the hyperplane that maximizes the margin between classes. But real data is often not linearly separable in its original space.

The **kernel trick** implicitly maps data to a higher-dimensional space where it becomes linearly separable — without actually computing the transformation. Instead, it computes dot products in the high-dimensional space directly using a kernel function.

**Common kernels**: RBF (Gaussian) — maps to infinite dimensions, captures complex non-linear boundaries. Polynomial — maps to polynomial feature space. Linear — no mapping, just a linear SVM.

**Example**: 2D data arranged in concentric circles isn't linearly separable. An RBF kernel implicitly maps it to a higher dimension where a hyperplane can separate the circles.

**Why it's clever**: Computing the actual high-dimensional transformation would be expensive. The kernel function computes the result of dot products in that space directly — O(n²) in the number of samples, not the dimensionality.

**Practical note**: SVMs with RBF kernels work well on small-to-medium datasets. For large datasets (100K+ samples), tree-based models or neural networks are more practical.

---

Q21. What is the difference between online learning and batch learning?

A21.
**Batch learning**: The model is trained on the entire dataset at once. Retraining means processing all data again. Suitable when data is static or you can afford periodic full retrains (nightly, weekly).

**Online learning**: The model updates incrementally as new data arrives — one sample or mini-batch at a time. The model evolves continuously without retraining from scratch. Useful for streaming data, concept drift (data distribution changes over time), or data too large to fit in memory.

**Algorithms that support online learning**: SGD-based models (linear regression, logistic regression with SGD), Vowpal Wabbit, online variants of Naive Bayes, and neural networks (they're naturally trained incrementally via mini-batch SGD).

**Concept drift**: The relationship between features and target changes over time — what predicted customer churn last year might not work this year. Online learning adapts naturally; batch learning requires detecting drift and triggering retraining.

---

Q22. What is Naive Bayes, and why is it called "naive"?

A22.
Naive Bayes is a probabilistic classifier based on Bayes' theorem: `P(class|features) ∝ P(features|class) * P(class)`.

It's called "naive" because it assumes all features are conditionally independent given the class. For text classification: the probability of a spam email containing "free" is independent of it containing "win," given that it's spam. This is obviously false — but the model works surprisingly well despite the wrong assumption.

**Why it works despite the wrong assumption**: For classification, you don't need accurate probability estimates — you just need the correct ordering (which class has the highest probability). Even with wrong independence assumptions, the ranking often remains correct.

**Variants**: Multinomial NB (word counts — text classification), Bernoulli NB (binary features — word presence/absence), Gaussian NB (continuous features — assumes normal distribution).

**Strengths**: Extremely fast training and prediction, works well with small data, handles high-dimensional data (text with 100K word features). A great baseline for text classification before reaching for more complex models.

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q23. What's the difference between classification and regression, and how do you decide which to use?

A23.
**Classification**: Predict a discrete category — spam/not-spam, cat/dog/bird, fraud/legitimate. Output is a class label (or probability distribution over classes).

**Regression**: Predict a continuous value — house price, temperature, revenue forecast. Output is a number.

**How to decide**: Look at your target variable. If it's categorical → classification. If it's a number on a continuous scale → regression.

**Edge cases**:
- Predicting a score from 1-5: Could be either. If you care about exact ranking, ordinal regression. If you treat each rating as a distinct category, classification.
- Predicting age brackets (18-24, 25-34): Classification, even though age is continuous — you've bucketed it.
- Predicting probability of default: Regression (outputs a continuous probability), but the downstream decision is binary (approve/deny).

**Metrics differ**: Classification uses accuracy, precision, recall, F1, AUC-ROC. Regression uses MAE, MSE, RMSE, R². Don't mix them up — accuracy makes no sense for regression.

**Algorithms overlap**: Random forests, gradient boosting, and neural networks can do both — just change the loss function (cross-entropy for classification, MSE for regression) and the output layer.

---

Q24. What are hyperparameters vs model parameters, and how do you tune hyperparameters?

A24.
**Model parameters**: Learned during training from data. Weights in a neural network, coefficients in linear regression, split points in a decision tree. You don't set these — the algorithm discovers them.

**Hyperparameters**: Set before training. They control the learning process itself. Learning rate, number of trees, max depth, regularization strength, batch size, number of layers. You choose these — the model can't learn them from data.

**Tuning approaches** (from simple to sophisticated):

1. **Manual tuning**: Educated guesses. Good starting point if you understand the algorithm.
2. **Grid search**: Try every combination of specified values. Exhaustive but exponentially expensive with more hyperparameters.
3. **Random search**: Randomly sample combinations. Surprisingly effective — Bergstra & Bengio showed it often beats grid search because it explores more values of each hyperparameter.
4. **Bayesian optimization** (Optuna, Hyperopt): Builds a probabilistic model of the objective function. Each trial is informed by previous results — it focuses on promising regions. Much more efficient for expensive models.
5. **Successive halving / Hyperband**: Start many configurations with small budget (few epochs). Eliminate poor performers early, give more budget to promising ones.

**Practical advice**: Start with reasonable defaults (XGBoost's defaults are usually 80% of the way there). Tune the most impactful hyperparameters first — for tree models: learning rate, n_estimators, max_depth. Use cross-validation during tuning to avoid overfitting to the validation set.

---

Q25. What is data leakage, and how do you prevent it?

A25.
Data leakage happens when information from the test set (or the future) leaks into the training process, making your model look better in development than it actually performs in production.

**Types of leakage**:

1. **Target leakage**: A feature that's derived from or correlated with the target but wouldn't be available at prediction time. Example: predicting hospital readmission and including "discharge_summary_sentiment" — which is written after the readmission decision.

2. **Train-test contamination**: Test data leaks into training. Scaling/normalizing before splitting (the scaler learns from test data), deduplication after splitting (same record in both sets).

3. **Temporal leakage**: Using future information. In time-series, random splitting mixes future data into training. Always split chronologically.

**Prevention**:
- Split data before any preprocessing. Put the split as early as possible in your pipeline.
- For time-series, use time-based splits. Never random.
- Examine features carefully: "Would I have this feature at prediction time?" If not, remove it.
- Use sklearn's `Pipeline` to ensure fit_transform only on training data.
- Be suspicious of unrealistically high metrics — if your model gets 99.5% accuracy, look for leakage before celebrating.

---

Q26. What is stratified sampling, and when should you use it?

A26.
Stratified sampling ensures that each split (train/test/validation) maintains the same proportion of each class as the original dataset.

If your dataset is 95% non-fraud and 5% fraud, a random 80/20 split might put only 3% fraud in the test set by chance. Stratified sampling guarantees both sets are 95%/5%.

```python
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)
```

**When to use it**:
- **Imbalanced classification**: The most important case. Without stratification, minority class may be underrepresented in one split.
- **Small datasets**: Random variation has bigger impact. Stratification ensures representative splits.
- **Multi-class**: Rare classes might be entirely absent from a random split.
- **Cross-validation**: Use `StratifiedKFold` instead of `KFold` for classification.

**When it doesn't matter**: Large balanced datasets — random splitting naturally maintains proportions. Regression — you can use stratified splitting on binned target values, but it's less critical.

---

### IMPORTANT

---

Q27. How do you interpret feature importance, and what is SHAP?

A27.
**Feature importance** tells you which features the model relies on most. But different methods give different answers:

**Built-in importance** (tree models): Based on how much each feature reduces impurity (Gini/entropy) or is used in splits. Fast but biased toward high-cardinality features and doesn't account for feature interactions.

**Permutation importance**: Shuffle one feature's values, measure how much model performance drops. If accuracy drops a lot, that feature is important. Model-agnostic, but slow and affected by correlated features.

**SHAP** (SHapley Additive exPlanations): Based on game theory — Shapley values from cooperative game theory. For each prediction, SHAP tells you how much each feature contributed (positively or negatively) to that specific prediction.

```python
import shap
explainer = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test)
shap.summary_plot(shap_values, X_test)
```

**SHAP properties**:
- Local explanations (per prediction) and global (aggregated across all predictions)
- Consistent — if a feature's real contribution increases, its SHAP value can't decrease
- Additive — SHAP values for all features sum to the difference between the prediction and the average prediction
- Computationally expensive for non-tree models (use KernelSHAP approximation)

Use SHAP when you need to explain individual predictions to stakeholders or debug model behavior. Use permutation importance for quick global feature ranking.

---

Q28. What are learning curves, and how do you use them to diagnose model issues?

A28.
A learning curve plots model performance against training set size. You train the model on increasing fractions of data and evaluate on both training and validation sets.

**High bias (underfitting)**: Training and validation scores are both low, close together, and plateau early. Adding more data won't help — the model is too simple. Fix: increase model complexity, add features, reduce regularization.

**High variance (overfitting)**: Training score is high, validation score is much lower. The gap is large but narrows as you add more data. Fix: get more data, simplify the model, increase regularization, use dropout.

**Good fit**: Both scores are high and converge. The gap is small.

```python
from sklearn.model_selection import learning_curve
train_sizes, train_scores, val_scores = learning_curve(
    model, X, y, train_sizes=np.linspace(0.1, 1.0, 10), cv=5
)
```

**Practical use**: Before throwing more data at a problem, check the learning curve. If you have a high-bias problem, more data is useless — invest in feature engineering or a more powerful model. If you have high variance, more data actually helps.

---

Q29. What are different distance metrics, and when do you use each?

A29.
**Euclidean distance** (L2): Straight-line distance in n-dimensional space. `sqrt(Σ(xi - yi)²)`. Most common, works when features are on similar scales. Sensitive to outliers and high dimensionality (curse of dimensionality).

**Manhattan distance** (L1): Sum of absolute differences. `Σ|xi - yi|`. Better in high dimensions — less affected by the curse of dimensionality. Preferred when features represent different things (like city block distances).

**Cosine similarity**: Measures angle between vectors, not magnitude. `cos(θ) = (A·B)/(|A||B|)`. Perfect for text (TF-IDF, embeddings) where document length shouldn't matter — a short and long document about the same topic should be similar.

**Minkowski distance**: Generalization — Euclidean (p=2) and Manhattan (p=1) are special cases. `(Σ|xi - yi|^p)^(1/p)`.

**Hamming distance**: Count of positions where values differ. For binary/categorical data. Used in error-correcting codes and categorical clustering.

**Mahalanobis distance**: Accounts for correlations between features and different variances. Like Euclidean but in the space warped by the data's covariance matrix. Useful for anomaly detection.

**Key consideration**: Always normalize/standardize features before computing distances (except cosine, which is scale-invariant). A feature in thousands (salary) would dominate a feature in single digits (age) with Euclidean distance.

---

Q30. What's the difference between a loss function and an evaluation metric?

A30.
**Loss function** (training objective): What the model directly optimizes during training. Must be differentiable for gradient-based methods. Examples: cross-entropy, MSE, hinge loss.

**Evaluation metric**: What you use to judge model quality for business decisions. Doesn't need to be differentiable. Examples: accuracy, F1-score, AUC-ROC, BLEU score.

They're often different because the business metric isn't differentiable or isn't directly optimizable:
- You care about **F1-score**, but train with **cross-entropy** (F1 isn't differentiable)
- You care about **AUC-ROC**, but train with **log loss** (AUC isn't easily optimizable)
- You care about **NDCG** (ranking quality), but train with pairwise or listwise losses

**Why the distinction matters**: A model that minimizes training loss might not maximize your evaluation metric. If your loss function and metric are misaligned, the model might converge to a solution that's mathematically optimal for the loss but poor for what you actually care about.

Example: In imbalanced datasets, cross-entropy loss treats all samples equally, but you might evaluate with precision@k because false positives are costly. In this case, consider focal loss, weighted cross-entropy, or training with a loss that better proxy's your metric.

---

### GOOD-TO-HAVE

---

Q31. What's the difference between multi-label and multi-class classification?

A31.
**Multi-class**: Each sample belongs to exactly one class out of many. Predicting an animal as cat, dog, or bird. The classes are mutually exclusive. Output: softmax (probabilities sum to 1), pick the highest.

**Multi-label**: Each sample can belong to multiple classes simultaneously. A movie can be Action AND Comedy AND Sci-Fi. The labels are independent. Output: sigmoid per label (each independently 0 or 1), threshold each.

**Key difference in architecture**:
- Multi-class: One output layer with softmax, cross-entropy loss
- Multi-label: One output per label with sigmoid, binary cross-entropy per label

**Evaluation differs too**: Multi-class uses accuracy, macro/micro F1. Multi-label uses Hamming loss (fraction of wrong labels), subset accuracy (all labels correct), per-label F1.

Practical example: Email classification — if each email goes to exactly one folder (Inbox, Spam, Social), it's multi-class. If each email can have multiple tags (Important, Urgent, Personal), it's multi-label.

---

Q32. What is target encoding, and when would you use it over one-hot encoding?

A32.
**Target encoding** (mean encoding): Replace a categorical feature's values with the mean of the target variable for that category. For a "city" feature predicting house price, replace "NYC" with the average house price in NYC.

**vs One-hot encoding**: One-hot creates a binary column per category. If "city" has 1000 unique values, that's 1000 new columns — high dimensionality, sparse, and tree models slow down.

**When to use target encoding**:
- High-cardinality categoricals (100+ categories): One-hot is impractical
- Tree-based models: They handle target encoding well
- When categories have a meaningful relationship with the target

**Risks**:
- **Overfitting**: If a category has few samples, its mean is noisy. A city with 2 houses gives a meaningless mean.
- **Target leakage**: You're encoding information about the target into features.

**Mitigation**:
- **Leave-one-out encoding**: For each row, compute the mean excluding that row
- **K-fold target encoding**: Compute means using out-of-fold data (like cross-validation)
- **Smoothing**: Blend category mean with global mean: `encoding = α * category_mean + (1-α) * global_mean`, where α depends on category count
- **Add noise**: Small Gaussian noise to prevent overfitting

Libraries: `category_encoders` in Python handles this + regularization.

---

Q33. Does correlation imply causation? How do you distinguish them in practice?

A33.
No. Correlation measures linear relationship between two variables. Causation means one variable directly influences the other. The classic examples: ice cream sales and drowning rates are correlated (both increase in summer), but ice cream doesn't cause drowning. Summer (a confounder) causes both.

**Why it matters in ML**: A model might learn spurious correlations that break when the data distribution shifts. If "number of firefighters" correlates with "fire damage" (bigger fires → more firefighters), a model might predict that sending fewer firefighters reduces damage.

**How to establish causation**:
1. **Randomized Controlled Trials** (A/B tests): The gold standard. Randomly assign treatment and control. If the outcome differs, the treatment caused it.
2. **Natural experiments**: Find situations where assignment was effectively random (e.g., a policy change in one region but not another).
3. **Causal inference methods**: When you can't run experiments:
   - **Instrumental variables**: Find a variable that affects X but not Y directly
   - **Difference-in-differences**: Compare before/after in treatment vs control groups
   - **Propensity score matching**: Match treated and untreated units on confounders
   - **Directed Acyclic Graphs (DAGs)**: Model the causal structure explicitly (DoWhy library)

**In practice**: Model is predicting well → don't assume the features cause the target. Validate causal claims with domain expertise and, when possible, A/B tests.