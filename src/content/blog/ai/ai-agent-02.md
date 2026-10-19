---
title: "Mastering smolagents, CodeAgents, Tools & Tool Calling"
description: "A practical deep dive into Hugging Face smolagents, CodeAgents, ToolCallingAgents, custom tools, model integration, tool sharing, and the foundations of agentic systems."
category: "AI Agents"
tags: ["Agentic AI", "smolagents", "CodeAgent", "ToolCallingAgent", "LLM Agents", "Python", "Hugging Face", "Tool Calling"]
---

# AI Agent Journey #2 — Mastering `smolagents`

## Introduction: From Agent Fundamentals to Building Real Agents

In AI Agent Journey #1, I learned how an AI agent uses an LLM to interpret a request, choose an action, execute a tool, observe the result, and continue until it can answer the user.

Now, the next question is:

**How can I implement these capabilities without writing the entire agent orchestration system from scratch?**

This is where `smolagents`, an open-source agent framework from Hugging Face, becomes useful.

The framework provides abstractions for working with language models, tools, multi-step execution, and different ways of representing actions. It is designed to keep the basic agent implementation relatively lightweight.

This journey uses Alfred, the butler of Wayne Manor, as a recurring example. Alfred needs to organize a party while the Wayne family is away. He must search for information, prepare a menu, calculate preparation times, find suitable services, and eventually combine multiple capabilities into a complete plan.

Although the story is fictional, the underlying techniques apply to practical systems such as research agents, coding assistants, retrieval systems, data-analysis agents, and browser automation.

## 1. What Is `smolagents`?

`smolagents` is a Python framework for building AI agents that can interact with their environment through tools and executable actions.

A conventional LLM application might work like this:

```text
User
  ↓
Prompt
  ↓
LLM
  ↓
Text response
```

A tool-using agent introduces an execution loop:

```text
User request
     ↓
Agent + LLM
     ↓
Select an action
     ↓
Execute code or call a tool
     ↓
Observe the result
     ↓
Continue if necessary
     ↓
Final answer
```

For example, if Alfred asks an agent to find suitable catering services, the agent can search the web, inspect results, compare options, and return a recommendation.

The framework handles much of the coordination between model outputs, tools, execution steps, and observations.

### What does the framework provide?

The central building blocks include:

- **Models:** Interfaces for connecting the agent to an LLM.
- **Agents:** Components that manage the multi-step task-solving process.
- **Tools:** Functions or objects that perform particular operations.
- **Memory and execution logs:** Records of the task and previous steps.
- **Model prompts:** Instructions that guide the agent's behavior.
- **Execution controls:** Settings that constrain how many steps an agent can take and what code it can execute.
- **Integrations:** Connections to external tools, model providers, Hugging Face Hub resources, and other ecosystems.

The exact APIs available depend on the installed version.

### Why is it called a lightweight framework?

The project emphasizes relatively few abstractions. This makes the basic mechanism easier to inspect and understand than a system that hides most of its orchestration behind many layers.

However, lightweight does not mean that every production concern is automatically solved. Authentication, authorization, execution isolation, evaluation, monitoring, and failure handling still require careful engineering.

## 2. Why Use `smolagents`?

Many open-source frameworks support agentic applications. The right choice depends on the task, the architecture, the tools, and the operational requirements.

### 2.1 Simplicity

`smolagents` offers a relatively straightforward interface for creating an agent.

A basic CodeAgent can be created with just a model and a tool:

```python
from smolagents import (
    CodeAgent,
    DuckDuckGoSearchTool,
    InferenceClientModel,
)

agent = CodeAgent(
    tools=[DuckDuckGoSearchTool()],
    model=InferenceClientModel(),
)

result = agent.run(
    "Find music recommendations for a superhero-themed party."
)

print(result)
```

The framework coordinates the model and search tool instead of requiring the developer to manually implement every interaction step.

The code is short, but the underlying operation can involve several model calls, tool executions, and observations.

### 2.2 Flexible model integration

An agent framework should not force every application to use a single model provider.

`smolagents` supports several model integration approaches, including the following.

| Model interface | General purpose |
|---|---|
| `InferenceClientModel` | Access supported hosted inference models through Hugging Face's inference ecosystem |
| `TransformersModel` | Use compatible models through the local Transformers ecosystem |
| `LiteLLMModel` | Connect through LiteLLM-supported model providers |
| `OpenAIServerModel` | Connect to a service exposing a compatible OpenAI-style API |
| `AzureOpenAIServerModel` | Connect to a compatible Azure OpenAI deployment |

These interfaces have different requirements. Some need credentials or network access; local model execution may require appropriate hardware, dependencies, and memory.


### 2.3 A code-first approach

The defining feature of `CodeAgent` is that it represents actions as Python code rather than requiring every action to be a JSON object.

For example, an agent may generate:

```python
results = web_search(
    query="Best catering services in Gotham City"
)

print(results)
```

The execution environment runs the generated code, and the agent can use the resulting output in subsequent steps.

This makes it possible to express loops, conditionals, intermediate variables, and combinations of tool calls.

### 2.4 Integration with the Hugging Face ecosystem

The framework can work with tools and resources from the Hugging Face Hub and can integrate with external tool ecosystems.

Depending on the tool and installed version, integrations can include:

- Community-created tools.
- Hugging Face Spaces.
- LangChain tools.
- MCP server tools.
- Hosted and local language models.

This reduces the amount of infrastructure that needs to be implemented from scratch.

### 2.5 When should I use it?

`smolagents` is worth considering when:

- I want to understand how an agent executes a task.
- I need a lightweight Python framework.
- My task requires several steps or tool calls.
- I want an agent that can compose Python operations.
- I need to experiment with different models.
- I want to integrate existing tools rather than build everything manually.

It may not be the best choice for every application. A fixed workflow may be more reliable when the steps are already known, while a different framework may better fit an application that requires specialized orchestration, persistence, or enterprise integrations.

## 3. Understanding the Two Main Agent Types

`smolagents` provides two principal agent types:

1. `CodeAgent`
2. `ToolCallingAgent`

Both are multi-step agents, but they represent actions differently.

The shared parent abstraction is `MultiStepAgent`. Conceptually, it manages repeated steps in which the model produces an action, the system executes it, and the outcome is added to the execution history.

### 3.1 What is a multi-step agent?

A multi-step agent does not have to solve a request in a single model response.

It can perform a sequence of operations:

```text
Step 1: Interpret the request
        ↓
Step 2: Search for information
        ↓
Step 3: Inspect the results
        ↓
Step 4: Perform another operation
        ↓
Step 5: Produce the final answer
```

Each step can depend on what happened in the previous step.

For example, Alfred might first search for catering companies, then inspect their websites, then compare the options, and finally recommend a suitable company.

This is an implementation of the broader Thought–Action–Observation pattern introduced in the first journey. The labels used internally by a particular framework can differ, but the conceptual cycle is similar.

### 3.2 What is an execution trace?

An execution trace is a record of what happened during an agent run.

It can include:

- The original task.
- Model responses.
- Generated actions.
- Tool inputs and outputs.
- Errors.
- Intermediate steps.
- The final answer.

Traces help developers understand why an agent chose an action, identify failed tool calls, and investigate unexpected results.

They are especially valuable because an agent may make several decisions before producing its final response.

## 4. CodeAgent: Writing Actions as Python Code

`CodeAgent` is the default agent type described in the course.

Instead of producing only a structured list of tool names and arguments, it generates Python code that invokes available tools and combines their results.

### 4.1 Example: Planning the party playlist

Alfred needs music recommendations for the party.

First, install the framework:

```bash
pip install -U smolagents
```

If the selected model requires hosted inference, configure the appropriate credentials and provider access.

Now create the agent:

```python
from smolagents import (
    CodeAgent,
    DuckDuckGoSearchTool,
    InferenceClientModel,
)

search_tool = DuckDuckGoSearchTool()

agent = CodeAgent(
    tools=[search_tool],
    model=InferenceClientModel(),
)

result = agent.run(
    "Search for music recommendations for a party at Wayne Manor."
)

print(result)
```

The agent has access to a web-search tool, so it can retrieve information rather than relying only on what the model remembers from training.

The exact output depends on the model, current search results, and tool implementation.

### 4.2 What happens internally?

The high-level process is:

1. The user submits a task.
2. The agent records the task and its system instructions.
3. The framework prepares the conversation context.
4. The model generates an action, represented as Python code.
5. The execution environment processes that code.
6. Tool results or errors are recorded.
7. The model receives updated context.
8. The process repeats until the task finishes or reaches its step limit.

In the framework, execution history is represented through memory-step objects. The course discusses steps such as `SystemPromptStep`, `TaskStep`, and `ActionStep`.

These objects help preserve the relationship between the original request, each action, and its corresponding result.

### 4.3 Why is Python code useful for actions?

Python can express more than a single function call.

Consider this task:

> Search for catering services and party themes, then combine the results.

A CodeAgent might produce code similar to:

```python
queries = [
    "Best catering services in Gotham City",
    "Luxury superhero party theme ideas",
]

results = []

for query in queries:
    result = web_search(query=query)
    results.append(result)

print(results)
```

This example illustrates the type of composition that code makes possible. The actual generated code depends on the model and the tool interface.

The code can:

