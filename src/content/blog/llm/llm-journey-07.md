---
title: "Let's Build GPT from Scratch"
description: "Understanding how Karpathy builds GPT using token embeddings, positional embeddings, self-attention, Transformer blocks, backpropagation, and text generation."
category: "LLM"
tags: ["LLM", "GPT", "Transformer", "PyTorch", "Self-Attention"]
---

# LLM Journey #7 — Let's Build GPT from Scratch

In this journey, I explored Andrej Karpathy's *Let's Build GPT from Scratch* lecture. The goal is to understand how a Transformer language model learns from text and generates new text using PyTorch.

## 1. What Are We Building?

The model learns to predict the next character from the preceding context.

For example:

```text
Input:  ROM
Target: E
```

After training, the model can generate text one character at a time.

The model learns statistical patterns from training text rather than storing a script of complete responses.

## 2. Text Tokenization

Neural networks work with numerical tensors, so we convert each unique character into an integer ID.

```python
chars = sorted(list(set(text)))
vocab_size = len(chars)

stoi = {ch: i for i, ch in enumerate(chars)}
itos = {i: ch for i, ch in enumerate(chars)}

encode = lambda s: [stoi[c] for c in s]
decode = lambda ids: ''.join([itos[i] for i in ids])
```

For example:

```text
"cab" → [2, 0, 1]
[2, 0, 1] → "cab"
```

- `stoi`: character to integer.
- `itos`: integer to character.
- `vocab_size`: number of unique characters.

This lecture uses character-level tokenization. Modern language models often use subword tokenization.

## 3. Creating Training Batches

The model learns by predicting the next character.

For the sequence `hello`:

```text
Input:  h e l l
Target: e l l o
```

The target sequence is shifted one position forward.

```python
x = torch.stack([
    data[i:i + block_size]
    for i in ix
])

y = torch.stack([
    data[i + 1:i + block_size + 1]
    for i in ix
])
```

Here, `x` contains input sequences and `y` contains the corresponding next-character targets.

The model can learn several next-character predictions from each sequence.

## 4. Creating Batches with `get_batch()`

Instead of processing the entire dataset at once, we sample batches.

```python
def get_batch(split):
    data = train_data if split == "train" else val_data

    ix = torch.randint(
        len(data) - block_size - 1,
        (batch_size,)
    )

    x = torch.stack([
        data[i:i + block_size] for i in ix
    ])

    y = torch.stack([
        data[i + 1:i + block_size + 1] for i in ix
    ])

    return x.to(device), y.to(device)
```

This function selects random chunks of text and creates their shifted targets.

- `batch_size`: number of sequences processed together.
- `block_size`: maximum context length used by the model.

## 5. Token Embeddings

An integer ID alone does not provide a useful learned representation. GPT converts each token into a vector.

```python
self.token_embedding_table = nn.Embedding(
    vocab_size, n_embd
)
```

For example, if `n_embd = 128`, each character receives a learned vector containing 128 values.

These embeddings are updated during training.

## 6. Positional Embeddings

Token embeddings tell the model *what* a character is. Positional embeddings tell it *where* that character occurs.

```python
self.position_embedding_table = nn.Embedding(
    block_size, n_embd
)

tok_emb = self.token_embedding_table(idx)

pos_emb = self.position_embedding_table(
    torch.arange(T, device=device)
)

x = tok_emb + pos_emb
```

GPT adds the token and positional embeddings together before passing them through the Transformer blocks.

## 7. Self-Attention

Self-attention is the central new concept in this lecture.

It allows each position to combine information from earlier positions in the sequence.

Karpathy creates three learned projections:

```python
self.key = nn.Linear(n_embd, head_size, bias=False)
self.query = nn.Linear(n_embd, head_size, bias=False)
self.value = nn.Linear(n_embd, head_size, bias=False)
```

Their intuition:

- **Query (Q):** What information am I looking for?
- **Key (K):** What information can I match against?
- **Value (V):** What information can I receive?

These are learned vectors, not literal questions or labels.

### Core attention calculation

```python
k = self.key(x)
q = self.query(x)
v = self.value(x)

wei = q @ k.transpose(-2, -1)
wei = wei * (k.shape[-1] ** -0.5)

wei = wei.masked_fill(
    self.tril[:T, :T] == 0,
    float("-inf")
)

wei = F.softmax(wei, dim=-1)

out = wei @ v
```

