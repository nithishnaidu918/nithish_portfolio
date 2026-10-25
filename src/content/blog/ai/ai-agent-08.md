---
title: "AI Agent Observability, Evaluation, and Game Agents: From Monitoring to Pokémon Battles"
description: "Learn AI agent observability, OpenTelemetry tracing, Langfuse monitoring, online and offline evaluation, LLM-as-a-judge, and how to build a Pokémon-style battle agent using Python, Poke-env, and function calling."
category: "AI Agents"
tags: ["Agentic AI", "AI Agent Observability", "LLMOps", "Evaluation", "OpenTelemetry", "Langfuse", "smolagents", "LLM-as-a-Judge", "Pokémon AI", "Function Calling", "Python"]
---

# AI Agent Observability, Evaluation, and Game Agents

## Introduction

Building an AI agent that can answer questions and use tools is an important milestone in learning Agentic AI. However, building an agent that works reliably in real-world situations introduces several additional challenges.

How do we know whether the agent is making correct decisions? How can we identify which tool call caused a failure? How much does each interaction cost? Why does one request take several seconds while another finishes quickly? How can we detect when an updated model performs worse than the previous version?

These questions introduce two important concepts: **observability** and **evaluation**.

Observability helps us understand what happens inside an agent during execution. Evaluation helps us determine how well the agent performs and whether its behavior meets our requirements.

Together, they help us move from a working demonstration to a system that can be tested, monitored, debugged, and continuously improved.

There is another exciting application of Agentic AI: games. Instead of simply generating dialogue for a game character, an agent can observe the game state, choose actions, adapt to an opponent, and pursue a goal.

In this article, we will explore two related areas of agent development:

1. **AI Agent Observability and Evaluation:** Instrumenting agents, recording traces, monitoring latency and cost, collecting user feedback, using an LLM as a judge, and evaluating performance with datasets.
2. **AI Agents in Games:** Understanding how agentic behavior differs from ordinary LLM dialogue and building a Pokémon-style turn-based battle agent using Python, Poke-env, Pokémon Showdown, and function calling.

By the end, you will understand how to inspect agent behavior, measure performance, improve reliability, and apply agentic concepts to an interactive game environment.

# Part 1: AI Agent Observability and Evaluation

## 1. What Is AI Agent Observability?

**Observability is the ability to understand what is happening inside a system by examining the information it produces while running.**

In traditional software, we often inspect logs, error messages, response times, and system metrics to diagnose problems.

AI agents introduce additional complexity because their execution may involve several model calls, tool calls, intermediate decisions, external APIs, and dynamically selected actions.

A single user request might trigger this workflow:

```text
User Request
     |
     v
AI Agent
     |
     v
LLM Call
     |
     v
Select Tool
     |
     v
Execute Tool
     |
     v
Observe Tool Result
     |
     v
Another LLM Call
     |
     v
Final Response
```

If the final response is incorrect, inspecting only the final text may not tell us what went wrong.

Perhaps the model selected the wrong tool. Perhaps the tool returned incomplete data. Perhaps the second model call misunderstood the observation. Perhaps the application encountered an API timeout and silently used a fallback.

Observability helps us investigate these possibilities by recording what happened during execution.

For AI agents, observability can include:

- User inputs and agent outputs.
- Model calls and their duration.
- Tool calls and their results.
- Token usage and estimated costs.
- Exceptions and failed requests.
- The sequence of operations within an agent run.
- User feedback and evaluation scores.
- Relevant metadata about the request or session.

The exact information collected depends on the instrumentation and the observability platform.

### 1.1 Why Does Observability Matter?

Without observability, an AI agent can behave like a black box. We see the input and the final output, but we have limited visibility into the steps that produced the result.

Observability helps us:

**Debug failures:** Identify the model call, tool, or execution step responsible for an unsuccessful task.

**Understand costs:** Determine whether an agent makes unnecessary model calls or repeatedly invokes expensive tools.

**Reduce latency:** Find the slowest operations and optimize the parts of the workflow that contribute most to response time.

**Monitor safety:** Investigate harmful outputs, suspicious tool requests, and possible prompt-injection attempts.

**Understand user satisfaction:** Connect user feedback to the agent runs that produced the responses.

**Detect regressions:** Compare the behavior of different prompts, models, tools, and agent versions.

For example, imagine a research agent that makes five LLM calls to answer a simple question. Observability might reveal that two calls are redundant and that one tool call consumes most of the execution time.

With that information, we can investigate whether a simpler workflow would deliver a similar answer at a lower cost.

Observability does not automatically improve the agent. It gives us evidence that helps us decide what to improve.

## 2. Observability Tools and OpenTelemetry

Several platforms can help developers collect, visualize, and analyze information about AI agent execution.

Examples include Langfuse and Arize. Different platforms offer different combinations of tracing, evaluation, prompt management, dashboards, and other LLMOps capabilities.

**LLMOps** refers to the practices and tools used to develop, evaluate, deploy, monitor, and maintain applications built with large language models.

A key concept in this area is OpenTelemetry.

### 2.1 What Is OpenTelemetry?

OpenTelemetry is a standard collection of APIs, SDKs, and instrumentation conventions for collecting telemetry data from software applications.

Telemetry is information produced by a running application that helps us understand its behavior.

Three common telemetry signals are:

- **Traces:** Show the sequence of operations associated with a request.
- **Metrics:** Represent measurements such as latency, request counts, and error rates.
- **Logs:** Record events, messages, and diagnostic information.

In an AI agent, these signals complement one another.

For example, a metric might reveal that the average response time has increased. A trace can help identify which operation is responsible. Logs can provide additional information about the error or event associated with that operation.

Frameworks such as smolagents can be instrumented to send execution information to compatible observability systems.

The purpose of using a standard such as OpenTelemetry is to reduce dependence on custom instrumentation for every individual component.

However, not every platform or framework supports every signal or attribute in the same way. The instrumentation and configuration determine what information is actually available.

## 3. Understanding Traces and Spans

Traces and spans are among the most important concepts in agent observability.

### 3.1 What Is a Trace?

A **trace represents an entire operation or workflow from beginning to end**.

For example, suppose a user asks:

"Find information about a historical event and summarize the main facts."

The agent might search the web, retrieve documents, make multiple model calls, and then produce a summary.

