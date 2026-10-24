---
title: "Fine-Tuning an LLM for Function Calling with LoRA"
description: "Learn how function calling works in AI agents, how LLMs learn to use tools, and how LoRA fine-tuning and special tokens help build function-calling models."
category: "AI Agents"
tags: ["Agentic AI", "LLM Fine-Tuning", "Function Calling", "LoRA", "Transformers", "SFT", "Python"]
---

# Fine-Tuning an LLM for Function Calling with LoRA

## Introduction

Function calling is an important capability in modern Large Language Models (LLMs). It allows a model to do more than generate text: it can produce structured requests that an AI application uses to execute functions, interact with external tools, and retrieve information.

In a typical AI agent, the model receives a user request, decides what action is required, calls a tool, observes the result, and continues working toward the objective.

However, there is an important distinction between an agent that is prompted to use tools and a model that has been specifically fine-tuned to produce function calls.

In a prompt-based approach, we provide the model with a list of available tools and rely on its existing capabilities to select and format the appropriate tool call.

With function-calling fine-tuning, we train the model on examples that teach it how to produce structured tool calls and interpret tool results.

In this article, we will explore:

1. How function calling works in LLMs.
2. The Thought → Act → Observe cycle.
3. How LLMs are trained and fine-tuned.
4. How LoRA reduces the cost of fine-tuning.
5. How special tokens help represent tool calls and observations.
6. How to approach fine-tuning a model for function calling.

The goal is to understand the concepts behind training a model to use tools, rather than relying only on prompt engineering.

## 1. What Is Function Calling?

Function calling is a technique that allows an LLM to request actions from its surrounding application by generating structured function calls.

For example, imagine asking an AI assistant:

"What's the status of my transaction T1001?"

The assistant may need to retrieve the transaction status from an external payment system. Instead of guessing the answer, it can generate a function call requesting the necessary information.

A simplified function might look like this:

```python
def retrieve_payment_status(transaction_id: str):
    # Retrieve the transaction status from a payment system.
    pass
```

The model can generate a structured request such as:

```json
{
  "name": "retrieve_payment_status",
  "arguments": {
    "transaction_id": "T1001"
  }
}
```

The surrounding application receives this request, executes the function, and returns the result to the model.

Suppose the tool returns:

```json
{
  "status": "Paid"
}
```

The model can then respond:

"Your transaction T1001 has been successfully paid."

The important point is that the LLM does not necessarily execute the function itself. It produces a structured request, and the application is responsible for executing the corresponding operation.

Function calling connects the language model's ability to understand instructions with the application's ability to interact with external systems.

### 1.1 Function Calling in AI Agents

In an AI agent, tools provide access to capabilities beyond text generation.

For example, an agent might have access to:

- A calculator.
- A web-search function.
- A database query function.
- A document-retrieval system.
- An API that retrieves payment information.

The agent needs to determine which tool is appropriate for the user's request and provide the correct arguments.

In an earlier prompt-based approach, the model receives descriptions of available tools and is expected to generalize from its existing training to use them.

Function-calling fine-tuning takes this a step further by training the model on examples of how to select tools, generate their arguments, and respond appropriately to tool results.

This does not eliminate the need for prompts, tool descriptions, or application logic. Instead, fine-tuning improves the model's ability to follow the expected function-calling format and behavior.

## 2. The Thought → Act → Observe Cycle

One of the fundamental patterns in agentic systems is the Thought → Act → Observe cycle.

It describes how an agent can reason about a task, perform an action, and use the resulting observation to decide what to do next.

### Step 1: Think

The model determines what information or action is required to fulfill the user's request.

For example:

"To answer the transaction-status question, I need to retrieve the status using the transaction ID."

This represents the decision to use a tool. It does not mean that every internal reasoning step must be exposed to the user.

### Step 2: Act

The model generates a structured function call with the appropriate name and arguments.

For example:

```json
{
  "name": "retrieve_payment_status",
  "arguments": {
    "transaction_id": "T1001"
  }
}
```

The application recognizes this as a tool request and executes the corresponding function.

### Step 3: Observe

The application returns the tool's result.

For example:

```json
{
  "status": "Paid"
}
```

The model receives this observation and uses it to continue the interaction.

It can now produce a final answer or make another tool call if additional information is required.

The complete cycle is:

