---
title: "LLM Journey #8 — Building a GPT Tokenizer"
description: "Understanding BPE, Unicode, pre-tokenization, SentencePiece, tiktoken, special tokens, and how tokenization connects to GPT."
category: "LLM"
tags: ["LLM", "GPT", "Tokenizer", "BPE", "SentencePiece", "PyTorch"]
---

# LLM Journey #8 — Building a GPT Tokenizer

## 1. What Is Tokenization?

In the previous journey, I explored how GPT uses token embeddings, positional embeddings, self-attention, Transformer blocks, and next-token prediction.

Before text enters GPT, it must be converted into numerical token IDs. This process is called **tokenization**.

```text
Raw text
   ↓
Tokenizer
   ↓
Token IDs
   ↓
Token embeddings
   ↓
GPT / Transformer
   ↓
Predicted token IDs
   ↓
Decoded text
```

For example, the text `"hello world"` might be represented by a few subword tokens instead of one token per character. The actual token IDs depend on the tokenizer and its vocabulary.

**Important distinction:** The tokenizer converts text into tokens. GPT learns to predict the next token.

## 2. Character-Level vs. Subword Tokenization

In the earlier GPT implementation, the vocabulary was built from individual characters.

```python
chars = sorted(list(set(text)))
vocab_size = len(chars)

stoi = {ch: i for i, ch in enumerate(chars)}
itos = {i: ch for i, ch in enumerate(chars)}

encode = lambda s: [stoi[c] for c in s]
decode = lambda ids: "".join([itos[i] for i in ids])
```

Here:

- `stoi` maps each character to an integer ID.
- `itos` maps each integer ID back to a character.
- `encode()` converts text into IDs.
- `decode()` converts IDs back into text.

Character-level tokenization is simple, but sequences can become long. Modern LLMs commonly use subword tokenization to represent frequently occurring words and word fragments more efficiently.

## 3. What Is BPE?

**BPE (Byte Pair Encoding)** is an algorithm that learns larger token units by repeatedly merging frequent adjacent pairs.

A simplified example:

```text
Initial units:
l o w   l o w   l o w

Possible learned merges:
l + o  → lo
lo + w → low

Result:
low | low | low
```

The tokenizer counts adjacent pairs, merges a frequent pair, and repeats the process until it reaches the desired vocabulary size or merge limit.

BPE does not simply split words into syllables. It learns reusable units from training data.

The exact merges depend on the training corpus, initial representation, and tokenizer configuration.

## 4. Why Bytes and Unicode Matter

Text is not always represented by one byte per character.

- Unicode defines characters and code points.
- UTF-8 is an encoding that represents Unicode text as bytes.
- Accented characters, Indian-language text, and emojis can require multiple UTF-8 bytes.

For example, the character `A` is represented by one byte in ASCII-compatible UTF-8, while `é` and `😊` use multiple bytes.

A byte-level tokenizer can represent arbitrary byte sequences, including text containing unfamiliar characters. This helps avoid relying on a fixed vocabulary of individual characters.

This is why understanding Unicode and UTF-8 is useful when studying real-world tokenizers.

## 5. What Is Pre-Tokenization?

Some tokenizers divide text into smaller chunks **before applying BPE merges**. This stage is called pre-tokenization.

For example, a rule-based tokenizer might divide:

```text
hello world! 123
```

into chunks resembling:

```text
hello | world | ! | 123
```

This is only an illustration; the actual boundaries depend on the tokenizer's rules.

Regular expressions can define how letters, numbers, punctuation, and whitespace are grouped. BPE then operates within the boundaries specified by that tokenizer's implementation.

Different pre-tokenization rules can produce different token sequences for the same text.

## 6. Understanding `minbpe`

Karpathy's [`minbpe`](https://github.com/karpathy/minbpe) is a compact educational implementation of BPE.

It helps explain how tokenizers learn merges and use those merges to encode and decode text instead of hiding everything behind a library call.

The general interface looks like this:

```python
ids = tokenizer.encode("hello world")
text = tokenizer.decode(ids)
```

The expected round trip is:

```text
Text → Encode → Token IDs → Decode → Original text
```

The exact initialization and training methods depend on the tokenizer class being used. The key idea is that encoding applies the learned tokenizer rules, while decoding converts token IDs back into text.

## 7. SentencePiece

SentencePiece is a tokenizer library that supports subword algorithms such as **BPE** and **Unigram**.

- **BPE:** Learns merges between adjacent units.
- **Unigram:** Learns a probabilistic model over candidate vocabulary pieces.

SentencePiece can process raw text without requiring the same conventional word-based pre-tokenization used by some other pipelines. Whitespace handling and normalization depend on the model configuration.

For example, a SentencePiece vocabulary may display a piece resembling:

```text
▁world
```

The visible `▁` marker represents a whitespace boundary in SentencePiece's token representation. It is not normally something you type instead of a space.

**Key distinction:** SentencePiece is a tokenizer framework, not another name for BPE.