The complete execution can be represented as one trace.

### 3.2 What Is a Span?

A **span represents an individual operation within a trace**.

Examples include:

- Calling an LLM.
- Executing a search tool.
- Retrieving documents.
- Calling an external API.
- Running a particular application function.

A trace may contain several spans, and spans can be organized hierarchically to show how operations relate to one another.

For example:

```text
Trace: Research Agent Request
|
+-- Span: Initial LLM Call
|
+-- Span: Search Tool
|
+-- Span: Second LLM Call
|
+-- Span: Document Retrieval
|
+-- Span: Final LLM Call
|
+-- Span: Return Response
```

This structure helps us understand the execution path rather than inspecting each operation in isolation.

### 3.3 Why Are Traces Useful?

Imagine that an agent takes 32 seconds to answer a question.

The total duration alone does not explain the cause.

A trace might reveal the following:

| Operation | Example duration |
|---|---:|
| Initial model call | 5 seconds |
| Search tool | 3 seconds |
| Second model call | 18 seconds |
| Document retrieval | 2 seconds |
| Final model call | 4 seconds |
| Total | 32 seconds |

These numbers are illustrative, not benchmark results.

The trace suggests that the second model call deserves investigation because it accounts for the largest portion of the total time.

We might test a faster model, simplify the prompt, reduce the context size, or investigate whether the agent could avoid that call.

This is more useful than optimizing a component simply because it looks complicated.

**Important:** The duration of a trace is not always equal to the sum of all span durations. Some operations may overlap or execute concurrently, and parent spans may include time spent in their child spans.

## 4. Key Metrics to Monitor in AI Agents

Observability becomes more useful when we know which measurements matter.

The appropriate metrics depend on the agent's purpose. A mathematical reasoning agent, a retrieval agent, and a customer-support agent may need different definitions of success.

### 4.1 Latency

**Latency measures how long an operation takes to complete.**

For an agent, we may measure:

- Total request duration.
- Duration of individual model calls.
- Tool execution time.
- Time spent waiting for external APIs.
- Time taken to produce the first response token, when applicable.

Latency directly affects user experience.

For example, a simple question-answering agent that takes 20 seconds to respond may feel unnecessarily slow.

Tracing helps us distinguish between time spent reasoning, executing tools, waiting for network responses, and performing other application operations.

Possible optimizations include:

- Using a faster model where quality remains acceptable.
- Reducing unnecessary model calls.
- Caching repeated results.
- Improving slow tool implementations.
- Running independent operations concurrently when safe.
- Reducing unnecessarily large prompts and retrieved contexts.

Optimization should be based on measurements. A faster workflow is not necessarily better if it produces significantly worse results.

### 4.2 Cost

Many hosted language models charge according to token usage. External APIs may also charge per request or operation.

An agent's cost can increase when it:

- Makes too many model calls.
- Generates unnecessarily long responses.
- Sends excessive context to the model.
- Repeatedly invokes the same tool.
- Enters a loop and continues making requests.
- Uses an expensive model for a task that a cheaper model could handle.

A simplified cost calculation is:

\[
C = \frac{T_i}{10^6}P_i + \frac{T_o}{10^6}P_o + C_{\text{tools}}
\]

where:

- \(T_i\) is the number of input tokens.
- \(T_o\) is the number of output tokens.
- \(P_i\) is the price per million input tokens.
- \(P_o\) is the price per million output tokens.
- \(C_{\text{tools}}\) represents other applicable tool or service costs.

Actual billing can include additional factors, such as cached tokens, different model pricing tiers, or provider-specific charges.

The formula illustrates how token usage can be converted into an estimated monetary cost.

Monitoring costs helps us answer questions such as:

- How much does one successful task cost?
- Which agent workflow is the most expensive?
- Does a new prompt increase token usage?
- Are repeated calls causing unexpected cost spikes?
- Does a more expensive model provide enough quality improvement to justify its cost?

The objective is not always to minimize cost. It is to understand the trade-off between cost and the quality of the result.

### 4.3 Request Errors

Error monitoring helps us identify how frequently requests fail and why they fail.

Common failures include:

- Model-provider errors.
- API timeouts.
- Invalid tool arguments.
- Missing permissions.
- Malformed responses.
- Network failures.
- Unexpected exceptions.
- Failure to parse a function call.

A robust agent should have appropriate error handling.

For example, if a model provider is temporarily unavailable, the application might retry the request or use a configured alternative provider.

Retries should be limited and should account for whether the operation is safe to repeat. Blindly retrying a tool that performs an irreversible action can cause duplicate operations.

Useful error metrics include total failures, failure rate, timeout rate, retry count, and failure categories.

### 4.4 Explicit User Feedback

Users can directly evaluate the agent's response.

Examples include:

- Thumbs up or thumbs down.
- A rating from one to five stars.
- A written comment.
- A report that an answer was incorrect.

This feedback helps us understand whether the agent is meeting users' expectations.

For example, if users consistently rate responses poorly after the agent retrieves documents, we can investigate the retrieval process, the quality of the documents, or how the model uses the retrieved information.

Feedback is most useful when it can be connected to the relevant agent trace and its execution details.

### 4.5 Implicit User Feedback

Users also communicate dissatisfaction through their behavior, even when they do not submit a rating.

Examples include:

- Rephrasing the same question immediately.
- Repeatedly requesting a correction.
- Clicking a retry button.
- Abandoning a workflow before completion.
- Repeating a search because the first answer was not useful.

These signals can help identify possible problems.

However, implicit feedback must be interpreted carefully. A user might rephrase a question to explore another perspective rather than because the original answer was incorrect.

Therefore, behavioral signals should be treated as evidence to investigate, not definitive proof of failure.

### 4.6 Accuracy and Task Success

Accuracy measures whether an agent produces correct or desirable results according to a defined criterion.

The definition depends on the task.

For a mathematical agent, success may mean producing the correct numerical answer.

For a retrieval agent, it may mean retrieving relevant documents and grounding the response in them.

For a coding agent, it may mean generating code that passes the required tests.

For a customer-support agent, it may mean resolving the customer's issue accurately and safely.

A simple success rate is:

\[
\text{Success Rate} =
\frac{\text{Successful Tasks}}{\text{Total Tasks}}
\times 100
\]

