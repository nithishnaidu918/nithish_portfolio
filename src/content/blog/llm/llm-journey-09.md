---
title: "Reproducing GPT-2 (124M)"
description: "Learning GPT-2 implementation, pretrained weight loading, training optimization, mixed precision, distributed training, and evaluation using PyTorch."
category: "LLM"
tags: ["LLM", "GPT-2", "PyTorch", "Transformers", "Deep Learning", "Model Training"]
---

# LLM Journey #9 — Reproducing GPT-2 (124M)

## 1. What Is GPT-2?

In the previous journey, I explored how to build a GPT tokenizer. Now, the focus is on reproducing GPT-2, understanding its implementation, training process, performance optimizations, and evaluation.

GPT-2 is a decoder-only Transformer language model that learns to predict the next token.

For example:

```text
Input:  The cat sat on the
Target: mat
```

The model produces scores for possible next tokens. During training, it learns to assign higher probabilities to the correct next token.

GPT-2 Small has approximately 124 million parameters.

| Configuration | Value |
|---|---:|
| Transformer blocks | 12 |
| Attention heads per block | 12 |
| Embedding dimension | 768 |
| Context length | 1,024 tokens |
| Vocabulary size | 50,257 |
| Parameters | Approximately 124M |

These values describe the original GPT-2 Small architecture.

## 2. Implementing GPT-2 with `nn.Module`

PyTorch's `nn.Module` is the base class used to build neural network models and custom layers.

A GPT-2 implementation contains:

- Token embeddings
- Positional embeddings
- Masked multi-head self-attention
- Feed-forward networks
- Residual connections
- Layer normalization
- A final language-model output head

A simplified model skeleton looks like this:

```python
import torch
import torch.nn as nn

class GPT(nn.Module):
    def __init__(self, config):
        super().__init__()

        # Create token embeddings, position embeddings,
        # Transformer blocks, final LayerNorm, and output head.

    def forward(self, idx):
        # Convert token IDs into embeddings.
        # Process them through Transformer blocks.
        # Return vocabulary logits.
        pass
```

The complete implementation must define the layers and their forward computations. The important objective is to reproduce the expected GPT-2 architecture and parameter shapes so that pretrained weights can be loaded correctly.

## 3. Loading Pretrained Hugging Face Weights

Training a model from scratch requires learning all its parameters from data. A pretrained checkpoint already contains learned parameters.

The process is conceptually:

```text
Pretrained GPT-2 checkpoint
          ↓
    Learned tensors
          ↓
  Your GPT-2 implementation
          ↓
   Generate text or evaluate
```

A checkpoint contains tensors for embeddings, attention projections, feed-forward layers, normalization parameters, and other components.

The parameter names and shapes must match the implementation. Some weights may also need to be mapped or transposed depending on how the source and destination layers store them.

**Important distinction:** Loading pretrained weights is not training. It initializes your implementation with learned parameters so you can verify that it behaves like the pretrained model.

## 4. The Forward Pass and Tensor Shapes

The forward pass converts input token IDs into logits.

```python
logits = model(idx)
```

If the input has shape `(B, T)`, the logits have shape `(B, T, V)`.

- `B` — batch size
- `T` — sequence length
- `V` — vocabulary size

For GPT-2:

```text
Input token IDs: (B, T)
Output logits:   (B, T, 50257)
```

Every position produces a score for every vocabulary token.

The model processes token embeddings and positional embeddings through its Transformer blocks, applies the final normalization, and projects the hidden representations into vocabulary scores.

## 5. Tokenization and Text Generation

GPT-2 uses its own tokenizer. A compatible tokenizer is essential because the pretrained model expects the token IDs associated with its vocabulary.

```python
import tiktoken

enc = tiktoken.get_encoding("gpt2")

text = "Hello, I'm a language model,"
tokens = enc.encode(text)

print(tokens)
print(enc.decode(tokens))
```

The encoded IDs become the initial prefix supplied to the model.

Using an unrelated tokenizer changes the meaning of the IDs and is not a drop-in replacement for GPT-2's tokenizer.

## 6. Autoregressive Sampling

GPT-2 generates text one token at a time. After predicting a token, it appends that token to the sequence and predicts the next one.

