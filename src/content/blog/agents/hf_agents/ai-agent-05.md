---
title: "Agentic RAG with Python: Building Alfred, a Tool-Using AI Agent"
description: "Learn Agentic RAG by building Alfred, a gala-hosting AI agent with guest information retrieval, web search, weather tools, Hugging Face Hub statistics, and conversation memory using smolagents, LlamaIndex, and LangGraph."
category: "AI Agents"
tags: ["Agentic RAG", "RAG", "AI Agents", "smolagents", "LlamaIndex", "LangGraph", "Python", "Hugging Face"]
---

# Agentic RAG: Building Alfred, a Tool-Using AI Agent

## 1. Introduction to Agentic RAG

Imagine hosting the most extravagant gala of the century. There are renowned scientists, artists, researchers, and influential people attending. The event includes lavish food, music, entertainment, and a fireworks display.

You have a friendly assistant named **Alfred**, who is responsible for helping you manage the event.

Alfred needs to answer questions such as:

- Who is Lady Ada Lovelace, and what is her background?
- Which guests are interested in science and technology?
- What is the weather forecast for the fireworks display?
- What are the latest developments in wireless energy?
- Which model has the most downloads from a particular organization on the Hugging Face Hub?

A general-purpose LLM might answer some questions from its training knowledge. However, it cannot automatically know the private guest list for your event, and its stored knowledge may not include the latest information.

Alfred therefore needs access to a knowledge base and tools that can retrieve information when required.

This is where **Retrieval-Augmented Generation (RAG)** and **Agentic RAG** become useful.

## 2. Understanding the Gala Requirements

Before implementing Alfred, let's identify what he needs to do.

### Knowledge about sports, culture, and science

A good host should be able to discuss a wide variety of topics and make guests feel comfortable.

Alfred needs access to useful information about the guests, their interests, their backgrounds, and their areas of expertise.

### Avoiding sensitive conversation topics

Some topics, such as politics and religion, may be inappropriate for this particular event. Alfred should be guided to avoid introducing those topics when helping the host prepare conversations.

This is an application-level requirement: retrieving information alone does not guarantee that the agent will follow the desired conversation policy.

### Understanding guest backgrounds

Alfred needs to retrieve details such as:

- Guest names
- Relationships with the host
- Biographical information
- Interests and professional achievements
- Email addresses
- Potential conversation starters

### Accessing current information

Alfred also needs tools to retrieve information that may change over time, including web search results, weather information, and Hugging Face Hub model statistics.

These requirements make Alfred a good example of an agent that combines a private knowledge base with external tools.

## 3. What Is Retrieval-Augmented Generation?

Large language models are trained on large amounts of text and other data. They can use what they learned during training to generate responses, but their internal knowledge is not automatically updated whenever new information becomes available.

RAG addresses this limitation by retrieving relevant information from an external knowledge source and providing it to the language model when generating a response.

A traditional RAG pipeline generally follows these steps:

1. Receive a question.
2. Search a knowledge base for relevant documents.
3. Retrieve the most relevant results.
4. Provide those results to the LLM as context.
5. Generate an answer using the retrieved information.

For example, suppose the host asks:

> Tell me about Lady Ada Lovelace.

The RAG system searches the gala's guest database, retrieves her record, and provides the relevant information to the LLM. The model can then produce an answer based on that record.

The important point is that the guest details come from the external dataset rather than relying exclusively on the model's training knowledge.

## 4. What Is Agentic RAG?

**Agentic RAG combines retrieval with an agent that can decide which tools or information sources to use.**

In a basic RAG system, the application usually follows a predefined retrieval process whenever a question arrives.

An Agentic RAG system gives an agent more control over the process. Depending on the question, it can decide to:

- Retrieve information from the guest database.
- Search the web for recent information.
- Request weather information.
- Retrieve model statistics from the Hugging Face Hub.
- Combine results from multiple tools.
- Ask a follow-up question if the request is ambiguous.
- Generate a final response after gathering the necessary information.

Consider these examples:

| User question | Appropriate tool |
|---|---|
| Who is Lady Ada Lovelace? | Guest-information retriever |
| What is the latest information about wireless energy? | Web search |
| What is the weather in Paris? | Weather tool |
| What is the most downloaded model from Facebook? | Hugging Face Hub statistics tool |
| Help me prepare for a conversation with Dr. Nikola Tesla. | Guest retriever and web search |

The agent determines which capabilities are useful for each request.

A simplified workflow looks like this:

```text
                 User Question
                       |
                       v
                +-------------+
                |     LLM     |
                | Agent Brain |
                +-------------+
                       |
              Decide which tool
                   to use
                       |
          +------------+------------+
          |            |            |
          v            v            v
    Guest RAG      Web Search    Weather Tool
          |            |            |
          +------------+------------+
                       |
                       v
               Tool Results
                       |
                       v
                +-------------+
                |     LLM     |
                | Agent Brain |
                +-------------+
                       |
                       v
                 Final Answer
```

The agent can use one tool or several tools before answering.

The central difference is that **RAG retrieves relevant information, while Agentic RAG lets an agent decide how retrieval and other tools should be used to solve the request.**

---

# Part 1: Creating a RAG Tool for Guest Information

## 5. Why Does Alfred Need a Custom RAG Tool?

A general-purpose LLM does not automatically have access to the guest database for our gala.

The guest list is specific to the event, the information may change, and certain details—such as email addresses—must be retrieved accurately.

We'll create a custom retrieval tool that searches a guest dataset and returns relevant information to Alfred.

The dataset contains four fields for each guest:

- **Name:** The guest's full name.
- **Relation:** How the guest is related to the host.
- **Description:** A biography or interesting facts about the guest.
- **Email:** The guest's contact information.

The guest database will act as our external knowledge source.

## 6. Organizing the Project

The example project uses a modular Python structure:

```text
agentic-rag/
|
├── app.py
├── tools.py
├── retriever.py
└── requirements.txt
```

Each file has a separate responsibility.

**`retriever.py`**

Loads the guest dataset, prepares documents, creates the retrieval system, and exposes the guest-information tool.

**`tools.py`**

Contains additional capabilities such as web search, weather information, and Hugging Face Hub statistics.

**`app.py`**

Initializes the LLM, imports the tools, creates Alfred, and handles user requests.

**`requirements.txt`**

Lists the Python packages needed by the project.

This separation makes the application easier to maintain and extend.

## 7. Step 1 — Load and Prepare the Guest Dataset

We'll use the Hugging Face `datasets` library to load the guest dataset.

For the LangChain implementation, each guest record is converted into a `Document` object.

A document contains the text that can be searched and optional metadata that helps identify the source.

```python
import datasets
from langchain_core.documents import Document

# Load the guest dataset
guest_dataset = datasets.load_dataset(
    "agents-course/unit3-invitees",
    split="train"
)

# Convert each guest record into a document
docs = [
    Document(
        page_content="\n".join([
            f"Name: {guest['name']}",
            f"Relation: {guest['relation']}",
            f"Description: {guest['description']}",
            f"Email: {guest['email']}"
        ]),
        metadata={"name": guest["name"]}
    )
    for guest in guest_dataset
]
```

Let's understand the important parts.

### Loading the dataset

```python
guest_dataset = datasets.load_dataset(
    "agents-course/unit3-invitees",
    split="train"
)
```

This loads the training split of the guest dataset.

The returned dataset contains records with guest names, relationships, descriptions, and email addresses.

### Creating a document

```python
Document(
    page_content="...",
    metadata={"name": guest["name"]}
)
```

`page_content` stores the searchable guest information.

`metadata` stores additional information about the document. Here, the guest's name is retained as metadata.

### Why convert records into documents?

Retrievers work with searchable text representations. Converting each guest record into a document gives the retrieval system a consistent format.

For example, a document might conceptually contain:

```text
Name: Lady Ada Lovelace
Relation: Friend
Description: Mathematician and computing pioneer.
Email: ada.lovelace@example.com
```

This is an illustrative representation, not a guarantee that every field in the live dataset has exactly these values.

## 8. Step 2 — Understand BM25 Retrieval

For this implementation, we'll use **BM25**, a text-retrieval algorithm that ranks documents according to how relevant their terms are to a search query.

BM25 does not require an embedding model. It uses information about query terms, document term frequency, and how common those terms are across the document collection.

For example, if the user asks:

> Tell me about Lady Ada Lovelace.

The retriever searches the guest documents and ranks records containing relevant terms.

A conceptual retrieval process looks like this:

```text
Question
   |
   v
Search query: "Lady Ada Lovelace"
   |
   v
BM25 Retriever
   |
   v
Rank guest documents
   |
   v
Return relevant records
```

BM25 is a useful starting point for keyword-oriented retrieval. However, it may not retrieve the best result when a question uses synonyms or concepts that do not share obvious keywords with the stored document.

For more advanced semantic search, you could use embedding-based retrieval, such as a retriever built with sentence-transformers.

## 9. Step 3 — Create the Guest Retriever Tool

We'll first implement the tool using **smolagents** and LangChain's `BM25Retriever`.

```python
from smolagents import Tool
from langchain_community.retrievers import BM25Retriever


class GuestInfoRetrieverTool(Tool):
    name = "guest_info_retriever"

    description = (
        "Retrieves detailed information about gala guests "
        "based on their name or relation."
    )

    inputs = {
        "query": {
            "type": "string",
            "description": (
                "The name or relation of the guest "
                "you want information about."
            )
        }
    }

    output_type = "string"

    def __init__(self, docs):
        super().__init__()
        self.retriever = BM25Retriever.from_documents(docs)

    def forward(self, query: str) -> str:
        results = self.retriever.invoke(query)

        if not results:
            return "No matching guest information found."

        return "\n\n".join(
            doc.page_content for doc in results[:3]
        )


# Initialize the tool
guest_info_tool = GuestInfoRetrieverTool(docs)
```

### How does this tool work?

**`name`**

```python
name = "guest_info_retriever"
```

This gives the tool a name that the agent can use when deciding which capability to invoke.

**`description`**

Explains what the tool does and when it should be used.

The description matters because an LLM-based agent uses tool descriptions to help choose the appropriate tool.

**`inputs`**

Defines the expected input. Here, the tool accepts a string called `query`.

**`BM25Retriever.from_documents(docs)`**

Creates the retriever from the prepared guest documents.

**`forward()`**

Runs the retrieval operation and returns the content of up to three matching documents.

The maximum of three results is a design choice. A larger result set may provide more context but can also introduce irrelevant information.

The tool returns a message when no results are found instead of returning an empty response.

## 10. Alternative Implementations with LlamaIndex and LangChain

The same retrieval idea can be implemented using different frameworks.

The overall workflow remains similar:

1. Load the dataset.
2. Prepare documents.
3. Create a BM25 retriever.
4. Wrap retrieval in a tool.
5. Provide that tool to an agent.

### 10.1 LlamaIndex implementation

LlamaIndex represents documents with its own `Document` class. In this version, the dataset is accessed through its column-oriented interface.

