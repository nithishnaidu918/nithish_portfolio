---
title: "LLM Journey #6 "
description: "Understanding how Karpathy moves from a bigram model to an MLP that uses multiple previous characters as context."
date: "2026-10-07"
category: "LLM"
tags: ["LLM", "Transformers", "Tokenization"]
---

## LLM Journey #6 — CNNs, Hierarchical Context & WaveNet
Description: Learning how Karpathy moves from a flat MLP to a CNN/WaveNet-style architecture by processing character context hierarchically with FlattenConsecutive, shared transformations, and PyTorch modules.


## 1. Where We Are Coming From
In Part 2, the model was essentially:
Characters
    ↓
Embeddings
    ↓
Flatten everything
    ↓
MLP
    ↓
Next character

All the character embeddings were combined and processed together.    Pasted markdown
In Part 5, Karpathy changes this idea.
Instead of processing the entire context at once:
whole context
     ↓
    MLP

the model processes small groups first, then combines those representations into larger groups.


## 2. The Main Idea: Hierarchical Processing
Suppose the context is:
a b c d

Instead of:
[a b c d]
     ↓
    MLP

we can process:
a b     c d
 \ /     \ /
 AB      CD
   \     /
    \   /
    ABCD

So the model goes:
Characters
    ↓
Small groups
    ↓
Larger groups
    ↓
Whole context

This is the central idea of Part 5.    Pasted markdown


## 3. Why Is This CNN-Like?
CNNs generally process local groups and gradually build larger representations.
For images:
Pixels
  ↓
Small patterns
  ↓
Edges
  ↓
Shapes
  ↓
Objects

For characters:
Characters
  ↓
Small character patterns
  ↓
Larger character patterns
  ↓
Whole word representation

So the model gradually increases the amount of context it can represent.    Pasted markdown


## 4. FlattenConsecutive
One of the most important new components is:
class FlattenConsecutive:    ...


Its job is simple:
Take consecutive positions and combine them together.

Suppose the tensor shape is:
(B, T, C)

where:
B = batch size
T = sequence positions
C = features per position

For example:
(32, 8, 10)

means:
32 examples
8 positions
10 features per position

   Pasted markdown


## 5. Combining Consecutive Positions
Suppose we combine every 2 consecutive positions.
Starting with:
(32, 8, 10)

we get:
(32, 4, 20)

Why?
8 positions ÷ 2 = 4 groups

and:
10 features × 2 = 20 features

So:
(B, T, C)
      ↓
(B, T/2, 2C)

   Pasted markdown


