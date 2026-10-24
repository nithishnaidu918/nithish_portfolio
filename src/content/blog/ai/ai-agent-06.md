---
title: "MCP and A2A Protocols: Connecting AI Agents, Tools, and Systems"
description: "Learn Model Context Protocol (MCP) and Agent2Agent (A2A), understand their architectures, explore practical Python examples, and discover how they enable interoperable agentic AI systems."
category: "AI Agents"
tags: ["Agentic AI", "MCP", "A2A", "LLM", "AI Agents", "Python", "Multi-Agent Systems"]
---

# MCP and A2A Protocols: Connecting AI Agents, Tools, and Systems

## Introduction

In previous articles, we explored how to build AI agents that use tools, retrieve information, and perform tasks with frameworks such as smolagents, LlamaIndex, and LangGraph.

However, building an agent is only one part of developing a complete agentic AI system.

Consider an AI agent that can search the web, retrieve documents, query databases, and analyze information. How can we connect it to different external systems without writing a separate integration for every application?

Now consider a different problem. Suppose we have multiple AI agents, each specializing in a particular task. How can these independently developed agents communicate and collaborate?

Two important protocols address these challenges:

- **Model Context Protocol (MCP):** Standardizes how AI applications connect to external tools, data, and capabilities.

- **Agent2Agent (A2A) Protocol:** Enables independently built AI agents to discover each other, communicate, and delegate tasks.

MCP was introduced by Anthropic in November 2024. Google announced A2A in April 2025.

These protocols address different aspects of interoperability and are useful concepts to understand when building modern agentic AI applications.

## 1. What Is Model Context Protocol (MCP)?

The Model Context Protocol is an open standard that defines how AI applications communicate with external tools, data sources, and services.

Think of MCP as a universal adapter for AI applications.

Just as USB-C provides a standardized way to connect different devices, MCP provides a standardized way for compatible AI applications to connect to external capabilities.

Without a shared protocol, developers often need to create custom integrations between individual AI applications and the systems they use.

MCP reduces this integration problem by defining a common communication interface.

### 1.1 Why Do We Need MCP?

Imagine that we are building an AI research assistant.

We want it to perform the following operations:

- Search GitHub repositories.
- Read files from a local directory.
- Query a PostgreSQL database.
- Search Hugging Face models and datasets.
- Retrieve information from a company knowledge base.

Without MCP, we might write separate integration code for each service.

For example:

```python
def search_github(query):
    # Call the GitHub API
    pass


def search_database(query):
    # Execute a database query
    pass


def search_documents(query):
    # Retrieve relevant documents
    pass
```

This approach can work well for a small application. However, as the number of tools and AI applications grows, maintaining separate integrations becomes more complicated.

MCP provides a common protocol through which compatible applications can discover and invoke capabilities exposed by MCP servers.

The server still needs an implementation for each underlying service, but compatible clients do not need to understand every service's custom integration protocol.

### 1.2 MCP Architecture

MCP uses a client-server architecture with three important roles.

**1. MCP Host**

The host is the AI application in which the model operates.

Examples include an AI assistant, an integrated development environment, or an agent application.

The host manages the model interaction and coordinates access to MCP servers.

**2. MCP Client**

The MCP client is the component that communicates with an MCP server.

It establishes the connection, discovers supported capabilities, and sends protocol requests.

An application can use multiple MCP clients to connect to different servers.

**3. MCP Server**

The MCP server exposes capabilities such as tools, resources, and prompts.

It connects the protocol to an actual implementation, such as a database, API, filesystem, or internal application.

The server might be a local process or a remote service, depending on its configuration.

The architecture can be represented as follows:

```text
                 User
                   |
                   v
            AI Application
               (Host)
                   |
                   v
              MCP Client
                   |
            MCP Protocol
                   |
                   v
              MCP Server
                   |
          +--------+--------+
          |        |        |
          v        v        v
       GitHub   Database  Documents
        API
```

The MCP protocol standardizes the communication between the client and server. The server's implementation determines how requests interact with the underlying systems.

### 1.3 How Does MCP Work?

Consider a user asking:

"Find the latest changes in my GitHub repository and summarize them."

The interaction can follow these steps:

1. The user submits a request to the AI application.
2. The application connects to an MCP server that exposes GitHub capabilities.
3. The MCP client discovers the tools available from the server.
4. The language model determines which available tool is appropriate.
5. The client sends a request to the server.
6. The server executes the corresponding operation using its underlying implementation.
7. The server returns the result.
8. The language model uses the result to prepare an answer.