```python
import datasets
from llama_index.core.schema import Document

guest_dataset = datasets.load_dataset(
    "agents-course/unit3-invitees",
    split="train"
)

docs = [
    Document(
        text="\n".join([
            f"Name: {guest_dataset['name'][i]}",
            f"Relation: {guest_dataset['relation'][i]}",
            f"Description: {guest_dataset['description'][i]}",
            f"Email: {guest_dataset['email'][i]}"
        ]),
        metadata={
            "name": guest_dataset["name"][i]
        }
    )
    for i in range(len(guest_dataset))
]
```

Next, create the retriever and wrap it in a LlamaIndex tool.

```python
from llama_index.core.tools import FunctionTool
from llama_index.retrievers.bm25 import BM25Retriever

bm25_retriever = BM25Retriever.from_defaults(nodes=docs)


def get_guest_info_retriever(query: str) -> str:
    """Retrieve guest information using a name or relation."""

    results = bm25_retriever.retrieve(query)

    if not results:
        return "No matching guest information found."

    return "\n\n".join(
        result.text for result in results[:3]
    )


guest_info_tool = FunctionTool.from_defaults(
    get_guest_info_retriever
)
```

The function is wrapped with `FunctionTool.from_defaults()`, making it available as a tool for a LlamaIndex agent.

**Important implementation note:** LlamaIndex's retriever and node APIs can vary between versions. If your installed version expects nodes rather than `Document` objects, convert the documents into the appropriate node type before creating the retriever.

### 10.2 LangChain implementation

LangChain also provides a `Tool` wrapper for exposing a Python function to an agent.

```python
from langchain_community.retrievers import BM25Retriever
from langchain_core.tools import Tool

bm25_retriever = BM25Retriever.from_documents(docs)


def extract_guest_text(query: str) -> str:
    """Retrieve detailed guest information."""

    results = bm25_retriever.invoke(query)

    if not results:
        return "No matching guest information found."

    return "\n\n".join(
        doc.page_content for doc in results[:3]
    )


guest_info_tool = Tool(
    name="guest_info_retriever",
    func=extract_guest_text,
    description=(
        "Retrieves detailed information about gala guests "
        "based on their name or relation."
    )
)
```

The `Tool` object exposes the function through a name, description, and callable implementation.

These examples demonstrate the same underlying idea: **a retriever becomes useful to an agent when it is exposed through a tool interface that the agent can invoke.**

---

# Part 2: Giving Alfred Access to More Tools

## 11. Why Does Alfred Need Multiple Tools?

Guest retrieval is not enough to answer every question.

If the host asks about current news, the guest database is not the right source. If the host asks about weather, Alfred needs a weather service. If the host asks about model popularity, Alfred needs access to model statistics.

We'll add three capabilities:

1. Web search
2. Weather information
3. Hugging Face Hub model statistics

The weather example below deliberately uses simulated data. It demonstrates tool integration, not real-time weather forecasting.

## 12. Tool 1 — Web Search

Web search allows Alfred to retrieve information that may not be available in the guest dataset or the LLM's existing knowledge.

For example, the host might ask:

> What are the latest developments in wireless energy transmission?

The agent can use a search tool to retrieve relevant web information and then summarize it.

### smolagents implementation

```python
from smolagents import DuckDuckGoSearchTool

search_tool = DuckDuckGoSearchTool()

results = search_tool(
    "Who's the current President of France?"
)

print(results)
```

The tool sends a query to DuckDuckGo and returns search results.

The example question is used to demonstrate how the tool works. The actual response depends on the search results available when it runs.

### LlamaIndex implementation

```python
from llama_index.tools.duckduckgo import (
    DuckDuckGoSearchToolSpec
)
from llama_index.core.tools import FunctionTool

tool_spec = DuckDuckGoSearchToolSpec()

search_tool = FunctionTool.from_defaults(
    tool_spec.duckduckgo_full_search
)

response = search_tool(
    "Who's the current President of France?"
)

print(response.raw_output[-1]["body"])
```

The tool specification exposes a DuckDuckGo search function, which is then wrapped as a LlamaIndex tool.

### LangChain implementation

```python
from langchain_community.tools import DuckDuckGoSearchRun

search_tool = DuckDuckGoSearchRun()

results = search_tool.invoke(
    "Who's the current President of France?"
)

print(results)
```

All three implementations provide web-search capability, but their interfaces and return formats differ.

Search results should be treated as retrieved evidence rather than automatically assumed to be correct. For important claims, the agent should rely on relevant, trustworthy results and communicate uncertainty when necessary.

## 13. Tool 2 — Weather Information

The gala includes an outdoor fireworks display. Alfred needs weather information to help the host decide whether conditions are suitable.

We'll create a custom weather tool.

For this learning example, the tool selects one of three predefined weather conditions randomly.

```python
from smolagents import Tool
import random


class WeatherInfoTool(Tool):
    name = "weather_info"

    description = (
        "Fetches simulated weather information "
        "for a given location."
    )

    inputs = {
        "location": {
            "type": "string",
            "description": (
                "The location to get weather information for."
            )
        }
    }

    output_type = "string"

    def forward(self, location: str) -> str:
        weather_conditions = [
            {"condition": "Rainy", "temp_c": 15},
            {"condition": "Clear", "temp_c": 25},
            {"condition": "Windy", "temp_c": 20}
        ]

        data = random.choice(weather_conditions)

        return (
            f"Simulated weather in {location}: "
            f"{data['condition']}, {data['temp_c']}°C"
        )


weather_info_tool = WeatherInfoTool()
```

