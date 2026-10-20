---
title: "LangGraph: Building Controlled AI Workflows with Python"
description: "Learn LangGraph from the basics through state, nodes, edges, conditional routing, email classification, document analysis, tool calling, ReAct agents, and tracing."
category: "AI Agents"
tags: ["LangGraph", "LangChain", "AI Agents", "LLM Workflows", "Python", "Tool Calling", "Generative AI"]
---

# Introduction to LangGraph

LangGraph is a framework developed by the LangChain team to manage the control flow of applications that integrate large language models (LLMs).

When building AI applications, calling an LLM once is often not enough. A real application may need to perform several steps:

1. Receive user input.
2. Analyze the input using an LLM.
3. Decide which action to perform next.
4. Call a tool or execute a Python function.
5. Observe the result.
6. Repeat when necessary.
7. Return the final response.

LangGraph helps organize these steps into a structured workflow.

Its main purpose is to give developers control over how an AI application executes, makes decisions, manages state, and moves between tasks.

## What We Will Learn

In this tutorial, we will explore:

1. What LangGraph is and when to use it.
2. The difference between LangChain and LangGraph.
3. The building blocks: state, nodes, edges, and StateGraph.
4. Conditional routing and workflow visualization.
5. Building an email classification workflow.
6. Drafting responses for legitimate emails.
7. Creating a document analysis agent.
8. Extracting text from images using a vision-language model.
9. Calling tools through an LLM.
10. Understanding the ReAct pattern.
11. Monitoring workflow executions with tracing.

By the end, you will understand how to create structured AI workflows and basic tool-using agents with Python.

# 1. What Is LangGraph?

LangGraph is a framework for orchestrating applications that use LLMs.

Think of it as a way to design a flowchart whose steps can contain Python functions, LLM calls, tool execution, conditional decisions, and human interactions.

For example, imagine an AI assistant that answers questions about documents. Different documents may require different processing methods:

- A text document may need ordinary text extraction.
- A PDF containing tables may require table extraction.
- A spreadsheet may require Python and pandas calculations.
- An image may require a vision-language model to understand its contents.

The application must determine which processing path to follow. LangGraph provides a structured way to represent that workflow.

## 1.1 LangChain vs. LangGraph

LangChain and LangGraph are related, but they serve different purposes.

**LangChain** provides interfaces and components for working with LLMs, prompts, tools, retrievers, and other AI components.

**LangGraph** focuses on orchestrating execution: which step runs, what state is passed between steps, and which step executes next.

| Concept | LangChain | LangGraph |
|---|---|---|
| Main purpose | Connect and use LLM components | Orchestrate multi-step workflows |
| Model calls | Provides model interfaces | Can use LangChain models or other components |
| Tool integration | Provides tool interfaces | Controls when tools execute |
| Conditional branching | Can be implemented in application logic | Explicitly represented in graph edges |
| State across steps | Depends on the components used | Central to graph execution |
| Human interaction | Can be implemented with application logic | Supports human-in-the-loop workflow patterns |
| Visualization | Depends on the approach | Graph structure can be visualized |

The packages can be used together or independently. LangGraph does not require every component to come from LangChain.

## 1.2 Control vs. Freedom

When designing an AI application, there is an important trade-off between freedom and control.

**Freedom** allows an LLM-based agent to decide how to solve a problem, which tools to use, and what actions to take.

**Control** allows the developer to define the permitted steps, conditions, transitions, and safeguards.

For example, an agent that freely writes and executes code may be flexible, but its behavior can be harder to predict.

A graph-based workflow can instead enforce a structure such as:

1. Validate the user's request.
2. Classify the request.
3. Select the appropriate processing path.
4. Execute the approved operation.
5. Check the result.
6. Return the response.

LangGraph is particularly useful when this structure matters.

### When should you use LangGraph?

LangGraph is useful for:

- Multi-step reasoning processes.
- Applications that need state across several steps.
- Workflows combining deterministic Python logic and LLM decisions.
- Human-in-the-loop approval processes.
- Applications with conditional branches and loops.
- Complex agent systems containing multiple components.

You could implement these workflows using ordinary Python functions and `if` statements. LangGraph provides abstractions that make complex flows easier to organize, inspect, visualize, and extend.

# 2. The Building Blocks of LangGraph

A LangGraph application is built around a graph. Understanding its four central concepts is essential:

1. State
2. Nodes
3. Edges
4. StateGraph

The graph typically begins at `START` and finishes at `END`, although it may contain conditional paths and loops before completion.

## 2.1 State

