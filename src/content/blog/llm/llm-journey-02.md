---
title: "LLM Journey #2 - Makemore, Bigram Language Model & Neural Network"
description: "Learning how a simple character-level language model goes from counting characters to a trainable neural network."
category: "LLM"
tags: ["LLM", "Transformers", "Tokenization"]
---

# LLM Journey #2 -  Bigram Language Model & Neural Network

Learning how a simple character-level language model goes from counting characters to a trainable neural network.

## 1. What is Makemore?
The goal is simple:
Give the model a collection of names and make it generate new names that look like the training names.

Makemore is a character-level autoregressive language model.
Instead of predicting the next word:
"I am going" → "home"

it predicts the next character:
"em" → "m"



## 2. Bigram
A bigram uses one character to predict the next character.

For:
emma

we create:
e → m  ,
m → m  ,
m → a

So:
input    target

e        m

m        m

m        a

The model only looks at one previous character.



## 3. Counting Bigrams
Suppose the dataset contains:
emma ,
emma ,
emma

We count how often characters follow each other.
e → m : 3 ,
m → m : 3 ,
m → a : 3

These counts can be stored in a matrix.
N[i][j]

means:
How many times did character j come after character i?

This count matrix is the first simple model.



## 4. Convert Counts into Probabilities
Counts are not probabilities yet.
Suppose:
m → a = 10 ,
m → e = 5 ,
m → i = 5

Total:
20

Normalize:
P(a | m) = 10 / 20 = 0.50   ,
P(e | m) = 5 / 20  = 0.25   ,
P(i | m) = 5 / 20  = 0.25

The probabilities add up to 1.
counts
   → 
normalize
   → 
probabilities



## 5. Generate a Name
Suppose the current character is:
m

The model predicts:
a → 0.50 ,
e → 0.25 ,
i → 0.25

We sample the next character.
For example:
m → a

Then use a to predict the next character.
Eventually:
m → a → r → i → a

Generated name:
maria



## 6. Likelihood
Now we need to measure how good the model is.
Suppose the real name is:
emma

The model predicts:
P(m | e) = 0.40 ,
P(m | m) = 0.30 ,
P(a | m) = 0.20

Probability of the sequence:
P(emma) = 0.40 × 0.30 × 0.20

P(emma) = 0.024

Higher probability means the model considers the sequence more likely.



## 7. Maximum Likelihood Estimation
Maximum Likelihood Estimation (MLE) means finding model parameters that give the training data the highest probability.
maximize P(training data)

For multiple examples:
P(x₁, x₂, ..., xₙ) = ∏ P(xᵢ)

So we want the model that assigns the highest probability to the training data.



## 8. Why Log?
Multiplying many probabilities becomes inconvenient:
0.4 × 0.3 × 0.2 × 0.1 × ...

Logarithms convert multiplication into addition:
log(ab) = log(a) + log(b)

Therefore:
log P(x₁, x₂, ...) = Σ log P(xᵢ)

This makes the calculation easier.



## 9. Negative Log Likelihood
We want to maximize probability.
But during neural-network training, we normally minimize a loss.
So we take the negative:
-log P

This gives:
Negative Log Likelihood (NLL).
The connection is:
Maximum likelihood
        → 
maximize probability
        → 
maximize log probability
        → 
minimize negative log probability
        → 
NLL loss


## 10. Average NLL
For all training examples:
loss = -(1/N) Σ log P(xᵢ)

This gives one number representing the model's performance.
Example:
loss = 2.31

Lower loss is better.



## 11. Smoothing
There is a problem with zero probabilities.
Suppose:
q → z = 0

Then:
P(z | q) = 0

and:
log(0) = -∞

To avoid this, a small count can be added before normalization.
For example:
q → z = 0

becomes:
q → z = 1

This is called additive/Laplace smoothing in this simple setting.



## 12. From Counting to a Neural Network
Instead of manually doing:
counts → normalize → probabilities

we let a neural network learn the parameters.
The basic process is:

character
    → 
one-hot encoding
    → 
weights W
    → 
logits
    → 
softmax
    → 
probabilities
    → 
loss


## 13. One-Hot Encoding
Suppose the vocabulary is:
[a, b, c, d, e]

Assign integers:
a = 0 ,
b = 1 ,
c = 2 ,
d = 3 ,
e = 4

One-hot representation:
a → [1, 0, 0, 0, 0]   ,
b → [0, 1, 0, 0, 0]   ,
c → [0, 0, 1, 0, 0]   ,
d → [0, 0, 0, 1, 0]   ,
e → [0, 0, 0, 0, 1]

Only one position is 1.



## 14. Weight Matrix
Suppose:
vocab_size = 5

Then:
W = 5 × 5

We calculate:
XW

where X is the one-hot input.
A one-hot vector effectively selects one row of W.
That row becomes the raw prediction for the next character.



## 15. Logits
The output of XW is not probabilities.
Example:
[-1.2, 0.5, 2.1, -0.7, 1.3]

These values are called logits.
They are raw scores and do not need to sum to 1.



