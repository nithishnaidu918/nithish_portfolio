---
title: "LLM Journey #5 "
description: "Doing BackProp manually and understanding what pytorch was doing with loss.backward()."
date: "2026-10-08"
category: "LLM"
tags: ["LLM", "Transformers", "Tokenization"]
---

## LLM Journey #5 — Understanding Backpropagation
Description: Learning what loss.backward() actually does by manually implementing backpropagation, understanding the chain rule, gradients, BatchNorm backward pass, and softmax + cross-entropy derivatives.

## 1. What this is About
In previous projects, we simply used:
loss.backward()


PyTorch calculated all the gradients for us.
Part 4 asks:
What is actually happening inside loss.backward()?

Instead of trusting autograd, we manually calculate the gradients and compare them with PyTorch's gradients.    Pasted markdown


## 2. Same Model as Part 3
The architecture is still essentially the same:
Characters
    ↓
Embedding
    ↓
Concatenate
    ↓
Linear
    ↓
BatchNorm
    ↓
tanh
    ↓
Linear
    ↓
Logits
    ↓
Cross Entropy
    ↓
Loss

The difference is that now we manually go backward through every operation.    Pasted markdown


## 3. Forward Pass
The forward pass can be broken into small operations:
emb = C[Xb]embcat = emb.view(emb.shape[0], -1)hprebn = embcat @ W1 + b1bnmeani = hprebn.mean(0, keepdim=True)bndiff = hprebn - bnmeanibnvar = bndiff.pow(2).mean(0, keepdim=True)bnvar_inv = (bnvar + 1e-5).pow(-0.5)bnraw = bndiff * bnvar_invhpreact = bngain * bnraw + bnbiash = torch.tanh(hpreact)logits = h @ W2 + b2


Then the logits are converted into probabilities and the loss is calculated.
The important idea is that every operation in the forward pass becomes something we must differentiate during the backward pass.    Pasted markdown


## 4. retain_grad()
Normally, PyTorch mainly keeps gradients for leaf tensors.
Karpathy wants to inspect intermediate gradients such as:
logits
h
hpreact
bnraw
bnvar
emb

So he uses:
t.retain_grad()


After:
loss.backward()


we can inspect:
t.grad


This allows us to compare our manually calculated gradients with PyTorch's gradients.    Pasted markdown


## 5. Backpropagation
The forward direction is:
C
↓
Embedding
↓
Concatenation
↓
Linear
↓
BatchNorm
↓
tanh
↓
Linear
↓
Loss

Backpropagation goes in the opposite direction:
Loss
↓
Linear
↓
tanh
↓
BatchNorm
↓
Linear
↓
Concatenation
↓
Embedding
↓
C

That is the basic idea of backpropagation.    Pasted markdown


## 6. The Main Rule: Chain Rule
The most important pattern is:
upstream gradient
        ×
local derivative
        ↓
downstream gradient

Mathematically:
\[
\text{gradient} =
\text{upstream gradient}
\times
\text{local derivative}
\]
For example:
h = torch.tanh(hpreact)


The derivative of tanh is:
\[
1-\tanh^2(x)
\]
So:
dhpreact = (1.0 - h**2) * dh


This is simply the chain rule.    Pasted markdown


## 7. Backprop Through a Linear Layer
Forward:
logits = h @ W2 + b2


Backward:
dh = dlogits @ W2.TdW2 = h.T @ dlogitsdb2 = dlogits.sum(0)


So from the gradient of logits, we calculate gradients for:
h
W2
b2

This is matrix calculus applied during backpropagation.    Pasted markdown


## 8. Backprop Through tanh
Forward:
h = torch.tanh(hpreact)


Backward:
dhpreact = (1.0 - h**2) * dh


The pattern is always:
upstream gradient
        ×
derivative of operation
        ↓
gradient of input



## 9. Why BatchNorm Is Hard
BatchNorm is the most difficult part of the manual backward pass.
Forward:
hprebn
   ↓
mean
   ↓
subtract mean
   ↓
square
   ↓
variance
   ↓
inverse standard deviation
   ↓
normalize
   ↓
gamma ×
   ↓
beta +
   ↓
hpreact

The difficulty is that one input affects:
1. Its own value
2. The batch mean
3. The batch variance
Therefore, the gradient has multiple paths that eventually need to be combined.    Pasted markdown
You don't need to memorize the giant BatchNorm derivative.
The important concept is understanding why multiple gradient paths exist.