**State is the information maintained and passed through the workflow.**

A node can read the current state, perform an operation, and return updates to it.

The state is defined by the developer, so its fields should represent the information the application needs to track.

```python
from typing_extensions import TypedDict

class State(TypedDict):
    graph_state: str
```

Here, `State` describes a dictionary containing a string field named `graph_state`.

An example state is:

```python
{"graph_state": "Hello"}
```

A node might update it to:

```python
{"graph_state": "Hello world"}
```

In a real application, state could contain the user's question, extracted document text, classification results, tool outputs, and conversation messages.

**Tip:** Design the state around the workflow's actual needs. Missing fields can make decisions difficult, while unnecessary fields make the workflow harder to maintain.

## 2.2 Nodes

**Nodes are Python functions that perform individual processing steps.**

A node usually:

1. Receives the current state.
2. Performs an operation.
3. Returns a dictionary containing state updates.

```python
def node_1(state):
    print("---Node 1---")

    return {
        "graph_state": state["graph_state"] + " I am"
    }


def node_2(state):
    print("---Node 2---")

    return {
        "graph_state": state["graph_state"] + " happy!"
    }


def node_3(state):
    print("---Node 3---")

    return {
        "graph_state": state["graph_state"] + " sad!"
    }
```

Each node updates the same state field in a different way.

Nodes can contain:

- LLM calls for generating text or making decisions.
- Tool calls for interacting with external systems.
- Ordinary Python calculations.
- Validation and conditional logic.
- Human approval or input steps.

LangGraph also provides special graph markers such as `START` and `END`. These are not ordinary application nodes that you need to define yourself.

## 2.3 Edges

**Edges define how execution moves from one node to another.**

### Direct edges

A direct edge always connects one node to the next specified node.

```python
builder.add_edge("read_email", "classify_email")
```

This means that after `read_email` completes, execution moves to `classify_email`.

### Conditional edges

A conditional edge selects the next node based on the current state or another decision.

```python
import random
from typing import Literal

def decide_mood(state) -> Literal["node_2", "node_3"]:
    if random.random() < 0.5:
        return "node_2"

    return "node_3"
```

This example chooses between two nodes with approximately equal probability.

The random decision is only for demonstration. In a practical application, the routing function would usually examine the state.

For example:

- If an email is spam, route to the spam handler.
- If an email is legitimate, route to response drafting.
- If a document contains a spreadsheet, route to a calculation step.
- If a user requests a summary, route to a summarization step.

The routing function's return value must match the destinations defined in the graph.

## 2.4 StateGraph

`StateGraph` is the builder used to define the workflow.

It allows you to register nodes, connect them with edges, define conditional routing, and compile the graph for execution.

Here is a complete example combining the previous concepts:

```python
from typing_extensions import TypedDict
from langgraph.graph import StateGraph, START, END


class State(TypedDict):
    graph_state: str


def node_1(state: State):
    print("---Node 1---")

    return {
        "graph_state": state["graph_state"] + " I am"
    }


def node_2(state: State):
    print("---Node 2---")

    return {
        "graph_state": state["graph_state"] + " happy!"
    }


def node_3(state: State):
    print("---Node 3---")

    return {
        "graph_state": state["graph_state"] + " sad!"
    }


def decide_mood(state: State):
    import random

    if random.random() < 0.5:
        return "node_2"

    return "node_3"


builder = StateGraph(State)

builder.add_node("node_1", node_1)
builder.add_node("node_2", node_2)
builder.add_node("node_3", node_3)

builder.add_edge(START, "node_1")

builder.add_conditional_edges(
    "node_1",
    decide_mood
)

builder.add_edge("node_2", END)
builder.add_edge("node_3", END)

graph = builder.compile()
```

Let's understand the structure:

- `StateGraph(State)` creates a graph builder using the specified state schema.
- `add_node()` registers processing functions.
- `add_edge()` creates direct connections.
- `add_conditional_edges()` defines a decision point.
- `compile()` prepares the graph for execution.

### Visualizing the graph

In a notebook environment, you can visualize the graph:

```python
from IPython.display import Image, display

display(
    Image(graph.get_graph().draw_mermaid_png())
)
```

Visualization helps you understand the workflow before running it.

### Invoking the graph

Execute the graph with an initial state:

```python
result = graph.invoke({
    "graph_state": "Hi, this is Lance."
})

print(result)
```

One possible output is:

```text
---Node 1---
---Node 3---

{'graph_state': 'Hi, this is Lance. I am sad!'}
```

The selected branch can vary because the routing function uses randomness.