## 16. Softmax
Softmax converts logits into probabilities.
Formula:
pᵢ = eᶻⁱ / Σ eᶻʲ

So:
logits
   → 
exponential
   → 
normalize
   → 
probabilities

Example:
logits = [1, 2, 3]

Approximately:
[0.09, 0.24, 0.67]

The probabilities sum to 1.



## 17. Why Exponential?
Exponentiation makes larger logits receive disproportionately larger values.
For:
1
2
3

we get approximately:
e¹ = 2.72
e² = 7.39
e³ = 20.09

Then we normalize these values to obtain probabilities.



## 18. Softmax + NLL
Suppose the correct next character is a.
If the model predicts:
P(a) = 0.8

then:
-log(0.8)

is a small loss.
If the model predicts:
P(a) = 0.01

then:
-log(0.01)

is a large loss.
So the model learns to give high probability to the correct next character.



## 19. Training
Initially:
W = random

So predictions are poor.
Training follows:
forward   →    probabilities   →   loss   →   backward   →   gradient  →   update W

PyTorch calculates the gradient:
loss.backward()


Then gradient descent updates the weights:
W.data += -learning_rate * W.grad


This process repeats many times.



## 20. Gradient Descent
The goal is to reduce the loss.
The gradient tells us which direction each parameter should move.

Formula:W = W - η ∂L/∂W

where:
W = weights  ,
L = loss  ,
η = learning rate

In simple code:
W.data += -learning_rate * W.grad



## 21. Learning Rate
The learning rate controls the size of each update.
For example:
gradient = 10

With:
learning_rate = 0.1

the change is:
1.0

With:
learning_rate = 0.001

the change is:
0.01

So:
large learning rate → big steps  ,
small learning rate → small steps



## 22. Regularization
We don't only want low training loss.
We also want reasonable weights.
Karpathy adds:
λ Σ W²

to the loss.
So:
loss = NLL + λ Σ W²

This is L2 regularization.
Its purpose is to discourage excessively large weights.


## 23. Why Regularization?
Without regularization, the model can make some weights extremely large.
That can make predictions excessively confident.
Regularization encourages the model to:
fit the data
    +
keep weights reasonable




## 24. Counting vs Neural Network
There are two approaches.
Counting
training data
    → 
count bigrams
    → 
normalize counts
    → 
probabilities

Neural Network
training data
    → 
one-hot
    → 
W
    → 
logits
    → 
softmax
    → 
probabilities
    → 
NLL
    → 
backpropagation
    → 
gradient descent
    → 
W

Both can learn essentially the same bigram distribution.
The neural-network approach gives us a path toward much more powerful models.



## 25. Complete Simple Version

```python
import torch
import torch.nn.functional as F

# Create training data
xs = []
ys = []

for word in words:
    chars = ["."] + list(word) + ["."]

    for ch1, ch2 in zip(chars, chars[1:]):
        ix1 = stoi[ch1]
        ix2 = stoi[ch2]
        xs.append(ix1)
        ys.append(ix2)

xs = torch.tensor(xs)
ys = torch.tensor(ys)

# One-hot encode input
xenc = F.one_hot(xs, num_classes=vocab_size).float()

# Initialize weights
W = torch.randn(vocab_size, vocab_size, requires_grad=True)
```





## 26. What Each Part Does

```python
xenc = F.one_hot(xs, num_classes=vocab_size).float()
```


One-hot encoding
```python
logits = xenc @ W
```


Raw model scores
```python
counts = logits.exp()
```


Exponentiation
```python
probs = counts / counts.sum(dim=1, keepdim=True)
```


Softmax / normalization
```python
loss = -probs[torch.arange(len(ys)), ys].log().mean()
```


Average Negative Log Likelihood
```python
loss += 0.01 * (W**2).mean()
```


L2 regularization
'''python
loss.backward()'''


Backpropagation
'''python
W.data += -0.1 * W.grad'''


Gradient descent



## 27. PyTorch Gives Us the Cleaner Version
Instead of manually calculating:
```python
counts = logits.exp()
probs = counts / counts.sum(dim=1,keepdim=True)
loss = -probs[torch.arange(len(ys)),ys].log().mean()```



we can use:
```python
loss = F.cross_entropy(logits, ys)```



CrossEntropyLoss handles the appropriate log-softmax and negative log-likelihood calculation.

Instead of manually updating:
```python
W.data += -0.1 * W.grad```

we normally use an optimizer:
```python
optimizer.zero_grad()
loss.backward()
optimizer.step()
```




## Key Takeaways
1. Makemore is a character-level language model.
2. A bigram predicts the next character from one previous character.
3. Bigram models can start with simple counting.
4. Counts can be converted into probabilities through normalization.
5. Likelihood measures how probable the training data is.
6. MLE tries to maximize that probability.
7. NLL converts this into a loss that can be minimized.
8. One-hot encoding represents characters numerically.
9. Weights → logits → softmax → probabilities forms the neural-network version.
10. Backpropagation calculates gradients.
11. Gradient descent updates the weights.
12. L2 regularization discourages excessively large weights.
13. PyTorch provides cleaner implementations through F.cross_entropy() and optimizers.