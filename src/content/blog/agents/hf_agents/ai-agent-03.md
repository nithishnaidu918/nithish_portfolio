---
title: "Mastering LlamaIndex, RAG, Tools, Agents & Workflows"
description: "A practical guide to LlamaIndex covering components, document ingestion, embeddings, vector stores, QueryEngine, tools, AgentWorkflow, multi-agent systems, asynchronous Python, and event-driven workflows."
category: "AI Agents"
tags: ["Agentic AI", "LlamaIndex", "RAG", "Agentic RAG", "LLM Agents", "Python", "Vector Databases", "Embeddings", "Workflows"]
---

# AI Agent Journey #3 — Mastering LlamaIndex, RAG, Tools, Agents & Workflows

## 1. Introduction to LlamaIndex

In the previous stages of our AI Agent Journey, we explored how agents use tools, retrieve information, and coordinate tasks. Now, we will learn another important framework for building LLM-powered applications: **LlamaIndex**.

LlamaIndex is a toolkit for building applications and agents that work with your own data. It provides components for loading documents, creating indexes, retrieving relevant information, connecting tools, and coordinating workflows.

Imagine Alfred, the helpful butler responsible for organizing a dinner party at Wayne Manor.

Alfred needs to know:

- Which guests are attending?
- What dietary preferences do they have?
- What menus worked well at previous events?
- What does the calendar say about the schedule?
- Which information should he retrieve before making a recommendation?

A language model alone may not have access to this private information. LlamaIndex helps connect the model to relevant data and provides a framework for deciding how that data should be used.

### 1.1 The three major areas of LlamaIndex

In this learning journey, we will focus on three major areas.

**1. Components**

Components are the building blocks of an LLM application. They include language models, embedding models, document loaders, indexes, retrievers, query engines, and other reusable elements.

**2. Agents and tools**

Tools give an agent specific capabilities, such as calculating a value, searching a database, or querying a knowledge base. An agent uses its model to decide which tools to use to accomplish a task.

**3. Workflows**

Workflows organize application logic into steps connected through events. They are useful when we need explicit control over the sequence of operations, branching, loops, and shared state.

These areas work together:

```text
                  User request
                       |
                       v
                  LlamaIndex
                       |
          +------------+------------+
          |            |            |
          v            v            v
      Components     Agents      Workflows
          |            |            |
          v            v            v
     Data, models,   Tool use    Steps, events,
     indexes, RAG    decisions   state, branching
          |            |            |
          +------------+------------+
                       |
                       v
                   Final result
```

The diagram shows the main relationships. In an actual application, a workflow may contain agents, agents may use query engines as tools, and components may be reused in several parts of the system.

## 2. What Makes LlamaIndex Useful?

Several capabilities make LlamaIndex useful for building LLM applications.

### 2.1 Structured workflows

LlamaIndex provides an event-driven workflow system. Instead of putting every operation into one large function, you can separate application logic into individual steps.

Each step can receive an event, perform some work, and emit another event. The emitted event can trigger the next step.

This approach is useful for complex applications that need explicit control over execution.

### 2.2 Document parsing

Real-world documents are not always clean text files. PDFs may contain tables, multiple columns, scanned pages, and complex layouts.

LlamaIndex provides integrations for document loading and parsing, including a managed parsing service called LlamaParse.

Advanced parsing can be useful when document structure matters. Availability and pricing depend on the service and plan being used.

### 2.3 Reusable components

LlamaIndex has a broad collection of integrations for models, embedding systems, vector stores, data readers, retrievers, and other tools.

This makes it possible to combine components instead of implementing every part of an LLM application from scratch.

### 2.4 Integration registry

LlamaHub is a registry of integrations, tools, and data connectors that can be used with LlamaIndex.

For example, an application might need:

- A language model hosted by an inference provider
- An embedding model
- A vector database
- A document reader
- A tool for interacting with an external service

These capabilities are often distributed across separate Python packages. Finding and installing the correct integration is an important part of building the application.

---

# Part 1 — Components and Retrieval-Augmented Generation

## 3. What Are Components in LlamaIndex?

A component is a reusable building block that performs a particular function within an LLM application.

Consider a system that answers questions about company documents. It might contain:

| Component | Responsibility |
|---|---|
| Document reader | Loads data from files or other sources |
| Text splitter | Divides documents into smaller chunks |
| Embedding model | Converts text into numerical vectors |
| Vector store | Stores vectors and associated information |
| Index | Organizes data for retrieval |
| Retriever | Finds relevant document chunks |
| Language model | Generates language-based responses |
| Query engine | Coordinates retrieval and response generation |
| Evaluator | Assesses the quality of responses |

These components can be combined into a complete application.

For this learning journey, the most important component is the **QueryEngine**, because it can become a tool that an agent uses to retrieve information from your data.

Before understanding QueryEngine, we need to understand RAG.

## 4. What Is Retrieval-Augmented Generation?

RAG stands for **Retrieval-Augmented Generation**.

Large language models learn general patterns and knowledge during training. However, a model may not know your private documents, recent internal updates, or information that was not included in its training data.

RAG addresses this limitation by retrieving relevant information from an external data source and providing that information to the model when it generates a response.

### 4.1 Example: Alfred's dinner-party assistant

Suppose Alfred receives this question:

> What menu should we prepare for tomorrow's dinner party?

The answer depends on information stored in several places:

- Guest dietary preferences
- Previous menus
- Available ingredients
- Event details

A RAG application can retrieve relevant records and provide them to a language model.

The model can then generate a recommendation using the retrieved context.

### 4.2 The RAG pipeline

```text
User question
     |
     v
Retrieve relevant information
     |
     v
Prepare the retrieved context
     |
     v
Send question and context to the LLM
     |
     v
Generate an answer
```

For example:

**Question:** What food should we prepare for the dinner party?

**Retrieved context:**
- One guest is vegetarian.
- Two guests have requested vegetarian options.
- A previous dinner used a successful vegetable-based menu.

**Generated response:** Prepare a vegetarian main course with suitable side dishes, while checking the remaining guests' dietary requirements.

The answer is grounded in the retrieved information rather than relying only on the model's general knowledge.

RAG does not guarantee that an answer is correct. If the retrieved data is incomplete, outdated, or irrelevant, the generated response may still be wrong.

## 5. The Five Main Stages of a RAG Pipeline

A practical RAG application usually involves five major stages:

1. Loading
2. Indexing
3. Storing
4. Querying
5. Evaluation

Understanding these stages helps you design and debug retrieval systems.

### 5.1 Loading

Loading means bringing data from its original source into the application.

Possible sources include:

- Text files
- PDFs
- Word documents
- Websites
- Databases
- APIs
- Internal knowledge bases

LlamaIndex provides document readers and integrations for many data sources.

A basic way to load files from a local directory is `SimpleDirectoryReader`.

```python
from llama_index.core import SimpleDirectoryReader

reader = SimpleDirectoryReader(
    input_dir="path/to/directory"
)

documents = reader.load_data()

print(f"Loaded {len(documents)} documents")
```

Here:

- `SimpleDirectoryReader` reads supported files from a directory.
- `load_data()` loads the content.
- `documents` contains LlamaIndex `Document` objects.

A `Document` represents a source document and can contain text and associated metadata.

For complex PDFs, ordinary text extraction may not preserve all tables or visual relationships. More advanced document parsing may be required.