```python
import torch
import torch.nn.functional as F

@torch.no_grad()
def generate(model, idx, max_new_tokens):
    model.eval()

    for _ in range(max_new_tokens):
        logits = model(idx)

        # Select predictions at the last position.
        next_token_logits = logits[:, -1, :]

        # Convert scores into probabilities.
        probs = F.softmax(next_token_logits, dim=-1)

        # Sample the next token.
        next_token = torch.multinomial(probs, num_samples=1)

        # Append it to the sequence.
        idx = torch.cat((idx, next_token), dim=1)

    return idx
```

This is the basic sampling loop. A full GPT-2 implementation may also crop the input to the maximum context length and apply other generation settings.

Important operations:

- `logits[:, -1, :]` selects the final position's predictions.
- `softmax()` converts scores into probabilities.
- `torch.multinomial()` samples a token.
- `torch.cat()` appends the new token.

## 7. Detecting the Available Device

Model parameters and input tensors must be on compatible devices.

```python
import torch

if torch.cuda.is_available():
    device = "cuda"
elif (
    hasattr(torch.backends, "mps")
    and torch.backends.mps.is_available()
):
    device = "mps"
else:
    device = "cpu"

model = model.to(device)
tokens = tokens.to(device)
```

- **CPU:** Useful for testing, but large models may run slowly.
- **CUDA:** Runs on a compatible NVIDIA GPU.
- **MPS:** Uses Apple's Metal backend on supported devices.

For large training jobs, GPU memory and hardware support strongly influence what configuration is practical.

## 8. Next-Token Training Data

During training, each input sequence is paired with a target sequence shifted one token forward.

For example:

```text
Input:  [The, cat, sat, on]
Target: [cat, sat, on, mat]
```

For a batch of sequences:

```text
Input tokens: (B, T)
Targets:      (B, T)
Logits:       (B, T, V)
```

The model predicts the next token at every position. This allows many next-token predictions to be trained in parallel during one forward pass.

## 9. Cross-Entropy Loss

Cross-entropy measures how well the predicted logits match the correct target token IDs.

```python
import torch.nn.functional as F

logits = model(x)

loss = F.cross_entropy(
    logits.reshape(-1, logits.size(-1)),
    y.reshape(-1),
)
```

Why reshape?

Suppose the batch size is 4, the sequence length is 8, and the vocabulary contains 50,257 tokens.

```text
Logits:  (4, 8, 50257)
Targets: (4, 8)

After reshaping:

Logits:  (32, 50257)
Targets: (32,)
```

Cross-entropy receives one row of vocabulary scores and one correct target ID for each token position.

A lower loss generally means the model assigns greater probability to the correct next tokens.

## 10. The Optimization Loop

The standard PyTorch training loop follows five steps:

```python
optimizer.zero_grad(set_to_none=True)

logits = model(x)

loss = F.cross_entropy(
    logits.reshape(-1, logits.size(-1)),
    y.reshape(-1),
)

loss.backward()
optimizer.step()
```

1. Clear previous gradients.
2. Run the forward pass.
3. Calculate the loss.
4. Compute gradients using backpropagation.
5. Update parameters using the optimizer.

### Why overfit a single batch first?

Before training on a large dataset, Karpathy demonstrates overfitting a small, fixed batch.

If the model cannot drive the loss down on repeated examples, possible problems include incorrect target alignment, bugs in the forward pass, disconnected gradients, or an optimizer issue.

This is a debugging technique, not the final training strategy.

## 11. DataLoaderLite

After validating the model on a small batch, training needs a stream of batches from a larger tokenized dataset.

A lightweight loader can select consecutive chunks and create shifted input-target pairs.

```python
def get_batch(data, batch_size, block_size, device):
    ix = torch.randint(
        len(data) - block_size - 1,
        (batch_size,),
    )

    x = torch.stack([
        data[i:i + block_size]
        for i in ix
    ])

    y = torch.stack([
        data[i + 1:i + block_size + 1]
        for i in ix
    ])

    return x.to(device), y.to(device)
```

This example assumes `data` is a one-dimensional tensor of token IDs. It illustrates batch construction; a production loader also needs to handle dataset boundaries, data splitting, and efficient iteration.

## 12. Weight Tying: `wte` and `lm_head`