The important point is that the graph determines the execution order, while the state carries information between nodes.

# 3. Building an Email Sorting Workflow

Let's build a practical workflow.

Alfred receives emails and must decide what to do with each one.

The application needs to:

1. Read an incoming email.
2. Classify it as spam or legitimate.
3. Explain why a message was classified as spam.
4. Categorize legitimate emails.
5. Draft a preliminary response for legitimate emails.
6. Display the email and draft for human review.

This example focuses on LangGraph's workflow orchestration. Although it uses an LLM, it is not a full tool-using agent because the email workflow does not give the model tools to call.

## 3.1 Install the dependencies

```python
%pip install langgraph langchain_openai
```

The example uses a chat model through `ChatOpenAI`. You will need the appropriate model API credentials configured in your environment.

## 3.2 Import the modules

```python
from typing import TypedDict, List, Dict, Any, Optional

from langgraph.graph import StateGraph, START, END
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage
```

## 3.3 Define the state

The workflow needs to track the original email, its classification, and the generated response.

```python
class EmailState(TypedDict):
    email: Dict[str, Any]

    email_category: Optional[str]
    spam_reason: Optional[str]
    is_spam: Optional[bool]

    email_draft: Optional[str]

    messages: List[Dict[str, Any]]
```

The fields have different responsibilities:

| Field | Purpose |
|---|---|
| `email` | Stores the sender, subject, and body |
| `email_category` | Stores the category of a legitimate email |
| `spam_reason` | Stores the explanation for a spam classification |
| `is_spam` | Records whether the email was classified as spam |
| `email_draft` | Stores the preliminary response |
| `messages` | Tracks prompts and model responses |

This state makes the workflow's data explicit.

## 3.4 Initialize the LLM

```python
model = ChatOpenAI(temperature=0)
```

The temperature setting is set to zero to encourage more consistent outputs. It does not guarantee that every model response will be identical in every situation.

## 3.5 Create the email-reading node

The first node reads the email from the state and prints basic information.

```python
def read_email(state: EmailState):
    """Read and log the incoming email."""

    email = state["email"]

    print(
        f"Alfred is processing an email from "
        f"{email['sender']} with subject: "
        f"{email['subject']}"
    )

    return {}
```

This node does not modify the state. It simply logs the email details.

Returning an empty dictionary means that this node provides no state updates.

## 3.6 Create the classification node

The classification node asks the LLM to determine whether the email is spam or legitimate.

For legitimate emails, the model is also asked to identify a category.

```python
def classify_email(state: EmailState):
    """Classify an email using an LLM."""

    email = state["email"]

    prompt = f"""
    As Alfred the butler, analyze this email and determine
    whether it is spam or legitimate.

    Email:
    From: {email['sender']}
    Subject: {email['subject']}
    Body: {email['body']}

    First, determine if this email is spam.
    If it is spam, explain why using the format:
    SPAM
    Reason: explanation

    If it is legitimate, respond with:
    NOT SPAM
    Category: inquiry, complaint, thank you, request,
    or information.
    """

    response = model.invoke([
        HumanMessage(content=prompt)
    ])

    response_text = response.content.lower()

    is_spam = (
        "spam" in response_text
        and "not spam" not in response_text
    )

    spam_reason = None

    if is_spam and "reason:" in response_text:
        spam_reason = (
            response_text.split("reason:", 1)[1].strip()
        )

    email_category = None

    if not is_spam:
        categories = [
            "inquiry",
            "complaint",
            "thank you",
            "request",
            "information"
        ]

        for category in categories:
            if category in response_text:
                email_category = category
                break

    new_messages = state.get("messages", []) + [
        {
            "role": "user",
            "content": prompt
        },
        {
            "role": "assistant",
            "content": response.content
        }
    ]

    return {
        "is_spam": is_spam,
        "spam_reason": spam_reason,
        "email_category": email_category,
        "messages": new_messages
    }
```

### How does classification work?

The node performs several operations:

1. Reads the email from the state.
2. Builds a prompt containing the sender, subject, and body.
3. Calls the LLM.
4. Interprets the returned text.
5. Extracts a spam reason or legitimate-email category.
6. Stores the decision and conversation history in the state.

**Important limitation:** The example uses simple string matching to interpret the model's response. If the model changes its wording, this logic can produce incorrect results.

For production applications, use structured outputs with a validated schema, explicit categories, error handling, and a review path for uncertain classifications. Spam detection should also be tested against malicious messages and false positives.

## 3.7 Create the spam-handling node