```text
User Request
     |
     v
   Think
     |
     v
Generate Function Call
     |
     v
    Act
     |
     v
Execute External Tool
     |
     v
  Observe Result
     |
     v
Continue or Respond
```

This cycle is useful because the model can use actual tool results rather than relying exclusively on information contained in its parameters.

A model may repeat the cycle several times before completing a task.

For example, it could search for a document, retrieve its contents, calculate a value, and then produce a final response.

## 3. How Function Calling Changes Conversation Structure

In an ordinary conversation, messages commonly alternate between the user and the assistant.

Consider this example:

```python
conversation = [
    {
        "role": "user",
        "content": "I need help with my order."
    },
    {
        "role": "assistant",
        "content": "I'd be happy to help. Could you provide your order number?"
    },
    {
        "role": "user",
        "content": "It's ORDER-123."
    }
]
```

The assistant generates text in response to the user's messages.

When function calling is introduced, the conversation needs to represent additional information:

- The assistant's request to call a function.
- The function's name and arguments.
- The tool's result.
- The assistant's response after receiving the result.

A simplified conversation might look like this:

```python
conversation = [
    {
        "role": "user",
        "content": "What's the status of my transaction T1001?"
    },
    {
        "role": "assistant",
        "content": "",
        "function_call": {
            "name": "retrieve_payment_status",
            "arguments": {
                "transaction_id": "T1001"
            }
        }
    },
    {
        "role": "tool",
        "name": "retrieve_payment_status",
        "content": {
            "status": "Paid"
        }
    },
    {
        "role": "assistant",
        "content": "Your transaction T1001 has been successfully paid."
    }
]
```

This is a conceptual representation of a function-calling conversation. Actual APIs may use different message formats and fields.

Notice that the function call is still represented as an assistant message. The tool response is represented separately so the model can distinguish the requested action from the observation returned by the application.

The conversation therefore contains more than ordinary natural-language messages. It also represents structured actions and their results.

### 3.1 Why Are Special Tokens Important?

Language models generate sequences of tokens. During fine-tuning, we can teach a model to recognize and produce particular token sequences associated with function calling.

Some models use dedicated special tokens. Others use different structured formats defined by their model architecture or chat template.

For example, a function-calling format may use markers to represent:

- The available tools.
- The beginning of a function call.
- The function-call arguments.
- The returned tool results.
- The end of a tool observation.

In the course material, the following illustrative markers are introduced:

| Marker | Purpose |
|---|---|
| `[AVAILABLE_TOOLS]` | Begins the list of available tools |
| `[/AVAILABLE_TOOLS]` | Ends the list of available tools |
| `[TOOL_CALLS]` | Marks a function call |
| `[TOOL_RESULTS]` | Marks the tool result |
| `[/TOOL_RESULTS]` | Marks the end of the tool observation |

These markers are illustrative of the format described in the course. They should not be assumed to be the literal special tokens used by every model or API.

The central idea is that structured markers help distinguish ordinary conversation from actions and observations.

For example, the model needs to learn that generating a request to retrieve a transaction status is different from generating a sentence telling the user that the transaction is paid.

This distinction is essential for reliable tool use.

## 4. Understanding the LLM Training Process

Before fine-tuning a model for function calling, it is important to understand the different stages through which language models may be trained.

A simplified training process contains three stages.

### 4.1 Pre-Training

During pre-training, a model learns patterns in large quantities of text and other training data.

A common objective is next-token prediction: given a sequence of tokens, the model learns to predict the next token.

For example, given:

"The capital of France is"

the model learns to assign a high probability to an appropriate continuation.

Pre-training gives the model broad language and knowledge capabilities. However, a base model does not necessarily follow instructions reliably or produce the structured function calls expected by an application.

### 4.2 Instruction Fine-Tuning

After pre-training, a model can be fine-tuned on examples of instructions and appropriate responses.

This helps the model learn to respond to user requests in a conversational format.

For example, an instruction-tuning dataset may contain:

**Instruction:**

"Explain what a database is."

**Expected response:**

"A database is an organized collection of information that can be stored, retrieved, and managed."

Instruction fine-tuning helps transform a general-purpose base model into a model that is more useful for following instructions.

In the Gemma model family used in the course, `google/gemma-2-2b` is the base model, while `google/gemma-2-2b-it` is the instruction-tuned version.