GPT-2 shares the token embedding matrix with the language-model output head. This is called **weight tying**.

```python
self.transformer.wte = nn.Embedding(
    vocab_size,
    n_embd,
)

self.lm_head = nn.Linear(
    n_embd,
    vocab_size,
    bias=False,
)

self.lm_head.weight = self.transformer.wte.weight
```

The embedding matrix maps token IDs to vectors. The output head maps hidden representations back to vocabulary scores.

Sharing the same parameter matrix reduces the number of independent parameters and ties input token representations to output prediction weights.

Because this parameter is shared, it should not be treated as two independent weight matrices.

## 13. Model Initialization

Random initialization gives trainable parameters their starting values. Karpathy discusses initialization with a standard deviation around `0.02` for relevant weights.

```python
torch.nn.init.normal_(
    weight,
    mean=0.0,
    std=0.02,
)
```

The exact initialization should follow the model implementation and parameter type.

Very large initial weights can produce extreme activations. Initializing every parameter to zero can also prevent neurons from learning distinct features.

Karpathy additionally discusses scaling certain residual-branch projection weights to control the accumulation of residual contributions in a deep Transformer.

Initialization affects the starting point of optimization; it does not replace training.

## 14. Timing Before Optimization

Before improving performance, measure the baseline.

```python
import time
import torch

torch.cuda.synchronize()
start = time.time()

# Run training steps here.

torch.cuda.synchronize()
elapsed = time.time() - start

print(f"Elapsed time: {elapsed:.2f} seconds")
```

CUDA operations are asynchronous, so synchronizing before and after the timed section gives a more meaningful measurement of GPU execution time.

The actual speedup depends on the GPU, batch size, sequence length, PyTorch version, and implementation.

## 15. Tensor Cores and TF32

Modern NVIDIA GPUs contain Tensor Cores that accelerate matrix operations used heavily by Transformers.

PyTorch supports a setting that can permit faster implementations of eligible float32 matrix multiplications on supported hardware:

```python
torch.set_float32_matmul_precision("high")
```

This is a performance-versus-precision setting. It does not guarantee that every operation uses TF32 or becomes faster on every device.

## 16. Mixed Precision: FP16 and BF16

Mixed precision allows eligible operations to use lower-precision formats to improve speed and reduce memory usage.

| Format | Important property |
|---|---|
| FP16 | 16-bit floating point with a more limited exponent range |
| BF16 | 16-bit floating point with an exponent range similar to float32 |

Karpathy uses autocast to enable lower-precision computation for eligible operations.

```python
with torch.autocast(
    device_type="cuda",
    dtype=torch.bfloat16,
):
    logits = model(x)

    loss = F.cross_entropy(
        logits.reshape(-1, logits.size(-1)),
        y.reshape(-1),
    )
```

This example assumes supported CUDA hardware and BF16-compatible operations.

BF16's wider exponent range can make it easier to use for large-scale training than FP16. FP16 training may require gradient scaling to reduce gradient underflow.

Mixed precision can improve training throughput, but support and numerical behavior depend on the hardware and operations.

## 17. `torch.compile` and Kernel Fusion

PyTorch normally dispatches a sequence of operations through its execution system. Some workloads spend significant time on Python overhead and many small GPU kernel launches.

```python
model = torch.compile(model)
```

`torch.compile` can optimize eligible computations and reduce overhead through compilation and kernel fusion.

Kernel fusion combines compatible operations so that intermediate results may not need to be written to memory between separate kernels.

Compilation also introduces overhead, so its benefits depend on how many training iterations are performed and which operations can be optimized.

## 18. FlashAttention

Standard self-attention computes attention scores, applies masking and softmax, and combines the resulting weights with value vectors.

A straightforward implementation may materialize an attention matrix shaped like:

```text
(B, H, T, T)
```

where `B` is batch size, `H` is the number of attention heads, and `T` is sequence length.

The quadratic sequence dimensions can make attention memory-intensive.

**FlashAttention** computes exact attention using a more memory-efficient algorithm that processes calculations in tiles, reducing expensive memory traffic.

It changes how attention is computed, rather than replacing attention with an approximation. It is especially valuable for long sequences and supported GPU workloads.

## 19. Why Pad the Vocabulary to 50,304?

