---
title: "Agent Fundamentals, LLMs, Tools & Workflows"
description: "A comprehensive introduction to AI agents, LLMs, chat templates, the ReAct cycle, tool calling, observations, and building blocks for agentic systems."
date: "2026-10-16"
category: "AI Agents"
tags: ["Agentic AI", "AI Agents", "LLM", "ReAct", "Tool Calling", "Hugging Face", "Python", "smolagents"]
---

# AI Agent Journey #1 — Agent Fundamentals, LLMs, Tools & Workflows

## Introduction

This learning journey introduces the foundations of AI agents. Before building an agent with a framework such as `smolagents`, I need to understand:

1. What an AI agent is and what makes a system agentic.
2. How Large Language Models (LLMs) work.
3. How messages and special tokens structure a conversation.
4. How agents decide what action to take.
5. How tools allow an LLM to interact with its environment.
6. How observations provide feedback after an action.
7. How to implement a basic agent workflow in Python.

The central idea is that **an LLM generates text, while an agent system connects the model to tools, executes actions, collects results, and manages the overall task.**

# Part 1 — What Is an AI Agent?

## 1. The Alfred Analogy

Imagine an assistant named Alfred. I ask:

> Alfred, I would like a coffee, please.

Alfred must understand the request, plan the required steps, use the necessary equipment, and complete the task.

```text
User requests coffee
        ↓
Understand the request
        ↓
Reason and plan
        ↓
Go to the kitchen
        ↓
Use the coffee machine
        ↓
Brew the coffee
        ↓
Bring the coffee to the user
```

Alfred does not merely describe how to make coffee. He can interact with his environment and carry out the task.

This is the key difference between a system that only generates an answer and a system that can take actions.

## 2. Formal Definition of an Agent

An **AI agent** is a system that uses an AI model to interact with its environment to achieve a user-defined objective. It combines reasoning, planning, and action execution, often through external tools.

An agent can be understood through two main components.

### The brain — AI model

The AI model interprets instructions, analyzes available information, decides what to do next, and may plan a sequence of actions.

An LLM is commonly used as the brain of a text-based agent.

### The body — capabilities and tools

The body represents what the agent can actually do.

Examples include:

- Searching the web.
- Reading files.
- Querying a database.
- Running Python code.
- Calling an API.
- Sending an email.
- Generating an image.
- Controlling a device.

An agent cannot perform an action unless its system provides a capability for it. A model may describe how to send an email, but it cannot actually send one unless an email tool or equivalent integration is available and permitted.

**Important:** An AI model and an AI agent are not the same thing. The model provides intelligence; the surrounding system provides the capabilities and execution loop.

## 3. The Spectrum of Agency

Not every system that uses an LLM is equally agentic. Agency can be understood as a spectrum.

| Level | What the model's output controls | Example |
|---|---|---|
| No control flow | The output is processed as ordinary text | Summarize an LLM response |
| Basic routing | The output determines which branch executes | Choose between a SQL path and a Python path |
| Tool calling | The output selects a function and its arguments | Call a calculator |
| Multi-step agent | The output influences repeated actions and whether to continue | Search, inspect results, search again |
| Multi-agent workflow | One agentic workflow can trigger another | A research agent delegates analysis to another agent |

A simple processor might look like this:

```python
def process_llm_output(llm_response):
    return llm_response.strip()
```

Here, the LLM output does not decide which operation the program executes.

A router gives the output some control over program flow:

```python
if llm_decision() == "research":
    run_research()
else:
    run_other_task()
```

A tool-calling system allows the model to select an operation:

```python
run_function(
    llm_chosen_tool,
    llm_chosen_arguments,
)
```

A multi-step agent repeats actions until the task is completed or another stopping condition is reached:

```python
while should_continue():
    action = choose_next_action()
    observation = execute_action(action)
```

These are conceptual examples, not complete implementations. In a real system, the model's proposed action must be validated before execution.

The key distinction is that **greater agency means the model's decisions influence more of the program's execution**, rather than merely determining the final text.

## 4. Which AI Models Can Power Agents?

LLMs are the most common model type used in text-based agents. They receive text and generate text.

Examples include GPT models from OpenAI, Llama from Meta, Gemini from Google, and other open or proprietary models.

Agents can also use other model types. A Vision-Language Model (VLM), for example, can process images as well as text.

For this introductory journey, the focus is on LLM-based agents.

# Part 2 — Understanding Large Language Models

## 5. What Is an LLM?

A **Large Language Model (LLM)** is an AI model trained on large amounts of data to learn patterns in language and generate text.

LLMs can perform tasks such as:

- Answering questions.
- Summarizing documents.
- Generating code.
- Translating text.
- Following instructions.
- Helping an agent decide which tool to use.

A model contains learned parameters. During training, these parameters are adjusted so the model becomes better at its training objective.

Many modern LLMs use the Transformer architecture, which relies heavily on the attention mechanism.

## 6. Three Main Transformer Architectures

The original Transformer architecture combined an encoder and a decoder. Modern Transformer models can be grouped into three broad architectural types.

| Architecture | Main purpose | Examples | Common applications |
|---|---|---|---|
| Encoder-only | Build contextual representations of input | BERT | Classification, semantic search, named-entity recognition |
| Decoder-only | Generate tokens autoregressively | GPT-style models, Llama | Text generation, chatbots, code generation |
| Encoder–decoder | Transform an input sequence into an output sequence | T5, BART | Translation, summarization, paraphrasing |

### Encoder-only models

An encoder processes input and produces contextual representations, often called embeddings or hidden states.

For example, a text classifier can use an encoder representation to predict whether a review is positive or negative.

### Decoder-only models

A decoder-only language model predicts the next token based on the preceding context.

This makes it suitable for generating text one token at a time.

### Encoder–decoder models

The encoder processes the input, and the decoder generates an output conditioned on that input.

For example, a translation model can encode an English sentence and decode its French translation.

These are broad categories. The best architecture depends on the model's training objective and intended task.

## 7. Next-Token Prediction

A central training objective for many decoder-only LLMs is **next-token prediction**.

Given a sequence of tokens, the model learns to predict the next token.

```text
Input:  The capital of France is
                         ↓
                    LLM predicts
                         ↓
Output candidate: Paris
```

The model does not directly jump from raw text to a complete answer. It generates a sequence of tokens, with each new token conditioned on the preceding context.

This is called **autoregressive generation**.

## 8. What Is a Token?

A token is a unit of text processed by the model. Depending on the tokenizer, it may represent a word, part of a word, punctuation, whitespace-related text, or another piece of text.

For example, a tokenizer might split:

```text
interesting
```

into pieces resembling:

```text
interest | ing
```

Another tokenizer might split the same text differently.

Subword tokenization lets a model represent common word fragments without requiring every possible word to have its own vocabulary entry.

The vocabulary size varies by model. A model with a vocabulary of roughly 32,000 tokens, for example, can represent many words by combining smaller pieces.

## 9. Special Tokens

Besides ordinary text tokens, language models use special tokens to represent structural information.

Depending on the model, special tokens can mark:

- The beginning of a sequence.
- The end of a message or sequence.
- The start of a new speaker's message.
- The start of an assistant response.
- Tool calls or other structured interactions.

Some examples of model-specific end markers are:

| Model or family | Example marker |
|---|---|
| GPT-style models | `<|endoftext|>` in certain tokenization and training contexts |
| Llama 3 | `<|eot_id|>` |
| DeepSeek models | Model-specific end markers |
| SmolLM2 chat format | `<|im_end|>` |
| Gemma | Model-specific turn-ending markers |

The exact meaning of an end marker depends on the model and its format. An end-of-message marker, end-of-turn marker, and end-of-sequence marker should not automatically be assumed to mean the same thing.

I do not need to memorize every model's special tokens. I need to understand that the tokenizer and model expect a specific format.

## 10. How Autoregressive Decoding Works

During generation, the model produces logits: a score for each possible next token.

A decoding strategy converts these scores into a choice of token.

```text
Prompt
  ↓
Tokenization
  ↓
Model processes the tokens
  ↓
Logits for the next token
  ↓
Decoding strategy selects a token
  ↓
Append the selected token
  ↓
Run the next generation step
  ↓
Stop when the generation condition is met
```

The process continues until a stopping condition is reached, such as a configured maximum token count or a relevant end token.

### Greedy decoding

Greedy decoding always selects the token with the highest score.

Conceptually:

```python
next_token = logits.argmax(dim=-1)
```

It is simple, but the locally highest-scoring token does not always lead to the best overall sequence.

### Sampling

Sampling chooses a token according to a probability distribution derived from the model's scores.

This can produce more varied outputs than greedy decoding.

### Beam search

Beam search tracks multiple candidate sequences and expands promising candidates. It can be useful for certain generation tasks, although it is not the standard decoding approach for every modern chat LLM.

## 11. What Is Attention?

Attention allows a Transformer to combine information from different positions in the input sequence.

Consider:

```text
The capital of France is ...
```

When predicting the next token, the model can use information associated with both `capital` and `France`.

Attention computes contextual relationships between tokens. It does not simply assign one permanent importance score to each word; the relationships depend on the input and the layer's learned computations.

Attention has been a central part of Transformer models since the original Transformer architecture.

## 12. Context Length

The **context length** is the maximum number of tokens a model can process in a single context, subject to the model and runtime configuration.

A conversation can contain:

- System instructions.
- Earlier user messages.
- Earlier assistant messages.
- Tool calls and their results.
- The latest user request.

All this content can consume context tokens.

If the conversation exceeds the available context window, the application may need to truncate, summarize, or otherwise manage the history.

## 13. Why Prompting Matters

The input given to an LLM is called a **prompt**.

Because the model predicts its output based on the supplied context, instructions, examples, formatting, and relevant background information can influence the response.

For an agent, a prompt may need to describe:

- The objective.
- The agent's role and behavior.
- Available tools.
- Tool arguments and expected results.
- When to call a tool.
- How to report the final answer.

A carefully designed prompt helps guide the model, but it does not guarantee correctness. Tool execution, validation, and evaluation remain important.

## 14. How LLMs Are Trained

LLMs commonly go through more than one training stage.

### Pretraining

During pretraining, a model learns from a large dataset using an objective such as next-token prediction.

This helps it learn statistical patterns in text, language structure, and other information represented in the training data.

### Fine-tuning

A pretrained model can be trained further on selected examples to improve performance for particular tasks.

Examples include instruction following, conversational behavior, code generation, and tool-use patterns.

A **base model** and an **instruction-tuned model** can therefore behave differently, even when they belong to the same model family.

A base model primarily learns to continue text. An instruction-tuned model has additionally been trained to respond to instructions in a more useful conversational format.

## 15. How Can We Use LLMs?

Two common approaches are:

### Run a model locally

Download and run a model on your own hardware.

Advantages include more control over deployment and potentially keeping data within your environment. The required memory, compute, and supported model formats depend on the model.

### Use a hosted API

Send requests to a model hosted by a provider.

This avoids managing the model's full inference infrastructure yourself, but depends on the provider's API, access permissions, costs, and service availability.

In this course, the examples use the Hugging Face ecosystem and its inference interfaces. Later sections introduce actual API calls and agent frameworks.

## 16. How LLMs Power Agents

An LLM can help an agent:

- Interpret the user's instruction.
- Decide which information is missing.
- Plan a sequence of steps.
- Select an available tool.
- Interpret the tool's output.
- Decide whether another action is necessary.
- Produce a final response.

However, the LLM does not automatically have access to every external system. The agent application must provide and execute the required tools.

**The LLM is the reasoning and language component; the agent framework coordinates its interaction with the environment.**

# Part 3 — Messages, Special Tokens, and Chat Templates

## 17. Why a Chat Is Converted into a Prompt

A chat interface displays separate messages, but the model must receive an input representation that follows its expected format.

A conversation might look like this in an application:

```python
conversation = [
    {"role": "user", "content": "I need help with my order"},
    {
        "role": "assistant",
        "content": "Could you provide your order number?",
    },
    {"role": "user", "content": "It's ORDER-123"},
]
```

The application formats the messages according to the selected model's chat template before sending them to the model.

The model processes the provided context for the current request. It does not automatically remember a previous API call unless the application supplies the relevant history or uses an appropriate memory mechanism.

## 18. The Three Main Message Roles

### System message

The system message defines the assistant's general behavior and instructions.

```python
system_message = {
    "role": "system",
    "content": (
        "You are a professional customer service agent. "
        "Always be polite, clear, and helpful."
    ),
}
```

For an agent, system instructions can also describe the available tools, required output format, and rules for deciding when to take actions.

Changing the system instructions can change the behavior the application requests from the model. It does not guarantee that the model will follow every instruction perfectly.

### User message

The user message contains the task or question.

```python
user_message = {
    "role": "user",
    "content": "I need help with my order.",
}
```

### Assistant message

The assistant message contains a response from the model or, in some formats, a structured action request.

```python
assistant_message = {
    "role": "assistant",
    "content": "Could you provide your order number?",
}
```

Conversations usually contain multiple messages, and the chat template determines how they are formatted for the specific model.

## 19. Chat Templates

A **chat template** converts a structured list of messages into the format expected by a model.

For example, a conversation with a system message, a user question, and an assistant response may be rendered with message-boundary tokens:

```text
<|im_start|>system
You are a helpful assistant.<|im_end|>
<|im_start|>user
What is a chat template?<|im_end|>
<|im_start|>assistant
A chat template formats messages for a language model.<|im_end|>
```

This is an illustrative ChatML-style format. Other models use different tokens and formatting rules.

A Llama-family template, for example, may use different message-header and turn-ending tokens. The same underlying conversation must be rendered according to the chosen model's format.

## 20. Base Models vs. Instruct Models

A base model is pretrained to learn from text, often through next-token prediction.

An instruction-tuned model is additionally trained to respond to instructions and conversational tasks.

For example, a model family may provide both a base version and an instruction-tuned version.

Chat templates are particularly important for instruction-tuned models because they define how system, user, and assistant messages should be presented.

A correct template helps the model recognize the intended conversational structure. An incorrect template can lead to unexpected behavior.

## 21. How to Apply a Chat Template with Transformers

The Hugging Face Transformers library provides a tokenizer method for formatting message lists.