- Call multiple tools.
- Store intermediate results in variables.
- Iterate over lists.
- Apply conditions.
- Reuse previous results.
- Combine data before returning an answer.

### 4.4 Four advantages of code-based actions

**Composability**

Actions can be combined using functions, loops, conditionals, and reusable variables.

**Object management**

Code can work with objects and structured data rather than forcing every intermediate result into a simple string.

**Generality**

Python can express a broad range of computational operations, provided the execution environment and available capabilities permit them.

**Familiarity to language models**

Many LLMs have been trained on substantial amounts of programming code, which can make code a natural format for expressing multi-step operations.

These are advantages, not guarantees that code-based agents will outperform every JSON-based agent on every task.

### 4.5 Using a custom menu tool

Alfred also needs a menu for the party.

We can define a Python function and expose it to the agent with the `@tool` decorator.

```python
from smolagents import (
    CodeAgent,
    InferenceClientModel,
    tool,
)


@tool
def suggest_menu(occasion: str) -> str:
    """Suggest a menu based on the occasion.

    Args:
        occasion: The type of occasion.
            Allowed values are:
            - "casual": Casual party menu.
            - "formal": Formal party menu.
            - "superhero": Superhero party menu.
            - "custom": Custom menu.
    """
    if occasion == "casual":
        return "Pizza, snacks, and drinks."

    elif occasion == "formal":
        return "Three-course dinner with dessert."

    elif occasion == "superhero":
        return "Buffet with varied food and healthy options."

    elif occasion == "custom":
        return "Create a custom menu for the butler."

    else:
        return (
            "Unsupported occasion. "
            "Choose casual, formal, superhero, or custom."
        )


agent = CodeAgent(
    tools=[suggest_menu],
    model=InferenceClientModel(),
)

result = agent.run(
    "Prepare a formal menu for the party."
)

print(result)
```

The menu function uses predefined responses. It does not search for recipes, calculate quantities, or place catering orders.

The agent's job is to decide when to call the function and how to use its return value.

### 4.6 Why do the docstring and allowed values matter?

The docstring provides the model with information about the tool's purpose and arguments.

The allowed values help the model choose a valid input rather than inventing an unsupported category.

This is an important tool-design principle:

**Clear tool interfaces reduce ambiguity and make reliable tool use easier.**

However, the function must still validate inputs. A docstring does not enforce a rule by itself.

### 4.7 Using Python imports inside a CodeAgent

Some tasks require libraries such as `datetime`, `pandas`, or `numpy`.

For example, Alfred wants to calculate when the preparations will finish.

Suppose the tasks take:

- Preparing drinks: 30 minutes.
- Decorating the mansion: 60 minutes.
- Setting up the menu: 45 minutes.
- Preparing the playlist: 45 minutes.

If all tasks are performed sequentially, the total duration is:

\[
30 + 60 + 45 + 45 = 180 \text{ minutes}
\]

That equals three hours.

A CodeAgent can use Python to calculate this. If it needs a module that is not authorized by the execution environment, that import must be permitted explicitly.

```python
from smolagents import CodeAgent, InferenceClientModel

agent = CodeAgent(
    tools=[],
    model=InferenceClientModel(),
    additional_authorized_imports=["datetime"],
)

result = agent.run(
    """
    The preparation tasks take:
    drinks: 30 minutes
    decorations: 60 minutes
    menu: 45 minutes
    playlist: 45 minutes

    Assume all tasks happen sequentially and start now.
    Calculate the total duration and the expected
    finishing time using Python's datetime module.
    """
)

print(result)
```

The example illustrates the authorization mechanism. The actual result depends on the current clock time and the model's generated code.

**Important:** Allowing an import is a security decision. Only authorize modules the application genuinely needs.

## 5. ToolCallingAgent: Writing Actions as Structured Tool Calls

`ToolCallingAgent` is the second principal agent type in `smolagents`.

Instead of generating Python code for its actions, it uses the model's tool-calling interface to produce structured calls, commonly represented as JSON-like tool names and arguments.

This is a common pattern across many LLM APIs.

### 5.1 CodeAgent versus ToolCallingAgent

Suppose Alfred wants two searches:

- Find the best catering services in Gotham City.
- Find superhero party theme ideas.

A CodeAgent could express the operations like this:

```python
queries = [
    "Best catering services in Gotham City",
    "Superhero party theme ideas",
]

for query in queries:
    print(web_search(query=query))
```

A ToolCallingAgent would represent the operations as separate structured tool calls, conceptually similar to:

```json
[
  {
    "name": "web_search",
    "arguments": {
      "query": "Best catering services in Gotham City"
    }
  },
  {
    "name": "web_search",
    "arguments": {
      "query": "Superhero party theme ideas"
    }
  }
]
```

The second example illustrates the structure conceptually. Actual provider tool-call payloads can differ in their serialization and schema.

The system interprets each tool call, executes the corresponding tool, and returns the result to the model.

### 5.2 Create a ToolCallingAgent

The course example uses a web-search tool:

```python
from smolagents import (
    ToolCallingAgent,
    WebSearchTool,
    InferenceClientModel,
)

agent = ToolCallingAgent(
    tools=[WebSearchTool()],
    model=InferenceClientModel(),
)

result = agent.run(
    "Search for music recommendations for a party at Wayne Manor."
)

print(result)
```

The exact availability and configuration of `WebSearchTool` depend on the installed framework version and its supported integrations.

The conceptual difference is that the agent requests a tool call through the model's structured tool-calling mechanism rather than writing a Python program to execute.

### 5.3 Why use ToolCallingAgent?

ToolCallingAgent is useful when:

- The provider has a reliable native tool-calling interface.
- The workflow consists mainly of discrete function calls.
- The application benefits from structured argument objects.
- Arbitrary generated code is unnecessary.
- The developer wants to avoid executing model-generated Python.

Because the system executes predefined tools rather than a general code snippet, this can reduce some code-execution risks. It does not eliminate other risks such as prompt injection, unsafe tool permissions, invalid arguments, or unintended external actions.

### 5.4 What are the trade-offs?

| Dimension | CodeAgent | ToolCallingAgent |
|---|---|---|
| Action representation | Python code | Structured tool calls |
| Loops and conditionals | Can express them directly in generated code | Usually requires separate calls or orchestration logic |
| Combining intermediate results | Natural through variables and expressions | Depends on the available tools and orchestration |
| Execution requirement | Code execution environment | Tool-call parsing and execution |
| Security considerations | Generated code must be carefully restricted | Tools and their permissions still need careful controls |
| Good fit | Computation and composable multi-step operations | Well-defined function calls and structured workflows |

Neither approach is universally better.

Choose based on the task, the model's capabilities, the execution environment, and the security requirements.

## 6. How Tools Work in `smolagents`

A tool is a callable capability that an agent can use to perform an operation.

Examples include:

- Searching the web.
- Visiting a webpage.
- Calculating a result.
- Querying a database.
- Generating an image.
- Retrieving documents.
- Calling an external API.

A tool is not simply a sentence in the prompt. It needs an implementation that actually performs the operation.

### 6.1 The four components of a tool

A tool interface should describe four things.

**1. Name**

The identifier the agent uses to refer to the capability.

Example: `web_search`.

**2. Description**

An explanation of what the tool does and when it should be used.

Example: "Searches the web for information related to a query."

**3. Input schema**

The arguments the tool accepts, including their types and descriptions.

Example: `query`, a string containing the search request.

**4. Output type**

The kind of result returned by the tool.

Example: a string containing search results.

A well-designed interface helps the model understand how to use the tool. The implementation must still validate the input and return meaningful results.

## 7. Method One: Define a Tool with `@tool`

The `@tool` decorator is a convenient way to turn a suitable Python function into a tool.

Consider a fictional list of catering services.

```python
from smolagents import (
    CodeAgent,
    InferenceClientModel,
    tool,
)


@tool
def catering_service_tool(query: str) -> str:
    """Return the highest-rated catering service
    from a predefined list.

    Args:
        query: A search phrase describing the catering
            service needed.
    """
    services = {
        "Gotham Catering Co.": 4.9,
        "Wayne Manor Catering": 4.8,
        "Gotham City Events": 4.7,
    }

    best_service = max(
        services,
        key=services.get,
    )

    return best_service


agent = CodeAgent(
    tools=[catering_service_tool],
    model=InferenceClientModel(),
)

result = agent.run(
    "Find the highest-rated catering service in Gotham City."
)

print(result)
```

In this example, the function returns `Gotham Catering Co.` because its predefined rating is the highest.

This is simulated data, not a real-world review search.

### What does `@tool` do?

The decorator makes the function available through the framework's tool interface and extracts useful metadata from its signature and docstring.

The function should have:

- A descriptive name.
- Type hints for its inputs and output.
- A clear description.
- An `Args:` section that explains every argument.
- A meaningful return value.

### Why is the tool description important?

Suppose a tool accepts a parameter called `query`, but the description does not explain what kind of query is expected.

The model may supply an unrelated string or use the tool at the wrong time.

A more explicit description communicates the intended use and constraints.

However, descriptions are guidance for the model, not security enforcement. The implementation must validate inputs independently.

## 8. Method Two: Define a Tool as a Python Class

A simple function is sufficient for many tools.

A class is useful when the tool needs more structure, configuration, or persistent state.

The framework's `Tool` base class provides fields for the name, description, input schema, output type, and execution method.