GPT-2's tokenizer vocabulary contains 50,257 tokens. Karpathy discusses using a padded model output dimension of 50,304 in the reproduction.

Some tensor dimensions align better with hardware-friendly multiples, potentially improving matrix-operation efficiency.

The extra output dimensions are padding, not new meaningful tokenizer entries. The tokenizer still produces IDs from the original vocabulary.

This optimization changes the model's output dimension for computational reasons; it does not change the tokenizer's vocabulary.

## 20. AdamW and Fused AdamW

AdamW is an optimizer that uses moving averages of gradients and squared gradients while applying weight decay separately from the gradient-based update.

```python
optimizer = torch.optim.AdamW(
    model.parameters(),
    lr=learning_rate,
    weight_decay=0.1,
    betas=(0.9, 0.95),
    eps=1e-8,
)
```

Important settings:

- `lr`: learning rate.
- `betas`: control the moving averages of gradients and squared gradients.
- `eps`: numerical-stability constant.
- `weight_decay`: controls parameter shrinkage.

When supported by the installed PyTorch version and device, a fused implementation can reduce optimizer kernel-launch overhead:

```python
optimizer = torch.optim.AdamW(
    model.parameters(),
    lr=learning_rate,
    weight_decay=0.1,
    betas=(0.9, 0.95),
    fused=True,
)
```

Fused AdamW changes execution efficiency, not the fundamental optimization objective.

## 21. Gradient Clipping

Occasionally, gradients can become unusually large and cause unstable parameter updates.

```python
optimizer.zero_grad(set_to_none=True)

logits = model(x)

loss = F.cross_entropy(
    logits.reshape(-1, logits.size(-1)),
    y.reshape(-1),
)

loss.backward()

torch.nn.utils.clip_grad_norm_(
    model.parameters(),
    max_norm=1.0,
)

optimizer.step()
```

Gradient clipping limits the global gradient norm when it exceeds the configured threshold. It helps reduce the risk of unusually large updates, although it cannot prevent every training problem.

## 22. Learning-Rate Warmup and Cosine Decay

The learning rate determines the scale of optimizer updates.

Karpathy discusses a schedule with two main phases:

- **Warmup:** Gradually increase the learning rate at the beginning.
- **Cosine decay:** Gradually reduce it toward a minimum during later training.

A simplified cosine-decay calculation is:

```python
import math

def cosine_lr(step, warmup_steps, max_steps, max_lr, min_lr):
    if step < warmup_steps:
        return max_lr * (step + 1) / warmup_steps

    progress = (step - warmup_steps) / (
        max_steps - warmup_steps
    )
    progress = min(max(progress, 0.0), 1.0)

    cosine = 0.5 * (1.0 + math.cos(math.pi * progress))

    return min_lr + cosine * (max_lr - min_lr)
```

This is a simplified schedule. A real training run should configure the warmup duration, total steps, maximum learning rate, and minimum learning rate appropriately.

Warmup reduces the risk of overly aggressive early updates. Cosine decay gradually lowers the update scale as training progresses.

## 23. Batch Size and Weight Decay

The effective batch size determines how many examples or tokens contribute to an optimizer update.

It can depend on:

- Micro-batch size
- Gradient accumulation steps
- Number of distributed workers

Larger effective batches can improve hardware utilization, but they also change optimization behavior.

Weight decay discourages unnecessary growth in parameters. In AdamW-based training, decay is commonly applied selectively to matrix-like weights rather than indiscriminately to biases and normalization parameters.

## 24. Gradient Accumulation

If a GPU cannot fit the desired batch size into memory, the batch can be divided into smaller micro-batches.

```python
optimizer.zero_grad(set_to_none=True)

accumulation_steps = 4

for _ in range(accumulation_steps):
    x_micro, y_micro = get_batch(
        train_data,
        micro_batch_size,
        block_size,
        device,
    )

    logits = model(x_micro)

    loss = F.cross_entropy(
        logits.reshape(-1, logits.size(-1)),
        y_micro.reshape(-1),
    )

    (loss / accumulation_steps).backward()

optimizer.step()
```

Dividing the loss by the number of accumulation steps makes the accumulated gradient represent an average across equally sized micro-batches.

The optimizer updates the weights only after the gradients from all micro-batches have accumulated.

