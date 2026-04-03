==============================
FILE: Deep Learning
==============================

### HIGH PRIORITY

---

Q1. What is a neural network? Explain the basic architecture (input, hidden, output layers).

A1.
A neural network is a function approximator composed of layers of interconnected nodes (neurons). Each neuron computes a weighted sum of its inputs, adds a bias, and passes the result through an activation function.

**Architecture**:
- **Input layer**: Receives raw features — one neuron per feature. For a 784-pixel image, 784 input neurons.
- **Hidden layers**: Where learning happens. Each neuron takes inputs from the previous layer, applies weights + bias + activation. Multiple hidden layers = "deep" learning. More layers = more abstract feature extraction.
- **Output layer**: Produces the final prediction. One neuron for regression, one per class for classification (with softmax for probabilities).

**Forward pass**: Input → multiply by weights → add bias → activation → next layer → ... → output.
**Backward pass (backpropagation)**: Compute loss → calculate gradients using chain rule → update weights using gradient descent.

The key insight: with enough neurons and layers, a neural network can approximate any continuous function (universal approximation theorem). The challenge is finding the right weights efficiently — that's what training does.

---

Q2. What are activation functions? Compare ReLU, sigmoid, and tanh.

A2.
Activation functions introduce non-linearity — without them, stacking layers would just be sequential linear transformations (equivalent to a single layer).

**Sigmoid**: σ(x) = 1/(1+e^(-x)). Output range: (0, 1). Used in output layer for binary classification (probability). **Problem**: Saturates at extremes — gradients near 0 for very large or small inputs (vanishing gradient). Outputs are not zero-centered — slows convergence.

**Tanh**: tanh(x). Output range: (-1, 1). Zero-centered (better than sigmoid for hidden layers). But still suffers from vanishing gradients at extremes.

**ReLU**: f(x) = max(0, x). Output range: [0, ∞). Simple, fast to compute. Solves vanishing gradient for positive values. **Problem**: "Dying ReLU" — neurons with negative input always output 0 and stop learning.

**Modern defaults**: ReLU for hidden layers (or Leaky ReLU / GELU). Sigmoid for binary output. Softmax for multi-class output. GELU is used in Transformers (smoother than ReLU, non-zero gradient for negative inputs).

---

Q3. What is backpropagation? How does it enable training?

A3.
Backpropagation calculates the gradient of the loss function with respect to each weight in the network — then gradient descent uses these gradients to update the weights.

**The chain rule is the core**: For a deep network, the loss depends on the output, which depends on the last hidden layer, which depends on the previous layer, ..., which depends on the weights of the first layer. The chain rule allows you to decompose this dependency and compute gradients layer by layer, from output back to input.

**Steps**:
1. **Forward pass**: Compute output and loss.
2. **Backward pass**: Compute ∂Loss/∂weights for each layer, starting from the output layer and propagating backwards.
3. **Weight update**: `w = w - learning_rate * gradient`.

**Vanishing gradient problem**: In deep networks with sigmoid/tanh activations, gradients get multiplied by small numbers at each layer. By the time they reach early layers, they're nearly zero — early layers barely learn. Solutions: ReLU, batch normalization, skip connections (ResNets), and careful initialization.

---

Q4. What is the vanishing gradient problem? How do modern architectures address it?

A4.
In deep networks, gradients during backpropagation pass through many layers via multiplication. If each layer's gradient is < 1 (sigmoid/tanh in saturated regions), the product shrinks exponentially. Layer 1's gradient becomes negligibly small — the first layers don't learn.

**Conversely, exploding gradients**: If gradients > 1 at each layer, they grow exponentially. Weights become NaN. Gradient clipping (cap gradient magnitude) addresses this.

**Solutions to vanishing gradients**:
- **ReLU activation**: Gradient is 1 for positive inputs — no shrinkage through layers.
- **Skip connections (ResNets)**: Shortcut connections that let gradients flow directly to earlier layers, bypassing the multiplication chain. `output = F(x) + x`.
- **Batch Normalization**: Normalizes layer inputs — prevents activations from saturating. Keeps gradients in a healthy range.
- **LSTM/GRU gates**: For RNNs — gates control gradient flow, preventing vanishing over long sequences.
- **Proper initialization**: Xavier/Glorot initialization keeps variance stable across layers. Kaiming/He initialization is designed for ReLU.