### Understanding the implementation

**Input:** A location such as Paris.

**Simulated weather data:** A list of predefined conditions.

**Random selection:** `random.choice()` selects one condition.

**Output:** A string describing the selected condition and temperature.

Because the condition is randomly selected, repeated calls for the same location may return different results.

This tool is suitable for learning how agents call custom tools. It is **not suitable for real decisions about fireworks, travel, or safety**.

A production implementation would call a real weather API and should retrieve relevant information such as the forecast time, precipitation, wind speed, and severe-weather warnings.

### LlamaIndex implementation

```python
import random
from llama_index.core.tools import FunctionTool


def get_weather_info(location: str) -> str:
    """Fetch simulated weather information."""

    weather_conditions = [
        {"condition": "Rainy", "temp_c": 15},
        {"condition": "Clear", "temp_c": 25},
        {"condition": "Windy", "temp_c": 20}
    ]

    data = random.choice(weather_conditions)

    return (
        f"Simulated weather in {location}: "
        f"{data['condition']}, {data['temp_c']}°C"
    )


weather_info_tool = FunctionTool.from_defaults(
    get_weather_info
)
```

### LangChain implementation

```python
from langchain_core.tools import Tool
import random


def get_weather_info(location: str) -> str:
    """Fetch simulated weather information."""

    weather_conditions = [
        {"condition": "Rainy", "temp_c": 15},
        {"condition": "Clear", "temp_c": 25},
        {"condition": "Windy", "temp_c": 20}
    ]

    data = random.choice(weather_conditions)

    return (
        f"Simulated weather in {location}: "
        f"{data['condition']}, {data['temp_c']}°C"
    )


weather_info_tool = Tool(
    name="get_weather_info",
    func=get_weather_info,
    description=(
        "Fetches simulated weather information "
        "for a given location."
    )
)
```

The key concept is that the agent does not need to know the implementation details of the weather tool. It needs a clear description of what the tool can do and what input it expects.

## 14. Tool 3 — Hugging Face Hub Statistics

Alfred also wants to impress AI researchers attending the gala.

He might be asked:

> What is the most downloaded model published by Facebook?

We can create a tool using `huggingface_hub.list_models()` to retrieve model information for a given author or organization.

```python
from smolagents import Tool
from huggingface_hub import list_models


class HubStatsTool(Tool):
    name = "hub_stats"

    description = (
        "Fetches the most downloaded model from "
        "a specified Hugging Face Hub author."
    )

    inputs = {
        "author": {
            "type": "string",
            "description": (
                "The username or organization name "
                "to find models from."
            )
        }
    }

    output_type = "string"

    def forward(self, author: str) -> str:
        try:
            models = list(
                list_models(
                    author=author,
                    sort="downloads",
                    direction=-1,
                    limit=1
                )
            )

            if not models:
                return f"No models found for author {author}."

            model = models[0]

            return (
                f"The most downloaded model by {author} "
                f"is {model.id} with "
                f"{model.downloads:,} downloads."
            )

        except Exception as error:
            return (
                f"Error fetching models for {author}: "
                f"{error}"
            )


hub_stats_tool = HubStatsTool()

# Example usage
print(hub_stats_tool("facebook"))
```

### How does it work?

**`list_models()`**

Queries the Hugging Face Hub for models associated with the specified author.

**`sort="downloads"`**

Requests sorting by download count.

**`direction=-1`**

Requests descending order in the API interface used by this example.

**`limit=1`**

Limits the results to one model.

**`model.id`**

Contains the model identifier.

**`model.downloads`**

Contains the reported download count, which is formatted with thousands separators for readability.

The returned statistics are live data and can change over time. Any example download count should be treated as illustrative rather than a permanent value.

The tool also handles errors and the case where no models are found.

### LlamaIndex implementation

```python
from llama_index.core.tools import FunctionTool
from huggingface_hub import list_models


def get_hub_stats(author: str) -> str:
    """Fetch the most downloaded model for an author."""

    try:
        models = list(
            list_models(
                author=author,
                sort="downloads",
                direction=-1,
                limit=1
            )
        )

        if not models:
            return f"No models found for author {author}."

        model = models[0]

        return (
            f"The most downloaded model by {author} "
            f"is {model.id} with "
            f"{model.downloads:,} downloads."
        )

    except Exception as error:
        return (
            f"Error fetching models for {author}: {error}"
        )


hub_stats_tool = FunctionTool.from_defaults(
    get_hub_stats
)
```

### LangChain implementation

```python
from langchain_core.tools import Tool
from huggingface_hub import list_models


def get_hub_stats(author: str) -> str:
    """Fetch the most downloaded model for an author."""

    try:
        models = list(
            list_models(
                author=author,
                sort="downloads",
                direction=-1,
                limit=1
            )
        )

        if not models:
            return f"No models found for author {author}."

        model = models[0]

        return (
            f"The most downloaded model by {author} "
            f"is {model.id} with "
            f"{model.downloads:,} downloads."
        )

    except Exception as error:
        return (
            f"Error fetching models for {author}: {error}"
        )


hub_stats_tool = Tool(
    name="get_hub_stats",
    func=get_hub_stats,
    description=(
        "Fetches the most downloaded model from "
        "a specified Hugging Face Hub author."
    )
)
```

The implementations follow the same pattern: query the Hub, extract the useful information, and return a string that the agent can incorporate into its response.

---

# Part 3: Integrating Alfred with an Agent Framework

## 15. Integrating Tools with smolagents

Now that the tools exist, we can create Alfred as a `CodeAgent`.