The important point is that MCP defines how the application and server communicate. The language model is responsible for deciding how to use the available capabilities, while the server performs the operation.

MCP does not automatically give an AI model access to every external service. Authentication, authorization, server configuration, and available tool implementations are still necessary.

### 1.4 The Three Main MCP Primitives

MCP defines three particularly important primitives: tools, resources, and prompts.

#### A. Tools

Tools allow an AI application to request an operation from an MCP server.

Examples include:

- Searching a repository.
- Querying a database.
- Calculating a result.
- Creating an issue.
- Retrieving information from an external API.

For example, a server might expose a tool called `search_repositories`.

The application can discover the tool's name, description, and input schema, then request its execution when appropriate.

Tools can perform read-only operations or actions that modify external systems. Applications should apply suitable permissions and approval controls to sensitive operations.

#### B. Resources

Resources expose information that an AI application can read and use as context.

Examples include:

- A project README.
- A document.
- A database record.
- Configuration information.
- Content from an internal knowledge base.

Resources help applications access relevant information without requiring every source to use the same underlying storage system.

#### C. Prompts

Prompts are reusable prompt templates exposed by an MCP server.

For example, a server might provide a template for reviewing a pull request or analyzing a document.

An application can discover and use these templates instead of maintaining every reusable prompt independently.

The three primitives have different purposes:

| Primitive | Main purpose | Example |
|---|---|---|
| Tools | Request operations | Search a GitHub repository |
| Resources | Access information | Read a project README |
| Prompts | Reuse prompt templates | Review a pull request |

These concepts are foundational when learning how MCP servers expose functionality to AI applications.

### 1.5 Understanding MCP Communication

MCP uses JSON-RPC messages for communication.

JSON-RPC provides a structured way to represent requests, responses, and notifications.

For example, a simplified request to invoke an addition tool could look like this:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "add",
    "arguments": {
      "a": 10,
      "b": 20
    }
  }
}
```

A corresponding simplified response could be:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "30"
      }
    ],
    "isError": false
  }
}
```

These examples illustrate the basic request-response idea. Actual messages must follow the protocol's applicable schemas and capabilities.

MCP also supports different transports, including standard input/output (stdio) and Streamable HTTP.

- **stdio:** Commonly used when a client launches and communicates with a local server process.
- **Streamable HTTP:** Used for communication with an HTTP-accessible MCP server.

You do not need to implement JSON-RPC or transport handling manually when using a suitable MCP SDK.

### 1.6 Building a Simple MCP Server in Python

Let's create a small MCP server that exposes an addition tool.

First, install the official Python SDK:

```bash
pip install "mcp[cli]"
```

Create a file called `server.py`:

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("Calculator")


@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two integers."""
    return a + b


if __name__ == "__main__":
    mcp.run()
```

Let's understand the code.

**Step 1: Create the server**

```python
mcp = FastMCP("Calculator")
```

This creates an MCP server named `Calculator`.

**Step 2: Register a tool**

```python
@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two integers."""
    return a + b
```

The decorator exposes the Python function as an MCP tool.

The function's name and description help a compatible client understand its purpose. Its parameters define the expected inputs.

**Step 3: Start the server**

```python
mcp.run()
```

This starts the server using its configured transport.

This example creates a server; it does not itself connect a language model to the tool. To test the complete workflow, connect it to an MCP-compatible client or host and follow the SDK's current setup instructions.

### 1.7 Where Can We Use MCP?

MCP is useful when an AI application needs standardized access to external capabilities.

Potential applications include:

- Connecting an AI coding assistant to GitHub.
- Exposing database queries to an AI application.
- Providing access to files and documents.
- Connecting an agent to a retrieval system.
- Integrating internal company APIs.
- Giving an AI application access to model and dataset discovery services.

For example, we could expose a document retriever from an Agentic RAG application as an MCP tool.

A compatible AI application could then discover and use that retriever through MCP, provided the relevant server and client capabilities are supported.

This allows us to separate the implementation of a capability from the particular AI application that uses it.

## 2. What Is Agent2Agent (A2A) Protocol?

MCP helps AI applications connect to external capabilities. However, another challenge appears when we build systems containing multiple independent agents.

The Agent2Agent protocol, commonly called A2A, is an open protocol designed to enable communication and collaboration between independently developed AI agents.

Google announced A2A in April 2025 to improve interoperability between agents built using different frameworks, technologies, and environments.

### 2.1 Why Do We Need A2A?

Imagine that we have built three AI agents.

**Research Agent**

Its responsibility is to search for information and summarize relevant findings.

**Data Analysis Agent**

Its responsibility is to analyze datasets, calculate metrics, and interpret results.

**Report Writing Agent**

Its responsibility is to transform research findings and analysis into a structured report.

Each agent may use a different framework and have its own tools, model, and internal workflow.

We now want to create a system in which these agents collaborate.

Without a common communication protocol, we might have to implement custom integration logic for every pair of agents.

As the system grows, these integrations can become difficult to maintain.

A2A provides a standardized way for agents to discover capabilities, send tasks, communicate updates, and exchange results.

### 2.2 A2A Architecture

Consider the following architecture:

```text
                    User
                      |
                      v
              Orchestrator Agent
                      |
             A2A Communication
             /        |        \
            v         v         v
      Research     Analysis    Writing
       Agent        Agent       Agent
            \         |         /
             \        |        /
                      v
               Final Report
