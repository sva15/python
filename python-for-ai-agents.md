# 🤖 Python for AI Agents — LangChain & LangGraph Decoder

## *Your Curriculum → Agent Code*

> **Goal:** Understand the Python syntax and patterns used in LangChain/LangGraph agent applications, mapped to the lessons you've already studied.

---

## 📋 Priority Learning Path

Not all 60 lessons are equally important for agent work. Here's your priority order:

### 🔴 Must-Know (Read These First)
| Priority | Lesson | Why It Matters for Agents |
|:--------:|--------|--------------------------|
| 1 | [L2-14: Type Hints](https://github.com/sva15/python/blob/main/level-2/lesson-14-type-hints.md) | Pydantic models, TypedDict state, function signatures |
| 2 | [L1-07: Dictionaries](https://github.com/sva15/python/blob/main/level-1/lesson-07-dictionaries.md) | LangGraph state is a dict. Everything flows through dicts |
| 3 | [L1-09: Functions](https://github.com/sva15/python/blob/main/level-1/lesson-09-functions.md) | Nodes in LangGraph are functions. Tools are functions |
| 4 | [L4-01: Decorators](https://github.com/sva15/python/blob/main/level-4/lesson-01-decorators.md) | `@tool`, `@chain`, `@node` — decorators are everywhere |
| 5 | [L3-08: Dataclasses](https://github.com/sva15/python/blob/main/level-3/lesson-08-dataclasses.md) | Pydantic's `BaseModel` works like dataclasses |
| 6 | [L1-11: Classes](https://github.com/sva15/python/blob/main/level-1/lesson-11-define-classes.md) | Tools, agents, and chains are classes |

### 🟡 Important (Read Next)
| Priority | Lesson | Why |
|:--------:|--------|-----|
| 7 | [L5-05: Async](https://github.com/sva15/python/blob/main/level-5/lesson-05-threading-async.md) | Streaming, async agents, parallel tool calls |
| 8 | [L3-01: Generators](https://github.com/sva15/python/blob/main/level-3/lesson-01-generator-functions.md) | Streaming LLM responses use `yield` |
| 9 | [L3-05: Exceptions](https://github.com/sva15/python/blob/main/level-3/lesson-05-exception-handling.md) | Retry logic, fallbacks, error handling |
| 10 | [L4-03: Inheritance](https://github.com/sva15/python/blob/main/level-4/lesson-03-inheritance.md) | Extending BaseTool, BaseModel, etc. |
| 11 | [L3-07: Enums](https://github.com/sva15/python/blob/main/level-3/lesson-07-enums.md) | Agent states, routing decisions |
| 12 | [L2-02: String Formatting](https://github.com/sva15/python/blob/main/level-2/lesson-02-string-formatting.md) | Prompt templates use f-strings |

### 🟢 Helpful (Read When Needed)
| Priority | Lesson | Why |
|:--------:|--------|-----|
| 13 | [L5-03: CSV/JSON](https://github.com/sva15/python/blob/main/level-5/lesson-03-csv-json.md) | Parsing LLM JSON output, structured data |
| 14 | [L3-06: Lambda](https://github.com/sva15/python/blob/main/level-3/lesson-06-lambda-functions.md) | Inline functions in chains |
| 15 | [L4-02: Closures](https://github.com/sva15/python/blob/main/level-4/lesson-02-closures.md) | Callback patterns, RunnableLambda |
| 16 | [L3-09: Context Managers](https://github.com/sva15/python/blob/main/level-3/lesson-09-context-managers.md) | Resource cleanup, tracing |
| 17 | [L2-05: *args/**kwargs](https://github.com/sva15/python/blob/main/level-2/lesson-05-args-kwargs.md) | Flexible function signatures |

---

## 🔧 Python Concepts → Agent Code Patterns

### Pattern 1: TypedDict — LangGraph State ⭐⭐⭐

**Lesson reference:** [Type Hints](https://github.com/sva15/python/blob/main/level-2/lesson-14-type-hints.md), [Dictionaries](https://github.com/sva15/python/blob/main/level-1/lesson-07-dictionaries.md)

```python
# LangGraph state is ALWAYS a TypedDict
# This is the MOST important pattern to understand

from typing import TypedDict, Annotated
from langgraph.graph.message import add_messages

# === What you learned ===
# A TypedDict is a dict with typed keys (Level 2, Lesson 14)
class PersonInfo(TypedDict):
    name: str
    age: int

# === How agents use it ===
# LangGraph state — the data flowing through your agent
class AgentState(TypedDict):
    messages: Annotated[list, add_messages]   # Chat history
    next_action: str                           # What to do next
    documents: list[str]                       # Retrieved docs
    final_answer: str                          # Agent's response

# KEY INSIGHT:
# - Every node in LangGraph receives this state dict
# - Every node returns updates to this state dict
# - Annotated[list, add_messages] means "append, don't replace"
```

**What to focus on:** Understand that `TypedDict` is just a regular Python dict with type annotations. When you see `AgentState`, think "dictionary with specific keys."

---

### Pattern 2: Decorators — `@tool` ⭐⭐⭐

**Lesson reference:** [Decorators](https://github.com/sva15/python/blob/main/level-4/lesson-01-decorators.md)

```python
# === What you learned ===
# A decorator wraps a function with extra behavior
def log_call(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        return func(*args, **kwargs)
    return wrapper

@log_call
def greet(name):
    return f"Hello {name}"

# === How agents use it ===
from langchain_core.tools import tool

@tool                                    # ← Decorator!
def search_database(query: str) -> str:
    """Search the firm's case database.     ← Docstring becomes tool description!
    
    Args:
        query: The search query string.     ← Becomes parameter description
    """
    results = db.search(query)
    return f"Found {len(results)} cases"

# KEY INSIGHT:
# @tool converts a regular Python function into a Tool object
# The DOCSTRING becomes the description the LLM reads
# The TYPE HINTS become the parameter schema
# The LLM decides WHEN to call this function based on the description
```

**What to focus on:** A decorator transforms a function. `@tool` transforms your function into something an LLM can understand and call.

---

### Pattern 3: Pydantic BaseModel — Structured Data ⭐⭐⭐

**Lesson reference:** [Dataclasses](https://github.com/sva15/python/blob/main/level-3/lesson-08-dataclasses.md), [Type Hints](https://github.com/sva15/python/blob/main/level-2/lesson-14-type-hints.md)

```python
# === What you learned (dataclass) ===
from dataclasses import dataclass

@dataclass
class Client:
    name: str
    revenue: float
    active: bool = True

# === How agents use it (Pydantic — same idea, more powerful) ===
from pydantic import BaseModel, Field

class SearchQuery(BaseModel):
    """Schema for structured LLM output."""
    query: str = Field(description="The search query")
    max_results: int = Field(default=10, description="Max results to return")
    filters: list[str] = Field(default_factory=list, description="Filter terms")

# Force LLM to output structured data:
from langchain_core.output_parsers import PydanticOutputParser

parser = PydanticOutputParser(pydantic_object=SearchQuery)
# LLM output: {"query": "patent law", "max_results": 5, "filters": ["2024"]}
# Parsed to: SearchQuery(query="patent law", max_results=5, filters=["2024"])

# KEY INSIGHT:
# Pydantic BaseModel = dataclass + validation + JSON serialization
# Think: "a class that validates its data and converts to/from JSON"
# LLMs return text → Pydantic parses it into structured Python objects
```

**What to focus on:** `BaseModel` is like `@dataclass` but with automatic validation and JSON conversion. Agent apps use it everywhere for structured LLM output.

---

### Pattern 4: Functions as Nodes — LangGraph ⭐⭐⭐

**Lesson reference:** [Functions](https://github.com/sva15/python/blob/main/level-1/lesson-09-functions.md), [Dictionaries](https://github.com/sva15/python/blob/main/level-1/lesson-07-dictionaries.md)

```python
# === What you learned ===
# Functions take input and return output
def process(data):
    result = transform(data)
    return result

# === How agents use it — nodes are just functions! ===
from langgraph.graph import StateGraph

class AgentState(TypedDict):
    messages: list
    documents: list[str]
    answer: str

# Each node is a FUNCTION that takes state and returns state updates
def retrieve_documents(state: AgentState) -> dict:
    """Node: search for relevant documents."""
    query = state["messages"][-1].content        # Read from state
    docs = vector_store.search(query)
    return {"documents": docs}                    # Return state UPDATE

def generate_answer(state: AgentState) -> dict:
    """Node: generate answer from documents."""
    docs = state["documents"]                    # Read from state
    response = llm.invoke(f"Answer using: {docs}")
    return {"answer": response.content}          # Return state UPDATE

def should_continue(state: AgentState) -> str:
    """Edge: decide what to do next."""
    if state["documents"]:
        return "generate"       # Go to generate_answer node
    return "end"                # Stop

# Build the graph — connect functions with edges
graph = StateGraph(AgentState)
graph.add_node("retrieve", retrieve_documents)   # Register functions
graph.add_node("generate", generate_answer)
graph.add_edge("retrieve", "generate")           # retrieve → generate
graph.add_edge("generate", "__end__")            # generate → done

# KEY INSIGHT:
# A LangGraph agent is just functions connected with arrows!
# Each function: receives state dict → returns partial state dict
# The graph decides which function runs next
```

**What to focus on:** Every node is a regular Python function. It receives a dict, does work, returns a dict. LangGraph just connects them.

---

### Pattern 5: Classes & Inheritance — Custom Tools ⭐⭐

**Lesson reference:** [Classes](https://github.com/sva15/python/blob/main/level-1/lesson-11-define-classes.md), [Inheritance](https://github.com/sva15/python/blob/main/level-4/lesson-03-inheritance.md)

```python
# === What you learned ===
class Animal:
    def speak(self): pass     # Base class

class Dog(Animal):            # Inherits from Animal
    def speak(self):
        return "Woof"

# === How agents use it — extending BaseTool ===
from langchain_core.tools import BaseTool
from pydantic import BaseModel, Field

class DatabaseSearchInput(BaseModel):
    query: str = Field(description="SQL-like search query")
    table: str = Field(default="cases", description="Table to search")

class DatabaseSearchTool(BaseTool):          # Inherits from BaseTool!
    name: str = "database_search"
    description: str = "Search the firm's database for case information"
    args_schema: type = DatabaseSearchInput
    
    def _run(self, query: str, table: str = "cases") -> str:
        """Override this method — it's what runs when the tool is called."""
        results = self.db.execute(f"SELECT * FROM {table} WHERE {query}")
        return str(results)

# KEY INSIGHT:
# BaseTool is a parent class. You INHERIT from it.
# You OVERRIDE _run() with your logic.
# name + description = what the LLM sees
# args_schema = what parameters the LLM can pass
```

---

### Pattern 6: Async/Await — Streaming & Parallel ⭐⭐

**Lesson reference:** [Threading & Async](https://github.com/sva15/python/blob/main/level-5/lesson-05-threading-async.md)

```python
# === What you learned ===
import asyncio

async def fetch_data():
    await asyncio.sleep(1)    # Non-blocking wait
    return "data"

# === How agents use it ===
# Streaming LLM responses (tokens arrive one at a time)
async for chunk in llm.astream("Tell me about patent law"):
    print(chunk.content, end="", flush=True)    # Print each token as it arrives

# Async tool execution (parallel tool calls)
async def research_agent(state):
    # Run multiple tools simultaneously!
    results = await asyncio.gather(
        search_tool.ainvoke("patent law"),
        search_tool.ainvoke("case precedents"),
        search_tool.ainvoke("filing deadlines"),
    )
    return {"documents": results}

# Async graph execution
result = await graph.ainvoke({"messages": [user_message]})

# KEY INSIGHT:
# async/await lets the agent:
# 1. Stream responses token-by-token (not wait for full response)
# 2. Call multiple tools at the same time
# 3. Handle multiple users simultaneously
# The "a" prefix means async: .invoke() → .ainvoke(), .stream() → .astream()
```

---

### Pattern 7: Generators — Streaming Responses ⭐⭐

**Lesson reference:** [Generators](https://github.com/sva15/python/blob/main/level-3/lesson-01-generator-functions.md)

```python
# === What you learned ===
def countdown(n):
    while n > 0:
        yield n        # Produces one value at a time
        n -= 1

for num in countdown(3):
    print(num)    # 3, 2, 1

# === How agents use it ===
# Stream agent events as they happen
for event in graph.stream({"messages": [user_input]}):
    for node_name, output in event.items():
        print(f"Node '{node_name}' produced: {output}")

# Each iteration gives you ONE step of the agent's execution:
# Node 'retrieve' produced: {"documents": [...]}
# Node 'generate' produced: {"answer": "The case law states..."}

# KEY INSIGHT:
# .stream() returns a generator — events arrive ONE AT A TIME
# This lets you show progress to the user as the agent works
# Without streaming, you'd wait for the ENTIRE agent to finish
```

---

### Pattern 8: Conditional Logic — Routing ⭐⭐

**Lesson reference:** [Booleans & Conditionals](https://github.com/sva15/python/blob/main/level-1/lesson-03-booleans-conditionals.md), [Enums](https://github.com/sva15/python/blob/main/level-3/lesson-07-enums.md)

```python
# === What you learned ===
if condition:
    do_this()
elif other_condition:
    do_that()
else:
    do_default()

# === How agents use it — conditional edges in LangGraph ===
from typing import Literal

def route_agent(state: AgentState) -> Literal["search", "answer", "clarify"]:
    """Decide next step based on state."""
    last_message = state["messages"][-1]
    
    if last_message.tool_calls:          # LLM wants to use a tool
        return "search"
    elif state.get("documents"):          # We have docs, generate answer
        return "answer"
    else:                                 # Need more info
        return "clarify"

# Register conditional edge
graph.add_conditional_edges(
    "agent",                    # From this node...
    route_agent,                # Run this function...
    {                           # Map return values to nodes:
        "search": "tool_node",
        "answer": "generate",
        "clarify": "ask_user",
    }
)

# KEY INSIGHT:
# Conditional edges are just if/elif/else inside a function
# The function returns a STRING that matches a node name
# This is how agents DECIDE what to do next
```

---

### Pattern 9: The Pipe Operator `|` — LCEL Chains ⭐⭐

**Lesson reference:** [Dunder Methods](https://github.com/sva15/python/blob/main/level-2/lesson-07-dunder-methods.md) (`__or__`)

```python
# === What you learned ===
# The | operator can be overloaded with __or__
class Step:
    def __or__(self, other):
        return Pipeline(self, other)

# === How agents use it — LangChain Expression Language (LCEL) ===
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser

prompt = ChatPromptTemplate.from_template("Summarize: {text}")
llm = ChatOpenAI(model="gpt-4o")
parser = StrOutputParser()

# The | operator chains components together:
chain = prompt | llm | parser
#       ↑         ↑      ↑
#    Format    Send to   Extract
#    prompt    LLM       text

result = chain.invoke({"text": "Long legal document..."})

# This is equivalent to:
formatted = prompt.invoke({"text": "Long legal document..."})
response = llm.invoke(formatted)
text = parser.invoke(response)

# KEY INSIGHT:
# | is Python's __or__ operator, overloaded by LangChain
# prompt | llm | parser = "format, then send to LLM, then parse"
# Each component's output becomes the next component's input
# Think of it as a Unix pipe: cat file | grep pattern | sort
```

---

### Pattern 10: Callbacks & Closures — Tracing ⭐

**Lesson reference:** [Closures](https://github.com/sva15/python/blob/main/level-4/lesson-02-closures.md), [Lambda](https://github.com/sva15/python/blob/main/level-3/lesson-06-lambda-functions.md)

```python
# === What you learned ===
def make_logger(prefix):
    def log(message):           # Closure — captures 'prefix'
        print(f"[{prefix}] {message}")
    return log

# === How agents use it — callbacks for monitoring ===
from langchain_core.callbacks import BaseCallbackHandler

class AgentMonitor(BaseCallbackHandler):
    def on_llm_start(self, serialized, prompts, **kwargs):
        print(f"📤 Sending to LLM: {prompts[0][:100]}...")
    
    def on_tool_start(self, serialized, input_str, **kwargs):
        print(f"🔧 Using tool: {serialized['name']}")
    
    def on_llm_end(self, response, **kwargs):
        print(f"📥 LLM responded: {response.generations[0][0].text[:100]}...")

# Attach monitor to see what the agent does:
result = agent.invoke(
    {"messages": [user_input]},
    config={"callbacks": [AgentMonitor()]}
)

# KEY INSIGHT:
# Callbacks let you observe the agent's behavior
# They use class inheritance (BaseCallbackHandler)
# You override methods: on_llm_start, on_tool_start, etc.
```

---

## 🗺️ Anatomy of a Complete Agent

Here's a minimal but complete LangGraph agent with **every Python concept labeled**:

```python
# ──────────────────────────────────────────────────
# COMPLETE LANGGRAPH AGENT — Annotated with Python concepts
# ──────────────────────────────────────────────────

# IMPORTS (Level 1, Lesson 10: Modules & Imports)
from typing import TypedDict, Annotated, Literal     # Type hints (L2-14)
from langchain_openai import ChatOpenAI
from langchain_core.messages import HumanMessage
from langchain_core.tools import tool                  # Decorator (L4-01)
from langgraph.graph import StateGraph
from langgraph.graph.message import add_messages
from langgraph.prebuilt import ToolNode


# PYDANTIC / TYPED DICT (L3-08: Dataclasses, L2-14: Type Hints)
class AgentState(TypedDict):
    """State that flows through the graph — it's just a dict!"""
    messages: Annotated[list, add_messages]


# DECORATED FUNCTION AS TOOL (L4-01: Decorators, L1-09: Functions)
@tool
def search_cases(query: str) -> str:            # Type hints! (L2-14)
    """Search the legal database for case information.
    
    Args:
        query: Natural language search query
    """
    # Your actual search logic here
    return f"Found 3 cases matching '{query}'"


@tool
def calculate_billing(hours: float, rate: float = 450.0) -> str:
    """Calculate billing amount for legal services."""
    total = hours * rate                         # Basic math (L1-01)
    return f"Total billing: ${total:,.2f}"       # f-string (L2-02)


# LLM SETUP (L1-13: Instantiate Objects)
llm = ChatOpenAI(model="gpt-4o")
tools = [search_cases, calculate_billing]        # List of tools (L1-04)
llm_with_tools = llm.bind_tools(tools)           # Method call (L3-02)


# NODE FUNCTIONS (L1-09: Functions, L1-07: Dictionaries)
def agent_node(state: AgentState) -> dict:
    """The 'thinking' node — LLM decides what to do."""
    response = llm_with_tools.invoke(state["messages"])  # Dict access (L1-07)
    return {"messages": [response]}                       # Return dict (L1-07)


# CONDITIONAL ROUTING (L1-03: Conditionals)
def should_continue(state: AgentState) -> Literal["tools", "end"]:
    """Decide: use a tool or finish?"""
    last_message = state["messages"][-1]         # List indexing (L1-04)
    
    if last_message.tool_calls:                  # if/else (L1-03)
        return "tools"                            # String return
    return "end"


# BUILD THE GRAPH (connecting functions with edges)
graph = StateGraph(AgentState)                    # Class instantiation (L1-13)

graph.add_node("agent", agent_node)               # Register function as node
graph.add_node("tools", ToolNode(tools))          # Prebuilt tool executor

graph.set_entry_point("agent")                    # Start here

graph.add_conditional_edges(                      # if/else routing
    "agent",
    should_continue,
    {"tools": "tools", "end": "__end__"}          # Dict mapping (L1-07)
)

graph.add_edge("tools", "agent")                  # After tools → back to agent

# COMPILE (creates the runnable agent)
app = graph.compile()


# RUN IT
result = app.invoke({
    "messages": [HumanMessage(content="Search for patent cases and bill 10 hours")]
})

# STREAM IT (L3-01: Generators)
for event in app.stream({"messages": [HumanMessage(content="Hello")]}):
    for node, output in event.items():            # Dict iteration (L1-07)
        print(f"[{node}]: {output}")
```

---

## 📊 Concept Frequency in Agent Code

How often each Python concept appears in typical agent applications:

| Concept | Frequency | Lessons |
|---------|:---------:|---------|
| **Dictionaries** | ████████████████████ | L1-07 |
| **Type hints** | ████████████████████ | L2-14 |
| **Functions** | ███████████████████ | L1-09 |
| **Decorators** (`@tool`) | ██████████████████ | L4-01 |
| **Classes/Inheritance** | █████████████████ | L1-11, L4-03 |
| **f-strings** | ████████████████ | L2-02 |
| **if/elif/else** | ███████████████ | L1-03 |
| **Lists** | ██████████████ | L1-04 |
| **Async/await** | █████████████ | L5-05 |
| **Generators** (streaming) | ███████████ | L3-01 |
| **Pydantic** (≈ dataclass) | ███████████ | L3-08 |
| **Exceptions** (try/except) | ██████████ | L3-05 |
| **Enums** (routing) | ████████ | L3-07 |
| **Lambda** | ███████ | L3-06 |
| ***args/**kwargs** | ██████ | L2-05 |

---

## 🎯 Recommended Reading Order

```
WEEK 1: Foundations
  L1-07 Dictionaries     → State management
  L1-09 Functions        → Nodes and tools
  L2-14 Type Hints       → TypedDict, annotations
  L2-02 String Formatting → Prompt templates

WEEK 2: Agent Building Blocks
  L4-01 Decorators       → @tool, @chain
  L3-08 Dataclasses      → Pydantic BaseModel
  L1-11 Classes          → Custom tools, handlers
  L4-03 Inheritance      → Extending LangChain classes

WEEK 3: Advanced Patterns
  L5-05 Async            → Streaming, parallel tools
  L3-01 Generators       → .stream() responses
  L3-05 Exceptions       → Error handling, retries
  L3-07 Enums            → Agent states, routing

WEEK 4: Polish
  L2-05 *args/**kwargs   → Flexible APIs
  L4-02 Closures         → Callbacks, RunnableLambda
  L5-03 CSV/JSON         → Parsing structured output
```