If the message is classified as spam, Alfred prints a notification.

```python
def handle_spam(state: EmailState):
    """Handle an email classified as spam."""

    print(
        "Alfred has marked the email as spam. "
        f"Reason: {state['spam_reason']}"
    )

    print("The email has been marked for spam handling.")

    return {}
```

This demonstration only prints a message. It does not actually move an email into a real spam folder.

A production implementation would need a properly authorized email integration to perform that action.

## 3.8 Create the response-drafting node

For a legitimate email, Alfred prepares a preliminary response.

```python
def draft_response(state: EmailState):
    """Draft a preliminary response."""

    email = state["email"]
    category = state["email_category"] or "general"

    prompt = f"""
    As Alfred the butler, draft a polite preliminary
    response to this email.

    Email:
    From: {email['sender']}
    Subject: {email['subject']}
    Body: {email['body']}

    Category: {category}

    Draft a brief, professional response that can be
    reviewed and personalized before sending.
    """

    response = model.invoke([
        HumanMessage(content=prompt)
    ])

    new_messages = state.get("messages", []) + [
        {
            "role": "user",
            "content": prompt
        },
        {
            "role": "assistant",
            "content": response.content
        }
    ]

    return {
        "email_draft": response.content,
        "messages": new_messages
    }
```

This node generates a draft for review; it does not send the email.

## 3.9 Create the notification node

The final node displays the email details and proposed response.

```python
def notify_mr_hugg(state: EmailState):
    """Display the email and its draft response."""

    email = state["email"]

    print("\n" + "=" * 50)

    print(
        f"Sir, you've received an email from "
        f"{email['sender']}."
    )

    print(f"Subject: {email['subject']}")
    print(f"Category: {state['email_category']}")

    print("\nI've prepared a draft response for your review:")
    print("-" * 50)

    print(state["email_draft"])

    print("=" * 50 + "\n")

    return {}
```

This step demonstrates the final review stage. A real application could display the draft in a web interface and allow the user to edit or approve it.

## 3.10 Define the routing logic

After classification, the workflow needs to choose between spam handling and response drafting.

```python
def route_email(state: EmailState) -> str:
    """Choose the next node based on classification."""

    if state["is_spam"]:
        return "spam"

    return "legitimate"
```

The routing function returns one of two labels:

- `"spam"` routes to the spam handler.
- `"legitimate"` routes to the response-drafting node.

## 3.11 Build the email graph

Now we connect all the nodes.

```python
email_graph = StateGraph(EmailState)

email_graph.add_node("read_email", read_email)
email_graph.add_node("classify_email", classify_email)
email_graph.add_node("handle_spam", handle_spam)
email_graph.add_node("draft_response", draft_response)
email_graph.add_node("notify_mr_hugg", notify_mr_hugg)

email_graph.add_edge(START, "read_email")

email_graph.add_edge(
    "read_email",
    "classify_email"
)

email_graph.add_conditional_edges(
    "classify_email",
    route_email,
    {
        "spam": "handle_spam",
        "legitimate": "draft_response"
    }
)

email_graph.add_edge("handle_spam", END)

email_graph.add_edge(
    "draft_response",
    "notify_mr_hugg"
)

email_graph.add_edge("notify_mr_hugg", END)

compiled_graph = email_graph.compile()
```

The graph now represents the entire workflow.

- Both types of email begin with reading and classification.
- Spam goes directly to the spam-handling node.
- Legitimate email goes through drafting and notification.
- Both branches terminate at `END`.

## 3.12 Run the application

Let's create two sample emails.

### Example A: Legitimate email

```python
legitimate_email = {
    "sender": "john.smith@example.com",
    "subject": "Question about your services",
    "body": (
        "Dear Mr. Hugg, I was referred to you by a "
        "colleague and I'm interested in learning more "
        "about your consulting services. Could we schedule "
        "a call next week? Best regards, John Smith"
    )
}
```

Initialize the graph's state and invoke it:

```python
legitimate_result = compiled_graph.invoke({
    "email": legitimate_email,
    "is_spam": None,
    "spam_reason": None,
    "email_category": None,
    "email_draft": None,
    "messages": []
})
```

The expected workflow is:

1. Alfred reads the email.
2. The LLM classifies it as legitimate.
3. The graph routes to `draft_response`.
4. The LLM generates a preliminary reply.
5. Alfred displays the reply for review.
6. The workflow reaches `END`.

The actual category and draft depend on the model's response.

### Example B: Spam email