## 8. The Llama 2 Tokenizer

Llama 2 uses a SentencePiece tokenizer model with its own learned vocabulary and token IDs.

Its basic workflow is:

```text
Text
  ↓
SentencePiece tokenizer
  ↓
Token IDs
  ↓
Embedding layer
  ↓
Transformer
  ↓
Output logits
  ↓
Next-token selection
  ↓
Decoded text
```

Unlike the earlier character-level tokenizer, a learned subword tokenizer can represent a word using one token or several tokens.

Tokenizers may also define special tokens such as:

- **BOS:** Beginning of sequence.
- **EOS:** End of sequence.
- **PAD:** Padding token, when defined by the model.

Llama 2's tokenizer vocabulary and token IDs are model-specific. Replacing its tokenizer with an unrelated one will not preserve the expected input-output behavior of the pretrained model.

## 9. `tiktoken`, GPT-2, and GPT-4

`tiktoken` is OpenAI's tokenizer library. It supports specific tokenization encodings, including byte-level BPE-based tokenization.

Karpathy compares tokenizer implementations and explains that GPT-2 and GPT-4 tokenization pipelines can differ in their pre-tokenization patterns, vocabulary, and special-token handling.

| Feature | `minbpe` | `tiktoken` | Llama 2 tokenizer |
|---|---|---|---|
| Main purpose | Educational BPE implementation | OpenAI tokenizer library | Tokenizer for Llama 2 |
| Vocabulary | Learned during tokenizer training | Defined by a particular encoding | Defined by the model's tokenizer |
| Tokenization details | Depend on the implementation | Depend on the encoding | Depend on the SentencePiece model and configuration |
| Freely interchangeable between pretrained models? | No | No | No |

Although these systems share related subword-tokenization ideas, their vocabularies, token IDs, normalization behavior, and special tokens can differ.

## 10. What Are Special Tokens?

Special tokens represent structural information rather than ordinary text alone.

Examples associated with the Llama 2 chat format include:

- `<s>` — beginning of sequence.
- `</s>` — end of sequence.
- `[INST]` and `[/INST]` — delimit an instruction.
- `<<SYS>>` and `<</SYS>>` — delimit system instructions in that format.

A useful distinction is that a visible marker in a chat prompt is not necessarily a dedicated token ID in the tokenizer vocabulary. The tokenizer and the chat-formatting layer work together.

Special tokens help the model and surrounding software interpret message boundaries and other structural information.

## 11. How Vocabulary Size Affects GPT

A tokenizer's vocabulary size affects both sequence length and model parameters.

Consider an embedding dimension of `384` and a vocabulary of `32,000` tokens.

```python
import torch.nn as nn

vocab_size = 32_000
embedding_dim = 384

token_embedding = nn.Embedding(vocab_size, embedding_dim)
```

The embedding table contains:

```text
32,000 × 384 = 12,288,000 parameters
```

The output layer must also produce a score for each possible vocabulary token.

The trade-off is:

- **Smaller vocabulary:** Often produces longer token sequences.
- **Larger vocabulary:** May shorten sequences, but increases embedding and output-layer sizes.

The best vocabulary depends on the training data, language, domain, and tokenizer design.

## 12. Training a Tokenizer for a Specific Corpus

A tokenizer can be trained on a corpus to learn token pieces that represent its text efficiently.

For example, a technical corpus may repeatedly contain particular programming terms or domain-specific expressions. A tokenizer trained on that corpus may learn useful pieces for them.

Tokenization also affects prompt compression: the same text can require different numbers of tokens under different tokenizers.

However, **training a new tokenizer does not automatically improve an existing pretrained model**. Its embeddings and output layer are associated with the original vocabulary and token IDs. Changing the tokenizer generally requires suitable model training or adaptation.

## 13. Tokenization Beyond Text

Tokenization becomes more complex in multimodal systems that work with images, audio, or video.

Text tokenizers map text into discrete IDs. Some multimodal architectures use learned representations and vector quantization to turn continuous representations into discrete codes.

A simplified example:

```text
Image / Audio / Video
        ↓
      Encoder
        ↓
Representation
        ↓
Discrete codes
        ↓
Token-like sequence
```

This is one possible approach, not a universal architecture for every multimodal model. Different systems represent non-text information in different ways.

## 14. Key Takeaways

- Tokenization converts text into token IDs before it enters a language model.
- Character-level tokenization is simple; subword tokenization can represent common text more compactly.
- BPE learns token units by merging frequent adjacent pairs.
- Unicode and UTF-8 explain how diverse text can be represented as bytes.
- Pre-tokenization defines boundaries before merge rules are applied.
- SentencePiece is a framework that supports algorithms such as BPE and Unigram.
- `minbpe`, `tiktoken`, and the Llama 2 tokenizer have different implementations and vocabularies.
- Special tokens provide structural information.
- Vocabulary size affects sequence length and model parameters.
- A tokenizer must match the vocabulary and token IDs expected by a pretrained model.