### 5.2 Indexing

Indexing organizes data so that it can be searched efficiently.

In many RAG applications, this involves creating embeddings for document chunks.

An embedding is a numerical vector that represents aspects of the meaning of a piece of text.

For example, a system might create vectors for these sentences:

- "A guest prefers vegetarian meals."
- "The visitor does not eat meat."
- "The car is painted red."

The first two sentences express similar ideas even though their wording differs. A suitable embedding model may represent them as vectors that are closer together than either is to the third sentence.

This makes semantic retrieval possible.

An index is not necessarily just a vector database. It is a data structure or organization strategy that supports searching and retrieving information.

### 5.3 Storing

Once the data has been processed and indexed, you often want to store the result.

Without persistence, you may have to repeat document loading, splitting, and embedding generation whenever the application restarts.

A persistent vector store can retain embeddings and associated metadata so that they can be reused.

### 5.4 Querying

Querying means using the indexed data to answer questions or retrieve relevant information.

LlamaIndex supports different query interfaces, including:

- `as_retriever()`
- `as_query_engine()`
- `as_chat_engine()`

We will explore these interfaces later.

### 5.5 Evaluation

Evaluation measures how well the retrieval system and its generated answers perform.

For example, you might check:

- Whether the retrieved passages contain relevant information
- Whether the answer is supported by the retrieved context
- Whether the response addresses the user's question
- Whether the response is factually correct
- How long the request takes

Evaluation becomes especially important when changing the embedding model, chunk size, retrieval strategy, or language model.

---

## 6. Loading and Splitting Documents

After loading documents, we usually divide them into smaller pieces called **Nodes**.

### 6.1 What is a Node?

A Node is a smaller unit of content derived from a document.

For example, a 30-page document might be divided into hundreds of nodes. Each node contains a portion of the original text and can retain metadata or references to its source document.

Why not simply send the entire document to the LLM?

Because large documents can:

- Exceed the model's context limit
- Increase token usage
- Contain information unrelated to the question
- Make retrieval less precise

Smaller chunks help the system retrieve the portions most relevant to a particular query.

### 6.2 Use SentenceSplitter

LlamaIndex provides `SentenceSplitter`, which splits documents into chunks while trying to respect natural sentence boundaries.

```python
from llama_index.core.node_parser import SentenceSplitter

splitter = SentenceSplitter(
    chunk_size=512,
    chunk_overlap=50,
)

nodes = splitter.get_nodes_from_documents(documents)

print(f"Created {len(nodes)} nodes")
```

The important parameters are:

- `chunk_size`: the target size of each chunk, measured in tokens by this splitter.
- `chunk_overlap`: the amount of overlapping content between neighboring chunks.

**Why use overlap?**

Suppose one chunk ends in the middle of an explanation and the next chunk continues that explanation. A small overlap can preserve context across the boundary.

Too little overlap can separate related information. Too much overlap can create unnecessary duplication.

The appropriate chunk size and overlap depend on the type of data, the embedding model, and the questions the application needs to answer.

### 6.3 Generate embeddings with an ingestion pipeline

LlamaIndex provides `IngestionPipeline` to apply transformations to documents.

A simplified example is:

```python
from llama_index.core import Document
from llama_index.core.ingestion import IngestionPipeline
from llama_index.core.node_parser import SentenceSplitter
from llama_index.embeddings.huggingface import HuggingFaceEmbedding

pipeline = IngestionPipeline(
    transformations=[
        SentenceSplitter(
            chunk_overlap=0
        ),
        HuggingFaceEmbedding(
            model_name="BAAI/bge-small-en-v1.5"
        ),
    ]
)

documents = [
    Document(
        text=(
            "LlamaIndex helps applications retrieve "
            "information from private documents."
        )
    )
]

nodes = await pipeline.arun(documents=documents)

print(f"Created {len(nodes)} nodes")
```

This example demonstrates the main idea: documents are transformed into nodes and the embedding model produces vector representations for those nodes.

The pipeline is asynchronous, so run it inside an environment that supports `await`, such as an asynchronous function or a notebook cell with asynchronous execution support.

The first time an embedding model is used, its model files may need to be downloaded. The required packages must also be installed.

---

## 7. Understanding Embeddings

Embeddings are one of the central ideas behind modern semantic retrieval.

Consider these two questions:

1. "What meals can we serve a vegetarian guest?"
2. "Which food options are suitable for someone who does not eat meat?"

The wording is different, but the meaning is similar.

A keyword-only search might miss some relevant matches if the documents use different terminology. Embedding-based retrieval can identify semantically similar text even when the exact words differ.

### 7.1 How semantic retrieval works

A typical semantic retrieval process is:

1. Convert each document chunk into an embedding vector.
2. Store the vectors in a vector store.
3. Convert the user's query into an embedding using the same compatible embedding model.
4. Compare the query vector with stored vectors.
5. Retrieve the most relevant chunks.

```text
Document chunks
      |
      v
Embedding model
      |
      v
Stored document vectors
      |
      |             User query
      |                  |
      |                  v
      |            Embedding model
      |                  |
      +--------+---------+
               |
               v
       Similarity search
               |
               v
       Relevant document chunks
```

### 7.2 Why use the same embedding model?

The document vectors and query vector must be compatible.

If the system embeds documents with one model and queries with an incompatible model, the resulting vectors may not be meaningful to compare.

For this reason, the ingestion and retrieval pipelines should use a compatible embedding setup.

---

## 8. Storing Vectors with ChromaDB

A vector store retains embeddings and related data so that the application can retrieve relevant information later.

In this example, we use ChromaDB.

Install the LlamaIndex Chroma integration and ChromaDB:

```bash
pip install llama-index-vector-stores-chroma chromadb
```

Create a persistent collection:

```python
import chromadb

from llama_index.vector_stores.chroma import ChromaVectorStore

db = chromadb.PersistentClient(
    path="./alfred_chroma_db"
)

chroma_collection = db.get_or_create_collection(
    "alfred"
)

vector_store = ChromaVectorStore(
    chroma_collection=chroma_collection
)
```

Let's understand the code.

- `PersistentClient` creates a Chroma client that persists its data in the specified directory.
- `get_or_create_collection()` retrieves an existing collection or creates it if it does not exist.
- `ChromaVectorStore` connects ChromaDB to LlamaIndex's vector-store interface.

The path `./alfred_chroma_db` is a local directory relative to the current working directory.

### 8.1 Attach the vector store to the ingestion pipeline

```python
from llama_index.core.ingestion import IngestionPipeline
from llama_index.core.node_parser import SentenceSplitter
from llama_index.embeddings.huggingface import HuggingFaceEmbedding

embed_model = HuggingFaceEmbedding(
    model_name="BAAI/bge-small-en-v1.5"
)

pipeline = IngestionPipeline(
    transformations=[
        SentenceSplitter(
            chunk_size=512,
            chunk_overlap=50,
        ),
        embed_model,
    ],
    vector_store=vector_store,
)

nodes = await pipeline.arun(
    documents=documents
)
```

The pipeline splits the documents, generates embeddings, and writes the processed nodes into the configured vector store.

In a real project, use consistent ingestion settings and avoid accidentally duplicating records every time the pipeline runs. Depending on your data source and ingestion strategy, you may need stable document IDs, deduplication, and update handling.