For example, if an agent successfully completes 90 out of 100 test tasks:

\[
\text{Success Rate} = \frac{90}{100}\times100 = 90\%
\]

This metric is meaningful only if the definition of a successful task is clear and consistently applied.

## 5. Instrumenting a smolagents Agent with Langfuse

Let's now look at a practical workflow for recording agent execution.

In this example, we use smolagents for the agent and Langfuse as the observability platform.

The general workflow is:

1. Install the required libraries.
2. Configure the observability credentials.
3. Initialize the client.
4. Instrument the agent framework.
5. Run an agent.
6. Inspect the resulting traces.

The exact SDK interfaces can change between versions, so code should be tested with compatible versions of the libraries.

### 5.1 Install the Required Libraries

A setup similar to the course example is:

```python
%pip install langfuse "smolagents[telemetry]" \
    openinference-instrumentation-smolagents \
    datasets "smolagents[gradio]" gradio --upgrade
```

These packages provide the components needed for agent execution, instrumentation, feedback interfaces, and dataset-based evaluation.

For a production project, use a tested dependency set rather than automatically upgrading every package during each deployment.

### 5.2 Configure Environment Variables

The observability client needs credentials to send trace information to the configured project.

A simplified example is:

```python
import os

os.environ["LANGFUSE_PUBLIC_KEY"] = "your-public-key"
os.environ["LANGFUSE_SECRET_KEY"] = "your-secret-key"
os.environ["LANGFUSE_HOST"] = "your-configured-host"

os.environ["HF_TOKEN"] = "your-hugging-face-token"
```

Replace the placeholder values with the appropriate credentials.

Environment variables keep credentials out of the source code, provided they are configured securely and not accidentally printed or committed to version control.

In production, use an appropriate secret-management mechanism and restrict access to credentials.

The Hugging Face token is relevant when the selected inference configuration requires it.

### 5.3 Initialize the Langfuse Client

After configuring the environment, initialize the client:

```python
from langfuse import get_client

langfuse = get_client()

if langfuse.auth_check():
    print("Langfuse client is authenticated and ready!")
else:
    print("Authentication failed. Check your credentials.")
```

The authentication check helps verify that the client can authenticate with the configured service.

If authentication fails, verify the host, credentials, project configuration, and network connectivity.

### 5.4 Instrument smolagents

The instrumentation layer captures supported execution information from the agent framework.

```python
from openinference.instrumentation.smolagents import (
    SmolagentsInstrumentor
)

SmolagentsInstrumentor().instrument()
```

Once instrumentation is configured, compatible agent operations can produce telemetry that the observability platform can record.

The exact information collected depends on the framework, instrumentation version, and provider configuration.

### 5.5 Test with a Simple Agent

Before monitoring a complex workflow, start with a simple test.

```python
from smolagents import CodeAgent, InferenceClientModel

agent = CodeAgent(
    tools=[],
    model=InferenceClientModel()
)

result = agent.run("Calculate 1 + 1.")
print(result)
```

The agent receives a simple request and attempts to produce the answer.

After running the test, inspect the configured observability dashboard.

You should verify that the expected trace information has been recorded before moving on to more complex workflows.

If no trace appears, investigate the instrumentation setup, authentication, framework compatibility, and telemetry delivery configuration.

Starting with a small test makes it easier to separate instrumentation problems from agent logic problems.

## 6. Inspecting a More Complex Agent Trace

Once the simple test works, we can introduce a tool.

For example, a search-enabled agent might look like this:

```python
from smolagents import (
    CodeAgent,
    DuckDuckGoSearchTool,
    InferenceClientModel
)

search_tool = DuckDuckGoSearchTool()

agent = CodeAgent(
    tools=[search_tool],
    model=InferenceClientModel()
)

result = agent.run(
    "Estimate how many Rubik's Cubes could fit inside "
    "Notre-Dame Cathedral. Explain your assumptions."
)

print(result)
```

This is an estimation task that requires the agent to reason about dimensions and may use a search tool to retrieve relevant information.

The exact result depends on the information available to the agent, its model, and its assumptions.

A recorded trace can contain spans for model calls and tool calls.

By inspecting those spans, we can investigate:

- Which tools were invoked.
- How many model calls occurred.
- How long individual operations took.
- Where execution slowed down.
- Whether an operation failed.
- How token usage changed between runs.

The key advantage is that we can inspect the execution path rather than judging the system only by its final answer.

## 7. Adding Metadata to Traces

Trace metadata helps organize and filter agent runs.

Useful attributes may include:

- User identifier.
- Session identifier.
- Agent version.
- Model configuration.
- Environment.
- Experiment name.
- Task category.

For example:

```python
from smolagents import (
    CodeAgent,
    DuckDuckGoSearchTool,
    InferenceClientModel
)

search_tool = DuckDuckGoSearchTool()

agent = CodeAgent(
    tools=[search_tool],
    model=InferenceClientModel()
)

with langfuse.start_as_current_span(
    name="Smolagent-Trace"
) as span:
    question = "What is the capital of Germany?"

    response = agent.run(question)

    span.update_trace(
        input=question,
        output=str(response),
        user_id="example-user-123",
        session_id="example-session-456",
        tags=["city-question", "testing-agents"],
        metadata={
            "agent_version": "v1",
            "environment": "development"
        }
    )

langfuse.flush()
```

This example illustrates the general pattern of creating a trace and attaching contextual information.

The exact span and trace APIs should be checked against the installed SDK version.

### Why Is Metadata Useful?

Suppose you deploy two agent versions.

Version A uses one model and prompt. Version B uses a different model and a revised prompt.

If you record the version and configuration in the trace metadata, you can compare their latency, cost, error rates, and evaluation scores.

This makes it easier to determine whether the new version actually improves the system.

Metadata must be handled carefully. Avoid recording passwords, access tokens, private user information, or other sensitive content unnecessarily. Even fields such as email addresses can create privacy and data-retention concerns.

## 8. Online Evaluation: Evaluating Agents in Production

Observability records what happens during agent execution. Evaluation determines whether the agent's behavior is acceptable.

**Online evaluation measures agent performance using real-world interactions while the application is in use.**

It can help identify problems that were not represented in the original test dataset.

Examples of online evaluation signals include:

- Task completion rate.
- User satisfaction.
- Response latency.
- Estimated cost.
- Error rate.
- Automatically generated quality scores.
- Repeated requests for clarification.

Online evaluation is valuable because user behavior and application requirements can change over time.

However, live interactions often do not come with known correct answers. Therefore, online evaluation may require user feedback, automated judging, business outcomes, or a combination of signals.

### 8.1 Collecting User Feedback with Gradio

Suppose we build a small chat interface for our agent.

We want users to rate the agent's responses using a thumbs-up or thumbs-down mechanism.

The general workflow is:

1. A user submits a question.
2. The agent generates a response.
3. The application records the agent trace.
4. The user rates the response.
5. The application records the rating against the corresponding trace.
6. We analyze feedback to identify recurring issues.

A simplified interface pattern is:

```python
import gradio as gr
from smolagents import CodeAgent, InferenceClientModel
from langfuse import get_client

langfuse = get_client()

model = InferenceClientModel()

agent = CodeAgent(
    tools=[],
    model=model,
    add_base_tools=True
)

def respond(message, history):
    with langfuse.start_as_current_span(
        name="Smolagent-Trace"
    ):
        output = agent.run(message)

    history = history + [
        {
            "role": "assistant",
            "content": str(output)
        }
    ]

    return history

with gr.Blocks() as demo:
    chatbot = gr.Chatbot(
        label="AI Agent",
        type="messages"
    )

    prompt_box = gr.Textbox(
        placeholder="Ask a question...",
        label="Your message"
    )

    prompt_box.submit(
        fn=respond,
        inputs=[prompt_box, chatbot],
        outputs=chatbot
    )

demo.launch()
```

This demonstrates the basic chat interface. A complete feedback implementation must also associate each rating with the correct trace.

For a multi-user application, avoid using a single global trace identifier. Concurrent requests could overwrite each other's identifiers and attach feedback to the wrong interaction.

Instead, maintain a reliable mapping between each response and its trace, using request-scoped or session-aware state as appropriate.

### 8.2 Turning Ratings into Evaluation Scores

A simple feedback system might map ratings to numeric values.

For example:

- Thumbs up: `1`
- Thumbs down: `0`

These values can be recorded against the corresponding trace.

Conceptually:

```python
if user_liked_response:
    score = 1
else:
    score = 0
```

Over time, these ratings can reveal patterns.

For example, perhaps users consistently dislike responses generated after a particular tool fails. We can investigate those traces and determine whether improved error handling or a different tool strategy would help.

User feedback is valuable, but it is not a perfect measure of factual correctness. Users can misunderstand an answer or rate a response according to preferences unrelated to its accuracy.

## 9. LLM-as-a-Judge

Another approach to evaluating agent outputs is **LLM-as-a-judge**.

In this method, a separate language model evaluates an agent's output against a defined criterion.

For example, a judge model could assess whether a response is:

- Relevant to the question.
- Helpful.
- Consistent with provided evidence.
- Correct according to a reference answer.
- Written in the requested style.
- Potentially harmful or toxic.

The judge is given an evaluation instruction and returns a score, label, or explanation.

### 9.1 How LLM-as-a-Judge Works

The workflow can be represented as follows:

```text
User Request
     |
     v
Agent Generates Response
     |
     v
Evaluation Prompt
     |
     v
Judge LLM
     |
     v
Score or Label
     |
     v
Record Evaluation Result
```

For example, we might ask the judge:

"Evaluate whether the answer is relevant to the user's question. Return a score from 1 to 5 and a short justification."

A structured evaluation could look like this:

```json
{
  "relevance_score": 5,
  "justification": "The response directly addresses the question."
}
```

This is an illustrative result, not an actual model evaluation.

### 9.2 Example: Checking for Toxicity

A toxicity evaluator might use an instruction such as:

```text
You are evaluating an AI assistant's response.

Determine whether the response contains toxic or abusive language.

Return:
- label: toxic or not_toxic
- reason: a brief explanation
```

The evaluator receives the agent's response and produces a classification.

That classification can be recorded as a score associated with the original trace.

### 9.3 Advantages of LLM-as-a-Judge

LLM-based evaluation can help scale quality checks when manually reviewing every response would be expensive.

It can also evaluate qualitative properties that are difficult to capture with simple exact-match metrics.

However, an LLM judge is not automatically reliable.

Possible problems include:

- Incorrect judgments.
- Bias toward particular response styles.
- Sensitivity to wording and prompt changes.
- Inconsistent scoring.
- High evaluation cost.
- Failure to recognize subtle factual errors.

A strong evaluation process should validate judge results against human-reviewed examples and use clear scoring criteria.

For important decisions, combine automated evaluation with deterministic checks or human review when appropriate.

## 10. Offline Evaluation

Online evaluation helps us understand real-world behavior. However, we should not wait until an agent is deployed to discover whether it works.

**Offline evaluation tests an agent using a controlled dataset before or alongside production deployment.**

The dataset usually contains inputs and, when available, expected outputs or reference labels.

For example, a mathematical reasoning agent might be tested on a collection of questions whose correct answers are already known.

The basic workflow is:

1. Prepare a test dataset.
2. Run the agent on each example.
3. Collect the generated responses.
4. Compare the responses with expected results or evaluate them using an appropriate scoring method.
5. Calculate aggregate metrics.
6. Investigate failures.
7. Repeat the evaluation after making changes.

Offline evaluation is repeatable, making it useful for comparing models, prompts, tools, and agent versions.

### 10.1 Why Use a Benchmark Dataset?

Suppose we develop an agent that solves mathematical word problems.

We want to know whether a new prompt improves its performance or merely makes its answers longer.

We can run both versions on the same set of questions and compare their results.

If the dataset contains reference answers, we can calculate how many problems the agent solves correctly.

A benchmark dataset should represent the types of tasks the agent is expected to handle.

It should also contain difficult cases and edge cases rather than only straightforward examples.

### 10.2 Using GSM8K

GSM8K is a dataset of grade-school mathematical word problems with answers and solution explanations.

It can be used to evaluate mathematical reasoning systems.

A simplified example of loading the dataset is:

```python
import pandas as pd
from datasets import load_dataset

dataset = load_dataset(
    "openai/gsm8k",
    "main",
    split="train"
)

df = pd.DataFrame(dataset)

print(df.head())
```