Modern networks with these techniques can be hundreds of layers deep (ResNet-152, GPT-3 with 96 layers).

---

Q5. What is batch normalization? Why does it help training?

A5.
Batch normalization normalizes the inputs to each layer (within a mini-batch) to have zero mean and unit variance, then applies a learnable scale and shift.

For each mini-batch: normalize to μ=0, σ=1, then `output = γ * normalized + β` where γ and β are learned parameters.

**Why it helps**:
- **Reduces internal covariate shift**: As weights update, the input distribution to each layer changes, making subsequent layers' job harder. BatchNorm stabilizes these distributions.
- **Allows higher learning rates**: Without BatchNorm, high learning rates cause instability. With it, you can train faster.
- **Regularization effect**: The mini-batch statistics add noise (each batch has slightly different mean/std), acting as implicit regularization. Some teams reduce dropout when using BatchNorm.

**Where it's placed**: Typically after the linear transformation and before the activation: `Linear → BatchNorm → ReLU`.

**At inference time**: Uses running averages of mean and variance computed during training — not the batch statistics.

**Alternative**: Layer Normalization (used in Transformers) — normalizes across features within a single sample, not across the batch. Works better for variable-length sequences and small batch sizes.

---

Q6. What are CNNs (Convolutional Neural Networks)? How do they process images?

A6.
CNNs exploit the spatial structure of images through three key operations:

**Convolutional layer**: Slides small filters (e.g., 3x3) across the image, computing dot products at each position. Each filter detects a specific feature (edges, textures, patterns). A layer with 64 filters produces 64 feature maps. Key property: weight sharing — the same filter is used across all positions, making CNNs translation-invariant.

**Pooling layer**: Reduces spatial dimensions (downsampling). Max pooling takes the maximum value in each 2x2 region, halving width and height. Reduces computation and adds spatial invariance.

**Fully connected layer**: Flattens the final feature maps and connects to the output for classification.

**Hierarchy of features**: Early layers detect low-level features (edges, corners). Middle layers detect mid-level features (textures, parts). Deep layers detect high-level features (faces, objects). This hierarchical feature extraction is why CNNs are so effective.

**Architectures**: LeNet (original), AlexNet (ImageNet breakthrough), VGG (deeper, simpler), ResNet (skip connections, 100+ layers), EfficientNet (optimized scaling).

---

Q7. What are RNNs, and why do they struggle with long sequences? How do LSTMs/GRUs solve this?

A7.
**RNNs** (Recurrent Neural Networks) process sequential data by maintaining a hidden state that carries information from previous time steps. At each step: `h_t = activation(W_h * h_{t-1} + W_x * x_t)`. The hidden state is the "memory."

**The problem**: For long sequences, the gradient must travel through many time steps during backpropagation. With vanilla RNNs, the vanishing gradient problem makes it impossible to learn long-range dependencies. After ~20-30 steps, the gradient is effectively zero.

**LSTM** (Long Short-Term Memory) solution: Introduces a cell state (a highway for information) and three gates:
- **Forget gate**: What to discard from the cell state.
- **Input gate**: What new information to add.
- **Output gate**: What part of the cell state to output as the hidden state.

The cell state allows gradients to flow through time with minimal modification — solving the long-range dependency problem.

**GRU** (Gated Recurrent Unit): A simplified LSTM with two gates (reset and update). Fewer parameters, often similar performance. Faster to train.

**Today**: For most sequence tasks, Transformers have replaced RNNs/LSTMs — they process all positions in parallel and handle long-range dependencies via attention.

---

### MEDIUM PRIORITY

---

Q8. What is dropout, and how does it prevent overfitting?

A8.
Dropout randomly "turns off" (sets to zero) a fraction of neurons during each training step. Typically 20-50% of neurons are dropped.

**Why it works**: Each training step uses a different subset of the network — effectively training an ensemble of sub-networks. The model can't rely on any single neuron or co-adaptation between neurons. Each neuron must be independently useful.