A `CodeAgent` can use Python code to coordinate its tools and solve a task.

```python
from smolagents import CodeAgent, InferenceClientModel

# Initialize the model
model = InferenceClientModel()

# Create Alfred with the available tools
alfred = CodeAgent(
    tools=[
        guest_info_tool,
        search_tool,
        weather_info_tool,
        hub_stats_tool
    ],
    model=model
)

# Ask a question
response = alfred.run(
    "What is Facebook and what is their most popular model?"
)

print("Alfred's Response:")
print(response)
```

### What happens here?

1. `InferenceClientModel()` initializes the model interface.
2. `CodeAgent()` receives the available tools.
3. The agent interprets the user's question.
4. It determines that the Hub statistics tool is useful for finding the most downloaded model.
5. It can use web search if it needs additional information about Facebook.
6. It combines the available evidence into a final answer.

The exact response depends on the model and live tool results.

## 16. Integrating Tools with LlamaIndex

LlamaIndex offers `AgentWorkflow.from_tools_or_functions()` to create an agent workflow from tools or callable functions.

```python
from llama_index.core.agent.workflow import AgentWorkflow
from llama_index.llms.huggingface_api import HuggingFaceInferenceAPI

# Initialize the LLM
llm = HuggingFaceInferenceAPI(
    model_name="Qwen/Qwen2.5-Coder-32B-Instruct"
)

# Create Alfred
alfred = AgentWorkflow.from_tools_or_functions(
    [
        guest_info_tool,
        search_tool,
        weather_info_tool,
        hub_stats_tool
    ],
    llm=llm
)

# Ask a question
response = await alfred.run(
    "What is Facebook and what is their most popular model?"
)

print(response)
```

The `await` keyword is used because this workflow call is asynchronous.

In a regular Python script, place this call inside an asynchronous function and run that function with `asyncio.run()`. In a notebook that already supports top-level `await`, you can execute the call directly.

LlamaIndex's agent interfaces and response objects can vary between versions. Inspect the returned object when extracting a final text response rather than assuming every version returns the same structure.

## 17. Integrating Tools with LangGraph

LangGraph provides a graph-based way to coordinate an LLM and its tools.

Instead of hiding the entire execution flow inside one agent abstraction, we explicitly define:

- The state that stores the conversation.
- The assistant node that calls the LLM.
- The tools node that executes tool calls.
- The edges that control how execution moves between nodes.

First, create the model and bind the tools.

```python
from typing import TypedDict, Annotated

from langchain_core.messages import AnyMessage, HumanMessage
from langchain_huggingface import HuggingFaceEndpoint, ChatHuggingFace
from langgraph.graph import START, StateGraph
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode, tools_condition

# Initialize the model endpoint
llm = HuggingFaceEndpoint(
    repo_id="Qwen/Qwen2.5-Coder-32B-Instruct",
    huggingfacehub_api_token=HUGGINGFACEHUB_API_TOKEN
)

chat = ChatHuggingFace(
    llm=llm,
    verbose=True
)

# All tools available to the agent
tools = [
    guest_info_tool,
    search_tool,
    weather_info_tool,
    hub_stats_tool
]

# Enable tool calling
chat_with_tools = chat.bind_tools(tools)
```

`HUGGINGFACEHUB_API_TOKEN` should be supplied securely through an environment variable or secret manager. Do not hard-code a real access token into source code.

### Define the agent state

```python
class AgentState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
```

The state stores the messages exchanged during the interaction.

`AnyMessage` allows the list to contain different message types, including human messages, AI messages, and tool messages.

`add_messages` tells LangGraph how to update the message history when a node returns additional messages.

### Define the assistant node

```python
def assistant(state: AgentState):
    response = chat_with_tools.invoke(
        state["messages"]
    )

    return {
        "messages": [response]
    }
```

The assistant sends the current message history to the LLM.

The model may return a normal answer or an AI message containing a tool call. LangGraph uses the message to determine what should happen next.

### Build the graph

```python
builder = StateGraph(AgentState)

# Nodes perform the work
builder.add_node("assistant", assistant)
builder.add_node("tools", ToolNode(tools))

# Start with the assistant
builder.add_edge(START, "assistant")

# Route according to whether the assistant requests a tool
builder.add_conditional_edges(
    "assistant",
    tools_condition
)

# After a tool executes, call the assistant again
builder.add_edge("tools", "assistant")

# Compile the graph
alfred = builder.compile()
```

### Invoke Alfred

```python
messages = [
    HumanMessage(
        content=(
            "Who is Lady Ada Lovelace, "
            "and what is her background?"
        )
    )
]

response = alfred.invoke({
    "messages": messages
})

print(response["messages"][-1].content)
```

### Understand the execution flow

```text
             START
               |
               v
         +-----------+
         | Assistant |
         +-----------+
               |
               v
       Does it request
          a tool?
          /     \
        Yes      No
         |        |
         v        v
    +---------+  END
    |  Tools  |
    +---------+
         |
         v
    +-----------+
    | Assistant |
    +-----------+
         |
         v
   Check the response
```

The `tools_condition` function routes execution to the tools node when the assistant message contains a tool call. Otherwise, the graph reaches its terminal path.

After the tools run, their results are added to the message history. The assistant is called again and can use those results to generate a final answer or request another tool.

This loop is an important part of tool-using agents.

---

# Part 4: Building Alfred's Complete Application

## 18. Keep the Tools and Retriever in Separate Modules

After testing each component independently, place the code into the project modules.

For example, `retriever.py` can expose a function that loads the guest dataset and returns the initialized guest tool.