### 4.3 Alignment

A model may undergo additional training to encourage its behavior to follow desired preferences, safety requirements, or product-specific expectations.

For example, a customer-service assistant may be trained to provide helpful and respectful responses.

The exact alignment methods vary by model and developer.

Not every model follows exactly the same training pipeline, and these stages are a simplified way to understand the broader process.

### 4.4 Why Start with an Instruction-Tuned Model?

For this tutorial, we start with an instruction-tuned model rather than a base model.

The objective is to teach the model how to use tools, not to teach it every aspect of conversational behavior from the beginning.

An instruction-tuned model already has some ability to follow user requests and generate conversational responses.

Fine-tuning it for function calling allows us to focus more directly on structured tool use.

The model still needs appropriate training examples to learn the new behavior.

## 5. Fine-Tuning an LLM for Function Calling

How do we train a model to use functions?

The answer begins with **training data**.

A model learns the desired behavior from examples that demonstrate how it should respond in different situations.

For function calling, the dataset should contain examples of user requests, available tools, function calls, tool results, and appropriate follow-up responses.

A simplified training example could look like this:

```python
training_example = {
    "user_request": "What is the status of transaction T1001?",
    "available_tools": [
        {
            "name": "retrieve_payment_status",
            "description": "Retrieve a transaction's payment status",
            "parameters": {
                "transaction_id": "string"
            }
        }
    ],
    "assistant_action": {
        "name": "retrieve_payment_status",
        "arguments": {
            "transaction_id": "T1001"
        }
    },
    "tool_result": {
        "status": "Paid"
    },
    "assistant_response": (
        "Your transaction T1001 has been successfully paid."
    )
}
```

This example demonstrates the intended workflow.

The model should learn to:

1. Understand the user's request.
2. Identify the appropriate tool.
3. Generate the correct function name.
4. Provide valid arguments.
5. Use the tool result to formulate a response.

In practice, a fine-tuning dataset must be formatted according to the model's chat template and the training objective.

The model is trained to predict the expected output tokens in these examples. It does not automatically execute the external tool during ordinary language-model training.

Instead, the application executes the generated call when the model is running.

### 5.1 Why Training Data Quality Matters

Suppose a dataset contains examples where the model calls the wrong function or generates invalid arguments.

Fine-tuning on those examples can teach undesirable behavior.

A good dataset should therefore include:

- Clear descriptions of available functions.
- Correct function names and argument structures.
- Examples of different user requests.
- Tool results in the expected format.
- Examples where no tool call is necessary.
- Appropriate responses after tool execution.

It is also useful to evaluate the model on examples it did not see during training.

This helps determine whether it has learned the general function-calling pattern rather than merely memorizing specific examples.

## 6. LoRA: Low-Rank Adaptation

Fine-tuning a large language model can require substantial computing resources.

A model may contain billions of parameters, and updating all of them can consume significant memory and training time.

LoRA, or Low-Rank Adaptation, is a parameter-efficient fine-tuning technique that reduces the number of trainable parameters.

Instead of updating all the original model weights, LoRA introduces small trainable matrices into selected layers while keeping the original weights frozen.

### 6.1 How LoRA Works

Suppose a Transformer layer contains a weight matrix \(W\).

In conventional full fine-tuning, the training process updates the original weights.

With LoRA, we keep the original matrix frozen and learn a low-rank update:

\[
W' = W + \Delta W
\]

The update can be expressed as:

\[
\Delta W = BA
\]

where \(A\) and \(B\) are smaller trainable matrices.

For a weight matrix with dimensions \(d \times k\), one common arrangement is:

\[
A \in \mathbb{R}^{r \times k}
\]

\[
B \in \mathbb{R}^{d \times r}
\]

Here, \(r\) is the rank of the adaptation, typically chosen to be much smaller than \(d\) and \(k\).

The number of parameters in the two adapter matrices is:

\[
r k + d r = r(k+d)
\]

By comparison, the original matrix contains:

\[
d k
\]

parameters.

When the rank is small, the number of trainable parameters can be much smaller than the number of parameters in the original matrix.

In practice, LoRA implementations commonly apply a scaling factor to the update. The exact configuration depends on the model and training setup.

### 6.2 What Happens During Training?

The LoRA training process can be understood in four steps:

