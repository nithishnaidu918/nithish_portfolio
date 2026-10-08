---
title: "LLM Journey #3 - MLP"
description: "Understanding how Karpathy moves from a bigram model to an MLP that uses multiple previous characters as context."
category: "LLM"
tags: ["LLM", "Transformers", "Tokenization"]
---


## LLM Journey #3 — MLP and Context in Language Models
Understanding how Karpathy moves from a bigram model to an MLP that uses multiple previous characters as context.



## 1. What is MLP?
In Journey #2, the model used:
1 character → next character

Now we use multiple previous characters:
3 characters → next character

Example:
e m m → ?

This gives the model more context.



## 2. Why Do We Need Context?
With only:
a → ?

the model has very little information.
But with:
m a r → ?

the model has more context to predict the next character.
So:
Bigram:
a → ?

MLP:
m a r → ?




## 3. Creating Training Examples
For the word:
emma

we add the start token:
. . . e m m a .

Using a context size of 3:
input       target

...    →     e   

..e    →     m

.em    →     m

emm    →     a

 mma    →     .

The model learns:
3 characters → next character




## 4. X and Y
X contains the input context.
Y contains the correct next character.

X → input
Y → target

For example:
X = [
    [0, 0, 0],
    [0, 0, 5],
    [0, 5, 13]
]

Y = [
    5,
    13,
    13
]




## 5. Embeddings
In Journey #2, we used one-hot encoding.
Now each character gets a small learned vector called an embedding.

Example:
e → [0.21, -0.13]  ,
m → [0.72,  0.31]  ,
a → [-0.4, 0.55]

Instead of a large one-hot vector, the model learns a compact representation for each character.



## 6. Embedding Matrix C
Karpathy creates an embedding matrix:
```python
C = torch.randn(vocab_size,embedding_dim)```



For example:
vocab_size = 27  ,
embedding_dim = 2

Then:
C.shape = (27, 2)

So C is basically a lookup table:

27 characters
      → 
each character
      → 
2 numbers




## 7. Looking Up Embeddings
Suppose:
ix = 5

Then:
C[ix]


returns the embedding for character 5.
For a context:
[5, 13, 13]

we get the embeddings for all three characters.



## 8. Concatenation
Suppose:
m → [0.2, 0.5]  ,
a → [0.7, 0.1]  .
r → [0.3, 0.8]

For:
m a r

we concatenate them:
[0.2, 0.5, 0.7, 0.1, 0.3, 0.8]

So:
3 characters × 2 embedding values = 6 values

This becomes the input to the MLP.



## 9. MLP Architecture
The basic flow is:
characters
    → 
embeddings
    → 
concatenate
    → 
Linear layer
    → 
tanh
    → 
Linear layer
    → 
logits
    → 
probabilities
    → 
loss




## 10. First Linear Layer
Suppose the concatenated embedding has 6 values.
We create:
```python
W1 = torch.randn(6, 100)
b1 = torch.randn(100)```

Then:
h = x @ W1 + b1

So:
6 inputs
   → 
100 neurons




## 11. Why tanh?
After the first linear layer:
```python
h = torch.tanh(h)```


tanh is an activation function.
It introduces non-linearity, allowing the network to learn more complex relationships.
Linear
   ↓
tanh





## 12. Second Linear Layer
Next:
```python
W2 = torch.randn(100, vocab_size)
b2 = torch.randn(vocab_size)```


Then:
logits = h @ W2 + b2


If there are 27 possible characters:
100 neurons
    → 
27 outputs

Each output represents one possible next character.



## 13. Softmax and Loss
The logits are converted into probabilities.
logits
   → 
softmax
   → 
probabilities

Then we calculate the loss:
```python
loss = F.cross_entropy(logits, Y)```


The goal is:
high probability for correct character
            → 
        low loss




## 14. Training
The entire network is trained using backpropagation.
loss.backward()


Gradients are calculated for:
C  ,
W1  ,
b1  ,
W2  ,
b2

Then the parameters are updated.
parameters
     → 
gradients
     → 
parameter update

So now the embeddings and neural-network weights are all learned.



## 15. PyTorch Version
The MLP can be represented more cleanly:
```python
model = torch.nn.Sequential(torch.nn.Linear(6, 100),torch.nn.Tanh(),torch.nn.Linear(100, 27))```


Training:
```python
logits = model(x)
loss = F.cross_entropy(logits, Y)
optimizer.zero_grad()
loss.backward()
optimizer.step()```


This follows the standard PyTorch training pattern.



## 16. Core Code
```python
import torch
import torch.nn.functional as F
#Parameters
vocab_size = 27
embedding_dim = 10
hidden_size = 200
#Parameters to learn
C = torch.randn(vocab_size,embedding_dim)
W1 = torch.randn(embedding_dim * 3,hidden_size)
b1 = torch.randn(hidden_size)
W2 = torch.randn(hidden_size,vocab_size)
b2 = torch.randn(vocab_size)
parameters = [C,W1,b1,W2, b2]

# Embedding lookup
emb = C[X]

# Concatenate the 3 character embeddings
embcat = emb.view(emb.shape[0], -1)

# First layer
hpreact = embcat @ W1 + b1

# Non-linearity
h = torch.tanh(hpreact)

# Output layer
logits = h @ W2 + b2

# Loss
loss = F.cross_entropy(logits, Y)

print(loss)

# Backpropagation
for p in parameters:
    p.grad = None

loss.backward()

# Update parameters
learning_rate = 0.1

for p in parameters:
    p.data += -learning_rate * p.grad
    
for step in range(10000):

    # Forward pass
    emb = C[X]
    embcat = emb.view(emb.shape[0], -1)

    h = torch.tanh(embcat @ W1 + b1)
    logits = h @ W2 + b2

    # Loss
    loss = F.cross_entropy(logits, Y)

    # Backward pass
    for p in parameters:
        p.grad = None

    loss.backward()

    # Update
    for p in parameters:
        p.data += -0.1 * p.grad