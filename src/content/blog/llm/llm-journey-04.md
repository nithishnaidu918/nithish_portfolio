---
title: "LLM Journey #4 - MLP"
description: "Understanding how Karpathy moves from a bigram model to an MLP that uses multiple previous characters as context."
date: "2026-10-07"
category: "LLM"
tags: ["LLM", "Transformers", "Tokenization"]
---

## LLM Journey #4 — Activations, Gradients & BatchNorm
Understanding what happens when the MLP becomes deeper, why gradients become unhealthy, and how Kaiming Initialization, BatchNorm, diagnostics, and PyTorchification help. Based on your Part 3 notes.    Pasted markdown

## 1. What Changes from Part 2?
In Part 2, we had:
Characters
    ↓
Embeddings
    ↓
Linear
    ↓
Tanh
    ↓
Linear
    ↓
Loss

Now Karpathy makes the network much deeper:
Embedding
    ↓
Linear
    ↓
BatchNorm
    ↓
Tanh
    ↓
Linear
    ↓
BatchNorm
    ↓
Tanh
    ↓
Linear
    ↓
BatchNorm
    ↓
Tanh
    ↓
...
    ↓
Output

The main question is:
Can activations and gradients remain healthy through many layers?


## 2. Same Dataset
The model still predicts the next character using 3 previous characters.
3 previous characters → next character

The dataset is divided into:
80% → training
10% → validation
10% → test


## 3. Deeper Network
Instead of one hidden layer, we now have multiple layers:
Input
  ↓
Linear
  ↓
Tanh
  ↓
Linear
  ↓
Tanh
  ↓
Linear
  ↓
Tanh
  ↓
Output

The purpose is to study what happens inside a deeper network.


## 4. The Problem: Bad Initialization
Weights start with random values.
But random does not automatically mean good initialization.
If a layer produces very large values:
2
5
10
-8
...

then tanh receives very large inputs.
This can cause problems.


## 5. Tanh Saturation
tanh produces values between -1 and 1.
For example:
```python
torch.tanh(torch.tensor(10.0))
```
is approximately:
1

Similarly:
tanh(-10) ≈ -1

When tanh gets close to -1 or 1, its derivative becomes very small.
The derivative is:
d/dx tanh(x) = 1 - tanh(x)²

So if:
tanh(x) ≈ 1

then:
1 - 1² = 0

Therefore:
Large activation
      ↓
Tanh saturation
      ↓
Very small gradient
      ↓
Learning becomes difficult

This is tanh saturation.    


## 6. Vanishing Gradients
Backpropagation passes gradients through the network.
If many layers produce very small gradients:
Output
  ↓
Small gradient
  ↓
Smaller gradient
  ↓
Even smaller
  ↓
...
  ↓
Early layers barely learn

This is called the vanishing-gradient problem.


## 7. First Fix: Smaller Weights
One simple solution is to initialize weights with a smaller scale.
Instead of:
```python
W = torch.randn(...)
```


we could use:
```python
W = torch.randn(...) * 0.2
```


This keeps the activations smaller and helps tanh stay in a healthier range.
But manually choosing 0.2 is only a hack.
We need a principled method.


## 8. Kaiming Initialization
Kaiming initialization chooses a suitable weight scale based on the number of inputs to a neuron.
A simplified formula is:
std(W) = gain / √fan_in

For the tanh network, Karpathy uses a gain of approximately:
5/3

Example:
```python
W1 = (
    torch.randn(n_embd * block_size, n_hidden)
    * (5 / 3)
    / ((n_embd * block_size) ** 0.5)
)
```
The goal is to keep activations at a sensible scale across layers.    Pasted markdown


## 9. What is Gain?
Different activation functions affect values differently.
So initialization uses a gain to adjust the scale.
For example:
tanh → gain ≈ 5/3
ReLU → gain ≈ √2

Main idea:
Kaiming initialization chooses a sensible starting weight scale based on fan-in and the activation function.

## 10. Initial Loss Calibration
The final layer can also produce very large logits.
That can make the model extremely confident before it has learned anything.
For example:
A → 0.9999
Everything else → almost 0