This loads the training split for demonstration purposes.

For a more meaningful evaluation, select an appropriate held-out split or a separate test set rather than measuring performance only on training examples.

The dataset contains questions and reference solutions. Evaluating an agent may require extracting the final numerical answer and comparing it with the reference answer using a consistent normalization procedure.

Exact string comparison can be misleading when answers use different formatting, so the evaluation method should match the task.

### 10.3 Registering a Dataset for Evaluation

An observability platform can help organize evaluation datasets and associate each test case with its results.

A conceptual dataset item might contain:

```python
{
    "input": {
        "question": "If a box contains 12 apples and 5 are removed, how many remain?"
    },
    "expected_output": {
        "answer": "7"
    },
    "metadata": {
        "category": "arithmetic"
    }
}
```

This is a simplified example of an evaluation record.

A complete evaluation dataset may include additional information such as a reference explanation, difficulty level, source, or expected tool behavior.

Organizing the dataset makes it easier to repeat experiments and compare different agent versions.

### 10.4 Running an Agent on a Dataset

Once the dataset is prepared, we can run the agent on each test case and record its output.

A simple local evaluation loop might look like this:

```python
results = []

for example in test_examples:
    question = example["question"]
    expected = example["expected_answer"]

    prediction = agent.run(question)

    results.append({
        "question": question,
        "expected": expected,
        "prediction": str(prediction)
    })
```

Here, `test_examples` is a collection of test records that you prepare beforehand.

The code collects the agent's predictions. It does not yet determine whether those predictions are correct.

The next step is to apply an appropriate evaluation function.

For a numerical task, that function might extract and compare final numerical answers. For a retrieval task, it might measure whether relevant documents were retrieved. For a coding task, it might execute tests in a controlled environment.

The evaluation method must be designed for the actual objective.

### 10.5 Comparing Multiple Experiments

Offline evaluation becomes especially useful when comparing configurations.

For example, we might compare:

- Two different models.
- Different system prompts.
- Different tool configurations.
- Different retrieval settings.
- Different agent frameworks.
- Different temperature settings.

Each configuration should be tested on the same evaluation cases whenever possible.

The results can then be compared using accuracy, task success, latency, cost, and other relevant metrics.

A model that produces slightly more accurate answers may also be significantly slower or more expensive. Evaluating several dimensions helps us understand the trade-offs.

## 11. Online and Offline Evaluation Together

Online and offline evaluation solve complementary problems.

| Aspect | Offline evaluation | Online evaluation |
|---|---|---|
| Environment | Controlled test environment | Real-world application usage |
| Data | Prepared test datasets | Live interactions |
| Ground truth | Often available | Often incomplete |
| Repeatability | High when configuration is fixed | Depends on live traffic |
| Main benefit | Regression testing and controlled comparisons | Detecting real-world problems |
| Main challenge | Test data may not represent every real-world situation | Reliable labels can be difficult to obtain |

A strong development process uses both approaches.

For example, offline tests might reveal that a new model improves performance on known tasks. Online monitoring might later reveal that the same model struggles with a new type of user request.

We can then collect representative failure cases, add suitable examples to the evaluation dataset, and test future changes against them.

The resulting cycle is:

```text
Offline Evaluation
        |
        v
Deploy Agent Version
        |
        v
Monitor Live Performance
        |
        v
Collect Feedback and Failures
        |
        v
Investigate Problems
        |
        v
Update Evaluation Dataset
        |
        v
Improve and Test Agent
        |
        +----------> Repeat
```

This process helps prevent regressions and encourages continuous improvement.

## 12. Common Mistakes in Agent Observability and Evaluation

### Mistake 1: Looking only at the final answer

A correct final answer does not necessarily mean the execution was efficient or safe. Inspect the trace to understand the operations that produced it.

### Mistake 2: Tracking latency without tracking cost

A workflow may be fast but expensive because it uses many model calls. Monitor both metrics.

### Mistake 3: Trusting a single benchmark

An agent may perform well on a small test set but fail on unfamiliar requests. Include representative tasks, edge cases, and examples based on real failures.

### Mistake 4: Treating every user rating as ground truth

Ratings are useful signals, but they do not always measure factual accuracy. Combine feedback with other evaluation methods.

### Mistake 5: Assuming an LLM judge is always correct

Judge models can make mistakes. Validate them using human-reviewed examples and explicit scoring criteria.

### Mistake 6: Ignoring privacy in traces

Traces may contain user inputs, generated responses, and tool outputs. Apply appropriate access controls, data minimization, retention policies, and redaction.

### Mistake 7: Recording telemetry without acting on it

Collecting metrics is only the first step. Use the information to identify problems, make changes, and verify that those changes improve performance.

# Part 2: AI Agents in Games

## 13. Introduction to LLMs and Agentic AI in Games

Large Language Models introduce new possibilities for game development.

Traditionally, many non-playable characters (NPCs) use predefined dialogue, scripted actions, state machines, or behavior trees.

These techniques are useful because they are predictable and can be designed for specific gameplay requirements.

However, they can limit the variety of conversations and decisions that a character can make.

LLMs allow game characters to generate more flexible dialogue and respond to player interactions in natural language.

Agentic AI extends this idea by allowing a system to observe its environment, select actions, pursue objectives, and adapt its behavior.

The difference is important: **an LLM can generate dialogue, while an agentic game character can use a decision-making loop to act within the game environment.**

## 14. Examples of LLMs in Games

Several demonstrations and games illustrate how generative AI can be used to create more interactive characters.

### 14.1 Covert Protocol

Covert Protocol is a technology demonstration featuring a private-detective scenario.

AI-powered characters can respond to player inquiries in real time, allowing conversations to influence how the player experiences the narrative.

The demonstration combines a game engine with AI-powered character technology.

Its main lesson is that NPC dialogue can become more dynamic than a fixed set of scripted responses.

### 14.2 NEO NPCs

NEO NPCs is a prototype presented by Ubisoft that explores generative AI for game characters.

The concept focuses on characters that can perceive aspects of their environment, remember interactions, and engage in more flexible conversations.

Such capabilities can make the game world feel more responsive to the player.

However, memory and perception must be implemented by the surrounding system; a language model does not automatically possess persistent memory or direct access to a game engine.

### 14.3 Mecha BREAK and NVIDIA ACE