```python
# retriever.py

import datasets

from langchain_core.documents import Document
from langchain_community.retrievers import BM25Retriever
from smolagents import Tool


class GuestInfoRetrieverTool(Tool):
    name = "guest_info_retriever"

    description = (
        "Retrieves detailed information about gala guests "
        "based on their name or relation."
    )

    inputs = {
        "query": {
            "type": "string",
            "description": "A guest name or relationship."
        }
    }

    output_type = "string"

    def __init__(self, docs):
        super().__init__()
        self.retriever = BM25Retriever.from_documents(docs)

    def forward(self, query: str) -> str:
        results = self.retriever.invoke(query)

        if not results:
            return "No matching guest information found."

        return "\n\n".join(
            doc.page_content for doc in results[:3]
        )


def load_guest_dataset():
    guest_dataset = datasets.load_dataset(
        "agents-course/unit3-invitees",
        split="train"
    )

    docs = [
        Document(
            page_content="\n".join([
                f"Name: {guest['name']}",
                f"Relation: {guest['relation']}",
                f"Description: {guest['description']}",
                f"Email: {guest['email']}"
            ]),
            metadata={"name": guest["name"]}
        )
        for guest in guest_dataset
    ]

    return GuestInfoRetrieverTool(docs)
```

The `load_guest_dataset()` function loads the data, constructs the documents, and returns a usable retrieval tool.

The application can now initialize the tool without duplicating the dataset preparation logic.

## 19. Create the Tools Module

The following is a simplified `tools.py` for smolagents. It keeps the external capabilities in one place.

```python
# tools.py

import random

from huggingface_hub import list_models
from smolagents import DuckDuckGoSearchTool, Tool


class WeatherInfoTool(Tool):
    name = "weather_info"

    description = (
        "Returns simulated weather information "
        "for a given location."
    )

    inputs = {
        "location": {
            "type": "string",
            "description": "Location to check."
        }
    }

    output_type = "string"

    def forward(self, location: str) -> str:
        conditions = [
            {"condition": "Rainy", "temp_c": 15},
            {"condition": "Clear", "temp_c": 25},
            {"condition": "Windy", "temp_c": 20}
        ]

        data = random.choice(conditions)

        return (
            f"Simulated weather in {location}: "
            f"{data['condition']}, {data['temp_c']}°C"
        )


class HubStatsTool(Tool):
    name = "hub_stats"

    description = (
        "Finds the most downloaded model from "
        "a Hugging Face Hub author or organization."
    )

    inputs = {
        "author": {
            "type": "string",
            "description": "Author or organization name."
        }
    }

    output_type = "string"

    def forward(self, author: str) -> str:
        try:
            models = list(
                list_models(
                    author=author,
                    sort="downloads",
                    direction=-1,
                    limit=1
                )
            )

            if not models:
                return f"No models found for author {author}."

            model = models[0]

            return (
                f"The most downloaded model by {author} "
                f"is {model.id} with "
                f"{model.downloads:,} downloads."
            )

        except Exception as error:
            return f"Error fetching model statistics: {error}"


# Initialize the tools
search_tool = DuckDuckGoSearchTool()
weather_info_tool = WeatherInfoTool()
hub_stats_tool = HubStatsTool()
```

The tools can now be imported into `app.py`.

## 20. Assemble Alfred

```python
# app.py

from smolagents import CodeAgent, InferenceClientModel

from tools import (
    search_tool,
    weather_info_tool,
    hub_stats_tool
)

from retriever import load_guest_dataset


# Initialize the model
model = InferenceClientModel()

# Load the guest retrieval tool
guest_info_tool = load_guest_dataset()

# Assemble Alfred
alfred = CodeAgent(
    tools=[
        guest_info_tool,
        weather_info_tool,
        hub_stats_tool,
        search_tool
    ],
    model=model,
    add_base_tools=True,
    planning_interval=3
)


# Example interaction
query = (
    "Tell me about Lady Ada Lovelace "
    "and help me prepare a conversation about computing."
)

response = alfred.run(query)

print("Alfred's Response:")
print(response)
```

Here, `planning_interval=3` configures the agent to perform planning periodically during execution, while `add_base_tools=True` enables the framework's additional base tools.

The exact supported options depend on the installed version of smolagents.

**Important:** This is an example of the smolagents version of the application. The LlamaIndex and LangGraph versions use their own tool wrappers and execution APIs. Do not mix tool objects from one framework into another unless they are explicitly adapted to the receiving framework.

---

# Part 5: End-to-End Examples

## 21. Example 1 — Retrieve Guest Information

**User:**

> Tell me about Lady Ada Lovelace.

Alfred searches the guest dataset and uses the retrieved record to prepare an answer.

An illustrative answer might describe Ada Lovelace's work in mathematics and computing, her association with Charles Babbage's Analytical Engine, and other details stored in the guest record.

The actual answer depends on the information returned by the dataset and the model's response.

This example demonstrates the role of retrieval: the agent can use event-specific information that was not necessarily available during LLM training.

## 22. Example 2 — Check Weather for Fireworks

**User:**

> What's the weather like in Paris tonight? Will it be suitable for our fireworks display?

Alfred can call the weather tool and use its result to prepare an answer.

For example, the simulated tool may return:

```text
Simulated weather in Paris: Rainy, 15°C
```

Alfred could then explain that the simulated result indicates rain.

Remember that this example does not access a real forecast. A production application must use a weather API that returns current or forecast conditions for the relevant time and location.

## 23. Example 3 — Find a Popular Hugging Face Model