```python
from transformers import AutoTokenizer

model_id = "HuggingFaceTB/SmolLM2-1.7B-Instruct"

tokenizer = AutoTokenizer.from_pretrained(model_id)

messages = [
    {
        "role": "system",
        "content": "You are an AI assistant focused on technical topics.",
    },
    {
        "role": "user",
        "content": "Can you explain chat templates?",
    },
]

rendered_prompt = tokenizer.apply_chat_template(
    messages,
    tokenize=False,
    add_generation_prompt=True,
)

print(rendered_prompt)
```

What the arguments mean:

- `messages` is the structured conversation.
- `tokenize=False` asks for the rendered prompt text instead of token IDs.
- `add_generation_prompt=True` adds the model-specific prefix needed to begin an assistant response when supported by the template.

The exact output depends on the tokenizer's configured chat template.

The application can then tokenize the rendered prompt and pass it to the model. Many higher-level pipelines handle these steps internally.

**Practical rule:** Use the chat template associated with the model you are using rather than manually inventing a format.

# Part 4 — The Thought–Action–Observation Cycle

## 22. The Core Agent Workflow

An agent often operates through a repeated cycle:

1. **Thought:** Decide what to do next.
2. **Action:** Execute a tool or other operation.
3. **Observation:** Receive the result from the environment.
4. **Updated decision:** Use the observation to decide whether to continue or finish.

```text
User request
     ↓
Thought: choose the next step
     ↓
Action: call a tool
     ↓
Observation: receive the result
     ↓
Is the objective complete?
     ├── No → choose another action
     └── Yes → return the final answer
```

The cycle is conceptually similar to a `while` loop. The application continues until the task is completed, a stopping condition is met, or a safety or execution limit is reached.

The model does not itself execute the tool. The agent framework interprets the proposed action, runs the tool, and returns the result to the model.

## 23. Alfred, the Weather Agent

Suppose the user asks:

> What's the current weather in New York?

Alfred has access to a tool named `get_weather`.

### Thought

The agent determines that current weather information is needed and that its weather tool is appropriate.

### Action

The model proposes a structured tool call:

```json
{
  "action": "get_weather",
  "action_input": {
    "location": "New York"
  }
}
```

This is a description of the requested action, not the weather result.

### Observation

The agent executes the tool. Suppose the actual tool returns:

```text
Current weather in New York:
Partly cloudy, 15°C, 60% humidity.
```

The agent adds this observation to the context provided to the model.

### Updated decision and final answer

The model can now use the actual result to answer:

```text
The current weather in New York is partly cloudy,
with a temperature of 15°C and 60% humidity.
```

The weather values above are illustrative, not a live report.

The important lesson is that the observation comes from the tool's execution. It must not be invented by the model.

## 24. Why the Cycle Matters

The cycle provides three important capabilities.

**Tool integration:** The agent can obtain information from APIs, databases, files, and other external systems.

**Feedback:** The agent can inspect the actual result of an action instead of assuming the action succeeded.

**Adaptation:** If a tool fails or returns incomplete information, the agent can decide to retry, use another tool, ask the user a question, or report the limitation.

This is the foundation of many agent frameworks.

# Part 5 — Thought, Chain-of-Thought, and ReAct

## 25. What Is a Thought?

In an agent workflow, a thought is the model's decision-making step before or after an action.

Conceptual categories include:

| Type | Example |
|---|---|
| Planning | Break a report into data collection, analysis, and writing |
| Analysis | Inspect an error message to identify a likely cause |
| Decision-making | Choose an option that satisfies the user's constraints |
| Problem-solving | Decide to profile code before optimizing it |
| Memory integration | Use relevant information from earlier conversation context |
| Reflection | Try a different approach after a failed tool call |
| Goal setting | Establish the criteria that define task completion |
| Prioritization | Address a security issue before a cosmetic improvement |

These are useful descriptions of decision-making behavior. They do not imply that every model exposes a complete or reliable internal reasoning trace.

## 26. Chain-of-Thought (CoT)

**Chain-of-Thought prompting** encourages a model to work through a problem in intermediate steps before giving an answer.

For example, a prompt may ask for a step-by-step solution to a mathematical problem.

```text
Question:
What is 15% of 200?

Reasoning outline:
10% of 200 = 20
5% of 200 = 10
15% of 200 = 20 + 10

Answer: 30
```

The example illustrates a reasoning outline. The important point is that the model can be prompted to use intermediate steps to solve a task.

CoT prompting does not require external tools. It can be used for problems solved from the information already provided to the model.

## 27. ReAct: Reasoning and Acting

**ReAct** combines reasoning with actions and observations.

Rather than trying to answer every question using only the original prompt, a ReAct-style agent can take an action, inspect the result, and then decide what to do next.

Example:

```text
Task:
Find the current weather in Paris.

Thought:
I need current weather data.

Action:
Call the weather tool for Paris.

Observation:
The tool returns the current weather data.

Thought:
I have enough information to answer.

Final answer:
Report the result returned by the tool.
```

This is a conceptual illustration. A real implementation uses the model's supported output format and the framework's action parser.

### CoT vs. ReAct

| Feature | Chain-of-Thought prompting | ReAct |
|---|---|---|
| Intermediate reasoning steps | Yes | Yes |
| External tool execution | Not required | Commonly used |
| Observation from the environment | Not required | Incorporated into the cycle |
| Typical use | Reasoning over provided information | Multi-step tasks that need external information or actions |

CoT and ReAct are prompting or workflow approaches, not the same thing as training a model.

Some models are trained to perform additional reasoning before answering. That is a training-level distinction, whereas CoT and ReAct describe ways of prompting or organizing model behavior.

**Practical distinction:** A model can reason about a weather question without a tool, but it needs an actual weather source to reliably retrieve current conditions.

# Part 6 — Actions and Observations

## 28. What Is an Action?

An **action** is a concrete operation an agent requests to change or inspect its environment.

Examples include:

- Search the web.
- Retrieve a document.
- Call a database.
- Execute a calculation.
- Run a program.
- Send a message.
- Interact with a user interface.
- Control a physical device.

An action may involve one tool or several tools. A tool is the capability used to perform an operation; an action is the step the agent takes toward its objective.

## 29. Three Ways to Represent Agent Actions

### JSON agents

The model emits structured data describing the tool and its arguments.

```json
{
  "action": "get_weather",
  "action_input": {
    "location": "New York"
  }
}
```

The application parses the output, validates the action, and invokes the corresponding function.

### Code agents

The model produces code that is executed by an external runtime.

```python
result = get_weather("New York")
print(result)
```

Code can express loops, conditionals, calculations, and combinations of multiple tools.

However, code generation introduces security risks. Generated code should run only in a suitably restricted environment with appropriate permissions and resource limits.

### Function-calling agents

The model emits a structured function call using a format supported by the model API.

The API or framework identifies the requested function and its arguments. The application executes the function and provides the result to the model.

Function calling is a structured form of tool invocation. It should not be assumed to work identically across all model providers.

## 30. The Stop-and-Parse Approach

An agent must distinguish a proposed action from the result of executing that action.

The stop-and-parse approach follows three steps:

1. **Generate:** The model produces a structured action request.
2. **Stop:** Generation ends at the appropriate action boundary.
3. **Parse and execute:** The application validates the request, calls the actual tool, and obtains the result.

For example:

```text
Model output:
Action:
{
  "action": "get_weather",
  "action_input": {
    "location": "New York"
  }
}
```