## 10. BatchNorm Parameters
The gradients for the BatchNorm scale and bias are:
dbngain = (bnraw * dhpreact).sum(0, keepdim=True)dbnbias = dhpreact.sum(0, keepdim=True)


These calculate the gradients for:
gamma → dbngain
beta  → dbnbias


## 11. Softmax + Cross Entropy
One of the most useful results in this lecture is the simplified gradient for softmax combined with cross entropy.
The result is:
\[
\frac{\partial L}{\partial logits}
=
\frac{p-y}{N}
\]
where:
p = predicted probabilities
y = target
N = batch size

The implementation is:
dlogits = F.softmax(logits, 1)dlogits[range(n), Yb] -= 1dlogits /= n


This avoids manually differentiating every individual softmax and logarithm operation.    Pasted markdown


## 12. Why -= 1?
Suppose the model predicts:
[0.2, 0.5, 0.3]

and the correct class is the second class:
[0, 1, 0]

Subtracting gives:
[0.2, -0.5, 0.3]

So:
correct class
     ↓
negative gradient
     ↓
push its logit upward

While the incorrect classes receive positive gradients and are pushed downward.    Pasted markdown


## 13. Why Does the Gradient Sum to Zero?
The probabilities sum to 1:
\[
\sum p_i = 1
\]
The target vector also sums to 1:
\[
\sum y_i = 1
\]
Therefore:
\[
\sum(p-y)=0
\]
So the gradients of the logits should sum to approximately zero.
This becomes a useful sanity check.    Pasted markdown


## 14. Manual Backpropagation
Eventually, loss.backward() is removed.
The gradients are calculated manually:
dlogits = F.softmax(logits, 1)dlogits[range(n), Yb] -= 1dlogits /= ndh = dlogits @ W2.TdW2 = h.T @ dlogitsdb2 = dlogits.sum(0)dhpreact = (1.0 - h**2) * dh# BatchNorm gradientsdbngain = (bnraw * dhpreact).sum(0, keepdim=True)dbnbias = dhpreact.sum(0, keepdim=True)# Continue backward...


Eventually the gradients reach:
dC
dW1
db1
dW2
db2
dbngain
dbnbias

Then parameters are updated:
p.data += -lr * grad


## 15. So What Is loss.backward()?
Now we can understand it much better.
When we write:
loss.backward()


PyTorch is effectively doing:
Loss
 ↓
dlogits
 ↓
dW2, db2, dh
 ↓
dhpreact
 ↓
BatchNorm gradients
 ↓
dW1, db1
 ↓
dEmbedding
 ↓
dC

Every step uses:
Chain rule + local derivative

That is essentially what loss.backward() automates.    Pasted markdown



## 16. torch.no_grad()
Once manual gradients are being calculated, we don't need PyTorch to build another computational graph.
So:
with torch.no_grad():    ...


means:
Don't track these operations for automatic gradient calculation.

The parameters can then be updated using the manually calculated gradients.    Pasted markdown


## 17. Learning Rate Decay
The training still uses learning-rate decay:
lr = 0.1 if i < 100000 else 0.01


Meaning:
First 100k steps → larger learning rate
Later steps     → smaller learning rate

Large updates help early training, while smaller updates help later optimization.    Pasted markdown


## 18. Evaluation
After training, the model is evaluated using:
Training loss
Validation loss

The notebook's shown run gets approximately:
Train ≈ 2.07
Validation ≈ 2.11

The exact numbers aren't the main point.
The important result is:
Manual gradients
       ↓
Network trains
       ↓
Loss decreases
       ↓
Manual backprop works


## 19. Sampling
After training, the model generates new names.
The process is:
Context
   ↓
Embedding
   ↓
MLP
   ↓
Logits
   ↓
Softmax
   ↓
Sample next character
   ↓
Update context
   ↓
Repeat

This is the same autoregressive idea from the earlier Makemore parts.    Pasted markdown


## Key Takeaways
1. Backpropagation = chain rule applied backward.

2. Every operation needs a local derivative.

3. Gradients flow from the loss toward the inputs.

4. Gradients from multiple paths are added.

5. Matrix operations have matrix derivatives.

6. tanh derivative = 1 - tanh².

7. Softmax + Cross Entropy:
   dlogits = probabilities - targets

8. BatchNorm is harder because each input
   affects batch statistics.

9. loss.backward() automates all these calculations.

10. Manual backprop shows what PyTorch is
    actually doing internally.