AI-powered character technologies can also support more natural interactions in games featuring futuristic or multiplayer settings.

NVIDIA's Avatar Cloud Engine, or ACE, is a technology platform for developing AI-powered digital characters.

These approaches explore how players might interact with characters using natural language and other forms of input.

The broader lesson is that AI-powered NPCs can combine language understanding, character animation, perception, and game-specific logic.

### 14.4 Suck Up!

Suck Up! is a game in which the player takes the role of a vampire attempting to convince AI-powered characters to invite them into their homes.

The interactions depend on persuasion and dialogue rather than only following a fixed sequence of conversation options.

This provides an example of how generative AI can introduce more varied interactions into a game.

### 14.5 What Do These Examples Teach Us?

These examples demonstrate the potential of LLM-powered characters, but flexible dialogue alone does not necessarily make a character an autonomous agent.

A character may produce convincing dialogue while still following a largely fixed behavioral structure.

To build an agentic character, we need additional mechanisms for observing the game state, planning actions, maintaining relevant state, and executing decisions.

## 15. From LLMs to AI Agents

Consider a traditional role-playing game NPC.

### An LLM-Powered NPC

The character may respond naturally when the player asks a question.

For example:

Player: "Where can I find help?"

NPC: "There is a healer near the northern gate. You should speak with her."

The response is more flexible than a fixed dialogue line, but the character may not take any independent action.

### An Agentic NPC

An agentic character may have goals and a mechanism for choosing actions.

For example, it might:

- Notice that a nearby character is in danger.
- Decide that help is needed.
- Move toward a designated location.
- Request assistance from another character.
- Change its plan when the environment changes.

The exact behavior depends on the game's action system, available observations, and agent design.

An agentic NPC can be designed around three important properties.

**Autonomy:** It can choose actions according to its goals and the current environment state.

**Adaptability:** It can change its behavior when the situation or player actions change.

**Persistence:** It can retain relevant state or memories across interactions, depending on the system's implementation.

These properties allow the character to participate in the game world rather than merely responding to dialogue.

## 16. The Main Challenge: Latency

One major limitation of LLM-driven agents in games is execution latency.

A fast-paced game may need to update its state and process input many times per second.

At 30 frames per second, a game has approximately:

\[
\frac{1000}{30} \approx 33.3 \text{ milliseconds per frame}
\]

This does not mean that every NPC decision must involve an LLM call, but it illustrates the tight timing requirements of real-time gameplay.

A multi-step agent that requires several model calls and tool executions may take much longer than this.

For example, an agent might need to:

1. Interpret the current game state.
2. Decide on a plan.
3. Generate a structured action.
4. Wait for a model response.
5. Validate the action.
6. Return the decision to the game.

If this entire process takes several seconds, it is unsuitable for directly controlling every frame of a fast-paced game.

### 16.1 Why Turn-Based Games Are a Good Starting Point

Turn-based games allow a decision to be made between player actions.

The agent can inspect the current state, reason about the available choices, and select an action before the next turn is resolved.

Pokémon-style battles are a useful example because players choose moves or switch Pokémon at discrete decision points.

The agent does not need to generate a new decision 30 times per second. It needs to produce a valid action when the game requests one.

This makes turn-based games a practical environment for experimenting with LLM-driven decision-making.

## 17. Building a Pokémon Battle Agent

Now let's explore how to build an agent that plays Pokémon-style battles.

The course material organizes the system around four main components:

1. **Poke-env:** A Python library that provides an interface for interacting with Pokémon battle environments.
2. **Pokémon Showdown:** A battle simulator that manages the game rules and battle state.
3. **LLMAgentBase:** A base class that connects an LLM to the battle environment and handles common decision logic.
4. **TemplateAgent:** A starter implementation where we provide the model-specific logic for selecting actions.

Each component has a distinct responsibility.

## 18. Understanding Poke-env

Poke-env is a Python interface designed for interacting with Pokémon battle environments. It has been used for reinforcement-learning bots and can also be adapted for LLM-based agents.

The library provides a `Player` class that handles interactions with the battle environment.

Our custom agent can inherit from this class rather than implementing all communication with the simulator from scratch.

Conceptually, the workflow is:

```text
Pokémon Showdown
        |
        v
     Poke-env
        |
        v
   Custom Agent
        |
        v
    LLM Decision
        |
        v
  Selected Action
        |
        v
Pokémon Showdown
```

Poke-env provides the game-facing interface. The LLM provides one possible mechanism for choosing actions.

The agent must still obey the rules and action constraints of the battle environment.

## 19. Understanding Pokémon Showdown

Pokémon Showdown is a battle simulator that supports Pokémon battles.

In our agent architecture, it serves as the environment in which the agent plays.

The simulator manages the battle state, resolves moves, and determines the effects of actions.

At each decision point, the agent receives information about the current battle and chooses an action.

For example, the agent might need to decide whether to:

- Use an available attack.
- Switch to another Pokémon.
- Respond to an opponent's strategy.
- Choose a different move after its active Pokémon becomes disadvantaged.

The simulator resolves the action and produces the updated battle state.

The agent then receives the next observation and makes another decision.

This creates an agent loop:

```text
Observe Battle State
        |
        v
Choose Action
        |
        v
Validate Action
        |
        v
Submit to Simulator
        |
        v
Resolve Turn
        |
        v
Receive Updated State
        |
        +---------> Repeat
```

The battle simulator is responsible for the game rules. The agent is responsible for selecting actions using the information available to it.

## 20. Designing LLMAgentBase

`LLMAgentBase` is a custom Python class that extends the Poke-env `Player` class.

Its purpose is to connect the LLM to the battle environment while centralizing the common logic needed by different agent implementations.

The base class can handle:

- Formatting the current battle state.
- Defining the available action tools.
- Requesting a decision from the model.
- Parsing the model's function call.
- Validating the requested action.
- Converting the action into a simulator-compatible command.
- Handling invalid responses and fallback behavior.

By separating these responsibilities, we can experiment with different model providers without rewriting the entire battle integration.

### 20.1 The Standard Tool Schema

The agent exposes two principal action tools in the course example:

**`choose_move`**

Selects an attack from the available moves.

**`choose_switch`**

Switches the active Pokémon to an eligible Pokémon from the team.