The principal components are:

| Component | Purpose |
|---|---|
| `name` | Identifies the tool |
| `description` | Explains its capability |
| `inputs` | Describes expected arguments and their types |
| `output_type` | Describes the returned value |
| `forward()` | Implements the actual operation |

### 8.1 Example: Superhero party theme generator

```python
from smolagents import (
    CodeAgent,
    InferenceClientModel,
    Tool,
)


class SuperheroPartyThemeTool(Tool):
    name = "superhero_party_theme_generator"

    description = (
        "Suggests a superhero-themed party idea "
        "based on a requested category."
    )

    inputs = {
        "category": {
            "type": "string",
            "description": (
                "The party category, such as "
                "'classic heroes', 'villain masquerade', "
                "or 'futuristic Gotham'."
            ),
        }
    }

    output_type = "string"

    def forward(self, category: str) -> str:
        themes = {
            "classic heroes": (
                "Justice League Gala: Guests dress as "
                "their favorite heroes."
            ),
            "villain masquerade": (
                "Gotham Rogues' Ball: A mysterious "
                "masquerade inspired by Batman villains."
            ),
            "futuristic gotham": (
                "Neo-Gotham Night: A cyberpunk-style "
                "event with neon decorations."
            ),
        }

        normalized_category = category.strip().lower()

        return themes.get(
            normalized_category,
            (
                "Theme not found. Try classic heroes, "
                "villain masquerade, or futuristic Gotham."
            ),
        )


party_theme_tool = SuperheroPartyThemeTool()

agent = CodeAgent(
    tools=[party_theme_tool],
    model=InferenceClientModel(),
)

result = agent.run(
    "Suggest a theme for a villain masquerade party."
)

print(result)
```

### 8.2 Why use a class?

The class approach makes it easier to group related behavior and configuration in one object.

It is particularly useful when a tool needs to:

- Initialize a retriever.
- Maintain a connection to a service.
- Store configuration.
- Reuse an internal resource.
- Implement a more complex operation.

For example, a document-retrieval tool might initialize its search index once and reuse it for many queries.

### 8.3 When should I use each method?

Use `@tool` for straightforward operations that can be expressed as a well-documented function.

Use the `Tool` class when the operation needs more structure, initialization, or internal state.

The framework's exact type requirements and available fields should be checked against the installed version.

## 9. The Default Toolbox

`smolagents` provides ready-made tools for common tasks.

The course lists tools such as:

- `PythonInterpreterTool`
- `FinalAnswerTool`
- `UserInputTool`
- `DuckDuckGoSearchTool`
- `GoogleSearchTool`
- `VisitWebpageTool`


### 9.1 Search tools

Search tools retrieve information from a search service.

Examples include `DuckDuckGoSearchTool` and `GoogleSearchTool`.

These tools can help an agent discover sources, but search results should not automatically be treated as verified facts.

Search-provider access, API credentials, and rate limits may apply.

### 9.2 Webpage tools

`VisitWebpageTool` retrieves content from a specified webpage.

It is useful when the agent needs to inspect a source beyond the search-result snippet.

For more reliable research, the agent should distinguish between search-result summaries and information retrieved from the actual source page.

### 9.3 PythonInterpreterTool

This tool provides a way to execute supported Python operations through the framework's interpreter interface.

It can help with calculations and data transformations.

It is not equivalent to unrestricted access to every capability of a normal Python process. The available operations and execution restrictions depend on the interpreter configuration.

### 9.4 FinalAnswerTool

This tool gives the agent a defined way to submit its final response through the framework's supported mechanism.

The final answer should be based on the task and relevant observations rather than an unsupported claim that a tool succeeded.

### 9.5 UserInputTool

A user-input tool can request additional information from a user when the workflow supports interaction.

This is useful when the agent cannot proceed safely or correctly without clarification.

For example, Alfred may need to ask how many guests are attending before calculating the amount of food required.

### 9.6 Add only the tools you need

A common mistake is importing many tools and assuming that the agent can automatically use them.

A tool must be included in the agent's configured tool list, or otherwise made available through the supported integration mechanism.

Keep the available tool set focused. Too many overlapping tools can make selection more difficult and increase the potential execution surface.

## 10. Sharing Custom Tools on the Hugging Face Hub

One useful feature of the ecosystem is the ability to share tools with other developers.

The course demonstrates the `push_to_hub()` method.

```python
party_theme_tool.push_to_hub(
    "YOUR_USERNAME/party_theme_tool",
    token="YOUR_HUGGING_FACE_TOKEN",
)
```

Replace the repository name with one you control.

The token should be provided securely, not committed to source control.

Sharing a tool allows others to discover and reuse it, subject to the repository's access settings and the tool's dependencies.

### Why share tools?

Reusable tools can prevent duplicated effort.

For example, a developer might create a tool that:

- Searches a specialized dataset.
- Generates a particular type of report.
- Queries a domain-specific knowledge base.
- Connects to an existing service.

Other developers can then integrate the capability into their own agent workflows.

Before sharing a tool, document its inputs, outputs, dependencies, permissions, limitations, and security considerations.

## 11. Importing a Community Tool

Instead of implementing every capability, an agent can load a tool that someone else has published.

The course illustrates this with an image-generation tool:

```python
from smolagents import (
    CodeAgent,
    InferenceClientModel,
    load_tool,
)

image_generation_tool = load_tool(
    "m-ric/text-to-image",
    trust_remote_code=True,
)

agent = CodeAgent(
    tools=[image_generation_tool],
    model=InferenceClientModel(),
)

result = agent.run(
    "Generate an image of a luxurious superhero-themed "
    "party at Wayne Manor."
)

print(result)
```

This example follows the course's integration pattern. Whether the specific community tool is still available and compatible depends on its current repository and the installed framework version.

### What does `trust_remote_code=True` mean?

It is a security-sensitive setting that permits remote repository code to be used in the loading process.

It should not be enabled casually.

Before trusting a remote tool, review its source code, dependencies, permissions, and external service behavior. A tool can have effects beyond the description that the model sees.

## 12. Import a Hugging Face Space as a Tool

A Hugging Face Space can expose an application through a Gradio interface.

`smolagents` provides a `Tool.from_space()` integration for supported Spaces.

The course demonstrates the pattern:

```python
from smolagents import (
    CodeAgent,
    InferenceClientModel,
    Tool,
)

image_generation_tool = Tool.from_space(
    "black-forest-labs/FLUX.1-schnell",
    name="image_generator",
    description="Generate an image from a prompt.",
)

model = InferenceClientModel(
    "Qwen/Qwen2.5-Coder-32B-Instruct"
)

agent = CodeAgent(
    tools=[image_generation_tool],
    model=model,
)

result = agent.run(
    "Improve the prompt and generate an image.",
    additional_args={
        "user_prompt": (
            "A grand superhero-themed party at Wayne Manor, "
            "with Alfred overseeing a luxurious gala."
        )
    },
)

print(result)
```

This is the course's integration pattern, not a guarantee that the named Space or its API remains available or unchanged.

The Space must expose a compatible interface. Some Spaces may require dependencies, authentication, usage limits, or additional configuration.

The `gradio_client` integration may also need to be installed, depending on the environment.

### Why is this useful?

Instead of writing an image-generation model from scratch, the agent can call a separate application that provides the capability.

The same principle can apply to supported Spaces for data analysis, text processing, and other tasks.

## 13. Reuse LangChain Tools

`smolagents` can integrate tools from other ecosystems, including LangChain-compatible tools.

This is useful when an existing tool already solves a problem and does not need to be rewritten.

The course demonstrates using `Tool.from_langchain()`.

First, install the required integration package:

```bash
pip install -U langchain-community
```

For the course's SerpAPI example, configure the required API key securely.

```python
import os

os.environ["SERPAPI_API_KEY"] = "YOUR_SERPAPI_KEY"
```

For actual projects, prefer environment variables or a secret manager over hardcoding credentials.

The course then demonstrates:

```python
from langchain.agents import load_tools
from smolagents import (
    CodeAgent,
    InferenceClientModel,
    Tool,
)

search_tool = Tool.from_langchain(
    load_tools(["serpapi"])[0]
)

model = InferenceClientModel()

agent = CodeAgent(
    tools=[search_tool],
    model=model,
)

result = agent.run(
    "Find luxury entertainment ideas for a superhero-themed event."
)

print(result)
```

This example illustrates the adapter pattern. The LangChain tool provides the underlying functionality, while the adapter exposes it through the `smolagents` tool interface.

## 14. Import Tools from MCP Servers

MCP stands for **Model Context Protocol**.

It is a protocol for connecting AI applications to external tools and contextual resources through a standardized interface.

Instead of implementing a separate integration for every external service, an application can connect to a compatible MCP server and discover the tools that it exposes.

The `smolagents` ecosystem supports importing tool collections from MCP servers.

Install the MCP integration:

```bash
pip install "smolagents[mcp]"
```

The course demonstrates a server launched using `uvx`:

```python
import os

from mcp import StdioServerParameters
from smolagents import (
    CodeAgent,
    InferenceClientModel,
    ToolCollection,
)

model = InferenceClientModel(
    "Qwen/Qwen2.5-Coder-32B-Instruct"
)

server_parameters = StdioServerParameters(
    command="uvx",
    args=["--quiet", "pubmedmcp@0.1.3"],
    env={
        "UV_PYTHON": "3.12",
        **os.environ,
    },
)

with ToolCollection.from_mcp(
    server_parameters,
    trust_remote_code=True,
) as tool_collection:
    agent = CodeAgent(
        tools=[*tool_collection.tools],
        model=model,
        add_base_tools=True,
    )

    result = agent.run(
        "Find and summarize relevant information "
        "from the available PubMed search tools."
    )

    print(result)
```

This is an illustrative adaptation of the course example. The particular MCP server must be available, its command and arguments must be valid, and its exposed tools must match the task. Some integrations may have additional requirements.

The course's original example asks a medical-information question. When working with medical sources, retrieval should not be confused with clinical validation, and the agent's output should not replace professional medical advice.

### What does `ToolCollection` do?

`ToolCollection` provides a way to load a group of tools exposed by an MCP server.

The expression:

```python
tools=[*tool_collection.tools]
```

expands the collection into the list of tools passed to the agent.

The `with` block manages the collection's lifecycle so that its resources can be cleaned up when execution leaves the block.

### Why use MCP?

MCP is useful when multiple applications need a standardized way to expose and consume tools.

For example, an application could connect to compatible servers for search, files, databases, or other external capabilities.

The exact available operations depend on the server.

### Security considerations

MCP is an integration protocol, not a guarantee that every connected tool is safe.

Before connecting to a server, consider:

- Who operates it.
- What tools it exposes.
- What data those tools can access.
- Whether the server requires credentials.
- Whether the tools can modify external state.
- Whether the server's code and dependencies are trustworthy.

Grant only the permissions required for the task.


## 15. Important Limitations to Remember

### A tool description is not a guarantee

A well-written docstring helps the model understand a tool. It does not prevent every invalid input or misuse.

### Search results are not automatically verified

Search tools retrieve information, but the agent may still need to inspect the original source and check whether the information is current and relevant.

### Tool imports do not automatically grant capabilities

The agent must be configured to use the tool through the framework's supported interface.

### Models can make mistakes

A model can choose the wrong tool, generate invalid code, misunderstand a tool result, or produce an incorrect conclusion.

### Execution security is essential

A CodeAgent can generate code, so the execution environment must be configured with suitable restrictions. Do not treat the framework as a guarantee of complete isolation.



## 16. Agentic RAG: Making Retrieval Intelligent

Before building more advanced agents, we need to understand **Agentic RAG**.

### 16.1 What is RAG?

RAG stands for **Retrieval-Augmented Generation**. It combines information retrieval with an LLM so that the model can answer questions using relevant external information instead of relying entirely on its pretrained knowledge.

A traditional RAG pipeline looks like this:

1. The user asks a question.
2. The application searches a knowledge base.
3. The retriever returns relevant documents.
4. The documents are added to the LLM's context.
5. The LLM generates an answer using the retrieved information.

For example, imagine Alfred is organizing a superhero party and has a collection of party-planning documents.

The user asks:

> What entertainment activities are suitable for a luxury superhero party?

A traditional RAG system searches the documents for relevant passages and gives those passages to an LLM. The LLM then generates recommendations.

The main limitation is that a simple pipeline may perform retrieval only once. If the first search produces incomplete results, the system may still generate an answer from insufficient information.

### 16.2 What is Agentic RAG?

**Agentic RAG uses an agent to control and improve the retrieval process.**

Instead of following one fixed retrieval path, the agent can decide what to search for, whether the retrieved information is sufficient, and what to do next.

For example:

1. The agent receives the question.
2. It identifies the important information required.
3. It searches the knowledge base.
4. It evaluates the retrieved passages.
5. If information is missing, it reformulates the query or searches another source.
6. It combines the useful evidence.
7. It produces a final answer.

The agent can potentially use different retrieval tools depending on the task.

For example, it might use a local knowledge base for internal documents and a web search tool for recent public information.

### 16.3 Traditional RAG vs Agentic RAG

| Feature | Traditional RAG | Agentic RAG |
|---|---|---|
| Retrieval | Usually follows a predefined pipeline | Can perform multiple retrieval steps |
| Query handling | Often uses the original query | Can reformulate or decompose queries |
| Tool selection | Usually predefined | Agent may select among available tools |
| Result evaluation | Often uses fixed logic | Agent can critique results and decide whether to search again |
| Complexity | Generally simpler | Generally more complex |
| Cost and latency | Often lower | Can be higher because of additional reasoning and retrieval |
| Best use | Predictable document question-answering | Complex questions requiring iterative research |

Agentic RAG does not automatically guarantee better answers. It adds flexibility, but it also introduces more opportunities for incorrect tool selection, unnecessary searches, and reasoning errors.

### 16.4 Building a basic search agent with smolagents

We can begin with a web-search agent.

Install the framework:

```bash
pip install -U smolagents
```

Then create the agent:

```python
from smolagents import (
    CodeAgent,
    DuckDuckGoSearchTool,
    InferenceClientModel,
)

search_tool = DuckDuckGoSearchTool()
model = InferenceClientModel()

agent = CodeAgent(
    model=model,
    tools=[search_tool],
    max_steps=8,
)

response = agent.run(
    "Search for luxury superhero-themed party ideas, "
    "including decorations, entertainment, and catering."
)

print(response)
```

**How does this work?**

- `DuckDuckGoSearchTool()` provides a search capability.
- `InferenceClientModel()` supplies the language model.
- `CodeAgent()` creates an agent that can write Python code to use tools.
- `max_steps=8` limits the number of agent execution steps.
- `agent.run()` starts the task and returns the final result.

The agent may perform several searches and combine the results rather than simply executing one search.

The exact behavior depends on the model, available tools, prompts, and framework version.

> Note: The example uses a model configured through Hugging Face's inference infrastructure. You may need to authenticate with Hugging Face and select a model/provider that is available to your account.

---

## 17. Building a Custom Knowledge Base for Agentic RAG

Web search is useful, but many applications need to retrieve information from their own documents.

Examples include:

- Company policies
- Product documentation
- Meeting transcripts
- Research papers
- Internal FAQs
- Technical learning notes

For V-Meet, for example, a retrieval agent could search saved meeting transcripts and notes to answer questions about previous meetings.

Let's understand how to create a small local retrieval tool.

### 17.1 What is a retriever?

A retriever accepts a query and returns documents or passages that are relevant to that query.

A basic retriever does not necessarily generate an answer. Its responsibility is to find useful information.

For example:

```text
Query:
"What entertainment is suitable for a superhero party?"

Retrieved passage:
"Luxury superhero parties can include a professional DJ,
themed decorations, and interactive virtual-reality experiences."
```

The agent can then use the retrieved passage to construct an answer.

### 17.2 What is BM25?

BM25 is a ranking algorithm commonly used for keyword-based information retrieval.

It scores documents according to how relevant their terms are to a search query. It considers factors such as term frequency, how rare a term is across the document collection, and document length.

BM25 is useful when important words in the query also appear in the documents.

For example, a query containing `"virtual reality entertainment"` may retrieve a document containing those terms.

However, BM25 does not inherently understand that two differently worded expressions can have the same meaning. A semantic retrieval system using embeddings can help with that limitation.

### 17.3 Install the retrieval dependencies

The following example uses LangChain's document utilities and BM25 retriever.

```bash
pip install -U smolagents langchain-core langchain-community rank-bm25
```

### 17.4 Create sample documents

First, prepare a few documents.

```python
from langchain_core.documents import Document

documents = [
    Document(
        page_content=(
            "A luxury superhero masquerade can include elegant "
            "decorations, themed lighting, and superhero-inspired "
            "visual projections."
        ),
        metadata={"category": "decorations"},
    ),
    Document(
        page_content=(
            "A professional DJ can provide music for a superhero "
            "party. The playlist should match the event's atmosphere."
        ),
        metadata={"category": "entertainment"},
    ),
    Document(
        page_content=(
            "Themed catering can include custom desserts, creative "
            "food presentations, and superhero-inspired menu names."
        ),
        metadata={"category": "catering"},
    ),
    Document(
        page_content=(
            "Interactive virtual-reality experiences can provide "
            "guests with immersive activities during the event."
        ),
        metadata={"category": "activities"},
    ),
]
```

Each `Document` contains:

- `page_content`: the text to search.
- `metadata`: optional structured information about that text.

Metadata becomes useful when you need filtering, source tracking, or citations.

### 17.5 Split large documents into chunks

Real documents can contain thousands of words. Sending an entire document to a model is often inefficient.

A common approach is to split documents into smaller chunks.

```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,
    chunk_overlap=50,
    add_start_index=True,
    strip_whitespace=True,
    separators=["\n\n", "\n", ".", " ", ""],
)

chunks = splitter.split_documents(documents)
```

Install the text-splitting package if needed:

```bash
pip install -U langchain-text-splitters
```

Understand the parameters:

- `chunk_size=500`: aims to keep each chunk near the specified character length.
- `chunk_overlap=50`: preserves some text between neighboring chunks.
- `add_start_index=True`: adds the starting character position to chunk metadata.
- `strip_whitespace=True`: removes unnecessary whitespace around chunks.
- `separators`: tells the splitter which boundaries to try first.

**Why use overlap?**

Imagine a sentence begins at the end of one chunk and finishes in the next. Without overlap, some context may be separated. Overlap helps preserve continuity.