The application parses the JSON, checks that `get_weather` is an allowed tool, validates the location argument, and invokes the function.

It then provides the actual function result as an observation.

The stopping mechanism depends on the model API and output format. It may use structured function calls, a stop sequence, a dedicated control signal, or another supported mechanism.

**Why this matters:** If generation continues without a control boundary, the model might generate text that looks like a tool result even though the tool was never executed.

## 31. Code Agents: Advantages and Risks

Code agents can express more complex actions than a single function name and argument list.

Their advantages include:

- **Expressiveness:** Loops, conditionals, and multi-step computations.
- **Modularity:** Code can combine reusable functions.
- **Debuggability:** Syntax errors and execution failures can often be inspected.
- **Integration:** Code can connect to libraries, APIs, and data-processing tools.

Their risks include:

- Unsafe operations.
- Unauthorized file or network access.
- Exposure of credentials.
- Resource exhaustion.
- Prompt injection that attempts to influence tool use.

A code agent should use a restricted execution environment, explicit permissions, timeouts, resource limits, and validation appropriate to the task. Libraries such as `smolagents` provide agent abstractions, but secure execution still depends on how the environment and tools are configured.

## 32. What Is an Observation?

An **observation** is the result or feedback an agent receives after an action.

Examples include:

| Observation type | Example |
|---|---|
| System feedback | Success message, error, status code |
| Data changes | Updated database record or file |
| Environmental data | Sensor readings or system metrics |
| Response data | API response, search results, calculation output |
| Time-based event | A deadline reached or a scheduled task completed |

After an action, the application typically:

1. Parses the action request.
2. Executes the tool.
3. Captures its output or error.
4. Adds the observation to the context.
5. Lets the model decide what to do next.

For example, a weather API response can become the next observation. If the response contains an error, the agent should not report a successful weather lookup.

Observations are feedback from the environment. They help the agent distinguish between what it intended to do and what actually happened.

# Part 7 — What Are Tools?

## 33. Definition of a Tool

A **tool is a callable capability made available to an LLM-powered agent to perform a specific operation.**

Common tools include:

| Tool | Purpose |
|---|---|
| Web search | Retrieve current information from the internet |
| Image generation | Generate an image from a description |
| Retrieval | Search documents or external knowledge sources |
| API interface | Interact with services such as GitHub or other APIs |
| Calculator | Perform precise arithmetic |
| Database query | Retrieve or modify data through an authorized interface |

Tools complement the model's capabilities.

For example, a calculator tool is often more reliable for exact arithmetic than asking a language model to perform every calculation using generated text.

A model's training data also has a time boundary. For current weather, changing prices, or recent events, a suitable external data tool is needed if the agent must provide up-to-date information.

## 34. The Four Parts of a Tool

A well-defined tool generally has:

1. **Name:** A clear identifier.
2. **Description:** What the tool does and when it should be used.
3. **Arguments:** The inputs it expects, including types.
4. **Output:** The result it returns, including its type where appropriate.

For example:

```python
def calculator(a: int, b: int) -> int:
    """Multiply two integers."""
    return a * b
```

This function has:

- Name: `calculator`
- Description: `Multiply two integers.`
- Input `a`: integer
- Input `b`: integer
- Output: integer

The annotations `int` are type hints. They describe the expected types but do not, by themselves, guarantee runtime validation.

## 35. How an LLM Uses a Tool

The model generally does not execute an ordinary Python function by itself.

Instead, the system makes the tool's description available to the model. The model can then produce a structured request to use it.

```text
User:
What is 12 × 8?

LLM:
Requests the calculator tool with a = 12 and b = 8.

Agent application:
Executes calculator(12, 8).

Tool result:
96

LLM:
Uses the result to answer the user.
```

The agent application manages the actual execution and passes the tool's result back to the model.

This is why tool calls can be hidden from the user interface while still being essential to the task.

## 36. Describing a Tool to the Model

A tool can be represented in a textual specification such as:

```text
Tool Name: calculator
Description: Multiply two integers.
Arguments: a: int, b: int
Output: int
```

This description tells the model which operation is available, what it does, which inputs to provide, and what output to expect.

When several tools are available, their descriptions should follow a consistent and precise format.

Ambiguous tool descriptions can cause the model to choose the wrong tool or provide incorrect arguments.

## 37. Why Use Type Hints and Docstrings?

Python functions already contain useful metadata:

- The function name identifies the capability.
- The docstring describes its purpose.
- Type hints describe its expected arguments and return value.

This information can be inspected programmatically and converted into a tool description.

The actual implementation language does not define what the LLM must see. The important information is the tool's name, purpose, inputs, and outputs.

A decorator can mark a function as a tool and help a framework extract this information automatically.

## 38. Building a Reusable `Tool` Class

The following simplified class stores a tool's metadata and the function it wraps.

```python
from typing import Callable


class Tool:
    def __init__(
        self,
        name: str,
        description: str,
        func: Callable,
        arguments: list,
        outputs: str,
    ):
        self.name = name
        self.description = description
        self.func = func
        self.arguments = arguments
        self.outputs = outputs

    def to_string(self) -> str:
        args_str = ", ".join(
            f"{arg_name}: {arg_type}"
            for arg_name, arg_type in self.arguments
        )

        return (
            f"Tool Name: {self.name}, "
            f"Description: {self.description}, "
            f"Arguments: {args_str}, "
            f"Outputs: {self.outputs}"
        )

    def __call__(self, *args, **kwargs):
        return self.func(*args, **kwargs)
```

Let's understand the components.

### `__init__()`

The constructor stores the tool's name, description, wrapped function, argument definitions, and output description.

### `to_string()`

This method converts the metadata into a readable text description that can be included in a system prompt.

### `__call__()`

This method makes the `Tool` instance callable. When the instance is invoked, it forwards the supplied arguments to the wrapped function.

We can create a calculator tool with the class:

```python
def calculator(a: int, b: int) -> int:
    """Multiply two integers."""
    return a * b


calculator_tool = Tool(
    name="calculator",
    description="Multiply two integers.",
    func=calculator,
    arguments=[("a", "int"), ("b", "int")],
    outputs="int",
)

print(calculator_tool.to_string())
print(calculator_tool(12, 8))
```

Expected output:

```text
Tool Name: calculator, Description: Multiply two integers.,
Arguments: a: int, b: int, Outputs: int

96
```

The class is an educational example. Real frameworks generally provide additional metadata extraction, validation, error handling, and execution controls.

## 39. What Does a `@tool` Decorator Do?

A decorator can wrap a normal Python function with additional behavior.

In an agent framework, a `@tool` decorator may inspect the function's name, type hints, and docstring to construct its tool metadata.

Conceptually:

```python
@tool
def calculator(a: int, b: int) -> int:
    """Multiply two integers."""
    return a * b
```

Some tool implementations expose a method such as:

```python
print(calculator.to_string())
```

This depends on the decorator implementation: the ordinary Python function does not automatically gain a `to_string()` method just because its parameters have type hints.

The key lesson is that a framework can automate the conversion of Python function metadata into a tool specification.

## 40. Model Context Protocol (MCP)

**Model Context Protocol (MCP)** is an open protocol that standardizes how applications expose tools and contextual resources to AI systems.

It helps provide:

- Reusable integrations.
- A consistent interface between compatible clients and tool servers.
- Flexibility when changing supported model providers or agent frameworks.
- A common way to connect models with external systems.

Without a shared protocol, developers may need to build separate integration layers for different frameworks.

MCP can reduce that duplication when the relevant applications and integrations support the protocol. It does not automatically make every tool compatible with every system or remove the need for access control and security.

## 41. Tools vs. Actions

These concepts are related but different.

- **Tool:** A capability the agent can call.
- **Action:** An operation the agent takes to progress toward its objective.

One action may use several tools. For example, researching a company might require searching the web, opening a page, extracting relevant information, and saving a report.

The tool is the available capability; the action is the step in the task workflow.


# Part 8 — Building an Agent from Scratch

## 43. Why Build an Agent Without a Framework?

In the previous sections, I learned the main components of an agent:

- An LLM that interprets instructions and selects actions.
- A system prompt that describes the available tools and workflow.
- Tools that execute specific operations.
- Observations that return the results of those operations.
- A loop that continues until the task is completed.

Before using an agent framework, it is useful to implement these components manually.

This helps me understand what a framework automates behind the scenes.

The course introduces a simplified, educational agent library and a serverless API. These are intended to teach the concepts, not to serve as a complete production implementation.

The course later introduces frameworks such as `smolagents`, LangGraph, and LlamaIndex.

## 44. What Is a Serverless Inference API?

A serverless inference API lets an application send a request to a model hosted by a provider without managing the model's full inference infrastructure.

In the Hugging Face ecosystem, the `huggingface_hub` package provides an `InferenceClient` interface for accessing supported models.

The general workflow is:

```text
Python application
        ↓
InferenceClient
        ↓
Hugging Face inference service
        ↓
Selected language model
        ↓
Generated response
        ↓
Python application
```

The model runs on the hosted service rather than necessarily running on my local computer.

Availability depends on the model, provider, access permissions, and current inference service support.

## 45. Install the Required Package

In a terminal, install the Hugging Face Hub client:

```bash
pip install -U huggingface_hub
```

The course examples also use Python's built-in `os` module to access environment variables.

## 46. Configure the Hugging Face Token

The course example retrieves a token named `HF_TOKEN`.

First, obtain an appropriately permissioned token from:

https://hf.co/settings/tokens

The source example uses a read token. The exact permissions required depend on the inference service and the model being accessed.

### On Google Colab

The course example stores the token in Colab Secrets.

1. Open the Secrets panel.
2. Create a secret named `HF_TOKEN`.
3. Store the token as its value.
4. Allow the notebook to access the secret.

### In a local environment

The example can read the token from an environment variable.

```python
import os

HF_TOKEN = os.environ.get("HF_TOKEN")
```

On Windows PowerShell, I can set it for the current terminal session:

```powershell
$env:HF_TOKEN = "YOUR_HUGGING_FACE_TOKEN"
```

Replace the placeholder with the real token. Do not commit the token to GitHub or paste it into a public notebook.

For a persistent or deployed application, use the platform's secret-management facilities rather than hardcoding credentials.

## 47. Create an Inference Client

The source notebook creates a client for the model `moonshotai/Kimi-K2.5`.

```python
import os
from huggingface_hub import InferenceClient

HF_TOKEN = os.environ.get("HF_TOKEN")

if not HF_TOKEN:
    raise RuntimeError(
        "Set the HF_TOKEN environment variable first."
    )

client = InferenceClient(
    model="moonshotai/Kimi-K2.5",
    token=HF_TOKEN,
)
```

What each part does:

- `InferenceClient` provides the interface for sending inference requests.
- `model` specifies the model identifier.
- `token` supplies the credentials used for the request.
- `HF_TOKEN` is read from the environment rather than embedded in the code.

**Important:** A valid token does not guarantee that every model is available through every inference provider. The selected model must be supported by the inference service being used.

## 48. Send a Chat Completion Request

The course uses the chat-completions interface.

```python
output = client.chat.completions.create(
    messages=[
        {
            "role": "user",
            "content": "The capital of France is",
        }
    ],
    stream=False,
    max_tokens=1024,
    extra_body={
        "thinking": {
            "type": "disabled"
        }
    },
)

print(output.choices[0].message.content)
```

The source notebook demonstrates a response:

```text
Paris.
```

This is an example of the expected result, not a guarantee that every model or API call returns exactly that text.

### Understanding the arguments

- `messages`: The conversation supplied to the model.
- `role`: Identifies who produced the message.
- `content`: Contains the message text.
- `stream=False`: Requests the completed response rather than streamed output.
- `max_tokens=1024`: Sets a generation limit supported by the selected API.
- `extra_body`: Passes additional provider-specific parameters.

The `thinking` parameter shown here is provider/model-specific. It is not a universal argument that every model supports.

### Reading the response

```python
print(output.choices[0].message.content)
```

The response object contains a list of choices. The expression accesses the first choice and prints its message content.

**Key lesson:** The `chat` interface helps apply the appropriate conversational formatting and provides a convenient way to communicate with supported models.

# Part 9 — Creating a Dummy Agent

## 49. What Makes This Different from a Normal LLM Call?

A normal LLM call asks the model to produce an answer.

A tool-using agent must do more:

1. Provide the model with the available tools.
2. Ask the model to select an action when necessary.
3. Stop generation at the action boundary.
4. Parse the proposed action.
5. Execute the actual Python function.
6. Insert the real result into the conversation.
7. Ask the model to continue.

The course demonstrates this process using a weather tool.

The first version uses a simplified system prompt that describes the available tool.

## 50. Write the Agent's System Prompt

```python
SYSTEM_PROMPT = """
Answer the following questions as best you can.

You have access to the following tool:

get_weather:
Get the current weather in a given location.

The tool action must be represented as a JSON object
with an "action" key and an "action_input" key.

Example:

{
    "action": "get_weather",
    "action_input": {
        "location": "New York"
    }
}

Use the following interaction format:

Question: the input question you must answer

Thought: decide the next step

Action:
A JSON object describing one tool call

Observation:
The result returned by the tool

The Thought, Action, and Observation cycle may repeat
when additional steps are needed.

When you know the final answer, finish with:

Final Answer: your answer to the original question
"""
```

This is a simplified version of the course's prompt. The original prompt also specifies that the model should perform one action at a time and use the exact `Final Answer:` marker.

### Why include tool descriptions in the prompt?

The model needs to know:

- Which tools are available.
- What each tool does.
- Which arguments it expects.
- How to represent an action.
- How to use the observation returned by the tool.
- How to signal that it has finished.

Without this information, the model may not know which operation it is allowed to request or how the application expects the request to be formatted.

A system prompt describes the workflow; it does not execute the workflow by itself.

## 51. Build the Conversation

The course creates a list containing the system instructions and the user's request.

```python
messages = [
    {
        "role": "system",
        "content": SYSTEM_PROMPT,
    },
    {
        "role": "user",
        "content": "What's the weather in London?",
    },
]
```

The `messages` list is the context supplied to the model.

The system message defines the expected behavior. The user message provides the actual task.

The API formats these messages for the selected model.

## 52. The First Attempt: A Subtle but Important Bug

The course first requests a response from the model:

```python
output = client.chat.completions.create(
    messages=messages,
    stream=False,
    max_tokens=200,
)

print(output.choices[0].message.content)
```