### 8.2 Create a VectorStoreIndex

Once the vector store contains data, create a LlamaIndex index that uses it.

```python
from llama_index.core import VectorStoreIndex
from llama_index.embeddings.huggingface import HuggingFaceEmbedding

embed_model = HuggingFaceEmbedding(
    model_name="BAAI/bge-small-en-v1.5"
)

index = VectorStoreIndex.from_vector_store(
    vector_store,
    embed_model=embed_model,
)
```

The `VectorStoreIndex` provides an interface for working with the stored vectors and retrieving relevant information.

The vector store persists the underlying records. The index object provides the LlamaIndex interface used by the application.

---

## 9. Querying an Index

Once the index is ready, we can use it in different ways.

### 9.1 `as_retriever()`

A retriever focuses on finding relevant information.

```python
retriever = index.as_retriever(
    similarity_top_k=3
)

results = retriever.retrieve(
    "What are the dietary preferences?"
)

for result in results:
    print(result.score)
    print(result.node.text)
```

A retriever typically returns `NodeWithScore` objects.

Each result can contain:

- The retrieved node
- A relevance or similarity score, when provided by the retrieval implementation

The score is useful for ranking or filtering, but its meaning depends on the underlying retrieval configuration. It should not automatically be interpreted as a calibrated probability that the passage is correct.

### 9.2 `as_query_engine()`

A query engine goes beyond retrieving passages. It combines retrieval with response generation.

```python
from llama_index.llms.huggingface_api import HuggingFaceInferenceAPI

llm = HuggingFaceInferenceAPI(
    model_name="Qwen/Qwen2.5-Coder-32B-Instruct"
)

query_engine = index.as_query_engine(
    llm=llm,
    similarity_top_k=3,
)

response = query_engine.query(
    "What information is available about the dinner party?"
)

print(response)
```

The query engine retrieves relevant content and uses the configured language model to produce a response.

The selected model must be available through the configured inference provider. Authentication and provider settings may be required.

### 9.3 `as_chat_engine()`

A chat engine supports conversational interaction and can use chat history alongside retrieved information.

For example:

```text
User: What food did we serve last time?

Assistant: The records mention a vegetarian menu.

User: Can we prepare something similar?

Assistant: I can use the previous menu information to suggest
a similar option, while checking the current guests' preferences.
```

A chat engine can use conversation history to interpret follow-up questions such as "something similar."

This does not mean every chat engine automatically remembers every conversation forever. Memory behavior depends on the configuration and the context supplied to it.

### 9.4 Choosing the right interface

| Interface | Main purpose |
|---|---|
| `as_retriever()` | Retrieve relevant nodes |
| `as_query_engine()` | Retrieve information and generate an answer |
| `as_chat_engine()` | Support conversational interactions with chat context |

Use a retriever when you need the underlying passages. Use a query engine when you want a generated response. Use a chat engine when the application needs conversational behavior.

---

## 10. Response Synthesis Strategies

A query engine does more than retrieve nodes and pass them to a model. It also needs a strategy for combining the retrieved information into a response.

LlamaIndex uses a `ResponseSynthesizer` to manage this stage.

Three commonly discussed strategies are `refine`, `compact`, and `tree_summarize`.

### 10.1 Refine

The refine strategy develops an answer by processing retrieved chunks sequentially.

Conceptually:

1. Generate an initial answer from the first chunk.
2. Provide the next chunk and refine the answer.
3. Continue processing additional chunks.
4. Return the final refined response.

This approach can incorporate information from multiple chunks, but it may require many model calls.

### 10.2 Compact

The compact strategy attempts to combine retrieved chunks into larger context groups before processing them.

This can reduce the number of model calls compared with processing every chunk independently.

It is useful when you want to provide the model with a consolidated context while keeping the response-generation process manageable.

### 10.3 Tree summarize

The tree-summarize strategy summarizes information in stages.

Conceptually, information from smaller groups is summarized and those summaries are combined into higher-level summaries.

This can be useful when dealing with a large amount of retrieved information.

### 10.4 Comparison

| Strategy | Main idea | Consideration |
|---|---|---|
| `refine` | Sequentially update an answer using retrieved chunks | Can require many model calls |
| `compact` | Combine chunks into larger groups before processing | Can reduce the number of calls |
| `tree_summarize` | Build summaries across multiple levels | Useful for combining larger amounts of information |

The best strategy depends on the size of the retrieved context, the model's context window, response quality requirements, and latency constraints.

---

## 11. Evaluating RAG Responses

Retrieval and generation can fail in different ways.

A retriever might return irrelevant documents. A language model might generate a response that is not supported by those documents. A response might be relevant to the question but factually incorrect.

LlamaIndex provides evaluators that help assess these different dimensions.

### 11.1 FaithfulnessEvaluator

This evaluator checks whether the answer is supported by the supplied context.

For example, if a retrieved document says that a guest prefers vegetarian food, an answer claiming that the guest requested seafood is not faithful to that evidence.

### 11.2 AnswerRelevancyEvaluator

This evaluator checks whether the answer addresses the question.

An answer may contain accurate information but still fail to answer what the user asked.

### 11.3 CorrectnessEvaluator

This evaluator assesses whether the answer is correct relative to the evaluation criteria and available reference information.

Correctness and faithfulness are related but not identical. An answer may faithfully repeat a source that is itself incorrect.

### 11.4 Example: Evaluate faithfulness

```python
from llama_index.core.evaluation import FaithfulnessEvaluator

evaluator = FaithfulnessEvaluator(
    llm=llm
)

response = query_engine.query(
    "What information is available about the dinner party?"
)

eval_result = evaluator.evaluate_response(
    response=response
)

print(eval_result.passing)
```

This example evaluates whether the generated response is supported by its context.

The evaluator itself uses a language model, so its judgment can also be imperfect. For important applications, combine model-based evaluation with independent checks and representative test cases.

---

## 12. Observability with LlamaTrace

Evaluation measures response quality. Observability helps developers understand how the system behaves while processing requests.

For example, you might want to investigate:

- Which components were called?
- Which documents were retrieved?
- How many model calls were made?
- How long did retrieval take?
- Where did an error occur?
- Did a change in the pipeline affect performance?

LlamaIndex supports instrumentation through integrations that collect execution traces. The course material demonstrates a tracing setup associated with Arize Phoenix and LlamaTrace.

The following example illustrates the configuration pattern:

```bash
pip install -U llama-index-callbacks-arize-phoenix
```

Configure the required API key through your environment rather than hardcoding a real secret in your source code.

```python
import os
import llama_index

os.environ["PHOENIX_API_KEY"] = os.getenv(
    "PHOENIX_API_KEY",
    "",
)

llama_index.core.set_global_handler(
    "arize_phoenix",
    endpoint=os.getenv(
        "PHOENIX_TRACE_ENDPOINT",
        "YOUR_CONFIGURED_TRACE_ENDPOINT",
    ),
)
```

The exact handler name, endpoint, authentication method, and package configuration depend on the tracing service and installed integration version. Configure these values according to the service you use.

**Important:** Observability may expose prompts, retrieved passages, model outputs, and other application data. Avoid recording sensitive information unnecessarily, and configure appropriate access controls and retention policies.

---

# Part 2 — Tools in LlamaIndex

## 13. What Is a Tool?