This can produce unnecessarily bad initial loss.
Karpathy reduces the scale of the final BatchNorm gain:
```python
layers[-1].gamma *= 0.1
```


This makes the initial predictions less extreme.    Pasted markdown


## 11. Batch Normalization
Now comes the main idea of Part 3.
Different layers can produce activations with different:
Mean
Variance
Scale

BatchNorm normalizes these activations.
Suppose a batch contains:
[10, 12, 14, 16]

Mean:
13

Subtract the mean:
[-3, -1, 1, 3]

Then divide by the standard deviation.
The values become roughly:
mean = 0
std = 1


## 12. BatchNorm Formula
First normalize:
x̂ = (x - μ) / √(σ² + ε)

Then BatchNorm learns two parameters:
y = γx̂ + β

So the process is:
x
 ↓
subtract mean
 ↓
divide by std
 ↓
γ ×
 ↓
+ β
 ↓
output


## 13. Gamma and Beta
BatchNorm learns:
γ → scale
β → shift

Why?
Because the network may want a distribution different from exactly:
mean = 0
std = 1

So BatchNorm gives the network control over the normalized values.    Pasted markdown


## 14. BatchNorm From Scratch
Karpathy implements the main idea manually:
```python
bnmeani = hpreact.mean(0, keepdim=True)
bnstdi = hpreact.std(0, keepdim=True)
hpreact = bngain * (hpreact - bnmeani) / bnstdi + bnbias
```


This is the core BatchNorm operation.


## 15. Why mean(0)?
Suppose:
```python
batch_size = 32
hidden_neurons = 200
```

Then:
```python
hpreact.shape == (32, 200)
```

We want to normalize each neuron across the batch.
So:
hpreact.mean(0)


calculates the mean for each of the 200 neurons across the 32 examples.


## 16. Running Mean and Standard Deviation
During training, BatchNorm can use the current batch.
But during inference, we may have only one example.
So Karpathy maintains:
running mean
running std

They are updated using a moving average:
```python
bnmean_running = 0.999 * bnmean_running + 0.001 * bnmeani
```


Training:
Use batch statistics

Inference:
Use running statistics



## 17. Training vs Evaluation
During training:
BatchNorm
    ↓
Current batch mean/std

During evaluation:
BatchNorm
    ↓
Running mean/std

This distinction is important when using BatchNorm.    Pasted markdown


## 18. Why Remove Linear Bias?
Normally:
y = xW + b

But BatchNorm immediately normalizes the output.
Therefore, the Linear bias becomes largely unnecessary.
So Karpathy uses:
```python
torch.nn.Linear(fan_in, fan_out, bias=False)
```


followed by BatchNorm.


## 19. Deep Architecture
The network becomes roughly:
Embedding
    ↓
Linear
    ↓
BatchNorm
    ↓
Tanh
    ↓
Linear
    ↓
BatchNorm
    ↓
Tanh
    ↓
Linear
    ↓
BatchNorm
    ↓
Tanh
    ↓
Output

The important pattern is:
Linear
   ↓
BatchNorm
   ↓
Tanh



## 20. Why BatchNorm Before Tanh?
We want to control the values going into tanh.
Without BatchNorm:
Linear
   ↓
Huge values
   ↓
Tanh saturation

With BatchNorm:
Linear
   ↓
BatchNorm
   ↓
Controlled values
   ↓
Tanh

This helps keep tanh in a healthier operating range.


## 21. Diagnostics
A major part of Part 3 is learning to inspect what is happening inside the network.
Karpathy checks:
1. Activations
2. Gradients
3. Weight distributions
4. Gradient-to-weight ratios
5. Update-to-weight ratios
These diagnostics help identify problems in deep networks.    Pasted markdown


## 22. Activation Statistics
Karpathy checks the outputs of the tanh layers.
For example:
```python
(t.abs() > 0.97).float().mean()
```


This tells us how many activations are close to -1 or 1.
If too many are saturated:
⚠️ Tanh saturation
      ↓
Small gradients
      ↓
Poor learning



## 23. Gradient Distribution
After:
```python
loss.backward()
```


we can inspect gradients.
For example:
```python
layer.out.grad
```