The model might produce something like:

```text
Thought: I need to check the weather in London.

Action:
{
    "action": "get_weather",
    "action_input": {
        "location": "London"
    }
}

Observation: The weather in London is partly cloudy.
Final Answer: It is partly cloudy in London.
```

The problem is that the model may have generated the observation itself.

No weather function has necessarily run. The model has merely generated text that looks like a tool result.

This is an example of **hallucinated tool execution**: the response claims that a tool returned data even though the application did not actually execute the tool.

The observation must come from the real function or external service, not from the model's imagination.

## 53. Prevent the Model from Inventing an Observation

The course demonstrates a stop sequence:

```python
output = client.chat.completions.create(
    messages=messages,
    max_tokens=150,
    stop=["Observation:"],
)

print(output.choices[0].message.content)
```

The intention is to stop generation immediately before the model begins writing the observation.

The response should now end after the action request, approximately like this:

```text
Thought: I need to check the weather in London.

Action:
{
    "action": "get_weather",
    "action_input": {
        "location": "London"
    }
}
```

Now the application has an opportunity to parse and execute the action.

**Important:** Stop-sequence behavior depends on the API and model. The application must still check that the returned text contains a complete, valid action. A stop sequence alone does not make an agent safe or reliable.

## 54. Implement the Weather Tool

The source notebook uses a dummy function rather than a real weather API.

```python
def get_weather(location):
    return (
        f"The weather in {location} is sunny "
        "with low temperatures."
    )
```

Call the function:

```python
result = get_weather("London")
print(result)
```

Expected output:

```text
The weather in London is sunny with low temperatures.
```

This is a hardcoded example. It does not retrieve actual weather information.

A production weather tool would need to call a real weather service, handle errors, and distinguish current data from unavailable or stale data.

### Why does this function matter?

The function represents the actual execution step.

The model can request `get_weather("London")`, but the application is responsible for invoking the function and capturing its result.

This separates the model's proposed action from the environment's actual response.

## 55. Append the Real Observation

After executing the function, the application adds the observation to the conversation.

The source notebook demonstrates the idea with:

```python
messages = [
    {
        "role": "system",
        "content": SYSTEM_PROMPT,
    },
    {
        "role": "user",
        "content": "What's the weather in London?",
    },
    {
        "role": "assistant",
        "content": (
            output.choices[0].message.content
            + "\nObservation:\n"
            + get_weather("London")
        ),
    },
]
```

This shows how the action request and its observation can be included in the context before asking the model to continue.

However, this is a simplified text-based implementation. In a more robust implementation, the application should:

1. Parse the action from the assistant's output.
2. Validate the tool name.
3. Validate the argument types.
4. Execute the selected function.
5. Capture its return value or error.
6. Record the action and observation in the conversation.
7. Request the next model response.

The tool should be executed only once for each validated action request.

The course's simplified example calls `get_weather("London")` directly because the action is already known. A general-purpose agent must extract the requested location from the parsed action instead of hardcoding it.

## 56. Ask the Model to Continue

After adding the observation, call the model again:

```python
output = client.chat.completions.create(
    messages=messages,
    stream=False,
    max_tokens=200,
)

print(output.choices[0].message.content)
```

The model can now produce a final response based on the supplied observation.

For the dummy function, it might answer:

```text
Final Answer: The weather in London is sunny
with low temperatures.
```

Again, this is based on the dummy function, not a real weather report.

### The complete conceptual workflow

```text
User asks a question
        ↓
Model receives system prompt and question
        ↓
Model proposes a tool call
        ↓
Application stops and parses the action
        ↓
Application executes the Python function
        ↓
Function returns its result
        ↓
Application appends the observation
        ↓
Model generates the final answer
```

This is the central mechanism behind many tool-using agents.

## 57. What the Dummy Agent Teaches

The example demonstrates several important ideas:

- The model needs a description of its available tools.
- The model proposes an action in a machine-readable format.
- The application, not the model, executes the function.
- The result must come from the executed function.
- The result is returned to the model as an observation.
- The model can then use the observation to formulate a final answer.

The example is deliberately small. It does not yet provide a complete production-grade action parser, robust retry policy, security boundary, or generalized multi-tool execution engine.

That is one reason agent frameworks are useful.

# Part 10 — Building the First Agent with `smolagents`

## 58. What Is `smolagents`?

`smolagents` is a Python library for building AI agents with a relatively lightweight interface.

The course introduces it after the manually implemented agent workflow.

One focus of `smolagents` is the **CodeAgent**, which can use generated code to carry out actions and observe the results of executing that code.

Instead of manually implementing every step of the agent loop, the framework handles much of the orchestration.

The developer still needs to choose the model, provide suitable tools, configure the agent, and decide how execution should be secured.

## 59. What Will the First Agent Do?

The course's starter Space provides a basic agent application with a Gradio user interface.

The goal is to make the agent capable of using tools such as:

- A custom Python tool.
- A timezone tool.
- A web-search tool.
- An image-generation tool.

The exact tools used depend on which ones are enabled and correctly configured.

The course encourages modifying the starter application rather than implementing every component from scratch.

## 60. Create a Hugging Face Space

The course uses this starter template:

https://huggingface.co/spaces/agents-course/First_agent_template

The workflow is:

1. Open the template.
2. Duplicate it to create a copy under your own Hugging Face account.
3. Open the duplicated Space.
4. Inspect its files.
5. Open `app.py`.
6. Configure the required secret.
7. Modify the agent and add tools.
8. Run the Space and test the agent.

Duplicating a Space creates a separate copy that you can customize. It does not mean that the original template is modified.

## 61. Configure the API Token in the Space

The course instructs you to obtain a Hugging Face token with inference permission.

Open the settings of your duplicated Space and locate the Variables and Secrets section.

Create a secret with the name:

```text
HF_TOKEN
```

Set its value to your token and save it.

The application can then use the token without placing it directly in the source code.

A secret should never be printed in application logs or committed to the repository.

## 62. Import the Required Libraries

The course example uses imports such as:

```python
from smolagents import (
    CodeAgent,
    DuckDuckGoSearchTool,
    InferenceClientModel,
    load_tool,
    tool,
)

import datetime
import requests
import pytz
import yaml
```

It also imports the template's final-answer tool and Gradio interface:

```python
from tools.final_answer import FinalAnswerTool
from Gradio_UI import GradioUI
```

The latter two imports depend on files supplied by the course template.

### What these imports are for

| Import | Purpose |
|---|---|
| `CodeAgent` | Creates an agent that can execute code-based actions through its configured runtime |
| `DuckDuckGoSearchTool` | Provides a web-search capability |
| `InferenceClientModel` | Connects the agent to a supported hosted inference model |
| `load_tool` | Loads a tool from a supported source |
| `tool` | Marks a Python function as a tool using the framework's tool interface |
| `datetime` | Works with dates and times |
| `requests` | Sends HTTP requests |
| `pytz` | Provides timezone handling |
| `yaml` | Reads the agent's YAML configuration |
| `FinalAnswerTool` | Provides a way for the agent to return a final answer in this template |
| `GradioUI` | Launches the template's Gradio interface |

**Version note:** Framework APIs and example imports can change. If an import fails, check the installed `smolagents` version and the actual files in the template rather than assuming every course example matches every later release.

## 63. Create a Custom Tool