Gradient accumulation reduces the activation memory required for each forward/backward pass, but it does not eliminate all memory costs.

## 25. Distributed Data Parallel (DDP)

Distributed Data Parallel enables multiple GPUs to process different batches while keeping their model replicas synchronized through gradient communication.

```python
from torch.nn.parallel import DistributedDataParallel as DDP

model = DDP(
    model,
    device_ids=[local_rank],
)
```

A multi-process training job can be launched with:

```bash
torchrun --standalone --nproc_per_node=4 train_gpt2.py
```

This example assumes a correctly configured distributed training script and four supported GPUs on one machine.

DDP can increase throughput when communication and computation are balanced, but it does not guarantee a fourfold speedup.

### Combining DDP and gradient accumulation

The effective batch size is:

```text
Effective batch size
= micro-batch size
  × accumulation steps
  × number of GPUs
```

For language models, token count is often a more useful measure:

```text
Tokens per update
= batch size per GPU
  × sequence length
  × accumulation steps
  × number of GPUs
```

This approach increases the effective batch size without requiring the entire batch to fit on one GPU.

## 26. Dataset and Training-Validation Split

The lecture discusses the distinction between reproducing GPT-2's architecture and reproducing its original training data.

Relevant datasets discussed include:

- **WebText:** Associated with the original GPT-2 training.
- **GPT-3 training mixture:** A broader collection of training data sources.
- **FineWeb-Edu:** Curated web text emphasizing educational content.

These datasets are not interchangeable. The dataset used depends on the particular reproduction and training configuration.

Training data updates model parameters. Validation data measures performance on held-out text without updating those parameters.

```python
model.eval()

with torch.no_grad():
    logits = model(x_val)

    val_loss = F.cross_entropy(
        logits.reshape(-1, logits.size(-1)),
        y_val.reshape(-1),
    )
```

`model.eval()` changes training-specific behavior such as dropout. `torch.no_grad()` avoids building a gradient graph.

If training loss improves while validation loss worsens, the model may be overfitting.

## 27. Sampling During Training

Loss is useful, but generated text provides an intuitive view of the model's learned behavior.

Karpathy periodically generates samples during training to inspect how output changes over time.

Early samples may look random or incoherent. As training progresses, patterns may become more recognizable.

However, lower validation loss does not guarantee that every generated sample will be good. Results also depend on the prompt and sampling settings.

## 28. HellaSwag Evaluation

HellaSwag is a multiple-choice benchmark that evaluates commonsense reasoning through sentence completion.

The model receives a context and candidate continuations. It evaluates the candidates and selects the one with the lowest normalized token-level loss.

Conceptually:

```python
candidate_losses = []

for candidate in candidates:
    loss = calculate_candidate_loss(
        model,
        context,
        candidate,
    )
    candidate_losses.append(loss)

prediction = candidate_losses.index(
    min(candidate_losses)
)
```

This is illustrative evaluation logic; a proper implementation must handle tokenization, context-target boundaries, candidate lengths, and the benchmark's scoring procedure correctly.

Normalizing loss helps compare candidates of different lengths rather than simply favoring candidates with fewer total token losses.

HellaSwag accuracy complements validation loss: validation loss measures next-token prediction on held-out text, while HellaSwag measures performance on a specific multiple-choice benchmark.

## 29. Key Takeaways

- GPT-2 is a decoder-only Transformer with approximately 124 million parameters in its Small configuration.
- Reproducing GPT-2 requires matching the architecture and tensor shapes before loading pretrained weights.
- The model learns by predicting shifted next-token targets using cross-entropy loss.
- Weight tying shares the token embedding matrix with the output head.
- Mixed precision, FlashAttention, `torch.compile`, and fused optimizers can improve training efficiency.
- Gradient clipping and learning-rate scheduling help manage training stability.
- Gradient accumulation and DDP increase effective batch size without requiring all data to fit on one GPU.
- Validation loss measures generalization, while HellaSwag provides a separate benchmark evaluation.
- Measuring performance is essential: optimizations should be evaluated on the actual hardware and workload.

**My main learning:** Reproducing GPT-2 is not only about understanding Transformer layers. It also requires careful parameter mapping, correct training targets, numerical stability, hardware-aware optimization, and meaningful evaluation.