A simplified representation of these tools is:

```python
STANDARD_TOOL_SCHEMA = {
    "choose_move": {
        "name": "choose_move",
        "description": "Select an available battle move.",
        "parameters": {
            "move_name": "string"
        }
    },
    "choose_switch": {
        "name": "choose_switch",
        "description": "Switch to an available Pokémon.",
        "parameters": {
            "pokemon_name": "string"
        }
    }
}
```

This is an illustrative schema, not a complete provider-specific function-calling specification.

The actual schema must match the function-calling format supported by the selected model provider.

The important design idea is that the model does not need to return an arbitrary paragraph explaining what it wants to do. It can return a structured action request.

For example:

```json
{
  "name": "choose_move",
  "arguments": {
    "move_name": "thunderbolt"
  }
}
```

The agent can parse this result and verify that the requested move is actually available.

## 21. Formatting the Battle State

The LLM cannot make a useful decision without information about the current situation.

The base agent therefore converts the battle object into a textual representation that the model can understand.

The course example includes information such as:

- The active Pokémon's species.
- Its types.
- Its remaining HP.
- Its status condition.
- Its stat boosts.
- The opponent's active Pokémon.
- Available moves.
- Available switches.
- Weather.
- Terrain.
- Side conditions affecting both players.

A simplified example of the formatted state might look like this:

```text
Your active Pokemon:
Species: Pikachu
Type: Electric
HP: 72%
Status: None

Opponent's active Pokemon:
Species: Squirtle
Type: Water
HP: 55%
Status: None

Available moves:
- thunderbolt
- quickattack

Available switches:
- Bulbasaur (HP: 90%)
- Charizard (HP: 65%)

Weather: None
Terrain: None
```

This example is illustrative. Actual moves and Pokémon information depend on the battle configuration.

The purpose of state formatting is to transform structured simulator data into a representation that the model can use to make a decision.

### 21.1 Why Include Available Actions?

Suppose the model decides to use a move that the active Pokémon does not know.

The request is invalid even if the model's strategic reasoning sounds convincing.

Providing the list of available moves reduces this problem by telling the model which actions are currently legal.

Similarly, the list of available switches helps the model choose a Pokémon that can actually be selected.

However, listing valid options does not guarantee that the model will always choose one. The application must still validate the result.

### 21.2 Handling the Opponent's State

The agent also needs information about the opponent.

Depending on the game rules and what the environment exposes, this may include:

- The opponent's active Pokémon.
- Its visible type information.
- Its remaining HP.
- Its known status condition.
- Relevant battle effects.

Some information may be unknown or hidden.

The agent should represent unavailable information accurately rather than inventing values.

This is important because an LLM can otherwise generate a confident strategy based on assumptions that the simulator has not confirmed.

## 22. Implementing the Core Decision Logic

The base agent's main decision method is `choose_move`.

Despite its name, the method can handle both attacking and switching.

Its responsibilities are:

1. Format the current battle state.
2. Request an LLM decision.
3. Extract the function name and arguments.
4. Check whether the requested action is valid.
5. Convert the action into a command for the simulator.
6. Use a fallback if the decision cannot be processed.

The general structure is:

```python
async def choose_move(self, battle):
    battle_state = self._format_battle_state(battle)

    decision_result = await self._get_llm_decision(
        battle_state
    )

    decision = decision_result.get("decision")

    if decision:
        function_name = decision.get("name")
        arguments = decision.get("arguments", {})

        if function_name == "choose_move":
            move_name = arguments.get("move_name")

            # Validate the requested move.
            # Convert a valid move into a simulator action.

        elif function_name == "choose_switch":
            pokemon_name = arguments.get("pokemon_name")

            # Validate the requested switch.
            # Convert a valid switch into a simulator action.

    # If the decision is invalid, use a safe fallback.
```

This is a conceptual outline rather than a complete implementation.

The course's base class includes additional details for resolving move names, finding eligible Pokémon, creating simulator orders, and handling errors.

### 22.1 Resolving a Move Name

The model may return a move name in a different format from the simulator's internal identifier.

For example, the model might produce a display name while the simulator expects a normalized move ID.

The agent can normalize the name and compare it against the available moves.

The preferred approach is to match the internal ID when possible. A fallback match against the display name may be useful, but it should be handled carefully because display names can be ambiguous or formatted differently.

If the requested move cannot be matched to an available move, the agent should reject the decision instead of submitting an invalid action.

### 22.2 Resolving a Pokémon Switch

Switching requires similar validation.

The agent must find the requested Pokémon among the available switches, not merely among every Pokémon that has ever appeared in the battle.

A Pokémon might be unavailable because it has fainted or because the current battle state does not permit the requested switch.

The application should validate the target against the simulator's current list of available switches.

### 22.3 Why Validation Is Essential

LLMs generate outputs probabilistically. Even a fine-tuned model can occasionally produce an invalid function name, missing argument, or unavailable move.

A reliable agent therefore separates the model's decision from the final action submitted to the simulator.

The model proposes an action. The application validates the action. Only a valid action is executed.

This is a general design principle that also applies to agents controlling databases, APIs, files, and other external systems.

## 23. Fallbacks and Error Handling

What happens when the model fails to produce a valid decision?

Possible causes include:

- The model provider returns an error.
- The response cannot be parsed.
- The function name is unknown.
- Required arguments are missing.
- The requested move is unavailable.
- The requested Pokémon cannot be selected.

The course's base agent uses a fallback strategy when a decision cannot be processed.

When valid moves or switches are available, it can choose a random legal action. If no such action is available, it can fall back to the simulator's default action behavior.

A simplified example is:

```python
if valid_action_found:
    return selected_action

if battle.available_moves or battle.available_switches:
    return self.choose_random_move(battle)

return self.choose_default_move(battle)
```

The exact return values and methods depend on the Poke-env version and the base class implementation.

A random fallback is useful for keeping the battle moving, but it is not necessarily strategically strong.

For a more advanced agent, we could implement a rule-based fallback that selects an available move or switch according to a simple heuristic.

The key principle is that a malformed LLM response should not automatically become an invalid game action.

## 24. Implementing TemplateAgent

Once the base class handles the game integration, we can implement a model-specific agent.

The course uses a `TemplateAgent` abstraction to demonstrate where the developer should add their own model-provider logic.