The course begins with a placeholder tool.

```python
def my_custom_tool(arg1: str, arg2: int) -> str:
    """A tool that does nothing yet.

    Args:
        arg1: The first argument.
        arg2: The second argument.
    """
    return "What magic will you build?"
```

This example teaches the basic structure of a tool function.

### Function name

`my_custom_tool` identifies the operation.

### Type hints

```python
arg1: str
arg2: int
```

These indicate the expected argument types.

```python
-> str
```

This indicates that the function is expected to return a string.

Type hints describe the interface; additional validation may still be necessary.

### Docstring

The docstring explains what the tool does and describes its arguments.

The course emphasizes that tool descriptions should be clear enough for the agent to determine when and how to use the tool.

### Return value

The function returns a string that can be provided to the agent as a tool result.

This starter tool does not perform a useful operation yet. It is intended to be modified.

## 64. Build a Timezone Tool

The course also provides a working example that returns the current local time in a specified timezone.

```python
import datetime
import pytz


def get_current_time_in_timezone(
    timezone: str,
) -> str:
    """Fetch the current local time in a specified timezone.

    Args:
        timezone: A valid timezone, such as
            'America/New_York'.
    """
    try:
        tz = pytz.timezone(timezone)

        local_time = datetime.datetime.now(
            tz
        ).strftime("%Y-%m-%d %H:%M:%S")

        return (
            f"The current local time in {timezone} "
            f"is: {local_time}"
        )

    except Exception as e:
        return (
            f"Error fetching time for timezone "
            f"'{timezone}': {str(e)}"
        )
```

Example call:

```python
print(
    get_current_time_in_timezone(
        "Asia/Kolkata"
    )
)
```

The output depends on the actual current time.

### How the tool works

1. The function receives a timezone name.
2. `pytz.timezone()` looks up the corresponding timezone.
3. `datetime.datetime.now(tz)` obtains the current time in that timezone.
4. `strftime()` formats the time as year-month-day and hour-minute-second.
5. The function returns the formatted result.
6. If an error occurs, the function returns an error message.

The tool returns a string, which makes it easy for the agent to use the result in a response.

The function is a useful example because it performs a real operation using the environment rather than returning a fixed placeholder.

For production use, errors should be handled deliberately, and the agent should recognize an error message as a failed operation rather than presenting it as a successful lookup.

## 65. Configure the Model

The course uses `InferenceClientModel` with the model identifier `Qwen/Qwen2.5-Coder-32B-Instruct`.

```python
model = InferenceClientModel(
    max_tokens=2096,
    temperature=0.5,
    model_id="Qwen/Qwen2.5-Coder-32B-Instruct",
    custom_role_conversions=None,
)
```

### `model_id`

Specifies the model the agent will use.

### `max_tokens`

Sets the maximum generation length according to the interface's supported configuration.

### `temperature`

Controls the sampling behavior where supported. Lower values generally make generation more conservative; higher values generally allow more variation.

### `custom_role_conversions`

Allows custom mapping of message roles when needed by a supported model interface. The example sets it to `None`.

The model must be accessible through the configured inference interface. Model availability and provider support can change.

## 66. Load the Final-Answer Tool

The course imports and creates the template's final-answer tool:

```python
final_answer = FinalAnswerTool()
```

This tool allows the agent to return a final answer through the framework's expected mechanism.

The example keeps this tool available in the agent's tool list.

The final-answer mechanism is important because the agent needs to know when it has completed the task rather than continuing to call tools unnecessarily.

## 67. Load the System Prompt from YAML

The course stores its prompt configuration in `prompts.yaml`.

```python
with open("prompts.yaml", "r") as stream:
    prompt_templates = yaml.safe_load(stream)
```

Let's break this down.

- `open()` opens the YAML file.
- `"r"` specifies read mode.
- `yaml.safe_load()` parses the YAML content into Python data structures.
- `prompt_templates` holds the resulting configuration.

A YAML prompt file can keep the agent's instructions separate from its Python implementation. This makes the instructions easier to inspect and modify.

The file must exist at the expected path and contain valid YAML. Otherwise, the application will fail when loading it.

## 68. Create the `CodeAgent`

The course configures the agent as follows:

```python
agent = CodeAgent(
    model=model,
    tools=[final_answer],
    max_steps=6,
    verbosity_level=1,
    grammar=None,
    planning_interval=None,
    name=None,
    description=None,
    prompt_templates=prompt_templates,
)
```

### Understanding the configuration

- `model`: The model used for agent decisions.
- `tools`: The tools the agent is allowed to use.
- `max_steps=6`: Limits the number of agent steps.
- `verbosity_level=1`: Configures the level of execution information displayed.
- `grammar=None`: Does not supply an additional custom grammar through this parameter.
- `planning_interval=None`: Does not set a custom planning interval through this parameter.
- `name=None`: Does not provide a custom agent name.
- `description=None`: Does not provide a custom agent description.
- `prompt_templates`: Supplies the prompt configuration loaded from YAML.

The `tools` list determines which capabilities are available. Merely importing a tool does not make it available to the agent.

**Important observation:** The course's minimal example contains `tools=[final_answer]`. Although the code imports a search tool and defines other functions, they are not automatically available unless they are added to the list.

For example, a configuration that intends to expose the timezone tool would need to include the function in the list, using the tool interface supported by the installed version of `smolagents`.

The number of steps also matters. A low limit may prevent a complicated task from completing, while an excessively high limit can allow unnecessary execution.

## 69. Add the Tools to the Agent

Conceptually, the tool list might contain:

```python
tools = [
    final_answer,
    get_current_time_in_timezone,
    my_custom_tool,
]
```

The exact objects accepted by `CodeAgent` depend on the framework version and how the functions are registered.

For the course's tool-decorator approach, a custom function can be defined with `@tool`:

```python
@tool
def get_current_time_in_timezone(
    timezone: str,
) -> str:
    """Get the current local time in a timezone.

    Args:
        timezone: A valid timezone name.
    """
    tz = pytz.timezone(timezone)

    return datetime.datetime.now(
        tz
    ).strftime("%Y-%m-%d %H:%M:%S")
```

This version demonstrates the decorator-based tool interface. It omits explicit error handling to keep the example focused on registration.

Before using the function in a real application, ensure that the framework recognizes the tool correctly and that invalid timezone inputs are handled appropriately.

## 70. Load an Image-Generation Tool

The course also demonstrates loading a tool from the Hugging Face Hub:

```python
image_generation_tool = load_tool(
    "agents-course/text-to-image",
    trust_remote_code=True,
)
```

This example illustrates how an agent can obtain capabilities beyond the functions defined locally.

The `trust_remote_code=True` argument is significant. It indicates that remote tool code may be executed, so the source must be trusted and inspected as appropriate before enabling it.

Loading a tool does not necessarily make it available to the agent. It must also be included in the agent's configured tool list.

A tool that calls an external service may require additional dependencies, credentials, permissions, or configuration.

## 71. Launch the Gradio Interface

The course ends its application setup by launching the interface:

```python
GradioUI(agent).launch()
```

This creates a user interface around the configured agent.

The conceptual application flow becomes:

```text
User enters a request in Gradio
        ↓
Agent receives the request
        ↓
Model decides what to do
        ↓
Agent invokes an available tool if needed
        ↓
Tool returns an observation
        ↓
Agent continues or finishes
        ↓
Gradio displays the response
```