```

The orchestrator coordinates the overall workflow. The specialist agents perform their assigned tasks and return results.

A2A provides the communication mechanism between the agents; the application still needs logic to decide how work should be distributed and how the results should be combined.

A2A does not require all agents to use the same model or framework.

### 2.3 Important Concepts in A2A

#### A. Agent Cards

An Agent Card describes an agent and the capabilities it offers.

It can include information such as:

- The agent's identity.
- Its advertised skills.
- Its communication endpoint.
- Supported input and output modes.
- Authentication requirements.

For example, a research agent might advertise skills such as web research, document summarization, and source comparison.

An orchestrator can inspect the Agent Card to determine whether the research agent is suitable for a particular task.

An Agent Card supports discovery; it does not guarantee that the agent will always produce correct or reliable results.

#### B. Tasks

Tasks represent units of work delegated between agents.

For example:

"Research recent developments in open-source language models and return a summary of the main findings."

The receiving agent may need to search multiple sources and analyze the information before returning a result.

Unlike a simple function call, a task can involve a longer-running process with a lifecycle that the client can track.

#### C. Messages

Messages allow agents to exchange information related to a task.

A message may contain a request, additional context, an update, or a response.

This enables agents to communicate without requiring the client to know every detail of the remote agent's internal implementation.

#### D. Artifacts

Artifacts are outputs produced by a task.

Examples include:

- A research report.
- A generated document.
- A structured dataset.
- An analysis result.
- Other task outputs supported by the application.

An orchestrator can pass an artifact from one stage of a workflow to another.

For example, the research agent can return a report that the writing agent uses to prepare the final document.

#### E. Task Status and Updates

Some agent tasks finish quickly, while others require more time.

A2A supports task-state updates and mechanisms for following progress.

This is useful when an agent must conduct extended research, process a large amount of information, or wait for an external operation to complete.

The exact interaction depends on the capabilities and communication mechanisms supported by the participating agents.

### 2.4 How Does A2A Work?

Suppose a user asks:

"Research recent developments in Agentic RAG, compare the main approaches, and prepare a technical report."

A possible workflow is:

1. The orchestrator receives the user's request.
2. It identifies the subtasks required to complete the request.
3. It discovers suitable agents using their advertised capabilities.
4. It delegates research to a research agent.
5. The research agent performs its work and returns its findings.
6. The orchestrator sends the relevant information to an analysis agent.
7. The analysis agent compares the approaches and returns its results.
8. The orchestrator delegates report preparation to a writing agent.
9. The writing agent generates the report.
10. The orchestrator combines the outputs and returns the final result.

The agents can use different underlying frameworks as long as they can communicate through compatible A2A interfaces.

This is particularly useful for enterprise systems in which different teams develop and operate different agents.

### 2.5 What Makes A2A Different from a Normal Function Call?

Consider this Python function:

```python
def calculate_sum(a, b):
    return a + b