A tool is a capability an agent can invoke to perform a specific operation.

Examples include:

- Calculating a result
- Searching a knowledge base
- Querying a database
- Reading a document
- Accessing an external service
- Sending a request to an API

An LLM can explain how to perform a calculation, but a calculator tool can execute the calculation directly. Similarly, an LLM can discuss a document, but a query engine can retrieve relevant passages from a document collection.

Tools connect the model to capabilities beyond text generation.

## 14. The Four Main Tool Categories

This learning module introduces four categories of tools in LlamaIndex.

### 14.1 FunctionTool

`FunctionTool` wraps a Python function so that an agent can call it.

It is suitable for custom operations that you can express as ordinary Python functions.

### 14.2 QueryEngineTool

`QueryEngineTool` wraps a query engine so that an agent can query a knowledge base.

This is especially useful for RAG applications.

An agent can decide when to search the knowledge base instead of querying it for every request.

### 14.3 ToolSpec

A `ToolSpec` groups related tools into a collection.

For example, an integration for a particular service might expose several operations through one tool specification.

### 14.4 Utility tools

Utility tools help manage large or complex tool outputs.

Sometimes an API returns far more information than an LLM needs. Passing the entire result into the model can increase latency, consume tokens, and crowd the context window.

Utility tools can help load, index, and search information on demand.

---

## 15. Creating a FunctionTool

A `FunctionTool` converts a Python function into a tool an agent can use.

Consider this example:

```python
from llama_index.core.tools import FunctionTool


def get_weather(location: str) -> str:
    """Get the weather for a given location."""
    print(f"Getting weather for {location}")

    return f"The weather in {location} is sunny"


weather_tool = FunctionTool.from_defaults(
    get_weather,
    name="my_weather_tool",
    description=(
        "Returns weather information for a given location."
    ),
)

result = weather_tool.call("New York")

print(result)
```

The example returns a fixed message. It does not connect to a real weather service.

Let's understand the important parts.

**Function name**

The Python function name helps identify the operation.

**Type annotations**

The `location: str` annotation communicates the expected input type.

**Docstring**

The docstring describes what the function does.

**Tool name**

The `name` parameter provides an explicit identifier for the tool.

**Tool description**

The `description` explains when the tool should be used.

### 15.1 Why do names and descriptions matter?

When an LLM chooses a tool, it relies on the information provided in the tool schema.

Compare these descriptions:

Weak:

```python
description = "Weather"
```

Better:

```python
description = (
    "Retrieve weather information for a specified location. "
    "Use this tool when the user asks about local weather."
)
```

The second description provides a clearer explanation of the tool's purpose.

Tool descriptions should accurately describe the actual behavior. A tool that returns mock data should not claim to provide live weather information.

### 15.2 Synchronous and asynchronous tools

A synchronous function runs using the ordinary function-call pattern.

An asynchronous function is defined using `async def` and can be awaited.

LlamaIndex supports both synchronous and asynchronous function patterns in appropriate contexts. Choose the pattern that matches the operation and the framework API being used.

For example, network requests and other I/O-bound operations may benefit from asynchronous execution when the underlying libraries support it.

---

## 16. Creating a QueryEngineTool

A `QueryEngineTool` allows an agent to query a LlamaIndex query engine.

This is an important connection between RAG and agentic behavior.

Instead of automatically retrieving documents for every user question, the agent can decide whether querying a particular knowledge base is necessary.

### 16.1 Create the query engine

Assume that `index` is a previously created `VectorStoreIndex` and `llm` is a configured language model.

```python
query_engine = index.as_query_engine(
    llm=llm,
    similarity_top_k=3,
)
```

The `similarity_top_k=3` parameter requests up to three relevant results from the similarity-based retrieval stage.

The precise retrieval behavior depends on the index and its configuration.

### 16.2 Wrap the query engine as a tool

```python
from llama_index.core.tools import QueryEngineTool

query_engine_tool = QueryEngineTool.from_defaults(
    query_engine=query_engine,
    name="dinner_party_knowledge",
    description=(
        "Search the dinner-party knowledge base for information "
        "about guests, preferences, previous menus, and event plans."
    ),
    return_direct=False,
)
```

The important parameters are:

- `query_engine`: the query engine the tool will call.
- `name`: the tool identifier.
- `description`: explains what information the tool can retrieve.
- `return_direct`: controls whether the tool result is returned directly or can be used as part of further agent processing, according to the framework's tool behavior.

The agent can now use the query engine as one of its available tools.

### 16.3 Why is this agentic RAG?

A traditional RAG pipeline might always retrieve documents before generating a response.

An agent with a `QueryEngineTool` can choose between different actions.

For example:

```text
User: What is 2 + 2?

Agent:
The answer is a simple calculation.
A document search may not be necessary.
```

For a different question:

```text
User: What dietary preferences were recorded for our guests?

Agent:
The answer depends on stored information.
Use the dinner-party knowledge tool.
```

The agent selects a tool based on the task. That decision-making capability is a central idea behind agentic RAG.

---

## 17. ToolSpecs: Collections of Related Tools

A `ToolSpec` is a way to group related tools together.

Think of a ToolSpec as a complete toolkit for a particular service or area of work.

For example, a business assistant might use a collection of tools for email-related tasks, while a research assistant might use a collection of tools for a particular research service.

### 17.1 Example: Google-related tools

The course demonstrates a Google integration package.

Install it:

```bash
pip install llama-index-tools-google
```

Then create a tool specification:

```python
from llama_index.tools.google import GmailToolSpec

tool_spec = GmailToolSpec()

tool_spec_list = tool_spec.to_tool_list()
```

`to_tool_list()` converts the specification into a list of tools that can be supplied to an agent.

You can inspect the tool metadata:

```python
[
    (tool.metadata.name, tool.metadata.description)
    for tool in tool_spec_list
]
```

This helps you understand which capabilities are exposed and how they are described.

The precise authentication requirements and available operations depend on the installed integration and its configuration.

**Security note:** Email tools may expose private messages or allow actions that affect other people. Grant only the required permissions and require confirmation for consequential operations.

---

## 18. Model Context Protocol (MCP) Tools

The Model Context Protocol, or MCP, provides a standard way for compatible applications to expose tools and other contextual capabilities to clients.

In an agent system, MCP can help connect a model-driven application to external tools without implementing every integration from scratch.

The LlamaIndex course demonstrates using an MCP tool specification.

Install the integration:

```bash
pip install llama-index-tools-mcp
```

A simplified setup looks like this:

```python
from llama_index.tools.mcp import (
    BasicMCPClient,
    McpToolSpec,
)

mcp_client = BasicMCPClient(
    "YOUR_MCP_SERVER_ENDPOINT"
)

mcp_tool_spec = McpToolSpec(
    client=mcp_client
)
```

The endpoint must point to a compatible MCP server configured for the transport supported by the client.

The course example uses a server exposing an SSE-style endpoint. The exact transports supported by MCP clients and integrations may differ across versions, so verify the server and client compatibility before running the example.

### 18.1 Use MCP tools with an agent

Once the MCP client and tool specification are configured, the general pattern is:

1. Connect to the MCP server.
2. Discover or load the tools it exposes.
3. Convert those capabilities into tools usable by the agent.
4. Supply the tools to the agent.
5. Allow the agent to invoke the tools as needed.

The specific agent-construction code depends on the agent class and the version of the integration.