**User:**

> What is Facebook, and what's their most popular model?

Alfred can use web search for general information about Facebook and the Hub statistics tool for the model ranking.

The Hub tool returns the most downloaded model found for the specified author or organization, along with the reported download count.

Alfred can combine the two results into one response.

This is an example of using multiple information sources to answer different parts of the same question.

## 24. Example 4 — Combine Guest Retrieval and Web Search

Consider a more complex request.

**User:**

> I need to speak with Dr. Nikola Tesla about recent advancements in wireless energy. Can you help me prepare?

Alfred can break this into smaller tasks.

**Step 1: Retrieve guest information**

The guest retriever may return the guest's relationship with the host, background, and interests.

**Step 2: Search for recent information**

The web-search tool retrieves information about wireless power transmission and related developments.

**Step 3: Combine the information**

The LLM can prepare conversation starters that connect the guest's background with the retrieved research.

An illustrative response might contain:

- A short summary of the guest's stored background.
- A summary of recent wireless-energy developments from the search results.
- Questions the host could ask about the subject.
- Conversation starters based on interests explicitly present in the guest record.

This example illustrates a central advantage of Agentic RAG: the agent can combine private, event-specific information with external information to solve a broader task.

**Data-quality note:** The guest records used in this learning scenario are demonstration data. Information returned by the dataset should not be interpreted as verified facts about real people without independent confirmation.

---

# Part 6: Conversation Memory

## 25. Why Does Alfred Need Memory?

Imagine this conversation:

**User:** Tell me about Lady Ada Lovelace.

**Alfred:** Retrieves and summarizes the relevant guest information.

**User:** What projects is she currently working on?

The second question uses the pronoun *she*. Alfred needs the previous conversation to understand that the user is referring to Lady Ada Lovelace.

Without conversation context, the agent might not know which person the user means.

Conversation memory helps the agent maintain context across multiple interactions.

However, conversation memory is different from RAG:

- **RAG** retrieves information from an external knowledge source.
- **Conversation memory** preserves or retrieves information from previous interactions.
- **Agent state** stores the information needed while a particular execution or conversation proceeds.

A system can use all three.

## 26. Memory with smolagents

In the smolagents example, you can preserve execution context by setting `reset=False` for a later call.

```python
from smolagents import CodeAgent, InferenceClientModel

model = InferenceClientModel()

alfred_with_memory = CodeAgent(
    tools=[
        guest_info_tool,
        weather_info_tool,
        hub_stats_tool,
        search_tool
    ],
    model=model,
    add_base_tools=True,
    planning_interval=3
)

# First interaction
response1 = alfred_with_memory.run(
    "Tell me about Lady Ada Lovelace."
)

print("First response:")
print(response1)

# Second interaction using the previous execution context
response2 = alfred_with_memory.run(
    "What projects is she currently working on?",
    reset=False
)

print("Second response:")
print(response2)
```

The second call is configured not to reset the agent's execution context.

The agent may use the earlier conversation to understand the reference to Ada Lovelace, then use the retrieval or search tools to gather additional information.

Preserving execution context does not guarantee that the model has current information about someone's projects. That information must still be retrieved from an appropriate source.

## 27. Memory with LlamaIndex

LlamaIndex workflows can use a `Context` object to preserve state.

```python
from llama_index.core.agent.workflow import AgentWorkflow
from llama_index.core.workflow import Context

alfred = AgentWorkflow.from_tools_or_functions(
    [
        guest_info_tool,
        search_tool,
        weather_info_tool,
        hub_stats_tool
    ],
    llm=llm
)

# Create a context for the workflow
ctx = Context(alfred)

# First interaction
response1 = await alfred.run(
    "Tell me about Lady Ada Lovelace.",
    ctx=ctx
)

print("First response:")
print(response1)

# Second interaction with the same context
response2 = await alfred.run(
    "What projects is she currently working on?",
    ctx=ctx
)

print("Second response:")
print(response2)
```

The same context object is passed into both workflow calls.

That allows the workflow to preserve relevant state between those interactions according to its configured memory behavior.

The exact response object and context behavior can vary across LlamaIndex versions, so consult the installed version's API when implementing this in a project.

## 28. Memory with LangGraph

In LangGraph, you can pass previous messages into the next invocation. The message history becomes part of the next input state.

```python
from langchain_core.messages import HumanMessage

# First interaction
response = alfred.invoke({
    "messages": [
        HumanMessage(
            content=(
                "Tell me about Lady Ada Lovelace. "
                "What's her background?"
            )
        )
    ]
})

print("First response:")
print(response["messages"][-1].content)

# Continue with the earlier messages
response = alfred.invoke({
    "messages": response["messages"] + [
        HumanMessage(
            content="What projects is she currently working on?"
        )
    ]
})

print("Second response:")
print(response["messages"][-1].content)
```

This example explicitly sends the previous messages back into the graph.

It demonstrates conversational context within the provided message history. It does not, by itself, create persistent storage across application restarts or independent sessions.

For longer-running applications, LangGraph also supports dedicated memory and checkpointing approaches. Those approaches can preserve conversation state without requiring the application to manually reconstruct the full history on every invocation.

### Comparing the three approaches

| Framework | Memory approach shown here |
|---|---|
| smolagents | Preserve execution context with `reset=False` |
| LlamaIndex | Reuse a workflow `Context` object |
| LangGraph | Pass previous messages into the next state, or use checkpointing |

These frameworks manage execution and memory differently. The common goal is to provide the agent with enough relevant context to interpret follow-up questions.

---