The template is responsible for:

- Initializing the model client.
- Defining the system prompt.
- Formatting the battle request.
- Sending the request to the model.
- Extracting the function call.
- Returning a structured decision to the base class.

A simplified structure is:

```python
class TemplateAgent(LLMAgentBase):
    def __init__(
        self,
        api_key=None,
        model="model-name",
        *args,
        **kwargs
    ):
        super().__init__(*args, **kwargs)

        self.model = model

        # Initialize the appropriate model-provider client.
        self.template_client = ...

        self.template_tools = list(
            self.standard_tools.values()
        )

    async def _get_llm_decision(self, battle_state):
        system_prompt = """
        You are an agent playing a turn-based battle.
        Choose only actions that are available.
        Return a valid structured function call.
        """

        user_prompt = f"""
        Current battle state:

        {battle_state}

        Select the most appropriate action.
        """

        # Send the request to the configured model.
        # Parse its function call.
        # Return the structured decision.
```

The ellipses indicate implementation details that depend on the model provider.

This template is a blueprint, not a directly executable implementation.

Different providers expose different request parameters, tool schemas, and response structures. The developer must adapt the request and response parsing to the chosen provider.

## 25. Connecting the LLM to Function Calling

The battle agent provides a practical example of the function-calling concepts discussed in the previous article.

The model receives a description of the available tools and a representation of the current battle state.

It chooses a function and produces its arguments.

For example, it might request:

```json
{
  "name": "choose_move",
  "arguments": {
    "move_name": "thunderbolt"
  }
}
```

The application parses the function call, checks whether the move is available, and converts it into a legal battle action.

If the model instead requests a switch, the application checks the requested Pokémon against the available switches.

This illustrates the relationship between LLM decision-making and deterministic application logic.

The LLM proposes what to do. The simulator-facing code determines whether the proposal can be executed.

This architecture allows the model to make flexible decisions while retaining control over the actions that enter the game environment.

## 26. Improving the Pokémon Agent's Strategy

A basic function-calling agent may choose legal moves but still perform poorly in battle.

Legal action selection and strategic decision-making are different problems.

To improve the agent, we can provide more useful state information and develop a better decision strategy.

### 26.1 Use Available Move Information

The course's battle-state formatter includes attributes such as move type, base power, accuracy, remaining PP, and move category.

These attributes can help the model compare the available moves.

For example, the agent might consider whether a move is powerful enough, whether it has sufficient accuracy, and whether its type is appropriate for the current matchup.

The model should not assume that the move with the highest base power is always the best choice. Accuracy, type interactions, remaining HP, and other battle conditions can influence the result.

### 26.2 Consider Switching

A useful agent should not always attack.

Switching may be appropriate when another Pokémon has a more favorable matchup or when the active Pokémon is at risk.

The model needs enough information about available switches to compare its options.

The final decision must still respect the legal choices provided by the simulator.

### 26.3 Maintain Relevant Battle History

The base agent can maintain a battle-history structure.

History may be useful for remembering previous observations or decisions during a battle, provided the application records and presents that information appropriately.

However, history should not be confused with verified current state.

The latest simulator observation should remain authoritative for current HP, available moves, and legal actions.

### 26.4 Reduce Unnecessary Model Calls

Each model call introduces latency and potentially additional cost.

A turn-based agent should gather the necessary state, make its decision, and return a valid action without performing unnecessary cycles.

Possible improvements include:

- Simplifying the state representation.
- Using concise prompts.
- Avoiding repeated calls with identical information.
- Choosing an appropriately sized model.
- Using deterministic heuristics for straightforward cases.
- Applying a stronger model only when a more complex decision is needed.

These optimizations should be evaluated against battle performance rather than assumed to improve it.

## 27. Observability for Game Agents

The observability concepts from Part 1 also apply to Pokémon battle agents.

A battle agent can be instrumented to record each decision and the resulting outcome.

For example, a trace might contain:

```text
Trace: Battle Turn 12
|
+-- Span: Format Battle State
|
+-- Span: Call LLM
|
+-- Span: Parse Function Call
|
+-- Span: Validate Move
|
+-- Span: Submit Action
|
+-- Span: Receive Battle Result
```

This helps us investigate problems that would otherwise be difficult to diagnose.

Suppose the agent loses a battle after repeatedly selecting ineffective moves.

We could inspect the recorded state, model response, and action to determine whether the model misunderstood the matchup, received incomplete information, or generated a poor strategy.

We could also track:

- Average decision latency.
- Invalid function-call rate.
- Number of fallback actions.
- Model calls per battle.
- Estimated cost per battle.
- Battle win rate.
- Performance against different opponents.

The win rate is an important outcome metric, but it should not be the only one.

An agent might win frequently against weak opponents while still making invalid decisions or performing poorly against stronger strategies.

Combining execution traces with outcome metrics helps provide a more complete picture.

## 28. Testing the Agent Against Other Players

Once the agent can interact with the battle simulator, the next step is to test it in actual battles.

The course describes a hosted environment in which developers can prepare their own agent implementation and challenge other agents.

The general workflow is:

1. Prepare a compatible copy of the battle-agent application.
2. Implement the custom agent in the appropriate Python file.
3. Register the agent in the application.
4. Select the agent in the interface.
5. Configure the correct battle username.
6. Send a battle invitation to an opponent.
7. Accept the battle and observe the agent's decisions.

The agent must be connected to the correct environment and configured with the required credentials and model settings.

A battle invitation alone does not guarantee that the match will begin; the opponent must be available and the invitation must be accepted.

### 28.1 Why Test Against Different Opponents?

A single battle provides very little evidence about an agent's overall strength.

The result may depend on the opponent's strategy, the teams involved, and the particular sequence of game events.

For a more useful evaluation, run multiple battles and record the outcomes.

A basic win-rate calculation is:

\[
\text{Win Rate} =
\frac{\text{Number of Wins}}{\text{Total Battles}}
\times 100
\]

For example, if an agent wins 60 out of 100 battles:

\[
\text{Win Rate} = \frac{60}{100}\times100 = 60\%
\]

This is an illustrative calculation, not a result from the course's agent.

For fair comparisons, use consistent battle rules and team configurations where possible. Record losses, invalid actions, timeouts, and fallback usage as well as wins.