## 6. The Code
Conceptually, the operation is:
x.view(    x.shape[0],    x.shape[1] // 2,    x.shape[2] * 2)


So:
(B, T, C)
      ↓
(B, T/2, 2C)

Nothing magical is happening.
We're simply grouping adjacent positions together.    Pasted markdown
For example:
[a][b][c][d]

becomes:
[ab][cd]



## 7. The New Architecture
Instead of the earlier:
Embedding
    ↓
Flatten
    ↓
Linear
    ↓
Tanh
    ↓
Linear

we now have something like:
Embedding
    ↓
FlattenConsecutive
    ↓
Linear
    ↓
BatchNorm
    ↓
Tanh
    ↓
FlattenConsecutive
    ↓
Linear
    ↓
BatchNorm
    ↓
Tanh
    ↓
Linear
    ↓
Output

The exact architecture is constructed from these layers in the notebook.    Pasted markdown


## 8. What Actually Changed?
Earlier MLP
a b c d e f
     ↓
flatten everything
     ↓
Linear

Part 5
a b c d e f
│ │ │ │ │ │
└─┴─┘ └─┴─┘
  ↓     ↓
 group  group
   \     /
    \   /
     ↓
larger group

The model now learns local relationships first, then combines them into larger relationships.    Pasted markdown


## 9. Tensor Shapes Are Very Important
This is one of the biggest lessons from Part 5.
Suppose:
(B, T, C) = (32, 8, 10)

After grouping pairs:
(32, 8, 10)
      ↓
(32, 4, 20)

After another grouping:
(32, 4, 20)
      ↓
(32, 2, 40)

And again:
(32, 2, 40)
      ↓
(32, 1, 80)

Now the entire context has been combined.
This is the hierarchical structure.    Pasted markdown


## 10. Seeing the Hierarchy
For eight positions:
a b c d e f g h

First:
a b     c d     e f     g h
 \ /     \ /     \ /     \ /
 AB      CD      EF      GH

Then:
AB + CD        EF + GH
      \          /
       \        /
        ABCDEFGH

So:
8 positions
     ↓
4 groups
     ↓
2 groups
     ↓
1 group

That is the tree-like hierarchy Karpathy is building.    Pasted markdown


## 11. Weight Sharing
A major CNN idea is weight sharing.
Suppose we have:
[a,b] [c,d] [e,f] [g,h]

Instead of using a different transformation for every group:
[a,b] → W1
[c,d] → W2
[e,f] → W3
[g,h] → W4

we reuse the same transformation:
[a,b] → W
[c,d] → W
[e,f] → W
[g,h] → W

This is weight sharing.    Pasted markdown
It means the model doesn't need separate parameters for every position.


## 12. FlattenConsecutive Is Not the Convolution
This distinction is important.
Karpathy uses:
FlattenConsecutive
        +
     Linear

to create this simplified CNN-like hierarchical operation.
FlattenConsecutive:
groups neighboring positions

while Linear:
learns a transformation over those groups

Together they create the simplified convolution-like structure used in the notebook.    Pasted markdown


## 13. Connection to WaveNet
Part 5 is inspired by the hierarchical idea behind WaveNet.
Don't think:
Part 5 = the original WaveNet.

Instead:
Part 5 builds a simpler CNN-like hierarchy that helps understand the ideas behind WaveNet.

The actual WaveNet architecture uses causal dilated convolutions, which Karpathy has not yet implemented in this notebook.    Pasted markdown


## 14. Causality
For a language model, when predicting:
a b c → ?

the model must not use future characters.
It can use:
a
a b
a b c

but not information from the future.
This is called causality.
The prediction must depend only on previous context.    Pasted markdown


## 15. What Does "Dilated" Mean?
Dilated convolutions allow the model to skip positions.
Instead of:
a b c d

a convolution could look at:
a   c   e   g

With a larger dilation, it can cover an even wider context.
This allows the receptive field to grow without requiring as many layers. Actual WaveNet uses causal dilated convolutions for this purpose.    Pasted markdown


## 16. nn.Module
Part 5 also reinforces an important PyTorch concept:
class MyLayer(nn.Module):    def __init__(self):        super().__init__()    def forward(self, x):        return ...


Then:
model = MyLayer()


PyTorch can automatically discover parameters inside the module.
This is the foundation of how real PyTorch models are organized.    Pasted markdown


## 17. Why nn.Module Matters
Imagine having:
W1
b1
W2
b2
W3
b3

Managing all of these manually becomes inconvenient.
With nn.Module:
model.parameters()


gives access to the trainable parameters.
Then an optimizer can use them:
optimizer = torch.optim.AdamW(    model.parameters())


So Part 5 also teaches how PyTorch organizes reusable neural-network components.    Pasted markdown


## 18. nn.Sequential
Karpathy also builds layers and applies them one after another:
for layer in layers:    x = layer(x)

This is essentially what:
nn.Sequential(...)
does.
It provides a convenient way to stack multiple layers together.   


## 19. Training Has Not Changed
This is important.
We already learned training in Parts 2–4.
The training process is still:
for step in range(num_steps):    logits = model(Xb)    loss = F.cross_entropy(logits, Yb)    optimizer.zero_grad()    loss.backward()    optimizer.step()


So Part 5 is not introducing a new optimizer or training algorithm.
It introduces a new architecture.    


## 20. What Happens to loss.backward()?
Nothing special.
The CNN-like layers are built from differentiable PyTorch operations.
So training remains:
CNN
 ↓
Forward
 ↓
Loss
 ↓
loss.backward()
 ↓
Gradients
 ↓
optimizer.step()

The same backpropagation system from Part 4 still works.    Pasted markdown


## 21. The Complete Architecture
Keep this mental model:
Characters
    ↓
Embedding
    ↓
(B, 8, C)
    ↓
FlattenConsecutive(2)
    ↓
(B, 4, 2C)
    ↓
Linear + BatchNorm + Tanh
    ↓
(B, 4, H)
    ↓
FlattenConsecutive(2)
    ↓
(B, 2, 2H)
    ↓
Linear + BatchNorm + Tanh
    ↓
(B, 2, H)
    ↓
FlattenConsecutive(2)
    ↓
(B, 1, 2H)
    ↓
Linear
    ↓
Logits
    ↓
CrossEntropyLoss

The exact dimensions depend on the notebook's hyperparameters; the important part is the hierarchical structure.    Pasted markdown



## 22. What You Should Remember
1. CNNs process local groups.

2. The same transformation can be reused
   across different positions.

3. This is called weight sharing.

4. Part 5 builds context hierarchically:
   
   local → larger → global

5. FlattenConsecutive groups neighboring positions:

   (B, T, C)
        ↓
   (B, T/2, 2C)

6. nn.Module provides a clean way to build
   reusable neural-network components.

7. Tensor shapes such as (B, T, C)
   are extremely important.

8. Training has not changed:

   zero_grad()
   ↓
   backward()
   ↓
   step()

9. Part 5 is CNN/WaveNet-inspired,
   not the exact original WaveNet.

10. The main new idea is hierarchical
    processing of context.

   Pasted markdown