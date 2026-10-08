---
title: "LLM Journey #1 - Foundations"
description: "Beginning a structured journey through the foundations of Large Language Models."
category: "LLM"
tags: ["LLM", "Transformers", "Tokenization"]
---

## LLM Journey #1

This journey focuses on the foundation behind neural-network training: autograd, derivatives, computational graphs, and backpropagation.



## 1. What is Karpathy Building?

In micrograd, Karpathy builds a very small version of backpropagation/autograd from scratch.

Forward Pass
    → 
Calculate Output
    → 
Calculate Loss
    → 
Backward Pass
    → 
Calculate Gradients
    → 
Update Weights

Main concepts:

Computational graph

Derivatives

Chain rule

Gradients

Backpropagation



## 2. Derivative

Suppose:

x = 3 , 
y = x * x

Mathematically:

y = x²

Derivative:

dy/dx = 2x

At x = 3:

dy/dx = 6

A derivative tells us:

If the input changes slightly, how does the output change?




## 3. Chain Rule

The chain rule is the core of backpropagation.

x → y → z

dz/dx = (dz/dy) × (dy/dx)

Example:

x = 2 ,
y = x * 3 ,
z = y * 4

Therefore:

dy/dx = 3 ,
dz/dy = 4

dz/dx = 4 × 3
      = 12

The gradient travels backward by multiplying local derivatives.




## 4. Computational Graph

Consider:

a = 2 ,
b = 3 ,
c = 4

d = a + b ,
e = d * c

Forward:

Inputs → Operations → Output

Backward:

Output → Gradients → Previous values

This is the basic idea behind backpropagation.




## 5. Addition Backward

For:

z = a + b

dz/da = 1 ,
dz/db = 1

So:

a.grad += dz ,
b.grad += dz




## 6. Multiplication Backward

For:

z = a * b

dz/da = b ,
dz/db = a

So:

a.grad += dz * b ,
b.grad += dz * a

This is the chain rule being applied locally.



## 7. Why +=?

A value can affect the output through multiple paths.

Example:

z = a + a

There are two paths from a to z.

Therefore the gradients must be accumulated:

a.grad += ... ,
a.grad += ...

This is why gradients accumulate.




## 8. tanh

Karpathy also uses tanh as an example of a nonlinear operation.

Forward:

y = tanh(x)

Derivative:

dy/dx = 1 - y²

Backward:

x.grad += y.grad * (1 - y**2)

The pattern is:

Upstream Gradient
       ×
Local Derivative
       → 
Input Gradient




## 9. What Does Micrograd Store?

Micrograd creates a computational graph. 

Each value keeps:

data
gradient
previous nodes
backward function

Conceptually:

a = Value(2) ,
b = Value(3)

c = a + b ,
d = c * 4

d.backward()

backward() walks backward through the graph and applies the chain rule.




## 10. Simplified Micrograd Code
'''python
class Value:

    def __init__(self, data, _children=()):
        self.data = data
        self.grad = 0
        self._prev = set(_children)
        self._backward = lambda: None

    def __add__(self, other):

        out = Value(
            self.data + other.data,
            (self, other)
        )

        def backward():
            self.grad += out.grad
            other.grad += out.grad

        out._backward = backward
        return out

    def __mul__(self, other):

        out = Value(
            self.data * other.data,
            (self, other)
        )

        def backward():
            self.grad += other.data * out.grad
            other.grad += self.data * out.grad

        out._backward = backward
        return out'''

The important part:

self.grad += ...   ,
other.grad += ...

This is gradient propagation.




## 11. PyTorch Does This Automatically

The same idea can be seen in PyTorch:

import torch

x = torch.tensor(2.0, requires_grad=True)

y = x * 3  ,
z = y * 4

z.backward()

print(x.grad)

Output:

tensor(12.)

Because:

dz/dx = 4 × 3 = 12

PyTorch's autograd calculates this automatically.



## 12. PyTorch Training Loop

The basic PyTorch training order is:

optimizer.zero_grad()

output = model(x)

loss = loss_function(output, y)

loss.backward()

optimizer.step()

Step 1 — Clear Gradients

optimizer.zero_grad()

Clear gradients from the previous iteration.

Step 2 — Forward Pass

output = model(x)

Generate predictions.

Step 3 — Calculate Loss

loss = loss_function(output, y)

Measure prediction error.

Step 4 — Backward Pass

loss.backward()

Calculate gradients through the computational graph.

Step 5 — Update Parameters

optimizer.step()

Update the weights using the gradients.



## 13. Complete PyTorch Example
```python
import torch
import torch.nn as nn
import torch.optim as optim
x = torch.tensor([[1.0]])
y = torch.tensor([[2.0]])
model = nn.Linear(1, 1)
loss_function = nn.MSELoss()
optimizer = optim.SGD(model.parameters(), lr=0.01)
for epoch in range(100):
    optimizer.zero_grad()
    output = model(x)
    loss = loss_function(output, y)
    loss.backward()
    optimizer.step()
    print(loss.item())```