```python
spam_email = {
    "sender": "winner@lottery-intl.com",
    "subject": "YOU HAVE WON $5,000,000!!!",
    "body": (
        "CONGRATULATIONS! You have been selected as the "
        "winner of our international lottery! To claim "
        "your prize, please send us your bank details "
        "and a processing fee of $100."
    )
}
```

Invoke the graph:

```python
spam_result = compiled_graph.invoke({
    "email": spam_email,
    "is_spam": None,
    "spam_reason": None,
    "email_category": None,
    "email_draft": None,
    "messages": []
})
```

The expected workflow is:

1. Alfred reads the email.
2. The LLM classifies it as spam.
3. The graph routes to `handle_spam`.
4. Alfred prints the classification and reason.
5. The workflow reaches `END`.

These are illustrative outcomes, not guaranteed model outputs. In a production application, the classification should be validated before automatically taking consequential actions.

## 3.13 Visualize the email workflow

You can inspect the compiled graph:

```python
from IPython.display import Image, display

display(
    Image(compiled_graph.get_graph().draw_mermaid_png())
)
```

This visualization makes the workflow's branches easier to understand and debug.

# 4. Observability and Tracing with Langfuse

As an AI application becomes more complex, debugging individual model calls is no longer enough.

You may need to understand:

- Which node executed.
- Which path the workflow followed.
- What inputs and outputs were produced.
- Where a failure occurred.
- How much time an execution took.
- Why the final result was unexpected.

**Observability** helps developers inspect and monitor application behavior.

Langfuse is an observability tool that can trace LLM and LangChain-based application executions.

## 4.1 Install the dependencies

```python
%pip install -q langfuse langchain
```

## 4.2 Configure credentials

Create a project in your tracing platform and configure its credentials as environment variables.

```python
import os

os.environ["LANGFUSE_PUBLIC_KEY"] = "YOUR_PUBLIC_KEY"
os.environ["LANGFUSE_SECRET_KEY"] = "YOUR_SECRET_KEY"
os.environ["LANGFUSE_HOST"] = "YOUR_LANGFUSE_HOST"
```

Replace these placeholders with your actual project values. Do not publish real credentials in your source code or portfolio repository.

For a real application, store credentials in environment configuration or a secrets manager.

## 4.3 Add the tracing callback

```python
from langfuse.langchain import CallbackHandler

langfuse_handler = CallbackHandler()
```

Pass the callback to the graph invocation:

```python
legitimate_result = compiled_graph.invoke(
    {
        "email": legitimate_email,
        "is_spam": None,
        "spam_reason": None,
        "email_category": None,
        "email_draft": None,
        "messages": []
    },
    config={
        "callbacks": [langfuse_handler]
    }
)
```

The callback allows supported LangChain and LangGraph execution details to be recorded by the configured tracing system.

Exact setup requirements may differ between installed versions, so follow the API expected by your environment.

### Why tracing matters

Tracing can help identify whether a problem occurred during:

- Email classification.
- Tool execution.
- Response generation.
- Conditional routing.
- A downstream integration.

For production applications, tracing should be combined with structured logs, error handling, monitoring, and careful management of sensitive data. Emails may contain personal information, so avoid recording full message bodies unless the data-handling policy permits it.

# 5. Building a Document Analysis Agent

Now let's move from a structured LLM workflow to an agent that can use tools.

Imagine Alfred is responsible for organizing Mr. Wayne's documents, training schedules, and meal plans.

Mr. Wayne leaves a handwritten or printed note containing a training schedule and meal information. Alfred must read the note, understand the request, perform calculations when necessary, and provide a useful answer.

Our document analysis system should be able to:

1. Accept an image document.
2. Extract text using a vision-language model.
3. Perform calculations using Python tools.
4. Analyze the extracted information.
5. Answer specific questions about the document.
6. Call tools when necessary and use their results.

This example introduces tool calling and the ReAct pattern.

## 5.1 Understand the workflow

The general flow is:

1. The user submits a question and provides an optional document.
2. The assistant node sends the conversation to the LLM.
3. The LLM either answers directly or requests a tool.
4. If a tool is requested, the tools node executes it.
5. The tool result is added to the conversation.
6. The assistant runs again with the updated messages.
7. The cycle continues until the model provides a response without requesting another tool.

This is different from the email workflow. The email workflow follows a fixed sequence of nodes with one conditional branch. The document analysis agent can repeatedly select tools based on the conversation.

## 5.2 Install the dependencies

```python
%pip install langgraph langchain_openai langchain_core
```

The examples use a vision-capable chat model. You will need the appropriate API credentials, and the selected model must support the required image and tool-calling features.