What happens here?

1. Queries and keys produce similarity scores.
2. Scaling controls the magnitude of the scores.
3. The causal mask blocks access to future positions.
4. Softmax converts the allowed scores into attention weights.
5. The weights combine the value vectors.

## 8. Causal Masking

During next-token prediction, the model must not look at future characters.

Karpathy creates a lower-triangular mask:

```python
self.register_buffer(
    "tril",
    torch.tril(torch.ones(block_size, block_size))
)
```

The mask prevents a position from using information from later positions.

This is essential because the model must learn to predict the next token without seeing the answer in advance.

## 9. Multi-Head Attention

One attention head learns one set of attention patterns. Multi-head attention runs several heads in parallel.

```python
self.heads = nn.ModuleList([
    Head(head_size) for _ in range(num_heads)
])

out = torch.cat(
    [h(x) for h in self.heads],
    dim=-1
)

out = self.proj(out)
```

Each head learns its own attention patterns. Their outputs are concatenated and projected back into the embedding dimension.

For example, with an embedding size of 128 and four heads, each head has a size of 32.

Multi-head attention is one model with several parallel attention heads, not several separate models.

## 10. Feed-Forward Network

Attention allows positions to exchange information. The feed-forward network then transforms each position's representation.

```python
self.net = nn.Sequential(
    nn.Linear(n_embd, 4 * n_embd),
    nn.ReLU(),
    nn.Linear(4 * n_embd, n_embd),
    nn.Dropout(dropout)
)
```

The hidden dimension expands to four times the embedding size and then returns to the original size.

- `Linear`: learns transformations.
- `ReLU`: introduces non-linearity.
- `Dropout`: helps reduce overfitting during training.

## 11. Residual Connections and LayerNorm

Karpathy's Transformer block uses residual connections and LayerNorm.

```python
x = x + self.sa(self.ln1(x))
x = x + self.ffwd(self.ln2(x))
```

Here:

- `self.sa`: self-attention.
- `self.ffwd`: feed-forward network.
- `self.ln1` and `self.ln2`: LayerNorm layers.
- `x + ...`: residual connections.

Residual connections preserve the original representation while adding the result of each transformation.

LayerNorm normalizes features within each token representation. Unlike BatchNorm, it does not rely on statistics across the batch.

## 12. The Transformer Block

These components are combined into a reusable block.

```python
class Block(nn.Module):
    def __init__(self, n_embd, n_head):
        super().__init__()

        head_size = n_embd // n_head

        self.sa = MultiHeadAttention(
            n_head, head_size
        )

        self.ffwd = FeedForward(n_embd)

        self.ln1 = nn.LayerNorm(n_embd)
        self.ln2 = nn.LayerNorm(n_embd)

    def forward(self, x):
        x = x + self.sa(self.ln1(x))
        x = x + self.ffwd(self.ln2(x))
        return x
```

A GPT model stacks several Transformer blocks. Each block applies attention and feed-forward processing.

## 13. GPT Model: Core Architecture

The model combines token embeddings, positional embeddings, Transformer blocks, LayerNorm, and an output layer.

```python
class GPTLanguageModel(nn.Module):
    def __init__(self):
        super().__init__()

        self.token_embedding_table = nn.Embedding(
            vocab_size, n_embd
        )

        self.position_embedding_table = nn.Embedding(
            block_size, n_embd
        )

        self.blocks = nn.Sequential(
            *[Block(n_embd, n_head)
              for _ in range(n_layer)]
        )

        self.ln_f = nn.LayerNorm(n_embd)

        self.lm_head = nn.Linear(
            n_embd, vocab_size
        )

    def forward(self, idx, targets=None):
        B, T = idx.shape

        tok_emb = self.token_embedding_table(idx)

        pos_emb = self.position_embedding_table(
            torch.arange(T, device=idx.device)
        )

        x = tok_emb + pos_emb
        x = self.blocks(x)
        x = self.ln_f(x)

        logits = self.lm_head(x)

        loss = None

        if targets is not None:
            B, T, C = logits.shape

            logits = logits.view(B * T, C)
            targets = targets.view(B * T)

            loss = F.cross_entropy(logits, targets)

        return logits, loss
```