We want to know whether gradients are:
Too small?
Too large?
Healthy?

Very small gradients can cause vanishing gradients.
Very large gradients can cause exploding gradients.


## 24. Gradient:data Ratio
Karpathy also compares gradient size with parameter size.
Conceptually:
std(gradient)
----------------
std(weight)

This helps answer:
Is the gradient too large or too small compared with the parameter?



## 25. Update:data Ratio
The update is approximately:
```python
update = learning_rate * gradient
```

We compare:
update
-------
parameter

The goal is a reasonable update size.
Tiny update
    ↓
Slow learning

Huge update
    ↓
Unstable training

Reasonable update
    ↓
Healthy learning

This is the practical balance to look for.


## 26. Learning-Rate Decay
Karpathy reduces the learning rate later in training.
Example:
```python
lr = 0.1 if i < 150000 else 0.01
```


So:
Early training
→ larger steps

Later training
→ smaller steps

This is learning-rate decay.


## 27. Mini-Batches
Instead of using the entire dataset every iteration, Karpathy selects a small random batch.
```python
ix = torch.randint(0, Xtr.shape[0], (batch_size,))
```


For example:
batch_size = 32

The process becomes:
Dataset
   ↓
Random 32 examples
   ↓
Forward
   ↓
Loss
   ↓
Backward
   ↓
Update

This is mini-batch gradient descent.


## 28. Why Mini-Batches?
Mini-batches provide:
Less computation per step
Less memory usage
Faster training
Noisy but useful gradients



## 29. PyTorchification
Karpathy then starts turning his manually written operations into reusable classes.
He creates concepts such as:
Linear
BatchNorm1d
Tanh

This makes the network easier to build and manage.


## 30. Custom Linear Layer
Conceptually:
```python
class Linear:
    def __init__(self, fan_in, fan_out):
        self.weight = ...
        self.bias = ...

    def __call__(self, x):
        return x @ self.weight + self.bias
```


A Linear layer is essentially:
Matrix multiplication + bias



## 31. Custom Tanh Layer
Very simple:
```python
class Tanh:
    def __call__(self, x):
        return torch.tanh(x)
```


Now layers can be combined:
```python
layers = [
    Linear(...),
    BatchNorm1d(...),
    Tanh(),
    Linear(...),
    BatchNorm1d(...),
    Tanh(),
]
```



## 32. Why Is This Important?
Instead of manually managing:
W1
b1
W2
b2
BN1
BN2
...

we can simply have:
```python
for layer in layers:
    x = layer(x)
```


This is the basic idea behind reusable neural-network layers.


## 33. Collecting Parameters
Each layer can provide its trainable parameters.
Conceptually:
```python
parameters = [
    p
    for layer in layers
    for p in layer.parameters()
]
```


Now training can work with all parameters automatically.


## 34. PyTorch Equivalent
What Karpathy builds manually is similar to:
```python
model = torch.nn.Sequential(
    torch.nn.Linear(...),
    torch.nn.BatchNorm1d(...),
    torch.nn.Tanh(),
    torch.nn.Linear(...),
    torch.nn.BatchNorm1d(...),
    torch.nn.Tanh(),
    torch.nn.Linear(...),
)
```


The important idea is:
PyTorch layers are abstractions over operations we can understand and implement ourselves.



## 35. Training Still Uses Backpropagation
Even with the deeper network:
```python
loss.backward()
```
is still used.
The basic training loop remains:
```python
for p in parameters:
    p.grad = None

loss.backward()

for p in parameters:
    p.data += -lr * p.grad
```


The difference is that there are now many more layers and parameters.


## 36. Evaluation
After training, we evaluate the model using:
Training loss
Validation loss

During evaluation, BatchNorm uses:
Running mean
Running std

rather than the current batch statistics.


## 37. Sampling
Finally, the trained model can generate names.
The process is:
Context
   ↓
Model
   ↓
Probabilities
   ↓
Sample next character
   ↓
Update context
   ↓
Repeat

Conceptually:
```python
probs = F.softmax(logits, dim=1)
ix = torch.multinomial(probs, num_samples=1)
```
Then the context is updated and the process continues until the end token is generated.