## 5.3 Import the required modules

```python
import base64
from typing import TypedDict, Annotated, Optional

from langchain_openai import ChatOpenAI
from langchain_core.messages import (
    AnyMessage,
    SystemMessage,
    HumanMessage
)

from langgraph.graph.message import add_messages
from langgraph.graph import StateGraph, START, END
from langgraph.prebuilt import ToolNode, tools_condition

from IPython.display import Image, display
```

Each import has a specific role:

- `base64` converts image bytes into an encoded string.
- `TypedDict` defines the structure of the graph's state.
- `Annotated` attaches additional behavior to a state field.
- `AnyMessage` represents different message types.
- `SystemMessage` defines the assistant's instructions.
- `HumanMessage` represents the user's input.
- `add_messages` defines how message updates are merged.
- `ToolNode` executes requested tools.
- `tools_condition` routes execution depending on whether the latest assistant message contains a tool call.

## 5.4 Define the agent's state

The document analysis agent needs to track the input file and the conversation.

```python
class AgentState(TypedDict):
    input_file: Optional[str]

    messages: Annotated[
        list[AnyMessage],
        add_messages
    ]
```

The `input_file` field contains an optional local path to a document, such as an image.

The `messages` field stores the conversation history.

### What does `add_messages` do?

In LangGraph, state fields can use reducers to determine how updates are combined.

Here, `add_messages` allows new messages to be added to the existing message history rather than simply replacing the entire list with the latest message.

This is important for an agent because the assistant needs access to earlier messages and tool results.

For example, the conversation may contain:

1. The user's original question.
2. The assistant's tool call.
3. The tool's result.
4. The assistant's final answer.

The graph needs to maintain this sequence for the tool-calling cycle to work.

## 5.5 Prepare the document extraction tool

Our first tool extracts text from an image using a vision-language model.

A vision-language model can process visual information and produce a text response. In this example, the model is instructed to extract the text visible in the image.

```python
vision_llm = ChatOpenAI(model="gpt-4o")


def extract_text(img_path: str) -> str:
    """Extract text from an image using a vision model."""

    try:
        with open(img_path, "rb") as image_file:
            image_bytes = image_file.read()

        image_base64 = base64.b64encode(
            image_bytes
        ).decode("utf-8")

        message = [
            HumanMessage(
                content=[
                    {
                        "type": "text",
                        "text": (
                            "Extract all the text from this image. "
                            "Return only the extracted text, "
                            "without explanations."
                        )
                    },
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": (
                                "data:image/png;base64,"
                                f"{image_base64}"
                            )
                        }
                    }
                ]
            )
        ]

        response = vision_llm.invoke(message)

        return str(response.content).strip()

    except Exception as e:
        print(f"Error extracting text: {e}")
        return ""
```

### How does the extraction tool work?

**Step 1: Read the image.**

```python
with open(img_path, "rb") as image_file:
    image_bytes = image_file.read()
```

The image is read in binary mode.

**Step 2: Encode the image.**

```python
image_base64 = base64.b64encode(
    image_bytes
).decode("utf-8")
```

Base64 converts the image bytes into a text representation that can be included in a data URL.

**Step 3: Create a multimodal message.**

The message contains both an instruction and the image.

**Step 4: Call the vision model.**

```python
response = vision_llm.invoke(message)
```

The model attempts to extract the visible text.

**Step 5: Return the extracted content.**

The function returns the model's text response. If an error occurs, the example prints the error and returns an empty string.

Returning an empty string makes the tool's result ambiguous: the document might contain no text, or extraction might have failed. A production version should return a structured result containing success status, extracted text, and any error information.

**Note:** This example uses a PNG data URL. If you want to support JPEGs or other image formats, make sure the MIME type matches the actual file format. PDF files also need appropriate page rendering or document-processing logic before they can be handled as images.

## 5.6 Create the calculation tool

The agent also needs to perform ordinary calculations.

```python
def divide(a: float, b: float) -> float:
    """Divide a by b."""

    if b == 0:
        raise ValueError("Cannot divide by zero.")

    return a / b
```

This is a normal Python function. It does not need an LLM to calculate the answer.

For example:

```python
print(divide(6790, 5))
```

Output:

```text
1358.0
```

The advantage of giving this function to the agent is that the model can request a precise calculation instead of trying to calculate it only through generated text.

## 5.7 Bind the tools to the model

Create a list of tools and bind them to the chat model.