# Part 7: Improving the Agentic RAG System

## 29. Improve the Retriever

BM25 is a useful baseline, but a larger or more complex dataset may need a more sophisticated retrieval system.

Possible improvements include:

- Embedding-based semantic retrieval.
- Hybrid search that combines keyword and semantic matching.
- Filtering by document metadata.
- Reranking retrieved documents.
- Returning only the most relevant fields.
- Evaluating retrieval quality using a representative set of questions.

For example, a semantic retriever may be more effective when a question describes a guest's interests using words that differ from the wording in the stored biography.

## 30. Add Conversation Starters

The current guest tool retrieves information. It could be extended to return conversation starters based on the guest's interests or background.

For example, if a guest record mentions computing, the agent could suggest a question about a computing project.

However, conversation starters should be grounded in retrieved information. The agent should not invent personal interests and present them as facts.

A useful extension is to return both the relevant guest record and a clearly labeled set of suggested questions.

## 31. Add Web Search for Unfamiliar Guests

The guest dataset may not contain every detail needed for a question.

Alfred can use web search to supplement the guest record when the user asks about recent projects, public achievements, or new research.

A robust implementation should distinguish between:

- Details retrieved from the event's guest database.
- Information found on the web.
- The model's own interpretation or suggestions.

This helps prevent external information or model-generated assumptions from being mistaken for verified guest records.

## 32. Integrate Multiple Knowledge Sources

A larger system could use separate retrieval sources for:

- Guest biographies.
- Event schedules.
- Menu and dietary requirements.
- Venue information.
- Guest preferences.
- Verified public biographies and publications.

The agent could select the relevant source according to the question.

For example, a question about a guest's dietary requirements should use the appropriate event record, while a question about a researcher's recent publication may require a separate, current source.

## 33. Add Real Weather Data

The current weather tool uses random, simulated values. To make Alfred useful for an actual event, replace the dummy implementation with a real weather API.

A production tool should consider:

- The correct location.
- The relevant forecast period.
- Precipitation and wind conditions.
- Weather warnings.
- API errors and missing data.
- How recently the forecast was updated.

Alfred should communicate when the information is unavailable rather than making up a forecast.

## 34. Improve Reliability and Safety

A production Agentic RAG application should also include:

- Input validation.
- Error handling for external APIs.
- Timeouts and retry limits.
- Retrieval quality checks.
- Logging and tracing.
- Evaluation using realistic questions.
- Protection for private guest data.
- Access controls for sensitive information.
- Clear limits on the actions tools can perform.

For example, an agent should not expose private email addresses to unauthorized users simply because the guest retriever can retrieve them.

Similarly, a search result should not be treated as authoritative just because a tool returned it.

---

# Part 8: Common Mistakes

## 35. Confusing RAG with Agentic RAG

A basic RAG pipeline retrieves documents and supplies them to an LLM.

Agentic RAG adds decision-making about which retrieval or other tool to use and how to coordinate the results.

Both approaches are useful. An agent is not automatically better for every task; a simple RAG pipeline may be sufficient when the retrieval workflow is predictable.

## 36. Assuming BM25 Understands Meaning Like an LLM

BM25 is a lexical retrieval algorithm. It ranks documents based on terms and their distribution, rather than directly comparing the semantic meaning of sentences through embeddings.

It can work well for name-based and keyword-based searches, but it may struggle with paraphrases and conceptually related phrases.

## 37. Treating Simulated Weather as a Forecast

The sample weather tool randomly chooses a predefined condition. It does not inspect the weather in the specified location.

Its output should never be used to make real event-safety decisions.

## 38. Assuming Tool Results Are Always Correct

A retriever can return irrelevant documents. A search engine can return outdated or misleading pages. A model-statistics API can fail or return data that changes over time.

The agent should use the tool results appropriately and communicate uncertainty when necessary.

## 39. Mixing Framework APIs

smolagents, LlamaIndex, and LangGraph have different interfaces for tool registration, agent execution, state, and memory.

For example, a smolagents `Tool` should not be assumed to work directly as a LangChain or LlamaIndex tool.

When switching frameworks, adapt the tool interface and test the execution flow.

## 40. Treating Conversation Context as Permanent Memory

Passing previous messages into the next invocation provides context for that invocation. It does not automatically create durable storage across application restarts.

For persistent memory, the application needs an appropriate storage or checkpointing mechanism.

---


# Final Takeaways

By building Alfred, we've explored how to combine retrieval and tools to create a more capable AI agent.

The main concepts are:

1. **RAG** provides the LLM with relevant information retrieved from an external knowledge source.
2. **Agentic RAG** allows an agent to choose retrieval strategies and other tools according to the user's request.
3. **BM25** provides a straightforward keyword-oriented retrieval baseline without requiring embeddings.
4. **Custom tools** expose capabilities such as guest retrieval, web search, weather information, and model statistics.
5. **smolagents, LlamaIndex, and LangGraph** offer different approaches to connecting an LLM with tools.
6. **Conversation memory** helps the agent interpret follow-up questions, while persistent memory requires an appropriate state-management mechanism.
7. **Reliable Agentic RAG** requires relevant data, sound tool selection, error handling, evaluation, and clear boundaries around private or uncertain information.

Alfred's gala is a teaching scenario, but the architecture applies to practical projects: customer-support assistants, internal company knowledge systems, research assistants, document-analysis tools, and AI applications that need to coordinate several sources of information.

The most important lesson is that an agent becomes useful not simply by having many tools, but by selecting appropriate tools, retrieving relevant information, and using the results responsibly to answer the user's actual question.