**At inference time**: All neurons are active, but weights are scaled by (1 - dropout_rate) to compensate for the additional activations. Some implementations use "inverted dropout" — scale during training instead.

**Where to apply**: After fully connected layers (most common). In CNNs, spatial dropout (drop entire feature maps) is more effective than per-neuron dropout. In Transformers, dropout is applied after attention and feed-forward layers.

**Trade-off**: Too much dropout → underfitting (model never has enough capacity to learn). Too little → overfitting. 0.1-0.3 for modern architectures. Not usually needed when using strong data augmentation or batch normalization.

---

Q9. What is transfer learning in deep learning? How does fine-tuning work?

A9.
Transfer learning takes a model pretrained on a large dataset (ImageNet with 14M images, or web-scale text) and adapts it to a new task with limited data.

**How fine-tuning works**:
1. Start with a pretrained model (ResNet trained on ImageNet).
2. Replace the final classification layer with a new one matching your number of classes.
3. Freeze early layers (they've learned generic features — edges, textures).
4. Train only the new layers on your small dataset.
5. Optionally, unfreeze some later layers and fine-tune with a very low learning rate.

**Why it works**: Lower layers learn universal features (edges in images, grammar in text). These transfer across tasks. You only need to learn the task-specific high-level features, which requires far fewer samples.

**Example**: A dermatology classifier. Rather than collecting millions of skin images, start with ResNet pretrained on ImageNet, fine-tune on 10K labeled skin images. The low-level features (texture, color patterns) transfer directly.

**In NLP**: BERT/GPT pretrained on massive text → fine-tuned on your specific task (sentiment analysis, named entity recognition) with a few thousand labeled examples. This transformed NLP.

---

Q10. What are GANs (Generative Adversarial Networks)? Explain the generator-discriminator setup.

A10.
A GAN consists of two neural networks competing against each other:

**Generator (G)**: Takes random noise as input and generates fake data (images, text). Its goal: produce outputs indistinguishable from real data.

**Discriminator (D)**: Takes real data and generated data and classifies them as real or fake. Its goal: correctly distinguish real from fake.

**Training**: D improves at detecting fakes → G improves at fooling D → D improves further → G improves further. At equilibrium, G produces data so realistic that D can't tell the difference (outputs 0.5 for everything).

**Applications**: Image generation (StyleGAN — photorealistic faces), image-to-image translation (pix2pix — sketches to photos), super-resolution (ESRGAN), data augmentation for small datasets.

**Challenges**: Mode collapse (generator produces only a few types of outputs), training instability (the balance between G and D is delicate), no clear convergence metric (loss doesn't reliably indicate quality).

**Modern alternatives**: Diffusion models (DALL-E, Stable Diffusion) have largely replaced GANs for image generation — more stable training, better diversity, and quality.

---

Q11. What is the difference between L1 loss and L2 loss for regression?

A11.
**L1 loss** (Mean Absolute Error): `Σ|y_pred - y_true|`. Robust to outliers — treats a prediction off by 100 the same as 100 predictions off by 1. Gradients are constant (±1), which can cause instability near the minimum.

**L2 loss** (Mean Squared Error): `Σ(y_pred - y_true)²`. Penalizes large errors more heavily — an error of 10 costs 100x more than an error of 1. Smooth gradient (∝ error magnitude). Sensitive to outliers — one extreme value can dominate the loss.

**When to use L1**: Data has outliers you don't want to dominate the model. Median regression — L1 minimizes the median absolute error.

**When to use L2**: Errors should be penalized proportionally. Clean data without extreme outliers. Most common default.

**Huber loss**: Combines both — L2 for small errors (smooth), L1 for large errors (robust). Best of both worlds. Has a hyperparameter δ that defines the transition point. Used increasingly as the default for regression tasks.

---

Q12. What is learning rate scheduling? What strategies exist?

A12.
Learning rate scheduling adjusts the learning rate during training — usually decreasing it as training progresses. Early training uses large steps (explore broadly), later training uses small steps (converge precisely).

**Strategies**:
- **Step decay**: Reduce LR by a factor every N epochs. `lr = initial_lr * 0.1^(epoch // 30)`. Simple, widely used.
- **Cosine annealing**: LR follows a cosine curve from initial value to near-zero. Smooth decay. Used in many modern architectures.
- **Warmup + decay**: Start with a very low LR, linearly increase for a few thousand steps (warmup), then decay. Critical for Transformers — prevents early instability.
- **ReduceOnPlateau**: Monitor validation loss; reduce LR when it stops improving. Adaptive, but reactive (reduces after the problem, not before).
- **One-cycle policy**: LR increases from low to high, then decreases from high to very low. Often achieves super-convergence — faster training.

**Modern optimizers** (Adam, AdamW) have per-parameter adaptive learning rates, reducing the need for aggressive scheduling. But even with Adam, cosine annealing or warmup + decay typically helps.

---

Q13. What are residual connections (skip connections), and why are they important?

A13.
A residual (skip) connection adds the input of a block directly to its output: `output = F(x) + x`. Introduced by ResNet.

**Why they're important**:
- **Gradient flow**: Gradients can bypass layers through the skip connection, flowing directly to earlier layers. This solves vanishing gradients in very deep networks.
- **Identity mapping**: If F(x) = 0, the block just passes the input through. The network can learn "do nothing" at a layer, making it easy to add depth without hurting performance.
- **Optimization landscape**: Skip connections make the loss surface smoother — easier for gradient descent to navigate.

**Before ResNet**: Networks deeper than ~20 layers actually performed worse than shallower ones — not from overfitting, but from optimization difficulty. ResNet enabled 152+ layer networks.

**Ubiquitous now**: Skip connections are in virtually every modern architecture — ResNet (CNN), Transformer (attention + skip, FFN + skip), U-Net (segmentation), DenseNet (dense connections).

---

Q14. What is data parallelism vs. model parallelism in distributed training?

A14.
When a model or dataset is too large for one GPU, you distribute training across multiple GPUs.

**Data parallelism**: The model is copied to each GPU. Each GPU processes a different mini-batch of data. Gradients are averaged across GPUs, and weights are synchronized. Most common approach. Works when the model fits in one GPU's memory.

**Model parallelism**: The model itself is split across GPUs — different layers on different GPUs. GPU 1 runs layers 1-20, GPU 2 runs layers 21-40. Necessary when the model doesn't fit in one GPU's memory (large language models). Communication overhead between GPUs can be a bottleneck.

**Pipeline parallelism**: A form of model parallelism where different micro-batches are processed by different GPU stages simultaneously — like a factory assembly line. Reduces idle time (GPU bubbles).

**Tensor parallelism**: Splits individual layers (matrices) across GPUs. A single matrix multiplication is distributed. Used in very large models (GPT-3, Megatron-LM).

Modern training of large models uses a combination: tensor parallelism within a node, pipeline parallelism across nodes, and data parallelism across groups of nodes.

---

### LOW PRIORITY

---

Q15. What is weight initialization? Why does it matter?

A15.
Weights are initialized before training begins. Bad initialization can cause vanishing/exploding activations from the very first forward pass.

**Zero initialization**: All neurons learn the same thing (symmetry problem). Never use for hidden layers.

**Random initialization**: Small random values. But if too small, activations shrink through layers; too large, they explode.

**Xavier/Glorot initialization**: Designed for sigmoid/tanh. Variance = 2/(fan_in + fan_out). Keeps variance stable across layers.

**He/Kaiming initialization**: Designed for ReLU. Variance = 2/fan_in. Accounts for ReLU killing half the neurons (zero for negative inputs).

In practice, frameworks handle this automatically — PyTorch uses Kaiming for ReLU layers by default. But understanding it explains why early deep learning papers struggled with training deep networks before proper initialization was developed.

---

Q16. What is an attention mechanism before Transformers (e.g., in seq2seq models)?

A16.
In traditional seq2seq models (encoder-decoder RNNs for translation), the encoder compresses the entire input sequence into a single fixed-size vector. For long sentences, this bottleneck loses information.

**Attention** (Bahdanau, 2014) allows the decoder to look at all encoder hidden states at each decoding step, focusing on the most relevant ones.

**How it works**: At each decoder step, compute a relevance score between the current decoder state and every encoder hidden state (dot product or learned function). Softmax the scores to get attention weights. Compute a weighted sum of encoder states — the "context vector." Pass it to the decoder for this step.

**Effect**: "To translate this word, pay more attention to these input words." For translating "chat" to "cat," the decoder attends highly to the French word "chat" and ignores padding.

This was the foundation for Transformers — self-attention generalizes this idea to allow every position to attend to every other position, and removes the recurrence entirely.

---

Q17. What are autoencoders, and what are their common architectures?

A17.
An autoencoder compresses input through a bottleneck (encoder) and reconstructs it (decoder). The bottleneck forces the network to learn a compact representation.

**Vanilla autoencoder**: Encoder → latent space → Decoder. Minimize reconstruction loss (MSE). The latent space captures the most important features.

**Variational autoencoder (VAE)**: The latent space is a probability distribution (Gaussian), not a fixed vector. Enables generating new samples by sampling from the distribution. Loss = reconstruction loss + KL divergence (keeps the latent distribution close to a standard normal).

**Denoising autoencoder**: Input is corrupted with noise; the model learns to reconstruct the clean version. Learns more robust, useful representations.

**Sparse autoencoder**: Adds a sparsity penalty — only a few neurons in the latent space should be active per input. Forces the network to learn distinct features.

**Use cases**: Dimensionality reduction (alternative to PCA — handles non-linearities), anomaly detection (high reconstruction error = anomaly), pretraining (learn representations before supervised fine-tuning), generation (VAE).

---

Q18. What is the difference between epoch, batch, and iteration?

A18.
**Epoch**: One complete pass through the entire training dataset. If you have 10,000 samples and train for 100 epochs, every sample is seen 100 times.

**Batch (mini-batch)**: A subset of the training data processed in one forward/backward pass. If batch size = 32 and you have 10,000 samples, one epoch has 10,000/32 ≈ 313 batches.

**Iteration**: One forward + backward pass on one batch. One epoch = 313 iterations (in the example above).

**Why batch size matters**: Small batches (32) → more noise in gradient estimates → better generalization but slower convergence. Large batches (1024+) → smoother gradients → faster training per epoch but may converge to sharp minima (worse generalization). The optimal batch size depends on the task, architecture, and available GPU memory.

**Common confusion**: "Steps" and "iterations" are usually synonymous. "Epoch" is a function of dataset size and batch size. Training for 100K steps with batch size 64 on 640K samples = 10 epochs.

---

Q19. What is mixed precision training?

A19.
Mixed precision training uses both 16-bit (FP16 or BF16) and 32-bit (FP32) floating point during training. Most computations happen in 16-bit; critical operations (loss computation, gradient accumulation) stay in 32-bit.

**Benefits**: 2x faster training (GPUs process 16-bit operations in half the time), 2x less memory (fit larger models or batches), minimal accuracy loss.

**How it works**: Forward pass in FP16 (faster). Compute loss in FP32 (precision). Backward pass in FP16 (faster). Accumulate gradients and update master weights in FP32 (accuracy).

**Loss scaling**: FP16 has a narrow range — small gradients underflow to zero. Loss scaling multiplies the loss by a large factor before backprop, then divides the gradients after. Dynamic loss scaling adjusts the factor automatically.

**BF16 (Brain Float 16)**: Same exponent range as FP32 with reduced precision. Doesn't need loss scaling. Supported on newer GPUs (A100, H100) and TPUs. Increasingly preferred over FP16.

In practice, mixed precision is essentially free performance — enable it with one line in PyTorch: `torch.cuda.amp.autocast()`.

==============================
ADDITIONAL MISSING Q&A (GAP FILL)
==============================

### CRITICAL

---

Q20. Compare SGD, Adam, RMSProp, and AdamW optimizers. When would you use each?

A20.
**SGD (Stochastic Gradient Descent)**: The simplest — update weights proportional to the gradient. `w = w - lr * gradient`. With momentum, it accumulates a velocity term that smooths updates and helps escape saddle points.

**RMSProp**: Adapts learning rate per parameter based on the magnitude of recent gradients. Divides by a running average of squared gradients. Parameters with large gradients get smaller learning rates. Good for RNNs.

**Adam (Adaptive Moment Estimation)**: Combines momentum (1st moment — mean of gradients) and RMSProp (2nd moment — variance of gradients). Adaptive learning rates + momentum. Fast convergence, robust to hyperparameter choices. The "default" optimizer.

**AdamW**: Adam with decoupled weight decay. Regular Adam applies weight decay through L2 regularization in the gradient — but this interacts poorly with adaptive learning rates. AdamW applies weight decay directly to the weights, independent of the gradient. Better generalization, especially for large models.

**When to use**:
- **SGD + momentum**: Often generalizes better for computer vision (ResNets, ConvNets). Requires careful learning rate scheduling. If you're willing to tune, SGD can outperform Adam on final accuracy.
- **Adam/AdamW**: Default for transformers and NLP. Faster convergence, less sensitive to learning rate. AdamW is preferred for any model with weight decay.
- **RMSProp**: Mostly superseded by Adam, but still used in specific cases (RL with A3C).

Typical learning rates: SGD ~0.01-0.1, Adam/AdamW ~1e-4 to 3e-4 for transformers, 1e-3 for smaller models.

---

Q21. What is cross-entropy loss and why is it used for classification?

A21.
Cross-entropy measures the difference between two probability distributions — the true labels and the model's predicted probabilities.

**Binary cross-entropy**: `L = -[y·log(p) + (1-y)·log(1-p)]` where y is the true label (0 or 1) and p is the predicted probability.

**Categorical cross-entropy**: `L = -Σ yi·log(pi)` across all classes. Only the true class contributes (since yi=0 for wrong classes).

**Why cross-entropy over MSE for classification?**
1. **Gradient behavior**: With sigmoid + MSE, gradients vanish when predictions are confident but wrong (sigmoid saturates → flat gradient). Cross-entropy's gradient is `(p - y)` — proportional to the error. Wrong predictions get strong gradients, enabling faster correction.
2. **Probabilistic interpretation**: Cross-entropy is the negative log-likelihood under a Bernoulli/categorical model. Minimizing it is equivalent to maximum likelihood estimation.
3. **Information theory**: It measures the extra bits needed to encode data from distribution y using model distribution p.

**Practical note**: In PyTorch, `nn.CrossEntropyLoss` combines `log_softmax` + `NLLLoss` and expects raw logits, not probabilities. Passing sigmoid/softmax outputs is a common bug.

---

Q22. How does the softmax function work, and what's the temperature parameter?

A22.
Softmax converts a vector of raw scores (logits) into a probability distribution: `softmax(zi) = exp(zi) / Σ exp(zj)`.

Each output is between 0 and 1, and all outputs sum to 1. It amplifies differences between logits — the largest logit gets the most probability mass.

**Numerical stability**: Computing `exp(1000)` overflows. Solution: subtract the max logit before exponentiating. `softmax(z) = exp(z - max(z)) / Σ exp(z - max(z))`. Mathematically equivalent, numerically stable.

**Temperature**: Divide logits by temperature T before softmax: `softmax(zi/T)`.
- **T < 1** (low temperature): Sharpens the distribution — model becomes more confident, picks the top class more aggressively. Used in inference for more deterministic outputs.
- **T > 1** (high temperature): Flattens the distribution — more randomness, more exploration. Used in text generation for diversity.
- **T → 0**: Approaches argmax (one-hot). **T → ∞**: Approaches uniform distribution.

**Knowledge distillation** uses temperature: The teacher model produces soft labels at high temperature (T=4-20), revealing which classes the model considers similar. The student learns from these soft targets, capturing richer information than hard labels.

---

Q23. What is gradient clipping, and why is it necessary?

A23.
Gradient clipping caps the magnitude of gradients during backpropagation to prevent exploding gradients.

**Two types**:
1. **Clip by value**: Clamp each gradient component to [-threshold, threshold]. Simple but can change gradient direction.
2. **Clip by norm** (preferred): If the global gradient norm exceeds a threshold, scale all gradients down proportionally: `if ||g|| > threshold: g = g * threshold / ||g||`. Preserves direction, just limits magnitude.

```python
# PyTorch
torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
```

**Why it's necessary**: In deep networks (especially RNNs and transformers), gradients can grow exponentially through long sequences. Without clipping, weights get massive updates, and training diverges (loss → NaN).

**Typical values**: `max_norm=1.0` is a common default. For transformers, 0.5-1.0. For RNNs, 1.0-5.0.

**Diagnosing**: If training loss spikes randomly and recovers, or goes to NaN, check gradient norms. Log `grad_norm` during training — if it occasionally jumps to 100x or 1000x the average, you need clipping.

Gradient clipping is a necessity for training transformers (used in every major LLM training recipe) and is the primary defense against exploding gradients (vanishing gradients are handled by architecture choices like residual connections).

---

### IMPORTANT

---

Q24. What are 1×1 convolutions, and why are they useful?

A24.
A 1×1 convolution applies a filter of size 1×1 across all channels. It doesn't look at spatial neighborhoods — it performs a linear combination across channels at each spatial position.

**Uses**:
1. **Dimensionality reduction**: Reduce channel count without changing spatial dimensions. An input of 256 channels → 64 channels through 64 1×1 filters. Massively reduces computation before expensive 3×3 or 5×5 convolutions. This is the key idea in **Inception/GoogLeNet** bottleneck layers.

2. **Adding non-linearity**: 1×1 conv + ReLU adds a non-linear transformation per pixel across channels. Increases network depth cheaply.

3. **Channel mixing**: Cross-channel feature interaction without spatial mixing. Used in **ResNet bottleneck blocks**: 1×1 (reduce) → 3×3 (spatial) → 1×1 (expand).

4. **Pointwise convolution**: In depthwise separable convolutions (MobileNet), the 1×1 conv is the "pointwise" step that mixes features from the per-channel depthwise step.

**Cost**: For an input of H×W×C_in going to H×W×C_out: computation is H × W × C_in × C_out — linear in both input and output channels. Much cheaper than a 3×3 (which multiplies by 9).

---

Q25. What is the U-Net architecture, and why is it effective for segmentation?

A25.
U-Net is an encoder-decoder architecture with skip connections, designed for semantic segmentation (pixel-level classification).

**Architecture** (U-shaped):
- **Encoder (contracting path)**: Standard conv blocks + max pooling. Captures "what" features are present but loses "where" — spatial resolution decreases.
- **Bottleneck**: Lowest resolution, highest-level features.
- **Decoder (expanding path)**: Upsampling (transposed convolutions) + conv blocks. Restores spatial resolution.
- **Skip connections**: Concatenate feature maps from encoder to corresponding decoder level. This is the key innovation — they provide precise localization information that was lost during downsampling.

**Why it works**: The encoder captures semantic context (what's in the image), the decoder recovers spatial detail (where it is), and skip connections bridge the two. Without skip connections, the decoder struggles to produce precise boundaries.

**Variants**: U-Net++ (nested skip connections), Attention U-Net (attention gates on skip connections), 3D U-Net (for volumetric medical imaging), V-Net (3D with residual connections).

**Impact**: Dominant in medical image segmentation (cell detection, organ segmentation, tumor detection) and widely used in satellite imagery, autonomous driving, and any dense prediction task.

---

Q26. What are depthwise separable convolutions?

A26.
Depthwise separable convolutions split a standard convolution into two steps, dramatically reducing computation:

**Standard conv** (3×3, C_in→C_out): One filter of size 3×3×C_in, applied C_out times. Cost: 3 × 3 × C_in × C_out × H × W.

**Depthwise separable**:
1. **Depthwise conv**: Apply one 3×3 filter per input channel independently. No cross-channel mixing. Cost: 3 × 3 × C_in × H × W.
2. **Pointwise conv**: Apply 1×1 conv across channels. Cost: C_in × C_out × H × W.
Total cost: (3² + C_out) × C_in × H × W ≈ **8-9× cheaper** than standard conv (for typical C_out=256).

**Used in**: MobileNet (designed for mobile/edge), Xception (extreme Inception), EfficientNet. Any architecture targeting efficiency.

**Trade-off**: Slightly lower representational power (the spatial and channel processing are decoupled), but the efficiency gain is enormous. In practice, the accuracy difference is small, especially when the model has enough capacity.

---

Q27. What is teacher forcing in sequence models?

A27.
Teacher forcing is a training technique for sequence-to-sequence models where you feed the ground truth previous token as input to the decoder at each step, rather than the model's own predicted token.

**Without teacher forcing** (autoregressive): At step t, the decoder uses its own prediction from step t-1 as input. If it makes a mistake early, subsequent predictions drift further off — exposure bias.

**With teacher forcing**: At step t, the decoder always gets the correct token from step t-1. Training is faster and more stable because the model never trains on its own mistakes.

**The problem — exposure bias**: During inference, the model must use its own predictions (no ground truth available). It was never trained on its own errors, so mistakes compound. The model has never seen a "wrong" input during training.

**Solutions**:
- **Scheduled sampling**: Start with 100% teacher forcing, gradually decrease to 0% during training. The model transitions from ground truth to its own predictions.
- **Sequence-level training**: Optimize entire sequences with BLEU/ROUGE rewards using reinforcement learning (REINFORCE algorithm).
- **Modern transformers**: Use teacher forcing during training (parallel — all ground truth tokens available) and autoregressive generation at inference. This works well enough that scheduled sampling is rarely used with transformers.

---

Q28. What is label smoothing, and how does it help?

A28.
Label smoothing replaces hard targets (one-hot: [0, 0, 1, 0]) with soft targets ([0.033, 0.033, 0.9, 0.033]). Instead of training the model to be 100% confident, you target (1 - ε) for the true class and distribute ε among other classes.

```python
# PyTorch
criterion = nn.CrossEntropyLoss(label_smoothing=0.1)
```

**Why it helps**:
1. **Prevents overconfidence**: Hard labels encourage the model to push logits to extreme values (infinity for correct class). This leads to sharp, overconfident predictions that generalize poorly.
2. **Better calibration**: Softened predictions are closer to true uncertainty.
3. **Regularization**: Acts as an implicit regularizer — prevents the model from becoming too sure about training examples.
4. **Improved generalization**: Consistently shown to improve test accuracy by 0.2-0.5% in image classification and translation tasks.

**Typical ε**: 0.1 is the standard choice (used in the original Transformer paper, Inception v3, and most modern architectures).

**Trade-off**: If you need the model's confidence scores for downstream decisions (like rejection at low confidence), label smoothing makes calibration slightly harder. But overall, it's a net positive for most classification tasks.

---

### GOOD-TO-HAVE

---

Q29. What are diffusion models at a high level?

A29.
Diffusion models generate data by learning to reverse a gradual noising process.

**Forward process** (fixed, not learned): Start with a clean image, add Gaussian noise step by step over T timesteps until it's pure noise. Each step adds a small amount of noise: `x_t = √(α_t) * x_{t-1} + √(1-α_t) * ε`.

**Reverse process** (learned): A neural network learns to denoise — given `x_t` (noisy image) and timestep t, predict the noise that was added. Starting from pure noise, iteratively denoise to generate a clean image.

**Training**: Sample a clean image, add noise at a random timestep, train the network (usually a U-Net) to predict the added noise. Loss is simple MSE between predicted and actual noise.

**Inference**: Start from random noise → denoise step by step → image emerges. This is slow (many steps), which led to:
- **DDIM**: Deterministic sampling, fewer steps
- **Latent diffusion** (Stable Diffusion): Run diffusion in a compressed latent space (encoded by a VAE), making it much faster
- **Classifier-free guidance**: Condition generation on text prompts; strengthening guidance produces more prompt-adherent images

**Why they won over GANs**: More stable training (no min-max game), better mode coverage (don't collapse to a subset of outputs), and higher quality. Dall-E 2, Stable Diffusion, Midjourney, and Sora all use diffusion models.

Trade-off: Slower generation than GANs (iterative denoising vs single forward pass), but quality and diversity are superior.