### 18.2 Why is MCP useful?

Imagine an application that needs to connect to several independent services.

Without a common integration protocol, the application may need custom integration code for every service.

MCP can provide a shared interface for exposing and discovering capabilities, reducing some of that integration work.

However, MCP does not automatically make every tool safe or compatible. The client, server, transport, authentication, and tool schemas still need to be configured correctly.

---

## 19. Utility Tools: Managing Large Outputs

Some tools return a large amount of information.

For example, an API may return thousands of records when the user only needs the five most relevant results.

Passing the entire response to an LLM can:

- Increase token consumption
- Slow down execution
- Fill the context window
- Make relevant information harder to identify

Utility tools help manage this problem.

The course introduces two approaches: `OnDemandToolLoader` and `LoadAndSearchToolSpec`.

### 19.1 OnDemandToolLoader

`OnDemandToolLoader` turns a compatible data loader into a tool that an agent can use.

Conceptually, the tool can:

1. Receive the parameters needed to load data.
2. Load the relevant data when requested.
3. Index or organize the data.
4. Query that data on demand.
5. Return the relevant results.

This is useful when loading and processing data is expensive and should only happen when necessary.

### 19.2 LoadAndSearchToolSpec

`LoadAndSearchToolSpec` takes an existing tool and exposes a loading operation and a separate search operation.

The general pattern is:

1. The loading tool calls the underlying tool.
2. Its output is organized into an index.
3. The search tool receives a query.
4. The search tool retrieves the relevant information from the indexed output.

This separation is useful when the underlying tool returns more data than the agent should process in a single response.

### 19.3 Why this matters

Consider a database tool that returns thousands of meeting records.

Instead of passing every record to the LLM, the system can load the data, index it, and return only the records relevant to the current question.

This is an important design principle for scalable LLM applications: **retrieve and process only the information needed for the current task.**

---

# Part 3 — Agents in LlamaIndex

## 20. What Is an Agent?

An agent is a system that uses an AI model to interact with its environment to accomplish a user-defined objective.

An agent can combine:

- Understanding the user's request
- Selecting appropriate tools
- Deciding which action to take
- Observing tool results
- Choosing a follow-up action
- Producing a final response

For example, Alfred might receive this request:

> Find the previous dinner menu, check the guests' dietary preferences, and suggest a suitable menu for tomorrow.

The agent may need to query a knowledge base, inspect the results, and use the information to produce a recommendation.

The agent is not simply retrieving information. It is deciding which capabilities to use and how to combine their results.

## 21. Three Agent Approaches in LlamaIndex

The course introduces three broad approaches.

### 21.1 Function-calling agents

Function-calling agents use a model that supports structured function or tool calls.

The model receives descriptions of available tools and can return a structured request identifying a tool and its arguments.

For example, the model might request:

```text
Tool: multiply
Arguments:
    a = 5
    b = 3
```

The framework executes the function and provides the result to the agent.

Structured tool calling can be useful because the model does not need to generate an entire executable Python program for each tool interaction.

The selected model must support the function-calling behavior required by the framework.

### 21.2 ReAct agents

ReAct stands for **Reasoning and Acting**.

A ReAct-style agent alternates between deciding what to do, taking an action, observing the result, and continuing as needed.

A simplified cycle looks like this:

```text
Understand task
      |
      v
Choose an action
      |
      v
Execute a tool
      |
      v
Observe the result
      |
      v
Choose the next action
      |
      v
Continue or finish
```

ReAct agents can work with models that support appropriate chat or text-completion behavior, even when the model does not provide a dedicated function-calling API.

They may require more prompting and output parsing than structured function-calling agents.

### 21.3 Advanced custom agents

Advanced agents provide more control over the decision-making process and execution logic.

They are useful when a standard agent loop is insufficient and the application needs custom behavior, specialized state management, or a more complex workflow.

### 21.4 Comparison

| Approach | Main characteristic | Useful when |
|---|---|---|
| Function-calling agent | Uses structured tool-call outputs | The model supports function calling |
| ReAct agent | Alternates between actions and observations | You need a flexible agent loop across compatible text/chat models |
| Advanced custom agent | Allows more specialized execution logic | The task needs custom control flow or state |

These are design approaches rather than a guarantee that one type will always outperform another. Model capabilities, tool quality, and task complexity all matter.

---

## 22. Creating an Agent with AgentWorkflow

The course uses `AgentWorkflow.from_tools_or_functions()` to create an agent from a set of functions or tools.

Let's start with a simple multiplication function.

```python
from llama_index.core.agent.workflow import AgentWorkflow
from llama_index.core.tools import FunctionTool
from llama_index.llms.huggingface_api import HuggingFaceInferenceAPI


def multiply(a: int, b: int) -> int:
    """Multiply two integers and return the result."""
    return a * b


llm = HuggingFaceInferenceAPI(
    model_name="Qwen/Qwen2.5-Coder-32B-Instruct"
)

multiply_tool = FunctionTool.from_defaults(
    multiply
)

agent = AgentWorkflow.from_tools_or_functions(
    [multiply_tool],
    llm=llm,
)
```

The agent has access to a multiplication tool.

When a request requires multiplication, the model can select the tool and supply the arguments.

### 22.1 Run the agent

LlamaIndex's workflow agents commonly use asynchronous execution.

```python
response = await agent.run(
    "What is 12 multiplied by 8?"
)

print(response)
```

The `await` keyword waits for the asynchronous operation to complete.

The framework manages the interaction between the model and its tools.

**Important:** This code assumes the installed LlamaIndex version supports the shown API and that the inference provider is configured correctly.

---

## 23. Stateless Agents and Context

Agents are stateless by default in the course's example. This means a separate call should not automatically be assumed to retain the conversation history of a previous call.

For example:

```python
response = await agent.run(
    "My name is Bob."
)

response = await agent.run(
    "What is my name?"
)
```

Without shared context or another memory mechanism, the second call may not know that the user previously supplied the name.

To retain context across calls, use a `Context` object.

### 23.1 Create and reuse a Context

```python
from llama_index.core.workflow import Context

ctx = Context(agent)

response = await agent.run(
    "My name is Bob.",
    ctx=ctx,
)

response = await agent.run(
    "What was my name again?",
    ctx=ctx,
)

print(response)
```

The shared context allows the workflow to retain relevant state across the two calls, according to the agent's context and memory behavior.

### 23.2 What does Context mean?

A `Context` provides a way to carry workflow state between executions or steps.

It is useful for:

- Conversational history
- Tracking progress
- Storing intermediate results
- Sharing state across workflow operations
- Coordinating multi-step tasks

Context is not automatically the same as permanent database storage. If the application needs durable memory across restarts, it requires an appropriate persistence design.

### 23.3 Understanding async Python

Python's `async` and `await` keywords support asynchronous programming.

A synchronous function runs through the ordinary function-call pattern. An asynchronous function can suspend while waiting for an operation and allow other work to proceed when the surrounding application is designed to support concurrency.

For example:

```python
async def get_information():
    result = await some_async_operation()
    return result
```

The expression `await some_async_operation()` waits for that operation to complete.

An asynchronous function must be called within an appropriate asynchronous execution context.

In notebooks that support top-level `await`, you can often run an asynchronous expression directly. In a regular Python script, an asynchronous entry point may be needed:

```python
import asyncio


async def main():
    response = await agent.run(
        "What is 12 multiplied by 8?"
    )
    print(response)


asyncio.run(main())
```

Use the execution pattern appropriate for your environment. Avoid calling `asyncio.run()` inside an event loop that is already running.

---

## 24. Building Agentic RAG with QueryEngineTools

We have now learned how to create:

- A document index
- A query engine
- A tool
- An agent

The next step is to combine them.

### 24.1 Wrap the query engine

Assume that `index` and `llm` have already been configured.

```python
from llama_index.core.tools import QueryEngineTool

query_engine = index.as_query_engine(
    llm=llm,
    similarity_top_k=3,
)

query_engine_tool = QueryEngineTool.from_defaults(
    query_engine=query_engine,
    name="persona_knowledge_base",
    description=(
        "Search the knowledge base for information about "
        "stored persona descriptions."
    ),
    return_direct=False,
)
```

The tool description tells the model what kind of information it can retrieve.

A vague description can cause the model to use the wrong tool or pass an unsuitable query.

### 24.2 Create an agent with the retrieval tool

```python
from llama_index.core.agent.workflow import AgentWorkflow

query_engine_agent = AgentWorkflow.from_tools_or_functions(
    [query_engine_tool],
    llm=llm,
    system_prompt=(
        "You are a helpful assistant with access to a "
        "knowledge base containing persona descriptions. "
        "Use the knowledge base when the question requires "
        "information stored in it."
    ),
)
```

The agent can now use the query engine to answer questions about the stored information.

### 24.3 Why is this more flexible than basic RAG?

In a basic RAG pipeline, retrieval often happens automatically for every query.

In an agentic RAG system, the agent can decide whether the knowledge base is needed and can potentially combine retrieval with other tools.

For example:

```text
Question about stored information
             |
             v
      Agent evaluates task
             |
             v
   Query knowledge-base tool
             |
             v
      Retrieve relevant data
             |
             v
       Generate response
```

The agent can also use a different tool when the question does not require the knowledge base, provided that the alternative capability is available.

---

## 25. Creating Multi-Agent Systems

Sometimes one agent has too many responsibilities.

A multi-agent system divides the work among agents with narrower roles.

For example:

- A calculator agent handles arithmetic.
- An information agent searches a knowledge base.
- A manager coordinates the task and combines the results.

LlamaIndex's `AgentWorkflow` supports workflows involving multiple agents.

Agents can be given names and descriptions. These help the workflow determine which agent is suited to a task.

### 25.1 Define tools for two agents

```python
def add(a: int, b: int) -> int:
    """Add two integers."""
    return a + b


def subtract(a: int, b: int) -> int:
    """Subtract the second integer from the first."""
    return a - b
```

These functions provide deterministic arithmetic operations.

The agent should use the tools instead of relying on the language model to perform arithmetic from memory.

### 25.2 Create a calculator agent

```python
from llama_index.core.agent.workflow import ReActAgent

calculator_agent = ReActAgent(
    name="calculator",
    description="Performs basic arithmetic operations.",
    system_prompt=(
        "You are a calculator assistant. "
        "Use your tools for arithmetic operations."
    ),
    tools=[add, subtract],
    llm=llm,
)
```

The calculator agent has a limited responsibility: performing arithmetic.

### 25.3 Create an information agent

```python
query_agent = ReActAgent(
    name="info_lookup",
    description="Looks up information about a specified topic.",
    system_prompt=(
        "Use the available retrieval tool to answer "
        "questions about the knowledge base."
    ),
    tools=[query_engine_tool],
    llm=llm,
)
```

This agent focuses on retrieving information from the knowledge base.

### 25.4 Combine the agents

```python
from llama_index.core.agent.workflow import AgentWorkflow

workflow = AgentWorkflow(
    agents=[
        calculator_agent,
        query_agent,
    ],
    root_agent="calculator",
)
```

The `root_agent` identifies the agent that receives the initial user message.

An agent may handle a request directly or hand off the request to another agent when appropriate.

### 25.5 Run the multi-agent workflow

```python
response = await workflow.run(
    user_msg="Can you add 5 and 3?"
)

print(response)
```

The root agent is the calculator in this example.

The framework coordinates the workflow, but whether a handoff occurs depends on the task, agent descriptions, model behavior, and configured tools.

**Important:** Naming an agent `calculator` does not automatically guarantee that it will route every request to the correct specialist. Test the handoff behavior with representative requests.

### 25.6 Why use specialized agents?

Specialization can help make a system easier to understand and maintain.

A calculator agent does not need to retrieve documents. An information agent does not need to implement every arithmetic operation.

This separation can reduce the complexity of each agent's instructions and responsibilities.

However, multi-agent systems also add overhead. If a task only requires one calculation, a single function call may be simpler and more reliable than orchestrating several agents.

---

# Part 4 — Creating Workflows in LlamaIndex

## 26. What Is a Workflow?

A workflow is a structured sequence of operations that processes information and controls what happens next.

LlamaIndex workflows use **steps** and **events**.

A step performs a unit of work. An event carries information and helps trigger subsequent steps.

For example, a document-processing workflow might:

1. Receive a document.
2. Extract the text.
3. Split the text into chunks.
4. Generate embeddings.
5. Store the results.
6. Return a completion event.

The main advantage is that the execution logic is explicitly organized rather than being hidden inside one large function.

### 26.1 Main benefits of workflows

**Clear organization**

Each step has a defined responsibility.

**Event-driven control flow**

A step can emit an event that triggers another step.

**Typed event data**

Events can declare the information they carry, helping make the workflow structure explicit.

**State management**

A shared context can store information that multiple steps need.

**Branching and loops**

A workflow can take different paths or repeat steps based on its logic.

**Asynchronous execution**

Workflows can use asynchronous operations where appropriate.

Workflows are useful when you want some of the flexibility of agentic systems while retaining explicit control over the overall execution process.

---

## 27. Installing and Creating a Basic Workflow

The course introduces a separate workflow utility package.

```bash
pip install llama-index-utils-workflow
```

In current LlamaIndex setups, workflow functionality is commonly exposed through the core package. Check the installed version and required packages before adding dependencies, since packaging may change.

### 27.1 Create a one-step workflow

```python
from llama_index.core.workflow import (
    StartEvent,
    StopEvent,
    Workflow,
    step,
)


class MyWorkflow(Workflow):

    @step
    async def my_step(
        self,
        ev: StartEvent,
    ) -> StopEvent:
        return StopEvent(
            result="Hello, world!"
        )


workflow = MyWorkflow(
    timeout=10,
    verbose=False,
)

result = await workflow.run()

print(result)
```

Let's understand each element.

**`Workflow`**

The base class for the workflow.

**`@step`**

Marks a method as a workflow step.

**`StartEvent`**

Represents the initial event that begins the workflow.

**`StopEvent`**

Represents the workflow's completion and carries the final result.

**`timeout=10`**

Sets a time limit for the workflow according to the framework's timeout behavior.

**`verbose=False`**

Controls the workflow's verbosity.

The `run()` method starts the workflow and returns the result when the workflow completes.

---

## 28. Connecting Multiple Steps with Events

A one-step workflow is simple, but many applications require multiple operations.

We can connect steps by defining a custom event that carries data from one step to another.

### 28.1 Define an event