This shows the central architecture. The `Head`, `MultiHeadAttention`, and `FeedForward` classes used above must also be defined as shown in the previous sections.

The model produces logits for every possible next character at every sequence position.

## 14. Logits and Cross-Entropy Loss

Logits are raw scores, not probabilities.

During training, GPT compares its predictions with the correct targets using cross-entropy.

```python
logits, loss = model(xb, yb)
```

The model produces logits with shape `(B, T, vocab_size)`. The implementation reshapes them to `(B * T, vocab_size)` and the targets to `(B * T,)`.

This allows cross-entropy to evaluate the next-character prediction at every position.

**Important:** Pass logits directly to `F.cross_entropy()`. You do not need to apply softmax first because PyTorch handles the required calculation internally.

## 15. Training the Model

The training process should feel familiar from earlier PyTorch lessons.

```python
optimizer = torch.optim.AdamW(
    model.parameters(),
    lr=learning_rate
)

for step in range(max_iters):
    xb, yb = get_batch("train")

    logits, loss = model(xb, yb)

    optimizer.zero_grad(set_to_none=True)

    loss.backward()

    optimizer.step()
```

What each line does:

- `get_batch()`: prepares training examples.
- `model(xb, yb)`: performs the forward pass and calculates loss.
- `optimizer.zero_grad()`: clears previous gradients.
- `loss.backward()`: calculates gradients.
- `optimizer.step()`: updates parameters using AdamW.

Training and validation loss help us assess whether the model is learning and generalizing.

## 16. Generating New Text

After training, GPT generates text by repeatedly predicting and sampling the next character.

```python
def generate(self, idx, max_new_tokens):
    for _ in range(max_new_tokens):
        idx_cond = idx[:, -block_size:]

        logits, loss = self(idx_cond)

        logits = logits[:, -1, :]

        probs = F.softmax(logits, dim=-1)

        idx_next = torch.multinomial(
            probs, num_samples=1
        )

        idx = torch.cat((idx, idx_next), dim=1)

    return idx
```

The process is:

1. Take the current context.
2. Run it through GPT.
3. Select the logits at the final position.
4. Convert logits into probabilities with softmax.
5. Sample the next character using `torch.multinomial()`.
6. Append the character and repeat.

Sampling allows variation instead of always choosing the most likely character.

## 17. Important PyTorch Functions

| Function | Purpose |
|---|---|
| `nn.Embedding()` | Learns token and position representations |
| `nn.Linear()` | Applies a learnable linear transformation |
| `F.softmax()` | Converts logits into probabilities |
| `F.cross_entropy()` | Measures prediction error |
| `torch.multinomial()` | Samples from a probability distribution |
| `torch.cat()` | Concatenates tensors |
| `nn.Dropout()` | Helps reduce overfitting |
| `nn.LayerNorm()` | Normalizes features within each representation |
| `loss.backward()` | Calculates gradients |
| `optimizer.step()` | Updates model parameters |
| `torch.no_grad()` | Disables gradient tracking in a code block or function |

## 18. Training vs. Generation

**Training**

```text
Input + Correct Targets
        ↓
    Forward Pass
        ↓
    Calculate Loss
        ↓
    Backpropagation
        ↓
    Update Parameters
```

**Generation**

```text
Current Context
       ↓
   Forward Pass
       ↓
Final-Position Logits
       ↓
     Softmax
       ↓
  Sample Next Token
       ↓
Append to Context
       ↓
      Repeat
```

Generation does not require a training loss or parameter updates.

## Key Takeaways

1. Tokenization converts text into integer IDs.
2. Token embeddings represent characters as learned vectors.
3. Positional embeddings provide information about sequence positions.
4. Self-attention combines information from earlier positions.
5. Causal masking prevents the model from looking into the future.
6. Multi-head attention learns multiple attention patterns in parallel.
7. Feed-forward layers transform token representations.
8. Residual connections and LayerNorm support Transformer training.
9. Cross-entropy measures next-token prediction error.
10. Backpropagation and AdamW train the model.
11. Softmax and multinomial sampling allow GPT to generate text.