```python
tools = [
    divide,
    extract_text
]

llm = ChatOpenAI(model="gpt-4o")

llm_with_tools = llm.bind_tools(
    tools,
    parallel_tool_calls=False
)
```

`bind_tools()` makes the tool definitions available to the model.

The model can decide to request a tool call. It does not directly execute the Python function itself. LangGraph's tool-execution node handles the requested call.

Setting `parallel_tool_calls=False` asks the model to avoid requesting multiple tool calls in parallel in this example.

## 5.8 Create the assistant node

The assistant node sends the system instructions and existing conversation to the model.

```python
def assistant(state: AgentState):
    textual_description_of_tools = """
    extract_text(img_path: str) -> str:
        Extract text from a local image file.

    divide(a: float, b: float) -> float:
        Divide a by b.
    """

    image = state["input_file"]

    sys_msg = SystemMessage(
        content=(
            "You are Alfred, a helpful butler who serves "
            "Mr. Wayne and Batman. You analyze documents "
            "and perform calculations using available tools. "
            f"Available tools: {textual_description_of_tools} "
            f"The optional input image path is: {image}. "
            "If the user asks about the document, use the "
            "text extraction tool when appropriate. "
            "Use the calculation tool for arithmetic. "
            "If you cannot access or extract the document, "
            "explain the limitation instead of inventing "
            "its contents."
        )
    )

    response = llm_with_tools.invoke(
        [sys_msg] + state["messages"]
    )

    return {
        "messages": [response]
    }
```

The assistant node does not execute the requested tools directly.

Instead, it asks the LLM to decide whether a tool is needed and returns the model's message. That message may contain a tool call or a normal response.

Because the state uses `add_messages`, the new assistant message is merged into the existing conversation.

## 5.9 Build the ReAct graph

We now have two main nodes:

- `assistant`: calls the LLM and receives its decision.
- `tools`: executes any requested tool calls.

Let's connect them.

```python
builder = StateGraph(AgentState)

builder.add_node("assistant", assistant)
builder.add_node("tools", ToolNode(tools))

builder.add_edge(START, "assistant")

builder.add_conditional_edges(
    "assistant",
    tools_condition
)

builder.add_edge("tools", "assistant")

react_graph = builder.compile()
```

The graph follows this logic:

- Execution starts at the assistant.
- If the assistant requests a tool, `tools_condition` routes execution to `tools`.
- The tools node executes the requested function and returns the tool result.
- The graph loops back to the assistant.
- If the assistant does not request a tool, `tools_condition` routes execution to `END`.

### Visualize the graph

```python
display(
    Image(
        react_graph.get_graph(
            xray=True
        ).draw_mermaid_png()
    )
)
```

The visualization helps you see the assistant-tools loop.

## 5.10 Understand the ReAct pattern

The document analysis agent follows the **ReAct pattern**, short for Reasoning and Acting.

The general cycle is:

1. **Reason:** The model determines what action is needed.
2. **Act:** The model requests an appropriate tool.
3. **Observe:** The tool returns its result.
4. **Repeat:** The model uses the result to decide what to do next or provide the final answer.

For example, suppose the user asks:

"According to the note, what should I buy for dinner?"

The agent may request the text extraction tool. After receiving the extracted text, the model can identify the relevant ingredients and prepare a concise shopping list.

The graph manages the execution loop, while the LLM selects the appropriate action.

The application should not rely on the model's internal reasoning as a verified explanation. For debugging, inspect tool calls, tool results, state changes, and final outputs.

# 6. The Document Analysis Agent in Action

Let's run the agent with two different requests.

## 6.1 Example A: Simple calculation

The user asks Alfred to divide 6790 by 5.

```python
messages = [
    HumanMessage(
        content="Divide 6790 by 5."
    )
]

result = react_graph.invoke({
    "messages": messages,
    "input_file": None
})

for message in result["messages"]:
    message.pretty_print()
```

The interaction may look like this:

```text
Human: Divide 6790 by 5.

AI Tool Call:
divide(a=6790, b=5)

Tool Response:
1358.0

Alfred:
The result of dividing 6790 by 5 is 1358.0.
```

The exact wording of the final response depends on the model.

The important sequence is:

1. The assistant recognizes that a calculation is needed.
2. The model requests the `divide` tool.
3. LangGraph executes the Python function.
4. The result is added to the conversation.
5. The assistant uses the result to answer the user.

## 6.2 Example B: Analyze a document image

Suppose a local image named `Batman_training_and_meals.png` contains Mr. Wayne's training and meal notes.

The user asks:

"According to the note provided in the image, what items should I buy for the dinner menu?"