1. Load the pre-trained model.
2. Freeze the original model parameters.
3. Insert LoRA adapters into selected layers.
4. Train the adapter parameters using the function-calling dataset.

The original model retains its learned knowledge, while the adapters learn adjustments that help the model perform the target task.

This can substantially reduce the memory needed to store trainable gradients and optimizer states compared with full fine-tuning.

### 6.3 What Happens During Inference?

After training, the model can be used with its trained LoRA adapters.

Depending on the implementation, the adapter weights can also be merged into the base model weights.

When the adapters are merged appropriately, they need not introduce a separate adapter computation during inference.

LoRA adapters are often much smaller than the original model, making them convenient to save, share, and deploy.

However, LoRA does not eliminate every resource requirement. The base model must still be loaded or otherwise made available, and training still requires appropriate compute and memory.

### 6.4 Why Use LoRA for Function Calling?

LoRA is useful because the objective is to adapt an existing model to a more specific behavior.

In this case, the desired behavior is to produce structured function calls and respond appropriately to tool observations.

Instead of updating every parameter in the model, we can train a smaller set of adapter parameters.

This makes experimentation more accessible, particularly when adapting a model on limited hardware.

## 7. Choosing a Model for Function-Calling Fine-Tuning

The course uses the instruction-tuned Gemma 2 model with approximately two billion parameters as its starting point.

The key distinction is between:

- A base model, which has primarily learned general token prediction.
- An instruction-tuned model, which has already been trained to follow instructions.

Starting from an instruction-tuned model reduces the amount of new behavior that the function-calling fine-tuning process needs to teach.

The target model must still learn the required output format, tool selection behavior, and argument structure from the training examples.

When selecting a model, consider:

- Its existing instruction-following ability.
- Its supported context length.
- Its compatibility with your training framework.
- Its license and deployment requirements.
- Its memory requirements.
- The function-calling format you intend to use.

A smaller model can be easier to experiment with, although its performance will depend on the task, training data, and evaluation setup.

## 8. Putting the Concepts Together

Let's revisit the transaction-status example.

The user asks:

"What's the status of my transaction T1001?"

The model has been fine-tuned on examples of function calling.

It receives a conversation containing the user's request and a description of the available tools.

The model generates a structured function call:

```json
{
  "name": "retrieve_payment_status",
  "arguments": {
    "transaction_id": "T1001"
  }
}
```

The application validates the request and executes the corresponding tool.

The tool returns:

```json
{
  "status": "Paid"
}
```

The application provides this observation to the model, which generates the final response:

"Your transaction T1001 has been successfully paid."

The model's fine-tuning helps it produce the appropriate action format. The application remains responsible for executing the function, managing permissions, and returning the tool result.

This separation between model behavior and application execution is fundamental to building reliable function-calling systems.

## 9. Common Misconceptions

### Misconception 1: The model executes the function itself

Not necessarily. The model generally generates a structured function-call request. The surrounding application executes the function and returns the result.

### Misconception 2: Function calling eliminates the need for prompts

Function-calling fine-tuning can improve the model's ability to use tools, but the application still needs suitable instructions, tool definitions, and conversation context.

### Misconception 3: LoRA trains the entire model

LoRA typically freezes the original model weights and trains additional low-rank adapter parameters.

### Misconception 4: Fine-tuning automatically makes an agent reliable

Fine-tuning can improve behavior, but reliability also depends on training-data quality, tool validation, error handling, permissions, and evaluation.

### Misconception 5: All models use the same function-calling tokens

Different models and APIs can use different message formats, chat templates, and special tokens. The format used during training must be compatible with the model and its intended inference environment.

## 10. What You Should Remember

The main concepts from this article are:

- Function calling allows an LLM to request structured actions from an application.
- The Thought → Act → Observe cycle describes how an agent decides on an action, executes it through a tool, and uses the resulting observation.
- Function-calling conversations represent tool requests and tool responses separately from ordinary user-facing text.
- Pre-training provides general language capabilities, instruction fine-tuning improves instruction following, and additional alignment may shape model behavior.
- Fine-tuning uses training examples to teach a model how to select tools and generate appropriate arguments.
- LoRA reduces the number of trainable parameters by introducing low-rank adapters into selected model layers.
- The application executes function calls and returns tool results; fine-tuning does not remove this responsibility.
- Evaluation, validation, and security controls remain essential for reliable tool use.