```python
from llama_index.core.workflow import Event


class ProcessingEvent(Event):
    intermediate_result: str
```

The `ProcessingEvent` contains an `intermediate_result` field.

A step can emit this event, and another step can receive it.

### 28.2 Create a two-step workflow

```python
from llama_index.core.workflow import (
    StartEvent,
    StopEvent,
    Workflow,
    step,
)


class MultiStepWorkflow(Workflow):

    @step
    async def step_one(
        self,
        ev: StartEvent,
    ) -> ProcessingEvent:
        return ProcessingEvent(
            intermediate_result="Step 1 complete"
        )

    @step
    async def step_two(
        self,
        ev: ProcessingEvent,
    ) -> StopEvent:
        final_result = (
            f"Finished processing: "
            f"{ev.intermediate_result}"
        )

        return StopEvent(
            result=final_result
        )


workflow = MultiStepWorkflow(
    timeout=10,
    verbose=False,
)

result = await workflow.run()

print(result)
```

### 28.3 Understand the event flow

```text
StartEvent
    |
    v
  Step One
    |
    v
ProcessingEvent
    |
    v
  Step Two
    |
    v
  StopEvent
```

The first step produces a `ProcessingEvent`.

The event contains the intermediate result, which becomes the input to the second step.

The second step uses that information to construct the final result.

### 28.4 Why type hints matter

The input and output type hints help describe which events a step accepts and emits.

They make the intended workflow structure explicit.

For example, a step that accepts `ProcessingEvent` is designed to process the data carried by that event rather than the initial start event.

The framework uses event types to connect compatible steps and determine which parts of the workflow can execute.

---

## 29. Creating Loops and Branches

Not every workflow is a straight line.

Sometimes a step must be repeated until a condition is satisfied. In other cases, the workflow needs to choose between multiple paths.

LlamaIndex workflows can express these patterns using event types and their relationships.

### 29.1 Example: Retry a step

Suppose a processing operation can fail temporarily.

We can model the result using two possible events:

- `ProcessingEvent`: processing succeeded.
- `LoopEvent`: processing should be attempted again.

```python
from llama_index.core.workflow import (
    Event,
    StartEvent,
    StopEvent,
    Workflow,
    step,
)


class ProcessingEvent(Event):
    intermediate_result: str


class LoopEvent(Event):
    loop_output: str


class RetryWorkflow(Workflow):

    @step
    async def step_one(
        self,
        ev: StartEvent | LoopEvent,
    ) -> ProcessingEvent | LoopEvent:

        if should_retry():
            return LoopEvent(
                loop_output="Retry the operation"
            )

        return ProcessingEvent(
            intermediate_result="Processing complete"
        )

    @step
    async def step_two(
        self,
        ev: ProcessingEvent,
    ) -> StopEvent:
        return StopEvent(
            result=ev.intermediate_result
        )
```

In this illustration, `should_retry()` represents application-specific logic and must be implemented separately.

The workflow can return a `LoopEvent` to trigger another execution of `step_one`, or return a `ProcessingEvent` to continue to `step_two`.

### 29.2 Why use the union operator?

The vertical bar in a type hint such as:

```python
StartEvent | LoopEvent
```

means the step can accept either event type.

Similarly:

```python
ProcessingEvent | LoopEvent
```

means the step can emit either of those event types.

This lets the workflow represent alternative paths.

### 29.3 Important production consideration

A loop must have a sensible exit condition.

If a step keeps emitting a retry event forever, the workflow may never complete.

Use bounded retries, timeouts, error classification, and explicit failure handling. A retry should only occur when repeating the operation is appropriate.

---

## 30. Drawing Workflows

Complex workflows can become difficult to understand by reading code alone.

LlamaIndex provides a utility for drawing the possible flows through a workflow.

The course demonstrates the following pattern:

```python
from llama_index.utils.workflow import draw_all_possible_flows

draw_all_possible_flows(
    workflow,
    "flow.html",
)
```

The function generates a visual representation of the workflow and stores it in an HTML file.

The workflow must be defined before it can be visualized.

A diagram can help you inspect:

- Which steps are present
- Which events connect the steps
- Where branching occurs
- Which paths can lead to completion
- Whether loops exist

This is particularly useful when debugging workflows with many events and conditional transitions.

---

## 31. State Management in Workflows

Events carry information between steps, but sometimes several steps need access to the same shared state.

For example, a document-processing workflow might need to retain:

- The original query
- The number of documents processed
- The current processing stage
- Intermediate results
- Retry counters

LlamaIndex provides `Context` and a store for managing workflow state.

### 31.1 Store and retrieve a value

```python
from llama_index.core.workflow import (
    Context,
    StartEvent,
    StopEvent,
    Workflow,
    step,
)


class QueryWorkflow(Workflow):

    @step
    async def query(
        self,
        ctx: Context,
        ev: StartEvent,
    ) -> StopEvent:

        await ctx.store.set(
            "query",
            "What is the capital of France?",
        )

        # Application-specific processing goes here.
        result = "Query received"

        stored_query = await ctx.store.get(
            "query"
        )

        return StopEvent(
            result={
                "query": stored_query,
                "status": result,
            }
        )
```

Here:

- `ctx.store.set()` stores a value.
- `ctx.store.get()` retrieves the value.
- `await` is used because the store operations are asynchronous.

The workflow can share this context with other steps as needed.

### 31.2 Why is shared state useful?

Without shared state, you might need to pass every piece of information through every event, even when several steps need access to the same data.

A shared context provides another way to retain that information.

Events and context serve related but different purposes:

- **Events** communicate information and control transitions between steps.
- **Context** provides shared workflow state.

A well-designed workflow uses each where it makes the most sense.

---

## 32. AgentWorkflow: Combining Agents with Workflows

So far, we have explored two related approaches:

1. Building workflows explicitly using steps and events.
2. Creating agents that use tools to complete tasks.

LlamaIndex also provides `AgentWorkflow`, which can coordinate one or more agents within a workflow.

This approach is useful when you want agents to collaborate and hand off tasks while using the workflow framework to manage execution.

### 32.1 How AgentWorkflow works

One agent must be designated as the root agent.

When a user message arrives:

1. The root agent receives the message.
2. It evaluates the task.
3. It may use its own tools.
4. It may hand off the task to another agent.
5. The receiving agent processes its part of the task.
6. The workflow continues until a response is returned.

The agents should have clear names, descriptions, tools, and responsibilities.

### 32.2 Example: Addition and multiplication agents

Let's define two functions.

```python
def add(a: int, b: int) -> int:
    """Add two integers."""
    return a + b


def multiply(a: int, b: int) -> int:
    """Multiply two integers."""
    return a * b
```

Create two specialized agents:

```python
from llama_index.core.agent.workflow import (
    AgentWorkflow,
    ReActAgent,
)
from llama_index.llms.huggingface_api import HuggingFaceInferenceAPI

llm = HuggingFaceInferenceAPI(
    model_name="Qwen/Qwen2.5-Coder-32B-Instruct"
)

multiply_agent = ReActAgent(
    name="multiply_agent",
    description="Multiplies two integers.",
    system_prompt=(
        "You are a multiplication assistant. "
        "Use your tool when multiplication is required."
    ),
    tools=[multiply],
    llm=llm,
)

addition_agent = ReActAgent(
    name="add_agent",
    description="Adds two integers.",
    system_prompt=(
        "You are an addition assistant. "
        "Use your tool when addition is required."
    ),
    tools=[add],
    llm=llm,
)
```