The Gradio interface provides the interaction layer. The agent and its tools provide the task-execution behavior.

# Part 11 — Putting the Starter Application Together

## 72. Understand the Main `app.py` Structure

The course's starter application combines the following pieces:

1. Imports.
2. Custom tool definitions.
3. The final-answer tool.
4. The model configuration.
5. Optional tools loaded from the Hub.
6. The YAML prompt configuration.
7. The `CodeAgent` configuration.
8. The Gradio interface launch.

The source example uses this overall structure:

```python
from smolagents import (
    CodeAgent,
    DuckDuckGoSearchTool,
    InferenceClientModel,
    load_tool,
    tool,
)

import datetime
import requests
import pytz
import yaml

from tools.final_answer import FinalAnswerTool
from Gradio_UI import GradioUI


def my_custom_tool(
    arg1: str,
    arg2: int,
) -> str:
    """A customizable example tool.

    Args:
        arg1: The first argument.
        arg2: The second argument.
    """
    return "What magic will you build?"


def get_current_time_in_timezone(
    timezone: str,
) -> str:
    """Get the current local time in a timezone.

    Args:
        timezone: A valid timezone name.
    """
    try:
        tz = pytz.timezone(timezone)

        local_time = datetime.datetime.now(
            tz
        ).strftime("%Y-%m-%d %H:%M:%S")

        return (
            f"The current local time in {timezone} "
            f"is: {local_time}"
        )

    except Exception as e:
        return f"Error fetching timezone: {e}"


final_answer = FinalAnswerTool()

model = InferenceClientModel(
    max_tokens=2096,
    temperature=0.5,
    model_id="Qwen/Qwen2.5-Coder-32B-Instruct",
    custom_role_conversions=None,
)

image_generation_tool = load_tool(
    "agents-course/text-to-image",
    trust_remote_code=True,
)

with open("prompts.yaml", "r") as stream:
    prompt_templates = yaml.safe_load(stream)

agent = CodeAgent(
    model=model,
    tools=[
        final_answer,
        get_current_time_in_timezone,
        my_custom_tool,
        image_generation_tool,
    ],
    max_steps=6,
    verbosity_level=1,
    grammar=None,
    planning_interval=None,
    name=None,
    description=None,
    prompt_templates=prompt_templates,
)

GradioUI(agent).launch()
```

**This is a structural illustration, not a guaranteed drop-in replacement for the course's exact starter file.** The original template's framework version, tool registration requirements, and import paths must match the code. In particular, some versions require custom functions to be decorated with `@tool`, and the template's `FinalAnswerTool` and `GradioUI` imports must exist.

The course's original minimal configuration uses only `tools=[final_answer]`. The expanded list above illustrates the intended idea of exposing additional capabilities, but it must be checked against the installed version before running.

The imports `requests` and `DuckDuckGoSearchTool` are not used in this illustrative configuration. They are relevant only if the application actually implements or registers functionality that uses them.

## 73. The Role of `prompts.yaml`

The system prompt is stored separately from `app.py`.

This separation has several advantages:

- The behavior instructions are easier to inspect.
- Prompt changes do not require rewriting the tool implementations.
- The same prompt configuration can be reused by compatible agents.
- The agent's action format and general behavior can be customized independently of the interface.

The YAML file must match the structure expected by the version of `smolagents` being used. A syntactically valid YAML file can still be incompatible with the framework if its keys or values do not match the expected schema.

## 74. What Should I Experiment With?

The course's objective is to become familiar with the Space and the agent by adding capabilities.

Useful experiments include:

1. Replace the placeholder tool with a real, harmless utility.
2. Ask the agent for the time in different valid timezones.
3. Add a supported web-search tool.
4. Try a supported image-generation tool.
5. Change the model, if the new model is available through the configured inference interface.
6. Modify the prompt to clarify the agent's role and permitted actions.
7. Observe how the agent behaves when a tool returns an error.
8. Adjust the maximum number of steps and observe how it affects multi-step tasks.

When experimenting, change one component at a time. That makes it easier to determine whether a problem comes from the model, prompt, tool, credentials, or execution configuration.

# Part 12 — Safety and Reliability

## 75. Why Generated Code Needs Protection

A CodeAgent may produce executable code to perform an action.

This creates a security concern because generated code can potentially attempt operations that were not intended by the developer.

Risks include:

- Reading or modifying files.
- Making network requests.
- Accessing secrets.
- Consuming excessive CPU or memory.
- Performing destructive operations.
- Being influenced by malicious instructions in external content.

A responsible implementation should apply suitable safeguards, such as:

- Restricted execution environments.
- Explicit tool permissions.
- Input validation.
- Network and filesystem restrictions where appropriate.
- Execution timeouts and resource limits.
- Careful secret management.
- Logging and monitoring.
- Human confirmation for sensitive or irreversible actions.

The course recommends learning about secure code execution in the `smolagents` documentation:

https://huggingface.co/docs/smolagents/tutorials/secure_code_execution

A framework can help organize an agent, but it should not be treated as a guarantee that all generated code is safe.

## 76. Why Tool Results Must Be Verified

A model may generate convincing text that describes an action or result without the action ever having happened.

For example, an agent could claim that it sent an email even though no email function was executed.

A reliable agent must distinguish between:

- A proposed action.
- An attempted action.
- A successful execution.
- A failed execution.
- A verified final outcome.

For important operations, the system should use the tool's actual return value or independently verify the resulting state.

For example, after an email-sending tool reports success, an application may need to inspect the provider's response or message identifier before telling the user that the operation completed.

The same principle applies to file creation, database updates, API requests, and other external actions.

# Final Summary — AI Agent Journey #1

In this journey, I learned how an LLM-powered agent works from its foundational concepts to a first framework-based implementation.

The most important ideas are:

1. **An agent is a system, not just a model.** It combines an AI model with tools and execution logic.
2. **LLMs generate text and predict tokens.** The application supplies the context and manages how model outputs are used.
3. **Chat templates structure conversations.** The chosen model expects a particular message format and special tokens.
4. **The Thought–Action–Observation cycle enables interaction.** The model proposes an action, the application executes it, and the result becomes an observation.
5. **Tools extend model capabilities.** They can perform calculations, retrieve information, call APIs, and execute other operations.
6. **The application must control tool execution.** It should parse and validate the model's action request instead of blindly executing generated text.
7. **Observations must reflect actual results.** A model-generated claim is not proof that a tool ran successfully.
8. **A dummy agent reveals the underlying mechanism.** It demonstrates prompts, action boundaries, function execution, and feedback.
9. **`smolagents` reduces orchestration work.** It provides abstractions for models, tools, agent execution, and user interfaces.
10. **Security and reliability remain essential.** Tools and generated code must be configured with appropriate restrictions and validation.

The main progression is:

```text
LLM fundamentals
        ↓
System prompts and chat templates
        ↓
Tool descriptions
        ↓
Structured action requests
        ↓
Tool execution
        ↓
Observations
        ↓
Repeated agent workflow
        ↓
Framework-based agent
```

The central lesson is that **agentic AI connects model decisions to controlled execution and feedback.** The LLM helps decide what should happen next, while the surrounding application determines which operations are available, executes them, and checks their results.