The appropriate chunk size depends on the document structure, retrieval method, and intended task. A character-based chunk size is not the same as a model's token count.

### 17.6 Create a custom retrieval tool

Now let's turn the retriever into a tool that an agent can use.

```python
from smolagents import Tool
from langchain_community.retrievers import BM25Retriever


class PartyPlanningRetrieverTool(Tool):
    name = "party_planning_retriever"

    description = (
        "Search the party-planning knowledge base for information "
        "about decorations, entertainment, catering, and activities."
    )

    inputs = {
        "query": {
            "type": "string",
            "description": "The information to search for.",
        }
    }

    output_type = "string"

    def __init__(self, docs, **kwargs):
        super().__init__(**kwargs)

        self.retriever = BM25Retriever.from_documents(
            docs,
            k=5,
        )

    def forward(self, query: str) -> str:
        results = self.retriever.invoke(query)

        if not results:
            return "No relevant documents were found."

        return "\n\n".join(
            document.page_content
            for document in results
        )
```

Let's understand the implementation.

**`name`**

The tool's identifier. The agent uses it to refer to the tool.

**`description`**

Explains the tool's purpose. This description helps the model decide when to use it.

**`inputs`**

Defines the expected input argument and its type.

**`output_type`**

Declares that the tool returns a string.

**`__init__()`**

Creates a BM25 retriever from the documents. The `k=5` setting requests up to five retrieved documents.

**`forward()`**

Runs the search and combines the retrieved passages into one string.

The tool returns the document text, not a fully generated answer. The agent remains responsible for using the results to answer the user's question.

### 17.7 Give the tool to an agent

```python
from smolagents import CodeAgent, InferenceClientModel

retrieval_tool = PartyPlanningRetrieverTool(chunks)

agent = CodeAgent(
    model=InferenceClientModel(),
    tools=[retrieval_tool],
    max_steps=8,
)

answer = agent.run(
    "What entertainment activities would work for a "
    "luxury superhero party?"
)

print(answer)
```

The flow is now:

```text
User question
     |
     v
CodeAgent
     |
     v
PartyPlanningRetrieverTool
     |
     v
BM25 searches the document chunks
     |
     v
Relevant passages return to the agent
     |
     v
Agent generates a response
```

This is a basic retrieval-enabled agent. It becomes more agentic when the agent evaluates the retrieved evidence and decides whether it needs additional searches, reformulated queries, or other tools.

### 17.8 Important production improvements

The example is intentionally small. A real knowledge-base implementation should consider the following improvements.

**Preserve source information.** Return document IDs, titles, and URLs alongside text so that answers can cite their sources.

**Handle empty results.** An agent should not invent facts when the retriever returns nothing relevant.

**Add semantic retrieval when appropriate.** Embedding-based retrieval can help find passages with similar meanings even when they use different words.

**Rerank results.** A cross-encoder can evaluate query-document pairs to improve the ordering of retrieved passages.

**Use query decomposition.** A complex question can be split into smaller searches. For example, a question about catering, entertainment, and decorations can trigger three focused queries.

**Validate the answer.** Check whether important claims are supported by retrieved evidence.

**Apply access control.** In a company application, retrieval must respect document permissions. An agent should never be able to retrieve private documents merely because it can call a search tool.

**Track provenance.** Store source identifiers and relevant passages so that answers can be audited later.

### 17.9 Query reformulation, expansion, decomposition, and reranking

These are related techniques, but they solve different problems.

| Technique | Meaning | Example |
|---|---|---|
| Query reformulation | Rewrites a query to make it clearer or more searchable | `"party activities"` becomes `"interactive entertainment for a luxury superhero party"` |
| Query expansion | Adds related terms or alternative wording | `"VR"` expands to `"virtual reality, immersive games"` |
| Query decomposition | Breaks a complex question into smaller questions | Search decorations, catering, and entertainment separately |
| Reranking | Reorders retrieved passages using a more detailed relevance model | Move the passage that directly answers the question above loosely related passages |
| Multi-step retrieval | Performs another retrieval step using earlier findings | Search for a venue, then search for the venue's capacity |

A capable Agentic RAG system can combine these methods, but it should not use every method for every question. Simple questions often need only one retrieval step.

---

## 18. Multi-Agent Systems: Several Agents Working Together

A single agent can search, calculate, analyze, and generate a response. However, as a task becomes more complicated, the agent's context can fill up and its responsibilities can become difficult to manage.

A **multi-agent system** distributes work among specialized agents.

In smolagents, an agent can manage other agents, allowing a higher-level agent to delegate subtasks and combine the results.

### 18.1 What is an orchestrator?

An orchestrator is the component responsible for coordinating the overall workflow.

Consider this example:

Alfred needs to find filming locations from Batman movies, estimate travel times, identify supercar factories, and plot the results on a map.

The work can be divided into roles:

- **Manager agent:** understands the task, delegates work, and prepares the final result.
- **Web agent:** searches the internet and visits relevant pages.
- **Calculation tool:** estimates travel time from geographic coordinates.
- **Analysis and plotting logic:** organizes results and creates a visualization.
- **Evaluation step:** checks whether the final output satisfies the task.

A simplified architecture looks like this:

```text
                     User task
                         |
                         v
                   Manager Agent
                    /          \
                   v            v
             Web Agent      Calculation Tool
                |
                v
        Search and Web Pages
                    \          /
                     v        v
                  Combine Results
                         |
                         v
                  Generate Map
                         |
                         v
                  Validate Output
                         |
                         v
                    Final Answer
```

The diagram represents responsibilities, not a requirement that every action happen in exactly this order.

### 18.2 Why use multiple agents?

**Specialization**

Each agent can focus on a smaller task. A web agent does not need to manage the entire plotting workflow.

**Separate memories**

Subtasks can have separate execution histories. This can reduce the amount of irrelevant information carried into each model call.

**Modularity**

You can improve the web-search agent without rewriting the plotting workflow.

**Different models for different jobs**

A less expensive model may be sufficient for a simple subtask, while a stronger reasoning model may be useful for coordination and final analysis.

**Better organization**

The manager can receive a compact result from a specialized agent instead of repeating every search step itself.

However, multiple agents are not automatically better. They introduce coordination overhead, extra model calls, more failure points, and potentially higher cost.

---

## 19. A Practical Multi-Agent Example: Research, Calculation, and Mapping

The following example follows the course's cargo-plane calculation and map-generation workflow.

**Important distinction:** The original task description mentions boat transfers in one place, but the provided calculation code models a cargo plane. The calculation below estimates cargo-plane travel time; it is not a boat-route calculator.

### 19.1 Install the required packages

```bash
pip install "smolagents[litellm]" plotly geopandas shapely kaleido
```

The exact extras and package requirements may vary with the installed smolagents version.

### 19.2 Create a deterministic calculation tool

Agents are useful for selecting actions and coordinating work. Mathematical calculations should still be implemented using ordinary code whenever possible.

The following tool estimates cargo-plane travel time using the haversine formula.

```python
import math
from typing import Optional, Tuple

from smolagents import tool


@tool
def calculate_cargo_travel_time(
    origin_coords: Tuple[float, float],
    destination_coords: Tuple[float, float],
    cruising_speed_kmh: Optional[float] = 750.0,
) -> float:
    """
    Estimate cargo-plane travel time between two points.

    Coordinates must be (latitude, longitude).
    The estimate uses great-circle distance, a 10% distance
    adjustment, and one additional hour for takeoff and landing.
    """

    def to_radians(degrees: float) -> float:
        return degrees * math.pi / 180

    lat1, lon1 = map(to_radians, origin_coords)
    lat2, lon2 = map(to_radians, destination_coords)

    earth_radius_km = 6371.0

    dlat = lat2 - lat1
    dlon = lon2 - lon1

    a = (
        math.sin(dlat / 2) ** 2
        + math.cos(lat1)
        * math.cos(lat2)
        * math.sin(dlon / 2) ** 2
    )

    c = 2 * math.asin(math.sqrt(a))
    distance_km = earth_radius_km * c

    adjusted_distance_km = distance_km * 1.1

    if cruising_speed_kmh is None or cruising_speed_kmh <= 0:
        raise ValueError("Cruising speed must be positive.")

    travel_time_hours = (
        adjusted_distance_km / cruising_speed_kmh
    ) + 1.0

    return round(travel_time_hours, 2)
```

Let's understand the calculation.

1. Convert latitude and longitude from degrees to radians.
2. Use the haversine formula to calculate great-circle distance.
3. Multiply the distance by `1.1` to model a 10% route adjustment.
4. Divide the adjusted distance by the assumed cruising speed.
5. Add one hour for takeoff and landing.
6. Round the result to two decimal places.

The result is an estimate based on assumptions, not a real flight schedule. It does not include actual airport locations, routing restrictions, weather, cargo handling, or live transportation data.

The tool can be tested independently:

```python
travel_time = calculate_cargo_travel_time(
    (41.8781, -87.6298),
    (-33.8688, 151.2093),
)

print(travel_time)
```

### 19.3 Configure a web research agent

The course uses Google search through a provider such as Serper or SerpAPI.

For Serper, configure the required environment variable before creating the tool:

```python
import os
from smolagents import (
    CodeAgent,
    GoogleSearchTool,
    InferenceClientModel,
    VisitWebpageTool,
)

# Configure SERPER_API_KEY in your environment first.

model = InferenceClientModel(
    "Qwen/Qwen2.5-Coder-32B-Instruct",
    provider="together",
    max_tokens=8096,
)

web_agent = CodeAgent(
    model=model,
    tools=[
        GoogleSearchTool(provider="serper"),
        VisitWebpageTool(),
        calculate_cargo_travel_time,
    ],
    name="web_agent",
    description="Browses the web to find and verify information.",
    max_steps=10,
    verbosity_level=0,
)
```

You can alternatively configure a supported SerpAPI integration with its corresponding API key. Check the current tool documentation for the provider syntax available in your installed version.

Search providers may have usage limits. A provider that works for one account or course notebook may not be available under another account's credentials.

### 19.4 Why not let one agent do everything?

The course first explores a single agent that gathers filming locations and other data, calculates travel times, and prepares a report.

A detailed prompt and a planning interval can help the agent revisit its plan during a long task:

```python
agent.planning_interval = 4

report = agent.run("""
Research Batman filming locations.

For each location, visit a source page to verify the information.
Collect the relevant coordinates and organize the findings.
Calculate estimated cargo-plane travel times using the
provided calculation tool.
Return a concise table with the locations and results.
""")

print(report)
```

This example assumes that `agent` has already been created with the appropriate model, tools, and configuration.

A planning interval is a mechanism for revisiting the plan during execution. It is not a guarantee that the model will reason correctly.

The course illustrates a practical problem: after a single agent collects a large amount of research, its context can become crowded. Combining that research with another large task can increase latency and token costs.

### 19.5 Split the work between a web agent and a manager

The manager can delegate research to a specialized web agent and concentrate on analysis and plotting.

The course uses a stronger reasoning model for the manager and gives it the imports needed for data analysis and visualization.

```python
from smolagents import CodeAgent, InferenceClientModel

manager_model = InferenceClientModel(
    "deepseek-ai/DeepSeek-R1",
    provider="together",
    max_tokens=8096,
)

manager_agent = CodeAgent(
    model=manager_model,
    tools=[calculate_cargo_travel_time],
    managed_agents=[web_agent],
    additional_authorized_imports=[
        "geopandas",
        "plotly",
        "shapely",
        "json",
        "pandas",
        "numpy",
    ],
    planning_interval=5,
    verbosity_level=2,
    max_steps=15,
)
```

This is the core delegation pattern:

- The web agent gathers information.
- The manager decides how to use the web agent.
- The manager can perform calculations with the supplied tool.
- The manager can use authorized libraries to organize results and create a map.

The precise model IDs, provider settings, and agent APIs can change. Check the current smolagents documentation and your inference provider before running the complete notebook.

### 19.6 Inspect the agent architecture

The course demonstrates:

```python
manager_agent.visualize()
```

This can display the manager's configuration, tools, authorized imports, and managed-agent relationships.

Architecture inspection is useful because it makes the system easier to understand before executing a complex task.

It also helps you check whether an agent has the capabilities it actually needs.

### 19.7 Add a final output check

A system should not assume that a successful model response means the task was completed correctly.

For example, the course demonstrates a final check that examines the agent's reasoning history and a saved map using a vision-capable model.

The checker verifies that the expected file exists, sends the map and a summary of agent steps to a multimodal model, and expects a `PASS` or `FAIL` decision.

Conceptually:

```text
Agent completes task
       |
       v
Map is saved to disk
       |
       v
Validation model inspects:
- task requirements
- agent execution summary
- map image
       |
       v
PASS or FAIL
       |
       v
Accept or raise an error
```

The course example uses `saved_map.png` and checks whether the plot follows the required visualization approach.

This kind of check is an example of model-assisted evaluation. It is not a mathematical proof of correctness. A validator can overlook errors or approve an inaccurate result, so important calculations and data should also be checked with deterministic tests.

### 19.8 What did we learn from the multi-agent example?

The key lesson is not simply that multiple agents can be created.

It is that **the structure of an agent system matters**.

A single agent can be sufficient for a small, predictable task. A manager and specialized agents can be more suitable when the workflow requires independent research, calculation, and analysis.

Choose the architecture according to the task instead of adding agents without a clear reason.

---

## 20. Vision Agents: Giving Agents the Ability to Work with Images

So far, most of our examples have used text, code, and tools. However, many tasks require visual information.

Examples include:

- Describing an image
- Understanding a chart or diagram
- Examining a screenshot
- Reading information from a visual interface
- Comparing objects in images
- Navigating a website using screenshots

A **vision agent** combines agent capabilities with a model that can process visual input.

### 20.1 What is a VLM?

VLM stands for **Vision-Language Model**.

A VLM accepts visual information, often alongside text, and produces language-based outputs.

A text-only model cannot necessarily interpret an image just because an image path appears in a prompt. The selected model and API must support the relevant image-input format.

The course example uses GPT-4o through `OpenAIServerModel`. Other models can be used if they support the required multimodal inputs and are compatible with the selected integration.

### 20.2 Pass images directly to an agent

A basic vision agent can receive images as part of the task.

The following is a simplified illustration of the course pattern:

```python
from smolagents import CodeAgent, OpenAIServerModel

model = OpenAIServerModel(model_id="gpt-4o")

agent = CodeAgent(
    tools=[],
    model=model,
    max_steps=20,
    verbosity_level=2,
)

response = agent.run(
    "Describe the important visual details in the supplied images.",
    images=images,
)

print(response)
```

Here, `images` must already contain images loaded into memory. The snippet shows the agent call, not the complete image-loading pipeline.

For example, an image can be loaded with Pillow:

```python
from PIL import Image

image = Image.open("example.png")
images = [image]
```

For remote images, you can use an appropriate HTTP client and load the resulting image bytes. Only retrieve images from sources you are authorized to access.

The agent can then use the image information to help complete the task.

### 20.3 Why image input is useful

Consider a screenshot of a meeting dashboard.

A vision-capable agent could be asked to:

- Describe the visible interface.
- Identify which buttons appear to be available.
- Summarize a chart displayed on the screen.
- Compare two interface screenshots.
- Help locate a particular visual element.

For a platform like V-Meet, this could eventually support screenshot-based interface assistance or visual analysis of meeting dashboards.

However, a model's description should not be treated as guaranteed ground truth. If the task involves an important action, the application should verify relevant details before acting.

---

## 21. Dynamic Visual Retrieval: Agents That See While Browsing

Passing an image to an agent at the beginning is only one approach.

A more advanced approach allows an agent to acquire visual information during execution.

Imagine that an agent needs to inspect a website. It may need to:

1. Search for the website.
2. Open a result.
3. Navigate to another page.
4. Capture a screenshot.
5. Interpret the screenshot.
6. Decide what to do next.

This creates a feedback loop between browsing and visual understanding.

### 21.1 Understanding the agent's execution history

The course explains three important types of execution steps.

**`SystemPromptStep`**

Stores the system prompt that guides the agent.

**`TaskStep`**

Records the user task and associated inputs.

**`ActionStep`**

Captures information about an action, its observations, and the resulting execution state.

The exact fields and implementation details may vary by framework version, but the central concept is that an agent maintains a structured execution history.

This history can contain visual observations, such as screenshots.

### 21.2 Store screenshots in the execution log

The course demonstrates a callback that captures the browser screenshot after an action.

The central operation is:

```python
step_log.observations_images = [image.copy()]
```

The screenshot becomes a visual observation associated with that step.

The callback can also record the current URL, helping the agent connect a screenshot with the page it came from.

A practical implementation should ensure that:

- A browser driver exists before attempting to capture a screenshot.
- The screenshot is copied so it remains available after the browser changes.
- Old screenshots are removed when they are no longer needed.
- The current URL is recorded safely.
- Browser errors are handled explicitly.

### 21.3 Why remove older screenshots?

Images can make agent history large.

If every screenshot is retained indefinitely, the system may consume more memory and create a larger input for the model.

The course example clears older screenshot observations so that recent visual context remains available without keeping every historical image.

This is an example of **context management**: preserving the information needed for the next decision while reducing unnecessary history.

The same principle applies to text logs, tool outputs, retrieved documents, and intermediate calculations.

---

## 22. Browser Agents: Letting an Agent Navigate Websites

A browser agent combines language-model reasoning with browser automation.

Instead of simply returning search results, it can interact with pages using tools that perform actions such as:

- Searching for a phrase on the current page
- Returning to the previous page
- Dismissing some pop-ups
- Capturing screenshots
- Reading information from a page
- Navigating through a sequence of pages

The course uses Selenium and Helium for browser automation.

### 22.1 Install the browser dependencies

The course example uses:

```bash
pip install "smolagents[all]" helium selenium python-dotenv
```

Browser automation also requires a compatible browser and driver setup. Check the installed Selenium and browser versions if the driver cannot start.

### 22.2 Example: Search for text on a page

A browser tool can search for a particular phrase and scroll the page to the matching element.

The course's `search_item_ctrl_f` tool demonstrates this concept.

A simplified version is:

```python
from smolagents import tool


@tool
def search_item_ctrl_f(text: str, nth_result: int = 1) -> str:
    """
    Find text on the current browser page and focus
    the requested matching element.
    """

    elements = driver.find_elements(
        "xpath",
        f"//*[contains(text(), '{text}')]",
    )

    if nth_result < 1 or nth_result > len(elements):
        return (
            f"Requested match {nth_result} was not found. "
            f"Found {len(elements)} matching elements."
        )

    element = elements[nth_result - 1]

    driver.execute_script(
        "arguments[0].scrollIntoView(true);",
        element,
    )

    return (
        f"Found {len(elements)} matching elements. "
        f"Focused on match {nth_result}."
    )
```

This simplified example assumes that `driver` is an initialized browser driver.

It also uses a direct XPath string, which can be fragile when text contains quotes or special characters. A production implementation should handle these cases safely and avoid relying on broad XPath matching for complex pages.

### 22.3 Example: Navigate backward

```python
@tool
def go_back() -> str:
    """Navigate to the previous browser page."""
    driver.back()
    return "Navigated to the previous page."
```

The browser driver performs the navigation. The tool returns a message that the agent can use in its next decision.

In a robust implementation, the tool should also handle cases where navigation fails or no previous page exists.

### 22.4 Example: Dismiss a pop-up

The course uses Helium/Selenium-related browser controls to dismiss certain pop-ups, including sending the Escape key.

A simplified Selenium implementation is:

```python
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.common.keys import Keys


@tool
def close_popups() -> str:
    """Attempt to dismiss a visible dialog using Escape."""

    ActionChains(driver).send_keys(Keys.ESCAPE).perform()

    return "Sent Escape to the browser."
```

This does not guarantee that every pop-up will close. Cookie consent banners, custom dialogs, and embedded interfaces may require different handling.

### 22.5 Build the browser agent

The course combines the search tool with custom navigation tools and a screenshot callback.

Conceptually, the agent is configured like this:

```python
from smolagents import (
    CodeAgent,
    DuckDuckGoSearchTool,
    OpenAIServerModel,
)

model = OpenAIServerModel(model_id="gpt-4o")

agent = CodeAgent(
    tools=[
        DuckDuckGoSearchTool(),
        go_back,
        close_popups,
        search_item_ctrl_f,
    ],
    model=model,
    additional_authorized_imports=["helium"],
    step_callbacks=[save_screenshot],
    max_steps=20,
    verbosity_level=2,
)
```

This example assumes that the browser driver, helper functions, and `save_screenshot` callback have already been defined.

The callback is responsible for capturing screenshots and recording them in the agent's execution history.

Depending on the installed version, the accepted callback parameter names and APIs may differ. Follow the current documentation when adapting the notebook.

### 22.6 How the complete browser workflow operates

```text
User gives a browsing task
          |
          v
Agent selects a search or navigation tool
          |
          v
Browser performs the action
          |
          v
Screenshot and page information are recorded
          |
          v
Vision-capable model interprets the observation
          |
          v
Agent chooses the next action
          |
          v
Repeat until completion or step limit
```

The important difference from a normal search application is that the agent can use what it observes after each action to guide its next step.

### 22.7 Important security and reliability limitations

Browser agents can interact with real websites, which introduces risks beyond ordinary text generation.

For example:

- A page may contain misleading instructions intended to manipulate the agent.
- A website may change its layout and break a tool.
- The agent may click the wrong control.
- A page may contain sensitive information.
- Automated actions may submit a form or trigger an irreversible change.

Use isolated browser sessions where appropriate. Restrict access to sensitive sites, require confirmation for consequential actions, and treat website content as untrusted input.

For identity verification or access control, a visual description from an LLM is not sufficient evidence that a person is who they claim to be. Such decisions require an appropriate, validated verification process and human oversight where necessary.

---

## 23. Observability: Understanding What Your Agent Actually Did

An agent may appear to work correctly in one test and fail on another.

Possible reasons include:

- It selected the wrong tool.
- A tool returned incomplete information.
- The model generated invalid code.
- A search result was misleading.
- A later step misunderstood an earlier result.
- The agent reached its maximum number of steps.
- A final answer was generated without enough supporting evidence.

Simply looking at the final answer does not reveal the entire execution path.

**Observability** helps developers inspect agent execution, diagnose failures, and understand system behavior.

### 23.1 What is tracing?

A trace records the sequence of operations involved in a request.

For an agent, a trace might include:

```text
User request
    |
    v
Model call
    |
    v
Search tool call
    |
    v
Search results
    |
    v
Second model call
    |
    v
Web-page visit
    |
    v
Final response
```

The trace helps answer questions such as:

- Which tools did the agent use?
- What arguments did it provide?
- How long did each operation take?
- Where did the workflow fail?
- How many model calls were made?
- Did the agent recover from an error?
- Did the final answer rely on relevant information?

### 23.2 OpenTelemetry and Langfuse

The course demonstrates using OpenTelemetry instrumentation with Langfuse.

- **OpenTelemetry** provides a standard approach for collecting telemetry from software systems.
- **Langfuse** provides tooling for observing and analyzing LLM application executions.
- **SmolagentsInstrumentor** instruments supported smolagents executions so that their traces can be collected.

Install the dependencies used in the course:

```bash
pip install \
    opentelemetry-sdk \
    opentelemetry-exporter-otlp \
    openinference-instrumentation-smolagents \
    langfuse
```

### 23.3 Configure your credentials

Create a Langfuse project and obtain the appropriate API keys.

The course uses environment variables similar to these:

```python
import os

os.environ["LANGFUSE_PUBLIC_KEY"] = "your-public-key"
os.environ["LANGFUSE_SECRET_KEY"] = "your-secret-key"

os.environ["LANGFUSE_HOST"] = (
    "https://cloud.langfuse.com"
)
```

The host depends on your Langfuse deployment or region.

**Security note:** Never commit real API keys to GitHub or publish them in a blog. For an application, configure secrets through environment variables or an appropriate secret manager.

### 23.4 Verify the connection

```python
from langfuse import get_client

langfuse = get_client()

if langfuse.auth_check():
    print("Langfuse authentication succeeded.")
else:
    print("Check the credentials and host.")
```

This checks whether the client can authenticate with the configured service.

### 23.5 Instrument smolagents

```python
from openinference.instrumentation.smolagents import (
    SmolagentsInstrumentor,
)

SmolagentsInstrumentor().instrument()
```

After instrumentation is initialized, supported agent runs can produce telemetry that is collected by the configured tracing system.

The exact trace fields and available instrumentation depend on the installed versions and configuration.

### 23.6 Why observability matters

Suppose an agent returns the wrong catering recommendation.

Without traces, you may only see the final incorrect answer.

With traces, you might discover that:

1. The agent selected the catering search tool.
2. The tool returned results for a different city.
3. The agent did not verify the location.
4. The final answer used the irrelevant result.

The fix is then clearer: improve the tool, its description, the query validation, or the agent's instructions.

The course also highlights inspecting traces to find errors that occurred during execution and were later recovered from. A final successful response does not necessarily mean that every intermediate step was successful.

Observability makes those intermediate behaviors visible.

---

## 24. Evaluating Agents: How Do We Know They Work?

Observability tells us what happened. **Evaluation tells us how well the system performed.**

These are related, but different, activities.

A trace may show that an agent used a search tool five times. Evaluation asks whether those searches were necessary, whether the results were correct, and whether the final answer satisfied the task.

### 24.1 Evaluate more than the final answer

Useful evaluation dimensions include:

**Task success**

Did the agent accomplish the requested task?

**Factual correctness**

Are the claims in its answer accurate?

**Tool correctness**

Did it select appropriate tools and pass valid arguments?

**Evidence quality**

Are important claims supported by reliable sources?

**Efficiency**

How many model calls, tool calls, tokens, and execution steps were required?

**Reliability**

Does the agent succeed consistently across different inputs?

**Safety**

Does it avoid unauthorized or destructive actions?

**Recovery**

Can it handle tool errors and missing information without inventing an answer?

### 24.2 Build a small test set

Suppose you are developing a party-planning agent.

Prepare a collection of representative questions and expected outcomes.

| Test case | Expected behavior |
|---|---|
| Request for decorations | Retrieves relevant decoration information |
| Request for catering | Uses catering information rather than unrelated entertainment results |
| Unknown topic | Clearly reports insufficient information |
| Conflicting sources | Identifies or resolves the conflict rather than silently choosing unsupported information |
| Invalid tool result | Handles the error without fabricating a successful result |
| Complex multi-part request | Addresses each requested component |

Run the agent on the same test set after changes to prompts, models, tools, or retrieval logic.

This makes regressions easier to identify.

### 24.3 Use deterministic checks wherever possible

LLM-based evaluation can help assess subjective or complex outputs, but deterministic tests are preferable for properties that can be checked exactly.

For example:

- Validate whether coordinates are numeric and within valid ranges.
- Check whether a required output file exists.
- Test whether a calculation returns the expected value within a tolerance.
- Verify that required fields are present in a structured response.
- Confirm that a tool rejects invalid input.
- Check whether every cited source corresponds to a retrieved document.

For important applications, combine unit tests, integration tests, traces, and model-based evaluation.

---

## 25. Making Agent Systems More Reliable

An agent is not reliable merely because it can use tools. The surrounding system needs explicit limits and safeguards.

### 25.1 Limit execution steps

The `max_steps` parameter places a limit on how many execution steps an agent may take.

For example:

```python
agent = CodeAgent(
    tools=[search_tool],
    model=model,
    max_steps=8,
)
```

This helps prevent an agent from continuing indefinitely.

A small value may stop a complex task too early. A very large value can increase cost and make loops more expensive. Choose the limit based on the task and monitor how often the agent reaches it.

### 25.2 Give tools precise descriptions

Tool descriptions help the model determine which tool to use.

A vague description such as:

```python
description = "Search tool"
```

provides little information.

A more useful description explains the tool's purpose, expected input, scope, and limitations.

For example:

```python
description = (
    "Search the local party-planning knowledge base for "
    "information about decorations, entertainment, and catering. "
    "Returns relevant document passages; it does not generate "
    "a final recommendation."
)
```

The more accurately the tool's description represents its actual behavior, the easier it is for the model to select it appropriately.

### 25.3 Use explicit error handling

A tool should not silently pretend that it succeeded when an operation fails.

For example:

```python
def divide_numbers(a: float, b: float) -> float:
    if b == 0:
        raise ValueError("The denominator cannot be zero.")

    return a / b
```

For external services, also consider timeouts, rate limits, network errors, malformed responses, and authentication failures.

Whether a tool should raise an exception or return a structured error depends on the framework and how the agent is expected to recover.

### 25.4 Separate decisions from calculations

Language models can decide which calculation to perform, but ordinary code should perform exact arithmetic whenever possible.

For example, a travel-planning agent can identify the relevant coordinates and request a calculation. A Python function should calculate the distance.

This division reduces avoidable numerical mistakes.

### 25.5 Validate external information

A web search result may be incomplete, outdated, or misleading.

When accuracy matters:

1. Open the original source.
2. Check whether it actually supports the claim.
3. Compare conflicting sources.
4. Preserve the source information.
5. Report uncertainty when verification is not possible.

### 25.6 Apply least privilege

An agent should receive only the tools, imports, data, and permissions needed for its task.

For example, a research agent generally does not need permission to delete files or send emails.

This is especially important for code agents because executing generated Python can have significant consequences.

---

## 26. Security: Authorized Imports Are Not a Complete Sandbox

Part 1 introduced `additional_authorized_imports`, which controls which imports a CodeAgent is permitted to use through the framework's execution mechanism.

For example:

```python
agent = CodeAgent(
    tools=[],
    model=model,
    additional_authorized_imports=["datetime"],
)
```

This can help restrict the available Python functionality.

However, **an import allowlist should not be treated as a complete security boundary**.

The safety of generated-code execution also depends on the executor, operating-system permissions, filesystem access, network access, installed packages, and other runtime capabilities.

For untrusted or production workloads:

- Prefer an appropriately isolated execution environment.
- Restrict network and filesystem access.
- Use narrowly scoped credentials.
- Avoid exposing secrets to generated code.
- Apply resource limits and timeouts.
- Review the current smolagents documentation for supported sandboxing options.
- Test failure and abuse scenarios before deployment.

The smolagents documentation has described execution options involving environments such as Docker and hosted sandboxes. Availability and configuration can change by version and provider.

Tool permissions also matter. A model can only be constrained effectively if the tools and surrounding application enforce the restrictions.

---

## 27. Putting the Pieces Together

We have now covered the major building blocks from the smolagents course.

| Component | Main responsibility |
|---|---|
| Language model | Interprets the task and proposes actions |
| CodeAgent | Uses generated Python code to perform tool actions |
| ToolCallingAgent | Uses structured tool-call representations |
| Tools | Connect the agent to external capabilities |
| Retriever | Finds relevant information from a knowledge base |
| Agentic RAG | Coordinates retrieval and evidence gathering iteratively |
| Manager agent | Delegates work and combines results |
| Vision model | Processes image-based information |
| Browser automation | Navigates and interacts with web pages |
| Callbacks | Execute additional logic at selected execution stages |
| Tracing | Records the agent's execution behavior |
| Evaluation | Measures task quality, correctness, and reliability |
| Security controls | Limit what the agent can access and do |

A practical agent system combines these pieces according to the requirements of the task.

For example, an internal research assistant might use:

1. A CodeAgent to coordinate the task.
2. A local retriever to search internal documents.
3. A web-search tool for public information.
4. A manager agent only if the work benefits from delegation.
5. A validation step to check the evidence.
6. Tracing to investigate failures.
7. Access controls to protect private documents.

Not every application needs a vision model, a browser agent, or multiple agents. Start with the simplest architecture that meets the requirements, then add capabilities when a demonstrated limitation justifies them.

---

## 28. How These Concepts Connect to V-Meet

These ideas can be applied to the V-Meet meeting platform as a future development direction.

### 28.1 Meeting knowledge assistant

A retrieval agent could search meeting transcripts and summaries.

Example:

> What deadlines were discussed in our last three project meetings?

A possible workflow:

1. Retrieve relevant meeting records.
2. Filter them according to the user's access permissions.
3. Search for deadlines and action items.
4. Combine the relevant evidence.
5. Produce a concise answer with meeting references.

### 28.2 Meeting summarization workflow

An agent could coordinate several specialized operations:

- Transcript retrieval
- Topic extraction
- Action-item extraction
- Summary generation
- Evidence checking
- Saving the final output

However, a deterministic pipeline may be preferable if the process is straightforward and predictable. Agent orchestration should solve a real coordination problem rather than add unnecessary complexity.

### 28.3 Visual meeting assistance

A vision-capable model could analyze screenshots of a meeting interface or help users understand charts shared during a meeting.

This requires a multimodal model and an appropriate way to capture and supply the image. It should not be assumed that a text-only model can perform visual analysis.

### 28.4 Observability for production

Tracing can help diagnose issues such as:

- A meeting assistant repeatedly retrieving irrelevant transcripts.
- A tool failing to save a summary.
- Excessive model calls for a simple request.
- An agent reaching its step limit.
- A generated answer that lacks supporting meeting evidence.

Private meeting transcripts and screenshots can contain sensitive information. Logging should therefore follow appropriate data-minimization, access-control, and retention practices.

These are possible applications of the course concepts, not features that are automatically provided by smolagents.

---

## 29. Common Mistakes to Avoid

**Mistake 1: Assuming an agent is always better than a fixed workflow.**

A fixed pipeline can be cheaper, faster, and easier to test when the task is predictable.

**Mistake 2: Treating retrieved text as automatically correct.**

Retrieval finds potentially relevant information. It does not prove that the information is accurate or current.

**Mistake 3: Adding more agents without defining their responsibilities.**

Each agent should have a clear role, an appropriate set of tools, and a useful output.

**Mistake 4: Giving every agent the same tools and context.**

This can increase complexity and expose capabilities that the agent does not need.

**Mistake 5: Assuming a successful final answer proves every step was correct.**

Inspect traces, validate outputs, and test intermediate behavior.

**Mistake 6: Treating authorized imports as complete isolation.**

Generated-code execution needs an appropriate security boundary.

**Mistake 7: Assuming browser actions always succeed.**

Websites change, selectors break, dialogs behave differently, and network requests fail.

**Mistake 8: Trusting model-based evaluation without independent checks.**

Use deterministic tests for calculations, data constraints, file existence, and required output structure.

**Mistake 9: Forgetting model and framework compatibility.**

Examples in course notebooks may use particular model IDs, providers, and API versions. Verify these details before reusing the code.

**Mistake 10: Ignoring cost and latency.**

Every additional model call, retrieval step, browser interaction, and delegated task can increase execution time and cost.

---

## 30. Practical Learning Exercises

Use these exercises to confirm that you understand the concepts rather than merely recognizing the code.

### Exercise 1 — Basic retrieval

Create a small knowledge base containing five documents about machine learning.

Build a BM25 retriever and search for:

- Classification
- Overfitting
- Gradient descent

Observe which passages are returned.

### Exercise 2 — Retrieval with an agent

Give your retriever to a CodeAgent.

Ask it to explain overfitting using the knowledge base. Check whether the answer is supported by the retrieved passages.

### Exercise 3 — Query decomposition

Ask:

> What are the differences between classification, regression, and clustering, and when should each be used?

Design a retrieval workflow that searches for each concept separately and combines the results.

### Exercise 4 — Tool design

Create a custom tool that calculates a simple metric, such as precision.

Give it a clear name, description, input schema, and output type.

Test valid inputs and edge cases.

### Exercise 5 — Multi-agent architecture

Create a research agent and a manager agent.

Give the research agent responsibility for gathering evidence. Give the manager responsibility for combining the results and preparing a report.

Compare the result with a single-agent implementation.

### Exercise 6 — Vision

Supply a screenshot to a compatible multimodal model and ask it to describe the visible interface.

Compare its description with the actual screenshot and identify any mistakes.

### Exercise 7 — Browser automation

Create a browser workflow that opens a public documentation page, searches for a heading, and records the URL.

Test what happens when the heading does not exist.

### Exercise 8 — Observability

Instrument an agent run and inspect its execution trace.

Identify the model calls, tool calls, errors, and total number of execution steps.

### Exercise 9 — Evaluation

Prepare a fixed set of questions and expected outcomes for your agent.

Run the same tests after changing its tools or prompts. Record which changes improve or reduce performance.

### Exercise 10 — Security

Review the capabilities available to your agent.

Remove unnecessary tools, restrict sensitive access, and identify which operations should require explicit user confirmation.