Combine them:

```python
workflow = AgentWorkflow(
    agents=[
        multiply_agent,
        addition_agent,
    ],
    root_agent="multiply_agent",
)
```

Run the workflow:

```python
response = await workflow.run(
    user_msg="Can you add 5 and 3?"
)

print(response)
```

The multiplication agent is the root agent in this example. It receives the initial request and may hand it off to the addition agent if it determines that the addition agent is better suited to the task.

The actual handoff depends on the model and the workflow configuration. A root agent's name does not guarantee that every request will be routed correctly.

### 32.3 Why define an agent's name and description?

The name identifies the agent, while the description explains its role.

For example:

```python
name="add_agent"
description="Adds two integers."
```

This makes the agent's intended responsibility explicit.

A narrow role can help reduce confusion when the system contains several agents with different capabilities.

### 32.4 Agents can also be tools

An agent's capabilities can be exposed to another agent in more advanced configurations.

This enables hierarchical designs in which one agent delegates a specialized subtask to another agent.

However, every additional layer introduces coordination complexity. Use agent delegation when it solves a real problem rather than merely because the framework supports it.

---

## 33. Sharing State Across Agents

The course also demonstrates initializing shared state when creating an `AgentWorkflow`.

For example, suppose we want to count how many times a tool function is called.

The state could contain:

```python
initial_state = {
    "num_fn_calls": 0
}
```

The workflow can use this state to track progress.

### 33.1 Update state from a tool

The following example illustrates the intended pattern:

```python
from llama_index.core.workflow import Context


async def add(
    ctx: Context,
    a: int,
    b: int,
) -> int:
    """Add two integers and count the function call."""

    state = await ctx.store.get("state")

    state["num_fn_calls"] += 1

    await ctx.store.set(
        "state",
        state,
    )

    return a + b
```

The function:

1. Receives the workflow context.
2. Retrieves the current state.
3. Increments the counter.
4. Stores the updated state.
5. Returns the arithmetic result.

The context-injection behavior and supported function signatures depend on the agent and workflow APIs in the installed version.

### 33.2 Initialize the multi-agent workflow state

The course demonstrates the following configuration pattern:

```python
workflow = AgentWorkflow(
    agents=[
        multiply_agent,
        addition_agent,
    ],
    root_agent="multiply_agent",
    initial_state={
        "num_fn_calls": 0
    },
    state_prompt=(
        "Current state: {state}. "
        "User message: {msg}"
    ),
)
```

The initial state provides a starting value. The state prompt describes how the current state and user message are presented to the agent.

### 33.3 Run the workflow with a shared context

```python
from llama_index.core.workflow import Context

ctx = Context(workflow)

response = await workflow.run(
    user_msg="Can you add 5 and 3?",
    ctx=ctx,
)

state = await ctx.store.get("state")

print(state["num_fn_calls"])
```

The result of the counter depends on which tools were actually called. It should not be assumed that the count increases simply because an agent was created.

This example demonstrates the broader principle: workflow state can record information that matters across agent operations.

---

# Part 5 — Putting Everything Together

## 34. How LlamaIndex Components, Tools, Agents, and Workflows Fit Together

Let's connect the concepts we have learned.

Imagine you are building a meeting knowledge assistant for a platform such as V-Meet.

The user asks:

> What action items were discussed in our previous project meeting?

A possible architecture is:

```text
                    User question
                         |
                         v
                    AgentWorkflow
                         |
                         v
                    Root agent
                         |
                         v
                QueryEngineTool
                         |
                         v
                 QueryEngine
                         |
                         v
                  Vector index
                         |
                         v
                Meeting transcripts
                         |
                         v
                Relevant text chunks
                         |
                         v
                 LLM generates answer
                         |
                         v
              Answer with source references
```

Each component has a distinct role.

**Document loader**

Reads meeting transcripts or other supported data sources.

**Ingestion pipeline**

Splits documents into nodes and generates embeddings.

**Vector store**

Stores the indexed information for later retrieval.

**Query engine**

Retrieves relevant information and generates a response.

**QueryEngineTool**

Exposes the query engine as a capability an agent can invoke.

**Agent**

Decides whether the query engine is needed and how to use its output.

**Workflow**

Coordinates the steps, shared state, and handoffs where necessary.

**Evaluator**

Checks whether the response is relevant, supported, and correct.

**Observability system**

Helps developers understand execution behavior and troubleshoot problems.

This architecture is only an example. The simplest application may need just a query engine, while a more complex system may benefit from an agent or a multi-agent workflow.

---

## 35. LlamaIndex vs. smolagents: Understanding the Difference

Both LlamaIndex and smolagents can be used to build LLM-powered agents, but they emphasize different abstractions.

| Area | LlamaIndex | smolagents |
|---|---|---|
| Data access | Strong focus on data ingestion, indexes, retrieval, and query engines | Supports retrieval through tools and agent workflows |
| Agent tool use | Functions, query engines, ToolSpecs, and integrations | Tools, custom Python functions, and external integrations |
| Agent approaches | Function-calling agents, ReAct agents, and custom agents | CodeAgent and ToolCallingAgent |
| Workflow design | Event-driven workflows and AgentWorkflow | Multi-step agent execution and managed-agent coordination |
| Knowledge-base integration | Query engines and indexes are central abstractions | Retrieval can be integrated as agent tools |
| Asynchronous execution | Workflow and agent APIs commonly use async execution | Depends on the tool and agent execution interfaces |
| Main design emphasis | Connecting LLM applications to data and orchestrating retrieval/workflows | Giving agents tool-use capabilities, including code-based actions |

Neither framework is universally better.

Choose based on the application requirements, integrations, execution model, and the level of control you need.

For a data-heavy application with document ingestion, retrieval, and query engines, LlamaIndex provides useful abstractions.

For a task that centers on tool use and code-based agent actions, smolagents may be a suitable choice.

You can also evaluate frameworks against the same representative tasks before making an architectural decision.

---


## 36. Final Summary

In this learning journey, we explored LlamaIndex as a framework for building LLM-powered applications and agents over data.

We learned that:

1. **Components** are reusable building blocks for data ingestion, embedding, retrieval, and response generation.
2. **RAG** retrieves relevant information from external data and supplies it to a language model.
3. **Nodes** represent smaller chunks of source documents that can be indexed and retrieved.
4. **Embeddings** represent text numerically and support semantic retrieval.
5. **Vector stores** retain embeddings and associated information for later retrieval.
6. **QueryEngine** combines retrieval and response generation.
7. **FunctionTool** turns Python functions into tools that agents can call.
8. **QueryEngineTool** allows agents to query a knowledge base when needed.
9. **ToolSpecs** organize related tool capabilities into reusable collections.
10. **MCP integrations** connect compatible agents to externally exposed tools.
11. **Function-calling and ReAct agents** provide different approaches to selecting and executing tools.
12. **Context** can preserve relevant state across interactions.
13. **AgentWorkflow** coordinates one or more agents and supports handoffs.
14. **Workflows** use steps and events to organize execution.
15. **Branches, loops, and state management** make workflows suitable for more complex processes.
16. **Evaluation and observability** help developers assess quality and understand execution behavior.