```

Calling this function executes a specific operation within the current program.

Now consider delegating a research task to an independent agent.

The receiving agent might need to search the web, read documents, reason about the findings, and produce a report. It may take time to complete the task.

The calling application needs a way to discover the agent's capabilities, communicate with it, and track the task's progress.

A2A is designed for this kind of agent-to-agent interoperability.

A2A does not replace ordinary function calls. It addresses a different problem: communication and task delegation between agents that may be independently developed and operated.

## 3. MCP vs. A2A: What Is the Difference?

MCP and A2A are complementary protocols, but they focus on different communication relationships.

| Aspect | MCP | A2A |
|---|---|---|
| Primary purpose | Connect AI applications to tools and data | Enable communication between independent agents |
| Main relationship | AI application and MCP server | Client agent and remote agent |
| Discovery | Discover tools, resources, and prompts | Discover agents and their capabilities |
| Typical operation | Invoke a tool or access a resource | Delegate and track a task |
| Example | Query a database through an MCP server | Ask a research agent to prepare findings |
| Typical use | Tool and context integration | Multi-agent collaboration |

A useful way to remember the distinction is:

**MCP connects an AI application to capabilities. A2A connects agents to other agents.**

The protocols can be used together in the same application.

## 4. How MCP and A2A Work Together

Let's build a conceptual architecture for an AI research platform.

The platform contains three agents:

- A Research Agent.
- A Data Analysis Agent.
- A Report Writing Agent.

The Research Agent needs access to a web-search tool and a document-retrieval system. The Data Analysis Agent needs access to datasets. The Report Writing Agent needs access to documents and report-generation capabilities.

We can use MCP to expose the tools and data each agent needs.

We can use A2A to let the agents delegate tasks and exchange results.

The combined architecture looks like this:

```text
                        User
                          |
                          v
                  Orchestrator Agent
                          |
                    A2A Protocol
                 /        |        \
                v         v         v
           Research    Analysis   Writing
            Agent       Agent      Agent
                |          |          |
               MCP        MCP        MCP
                |          |          |
                v          v          v
           Search and   Datasets   Documents and
           Retrieval               Report Tools
```

This design separates two responsibilities:

- MCP handles access to external capabilities.
- A2A handles communication and delegation between agents.

For example, the Research Agent could use an MCP server to retrieve information, then send its findings to the Analysis Agent through A2A.

The Analysis Agent could process the findings and return an artifact that the Report Writing Agent uses to create the final report.

This architecture can make complex AI systems more modular. However, the protocols alone do not guarantee good orchestration, accurate results, secure execution, or efficient collaboration. Those still depend on the system's design.

## 5. Security Considerations

Both protocols introduce important security considerations because agents may interact with external systems and perform operations on behalf of users.

When building MCP-based applications, consider:

- Authenticating clients and servers appropriately.
- Limiting access to the resources and tools each application needs.
- Protecting API keys and credentials.
- Requiring approval for sensitive or destructive actions.
- Validating tool inputs and outputs.
- Avoiding unnecessary exposure of private data.

When building A2A systems, consider:

- Verifying the identity of remote agents.
- Enforcing authorization before delegating sensitive tasks.
- Validating returned messages and artifacts.
- Limiting the actions that delegated agents can perform.
- Handling failed, delayed, or untrusted agent responses.
- Avoiding the assumption that a discovered agent is automatically trustworthy.

For both protocols, tool outputs and agent responses should be treated as potentially untrusted data.

For example, a document retrieved by an agent might contain malicious instructions. The application should not automatically treat those instructions as authorized commands.

Security should be considered from the beginning rather than added after the system has been built.

## 6. What Should You Learn Next?

If you are learning Agentic AI with Python, a good approach is to understand the underlying concepts before trying to build a large multi-agent system.

### Step 1: Learn MCP Fundamentals

Start with:

- MCP hosts, clients, and servers.
- Tools, resources, and prompts.
- JSON-RPC basics.
- Local and remote transports.
- Tool discovery and invocation.
- Authentication and permissions.

Then build a simple MCP server that exposes a Python function.

### Step 2: Build a Practical MCP Integration

Extend the calculator example into something more useful.

For example, create a server that exposes a document-retrieval tool.

The tool could accept a query, search your documents, and return relevant passages.

Then connect it to a compatible MCP client and verify that the client can discover and invoke the tool.

### Step 3: Learn A2A Fundamentals

Focus on:

- Agent Cards.
- Agent discovery.
- Tasks and messages.
- Task lifecycle and status updates.
- Artifacts and returned results.
- Communication and authentication between agents.

Start with two agents rather than building a large multi-agent system immediately.

### Step 4: Build a Two-Agent Application

Create a Research Agent and a Report Writing Agent.

The Research Agent should gather and organize information. The Report Writing Agent should transform the findings into a structured report.

Initially, you can connect them using ordinary Python functions to understand the workflow.

Then implement communication using a compatible A2A implementation to learn what the protocol adds.

### Step 5: Combine MCP and A2A

Finally, expose the Research Agent's search or retrieval capability through MCP and use A2A for communication between the Research Agent and the Report Writing Agent.

This will help you understand the difference between connecting an agent to a tool and connecting one agent to another.