```python
messages = [
    HumanMessage(
        content=(
            "According to the note provided in the image, "
            "what items should I buy for the dinner menu?"
        )
    )
]

result = react_graph.invoke({
    "messages": messages,
    "input_file": "Batman_training_and_meals.png"
})

for message in result["messages"]:
    message.pretty_print()
```

The example assumes that the image file exists at the specified path and that the model can access it through the extraction tool.

The intended interaction is:

```text
Human:
According to the note, what items should I buy for dinner?

AI Tool Call:
extract_text(img_path="Batman_training_and_meals.png")

Tool Response:
Extracted text from the training and meal notes.

Alfred:
A shopping list based on the extracted dinner menu.
```

The actual shopping list must be derived from the real document's extracted text. The image contents are not included in this code example, so a specific list cannot be verified here.

### Important implementation detail

The agent receives the image path in its state and system instructions. The extraction tool then opens that local file.

For this to work:

- The path must be valid in the Python environment.
- The process must have permission to read the file.
- The tool must receive the correct path.
- The image must be in a supported format.
- The model must support the multimodal request.

The example's error handler returns an empty string if extraction fails. A more robust implementation should distinguish extraction failures from successfully extracted empty content and ask the user to provide another file when necessary.

# 7. Important Lessons from the Document Analysis Agent

The example demonstrates several important principles for building tool-using AI applications.

## 7.1 Define clear tools

Each tool should have a specific responsibility.

Examples:

- `extract_text()` extracts text from an image.
- `divide()` performs a mathematical operation.

Small, clearly defined tools are easier to test, debug, and maintain.

## 7.2 Maintain state

The agent needs to retain the conversation between tool calls.

Without the message history, the model may not have the original question or the results returned by earlier tools.

The `add_messages` reducer allows new messages to be added to the existing conversation.

## 7.3 Handle errors explicitly

A tool can fail because of a missing file, an API error, invalid input, or an unsupported format.

Production workflows should represent errors explicitly and decide whether to retry, request clarification, or terminate with an error message.

Do not silently treat every failure as a valid empty result.

## 7.4 Keep tool execution under control

Tool calling gives the model a way to request actions, but the application is still responsible for executing them.

Before exposing tools in a production application, consider:

- Input validation.
- File-access restrictions.
- Permissions.
- Timeouts.
- Tool execution limits.
- Logging and monitoring.
- Human approval for sensitive actions.

A model's request to call a tool should not automatically authorize every possible operation.

# 8. Comparing the Two LangGraph Applications

We have now built two different applications using the same graph framework.

| Feature | Email sorting workflow | Document analysis agent |
|---|---|---|
| Main purpose | Classify emails and draft replies | Answer questions about documents |
| Uses an LLM | Yes | Yes |
| Uses tools | No tool-calling loop in this example | Yes |
| Routing | Spam vs. legitimate | Tool call vs. direct response |
| State | Email, category, spam result, draft, messages | Input file and conversation messages |
| Execution pattern | Mostly fixed workflow | Repeated assistant-tools loop |
| Typical use | Email triage and drafting | Document understanding and calculations |

The email workflow demonstrates explicit orchestration of a predefined process.

The document analysis agent demonstrates how a model can select tools within a controlled graph.

Both approaches are useful. The right design depends on whether your application needs a predictable fixed process, model-directed tool selection, or a combination of the two.


# 9. Key Takeaways

LangGraph provides a structured way to build applications that use LLMs.

The most important concepts are:

- **State:** Holds the information shared between workflow steps.
- **Nodes:** Execute Python functions, model calls, tool calls, and other operations.
- **Edges:** Define transitions between nodes.
- **Conditional routing:** Chooses the next step based on a decision or the current state.
- **StateGraph:** Defines and organizes the complete workflow.
- **ToolNode:** Executes tool calls requested by the model.
- **`add_messages`:** Helps maintain conversation history across steps.
- **ReAct:** Uses a repeated reasoning, action, and observation pattern.
- **Tracing:** Helps developers inspect and debug executions.

The email sorting workflow showed how to build a predictable process with conditional branches. The document analysis agent showed how an LLM can request tools and use their results through a graph-managed loop.

Together, these examples provide a foundation for more advanced applications involving document analysis, retrieval-augmented generation, multi-agent workflows, human approval, persistent state, and production monitoring.

Alfred's work at Wayne Manor demonstrates the central idea: an LLM provides language understanding and decision-making, tools provide concrete capabilities, and LangGraph organizes how the application moves from one step to the next.
