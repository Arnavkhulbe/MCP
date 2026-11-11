
# Building Custom MCP Clients Using LangChain MCP Adapters

## 1. Introduction

In previous lectures, we learned:

- Why MCP (Model Context Protocol) is needed.
- MCP architecture and its components.
- The MCP lifecycle.
- How to use existing MCP Servers.
- How to build Local MCP Servers.
- How to build and deploy Remote MCP Servers.

Until now, we used **Claude Desktop** as our MCP Host because it already contains an MCP Client.

But what if we want to build our own chatbot and connect it to MCP Servers?

For that, we need to create our **own MCP Client**.

### Goal of This Lecture

We will build a custom AI chatbot that:

1. Supports normal conversations.
2. Connects to a Local Maths MCP Server.
3. Connects to a Remote Expense Tracker MCP Server.
4. Can connect to third-party MCP Servers.
5. Uses an LLM to select and execute tools.
6. Provides a Streamlit-based graphical interface.

---

## 2. MCP Playlist Recap

The MCP playlist was divided into three major parts:

### WHY

Understand why MCP is needed.

### WHAT

Understand:

- MCP architecture.
- MCP lifecycle.
- How MCP communication works.

### HOW

Learn how to:

- Use MCP Servers.
- Create Local MCP Servers.
- Create Remote MCP Servers.
- Deploy Remote MCP Servers.
- Build Custom MCP Clients.

This lecture covers the final topic: **Building MCP Clients**.

---

# PART 1: Understanding MCP Clients

## 3. What Is an MCP Client?

An MCP Client is a component that establishes a connection with an MCP Server and communicates with it using the MCP protocol.

It allows an application to discover and execute capabilities exposed by MCP Servers.

### Architecture

```text
                  USER
                    |
                    v
              AI CHATBOT
                (Host)
                    |
                    v
               MCP CLIENT
                    |
          +---------+---------+
          |                   |
          v                   v
    LOCAL MCP SERVER    REMOTE MCP SERVER
          |                   |
          v                   v
      Maths Tools        Expense Tools
```

### Important Point

The MCP Client does not need to be Claude Desktop.

We can create our own MCP Client and integrate it into our own application.

---

## 4. Libraries for Building MCP Clients

The lecture discusses three options:

1. Official MCP Python SDK.
2. FastMCP.
3. LangChain MCP Adapters.

We will use:

**LangChain MCP Adapters**

### What Is LangChain MCP Adapters?

It is a lightweight library that makes MCP tools compatible with LangChain and LangGraph.

This allows us to integrate MCP Servers into applications built using the LangChain ecosystem.

### Why Use It?

The instructor selected this library for two reasons:

- It provides a relatively simple way to create MCP Clients.
- It integrates closely with LangChain and LangGraph.

### Important Class

```python
MultiServerMCPClient
```

This class allows us to configure connections to multiple MCP Servers.

---

## 5. Project Architecture

Our chatbot will initially connect to two MCP Servers.

### Server 1: Local Maths MCP Server

Provides arithmetic tools:

- Add
- Subtract
- Multiply
- Divide
- Power
- Modulus

**Transport:** STDIO

### Server 2: Remote Expense Tracker MCP Server

Provides expense tracking tools:

- Add Expense
- List Expenses
- Summarize Expenses

**Transport:** Streamable HTTP

### Architecture

```text
                      USER
                        |
                        v
                 CUSTOM CHATBOT
                        |
                        v
                       LLM
                        |
                        v
               MultiServerMCPClient
                        |
              +---------+---------+
              |                   |
              v                   v
       LOCAL MCP SERVER    REMOTE MCP SERVER
              |                   |
              v                   v
         Maths Tools         Expense Tools
              |                   |
              v                   v
            STDIO                HTTP
```

Later, we will also connect a third-party Manim MCP Server.

---

# PART 2: Setting Up the Project

## 6. Prerequisites

We need:

- Python
- UV package manager
- VS Code
- LangChain
- LangChain OpenAI
- LangChain MCP Adapters
- python-dotenv
- Streamlit
- A working Local Maths MCP Server
- A deployed Remote Expense Tracker MCP Server
- An OpenAI API key for the lecture's implementation

---

## 7. Project Setup

### Step 1: Install UV

```bash
pip install uv
```

### Step 2: Create a Project Folder

Create a new folder for the MCP Client project.

Open it in VS Code.

### Step 3: Initialize UV

```bash
uv init .
```

### Step 4: Install Dependencies

```bash
uv add langchain langchain-openai langchain-mcp-adapters python-dotenv streamlit
```

### Step 5: Create the Client File

```text
client1.py
```

We will build the client step by step inside this file.

---

# PART 3: Connecting to a Local MCP Server

## 8. Understanding Async Programming

The MCP Client uses Python's asynchronous programming features.

Important keywords:

```python
async
await
```

### async

Defines an asynchronous function.

```python
async def main():
    pass
```

### await

Waits for an asynchronous operation to complete without blocking the event loop in the usual synchronous way.

```python
tools = await client.get_tools()
```

### asyncio.run()

Runs an asynchronous function from synchronous Python code.

```python
import asyncio

async def main():
    print("MCP Client Started")

if __name__ == "__main__":
    asyncio.run(main())
```

This is the basic structure of our client.

---

## 9. Importing MultiServerMCPClient

```python
import asyncio

from langchain_mcp_adapters.client import MultiServerMCPClient
```

### Why MultiServerMCPClient?

Because our application will eventually connect to multiple MCP Servers.

We start with one server and then add more.

---

## 10. Configuring the Local Maths MCP Server

Before connecting, we must provide the server configuration.

Example:

```python
servers = {
    "math": {
        "transport": "stdio",
        "command": "uv",
        "args": [
            "run",
            "fastmcp",
            "run",
            "PATH_TO_MATH_SERVER/main.py"
        ]
    }
}
```

Replace:

```text
PATH_TO_MATH_SERVER/main.py
```

with the actual path to your Maths MCP Server.

### Understanding the Configuration

| Key | Meaning |
|---|---|
| `math` | Name assigned to the server. |
| `transport` | Communication transport. |
| `stdio` | Local process communication. |
| `command` | Executable used to start the server. |
| `args` | Arguments passed to the executable. |

### Important Concept

For a local STDIO connection, the MCP Client launches the server process using the supplied command.

The command effectively starts:

```bash
uv run fastmcp run PATH_TO_MATH_SERVER/main.py
```

If `uv` is not available through the process PATH, its full executable path may be required.

---

## 11. Creating the MCP Client

Once the server configuration is ready, create the client.

```python
client = MultiServerMCPClient(servers)
```

Now the client knows how to connect to the configured server.

### Fetching Available Tools

```python
tools = await client.get_tools()
```

This retrieves the tools exposed by the MCP Server.

### Complete Example

```python
import asyncio
from langchain_mcp_adapters.client import MultiServerMCPClient

async def main():

    servers = {
        "math": {
            "transport": "stdio",
            "command": "uv",
            "args": [
                "run",
                "fastmcp",
                "run",
                "PATH_TO_MATH_SERVER/main.py"
            ]
        }
    }

    client = MultiServerMCPClient(servers)

    tools = await client.get_tools()

    print(tools)

if __name__ == "__main__":
    asyncio.run(main())
```

### What Happens?

1. The MCP Client starts the Local Maths Server.
2. The client establishes a connection.
3. The server exposes its tools.
4. The client retrieves the available tools.
5. The tool definitions are printed.

The tools include:

```text
add
subtract
multiply
divide
power
modulus
```

---

## 12. Converting the Tool List into a Dictionary

The client returns a list of tool objects.

The instructor converts this list into a dictionary.

### Code

```python
named_tools = {}

for tool in tools:
    named_tools[tool.name] = tool
```

### Why?

Because we want to retrieve a particular tool using its name.

The resulting structure is conceptually:

```python
named_tools = {
    "add": add_tool,
    "subtract": subtract_tool,
    "multiply": multiply_tool,
    "divide": divide_tool,
    "power": power_tool,
    "modulus": modulus_tool
}
```

This makes it easier to execute a tool selected by the LLM.

---

# PART 4: Connecting the LLM to MCP Tools

## 13. Initializing the LLM

Until now:

- The MCP Client is connected to the server.
- The client can discover available tools.

But who decides which tool should be used?

**The LLM.**

The lecture uses an OpenAI model through LangChain.

### Create a .env File

```env
OPENAI_API_KEY=your_api_key_here
```

Do not commit this file to GitHub.

### Import Dependencies

```python
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
```

### Load Environment Variables

```python
load_dotenv()
```

### Initialize the Model

```python
llm = ChatOpenAI(model="gpt-5")
```

The lecture demonstrates a GPT-5 model.

---

## 14. Binding MCP Tools to the LLM

We need to make the MCP tools available to the LLM.

For that, LangChain provides:

```python
bind_tools()
```

### Code

```python
llm_with_tools = llm.bind_tools(tools)
```

### Important Distinction

We pass:

```python
tools
```

Not:

```python
named_tools
```

Because `bind_tools()` expects tool definitions, while `named_tools` is our lookup dictionary for executing tools.

### What Happens?

The LLM receives information about:

- Tool names.
- Tool descriptions.
- Tool input parameters.

It can then decide whether a tool is needed for a user's question.

---

## 15. Sending a Prompt to the LLM

Example:

```python
prompt = "Using the math tool, what is the product of 12 and 15?"

response = await llm_with_tools.ainvoke(prompt)

print(response)
```

### Why Mention "Using the Math Tool"?

In the lecture, the LLM initially answered the simple calculation directly.

It did not call a tool.

After explicitly asking it to use the Maths tool, the LLM generated a tool call.

### Example Tool Call

```python
{
    "name": "multiply",
    "args": {
        "a": 12,
        "b": 15
    },
    "id": "tool_call_id"
}
```

The actual ID is generated by the model/provider.

### Important Concept

The LLM does not necessarily execute the tool itself.

It generates a request describing:

1. Which tool to call.
2. What arguments to pass.
3. Which tool-call ID identifies the request.

The application must execute that request.

---

# PART 5: Executing MCP Tools

## 16. Understanding response.tool_calls

The tool calls are available through:

```python
response.tool_calls
```

This is a list because the LLM may request multiple tool calls.

Example:

```python
response.tool_calls[0]
```

Retrieves the first tool call.

### Extracting the Tool Name

```python
selected_tool = response.tool_calls[0]["name"]
```

### Extracting Arguments

```python
selected_tool_args = response.tool_calls[0]["args"]
```

### Extracting Tool Call ID

```python
selected_tool_id = response.tool_calls[0]["id"]
```

These values tell us which tool to execute and how to identify its result.

---

## 17. Executing the Selected Tool

Remember the dictionary:

```python
named_tools = {
    tool.name: tool
    for tool in tools
}
```

We can retrieve the selected tool using its name.

### Code

```python
tool_result = await named_tools[selected_tool].ainvoke(
    selected_tool_args
)
```

### Understanding the Code

```python
named_tools[selected_tool]
```

Retrieves the required tool object.

```python
.ainvoke(selected_tool_args)
```

Executes the tool asynchronously using the arguments selected by the LLM.

### Example

```text
Selected Tool: multiply

Arguments:
a = 12
b = 15

Tool Result:
180
```

At this point, the MCP Server has executed the requested operation.

But we still need to send the result back to the LLM.

---

# PART 6: Returning the Tool Result to the LLM

## 18. What Is a ToolMessage?

LangChain provides a message type called:

```python
ToolMessage
```

It represents the result of a tool execution.

### Import

```python
from langchain_core.messages import ToolMessage
```

### Creating a ToolMessage

```python
tool_message = ToolMessage(
    content=str(tool_result),
    tool_call_id=selected_tool_id
)
```

### Important Parameters

| Parameter | Meaning |
|---|---|
| `content` | Result returned by the tool. |
| `tool_call_id` | ID linking the result to the original tool call. |

### Why Is tool_call_id Important?

The LLM may request multiple tools.

Each tool call has an ID.

The corresponding ToolMessage uses that ID so the model can identify which result belongs to which tool request.

The lecture initially encountered an error because the required tool-call ID was missing.

After adding `tool_call_id`, the issue was resolved.

---

## 19. Sending the Complete History Back to the LLM

Now we have:

1. Original user prompt.
2. Initial LLM response containing the tool call.
3. ToolMessage containing the execution result.

We send all three back to the LLM.

### Code

```python
final_response = await llm_with_tools.ainvoke([
    ("human", prompt),
    response,
    tool_message
])

print(final_response.content)
```

### Why Send the Complete History?

Because the LLM needs to know:

- What the user asked.
- Which tool it requested.
- What result the tool returned.

It can then generate a final natural-language answer.

### Final Output

```text
The product of 12 and 15 is 180.
```

---

## 20. Complete MCP Tool Execution Flow

```text
                  USER PROMPT
                       |
                       v
                      LLM
                       |
                       v
              TOOL SELECTION
                       |
                       v
                 TOOL CALL
                (name, args)
                       |
                       v
                 MCP CLIENT
                       |
                       v
                  MCP SERVER
                       |
                       v
                 TOOL EXECUTES
                       |
                       v
                 TOOL RESULT
                       |
                       v
                  ToolMessage
                       |
                       v
                      LLM
                       |
                       v
                FINAL RESPONSE
```

### Summary

1. User sends a prompt.
2. LLM decides which tool to call.
3. The application extracts the tool details.
4. MCP Client executes the selected tool.
5. MCP Server returns the result.
6. The application creates a ToolMessage.
7. The result is sent back to the LLM.
8. LLM generates the final answer.

---

# PART 7: Handling Normal Questions

## 21. Problem: Not Every Question Needs a Tool

Suppose the user asks:

```text
What is the capital of India?
```

The LLM can answer directly.

No MCP tool is required.

But our initial code assumed that every response contained a tool call.

It attempted to access:

```python
response.tool_calls[0]
```

If the list was empty, the code failed.

### Solution

Check whether the LLM generated any tool calls.

```python
if not response.tool_calls:
    print(response.content)
    return
```

### Explanation

If no tool calls exist:

- Print the LLM's response.
- Exit the function.

Otherwise:

- Continue with the tool execution logic.

This allows the chatbot to handle both normal conversations and MCP tool calls.

---

# PART 8: Handling Multiple Tool Calls

## 22. Problem: The LLM May Request Multiple Tools

Our original implementation handled only one tool call.

But an LLM can request multiple tools in a single response.

For example, it might request:

```text
add
multiply
subtract
```

To handle this, we use a loop.

### Code

```python
tool_messages = []

for tool_call in response.tool_calls:

    selected_tool = tool_call["name"]
    selected_tool_args = tool_call["args"]
    selected_tool_id = tool_call["id"]

    tool_result = await named_tools[selected_tool].ainvoke(
        selected_tool_args
    )

    tool_message = ToolMessage(
        content=str(tool_result),
        tool_call_id=selected_tool_id
    )

    tool_messages.append(tool_message)
```

### Sending Multiple Tool Results

```python
final_response = await llm_with_tools.ainvoke([
    ("human", prompt),
    response,
    *tool_messages
])
```

### Benefit

The application can now handle multiple tool calls from a single LLM response.

**Note:** This implementation executes the requested tools one after another. It does not automatically run them in parallel.

---

# PART 9: Complete Basic MCP Client

## 23. client1.py

The following code combines the main logic developed in the lecture.

It connects to the Local Maths MCP Server, fetches its tools, binds them to an LLM, executes tool calls, and returns the final response.

```python
import asyncio

from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.messages import ToolMessage
from langchain_mcp_adapters.client import MultiServerMCPClient

load_dotenv()


async def main():

    # Step 1: Configure the MCP Server
    servers = {
        "math": {
            "transport": "stdio",
            "command": "uv",
            "args": [
                "run",
                "fastmcp",
                "run",
                "PATH_TO_MATH_SERVER/main.py"
            ]
        }
    }

    # Step 2: Create MCP Client
    client = MultiServerMCPClient(servers)

    # Step 3: Fetch tools
    tools = await client.get_tools()

    # Step 4: Create tool lookup dictionary
    named_tools = {
        tool.name: tool
        for tool in tools
    }

    # Step 5: Initialize LLM
    llm = ChatOpenAI(model="gpt-5")

    # Step 6: Bind tools to LLM
    llm_with_tools = llm.bind_tools(tools)

    # Step 7: User prompt
    prompt = (
        "Using the math tool, "
        "what is the product of 12 and 15?"
    )

    # Step 8: Ask LLM
    response = await llm_with_tools.ainvoke(prompt)

    # Step 9: Handle normal response
    if not response.tool_calls:
        print(response.content)
        return

    # Step 10: Execute tool calls
    tool_messages = []

    for tool_call in response.tool_calls:

        selected_tool = tool_call["name"]
        selected_tool_args = tool_call["args"]
        selected_tool_id = tool_call["id"]

        tool_result = await named_tools[
            selected_tool
        ].ainvoke(selected_tool_args)

        tool_message = ToolMessage(
            content=str(tool_result),
            tool_call_id=selected_tool_id
        )

        tool_messages.append(tool_message)

    # Step 11: Send tool results to LLM
    final_response = await llm_with_tools.ainvoke([
        ("human", prompt),
        response,
        *tool_messages
    ])

    # Step 12: Print final response
    print(final_response.content)


if __name__ == "__main__":
    asyncio.run(main())
```

### Run the Client

```bash
uv run client1.py
```

### Expected Result

```text
The product of 12 and 15 is 180.
```

This demonstrates the basic MCP Client workflow.

---

# PART 10: Connecting Multiple MCP Servers

## 24. Adding a Remote Expense Tracker MCP Server

Now we will connect a second MCP Server.

The first server is local.

The second server is remote.

### Local Server

```python
"math": {
    "transport": "stdio",
    "command": "uv",
    "args": [
        "run",
        "fastmcp",
        "run",
        "PATH_TO_MATH_SERVER/main.py"
    ]
}
```

### Remote Server

```python
"expense": {
    "transport": "streamable_http",
    "url": "YOUR_REMOTE_MCP_SERVER_URL"
}
```

### Complete Configuration

```python
servers = {
    "math": {
        "transport": "stdio",
        "command": "uv",
        "args": [
            "run",
            "fastmcp",
            "run",
            "PATH_TO_MATH_SERVER/main.py"
        ]
    },

    "expense": {
        "transport": "streamable_http",
        "url": "YOUR_REMOTE_MCP_SERVER_URL"
    }
}
```

Replace the URL with your deployed Expense Tracker MCP Server URL from FastMCP Cloud.

### What Changes in Our Code?

Only the server configuration needs to be extended.

The rest of the main logic remains largely the same.

```python
client = MultiServerMCPClient(servers)

tools = await client.get_tools()
```

The client now fetches tools from both servers.

---

## 25. Available Tools After Connecting Both Servers

### Maths Tools

```text
add
subtract
multiply
divide
power
modulus
```

### Expense Tracker Tools

```text
add_expense
list_expenses
summarize_expenses
```

### Displaying Tool Names

```python
print([tool.name for tool in tools])
```

### Testing the Expense Tracker

Example prompt:

```python
prompt = "Add an expense of ₹800 for groceries on 4 November."
```

### Execution Flow

```text
User Prompt
     |
     v
    LLM
     |
     v
Select add_expense
     |
     v
MCP Client
     |
     v
Remote Expense MCP Server
     |
     v
Expense Database
     |
     v
Tool Result
     |
     v
    LLM
     |
     v
Final Response
```

The LLM identifies the relevant tool, and the client executes it through the Remote MCP Server.

---

# PART 11: Connecting Third-Party MCP Servers

## 26. Adding a Manim MCP Server

The instructor also demonstrates connecting a third-party Manim MCP Server.

Manim is used to create mathematical animations.

The important point is that this server was created by another developer.

We can still connect it to our own MCP Client.

### How?

Add the third-party server's configuration to the existing `servers` dictionary.

Conceptually:

```python
servers = {
    "math": {
        # Local Maths Server Configuration
    },

    "expense": {
        # Remote Expense Server Configuration
    },

    "manim": {
        # Third-Party Manim Server Configuration
    }
}
```

Use the actual Manim MCP Server command and arguments from its setup instructions.

The lecture reuses the configuration demonstrated in an earlier video rather than providing all of its source code in the transcript.

### New Tools

After connecting the Manim MCP Server, the client discovers additional tools.

The lecture demonstrates tools for:

- Executing Manim code.
- Cleaning up temporary Manim files.

### Example Prompt

```text
Draw a triangle rotating in place using the Manim tool.
```

The LLM selects the relevant Manim tool, and the server generates the animation.

### Key Concept

The MCP Client can integrate tools from:

- Servers we created ourselves.
- Local MCP Servers.
- Remote MCP Servers.
- Third-party MCP Servers.

We do not need to rewrite the functionality of each server.

We only need its compatible configuration and access.

---

# PART 12: Building a Streamlit GUI

## 27. Why Use Streamlit?

Until now, we have interacted with our MCP Client through the terminal.

The instructor's final goal is to create a chatbot with a graphical user interface.

For this, the lecture uses **Streamlit**.

### What Is Streamlit?

Streamlit is a Python framework for creating interactive web applications.

### Our GUI Chatbot

The application will support:

1. Normal conversations.
2. Maths calculations through the Local MCP Server.
3. Expense tracking through the Remote MCP Server.
4. Animation generation through the Manim MCP Server.

---

## 28. Creating client2.py

The instructor creates another file:

```text
client2.py
```

This file contains the Streamlit version of the MCP Client.

### Important Point

The underlying MCP logic remains largely the same.

We still:

1. Configure MCP Servers.
2. Create the MultiServerMCPClient.
3. Fetch tools.
4. Initialize the LLM.
5. Bind the tools.
6. Process user prompts.
7. Execute requested tools.
8. Return the final response.

The major change is the user interface.

Instead of receiving prompts through terminal code, the user interacts with a Streamlit chatbot.

### Note

The lecture does not walk through the complete Streamlit implementation line by line.

It focuses on MCP Client logic and demonstrates the finished GUI.

---

## 29. Running the Streamlit Application

Run:

```bash
streamlit run client2.py
```

Or, using UV:

```bash
uv run streamlit run client2.py
```

Streamlit starts the web application.

### Example Queries

#### Normal Conversation

```text
Hello! How are you?
```

The LLM can respond without using MCP tools.

#### Expense Tracking

```text
Show me all expenses from the last two weeks.
```

The LLM selects the appropriate Expense Tracker tool.

#### Mathematical Calculation

```text
What is the remainder when a number is divided by 23?
```

The LLM can use the Maths MCP Server when a complete calculation is supplied.

#### Animation Generation

```text
Make an animation video of a circle using the Manim tool.
```

The LLM selects the Manim MCP Server tool.

---

# PART 13: Final Application Architecture

## 30. Complete MCP Chatbot Architecture

```text
                         USER
                           |
                           v
                    STREAMLIT GUI
                           |
                           v
                     CHATBOT LOGIC
                           |
                           v
                          LLM
                           |
                           v
                   TOOL SELECTION
                           |
                           v
                 MultiServerMCPClient
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
        MATH SERVER   EXPENSE SERVER   MANIM SERVER
             |             |             |
             v             v             v
           STDIO          HTTP          STDIO
             |             |             |
             v             v             v
       Arithmetic      Expense DB      Animation
       Operations                       Generation
             |             |             |
             +-------------+-------------+
                           |
                           v
                       TOOL RESULTS
                           |
                           v
                          LLM
                           |
                           v
                     FINAL RESPONSE
                           |
                           v
                    STREAMLIT GUI
                           |
                           v
                         USER
```

---

## 31. Project Structure

A possible project structure based on the lecture:

```text
mcp-client-project/
|
|-- client1.py
|-- client2.py
|-- .env
|-- .gitignore
|-- pyproject.toml
|-- uv.lock
|-- README.md
```

### File Descriptions

| File | Purpose |
|---|---|
| `client1.py` | Terminal-based MCP Client. |
| `client2.py` | Streamlit-based MCP Chatbot. |
| `.env` | Stores API keys. |
| `.gitignore` | Prevents sensitive and unnecessary files from being committed. |
| `pyproject.toml` | Project dependencies and configuration. |
| `uv.lock` | Dependency lock file. |
| `README.md` | Project documentation. |

### Recommended .gitignore

```gitignore
.env
.venv/
__pycache__/
*.pyc
```

Never upload API keys to GitHub.

---

# PART 14: Important Concepts

## 32. Important Functions and Classes

| Function / Class | Purpose |
|---|---|
| `MultiServerMCPClient` | Configures connections to multiple MCP Servers. |
| `client.get_tools()` | Retrieves tools from MCP Servers. |
| `ChatOpenAI()` | Initializes the LLM. |
| `llm.bind_tools()` | Makes tool definitions available to the LLM. |
| `llm.ainvoke()` | Asynchronously invokes the LLM. |
| `response.tool_calls` | Contains tool calls requested by the LLM. |
| `tool.ainvoke()` | Executes a selected tool asynchronously. |
| `ToolMessage()` | Represents a tool execution result. |
| `asyncio.run()` | Runs the asynchronous main function. |
| `load_dotenv()` | Loads environment variables from a `.env` file. |

---

## 33. MCP Client vs MCP Server

| MCP Client | MCP Server |
|---|---|
| Establishes communication with MCP Servers. | Exposes capabilities through MCP. |
| Discovers available tools. | Defines and provides tools. |
| Sends tool execution requests. | Executes requested tools. |
| Receives tool results. | Returns execution results. |
| Can connect to multiple servers. | Can serve compatible MCP Clients. |

---

## 34. Tool Binding vs Tool Execution

These are two different operations.

### Tool Binding

```python
llm_with_tools = llm.bind_tools(tools)
```

This makes the tool definitions available to the LLM.

It does not execute the tools.

### Tool Execution

```python
result = await named_tools[tool_name].ainvoke(arguments)
```

This actually executes the selected tool.

### Difference

| Tool Binding | Tool Execution |
|---|---|
| Gives the LLM knowledge of available tools. | Runs the selected tool. |
| Happens before tool selection. | Happens after the LLM requests a tool. |
| Uses `bind_tools()`. | Uses `ainvoke()` on the tool. |

---

## 35. Why Do We Need ToolMessage?

The tool execution result must be returned to the LLM in a structured format.

For this, we use:

```python
ToolMessage(
    content=str(tool_result),
    tool_call_id=selected_tool_id
)
```

The `tool_call_id` links the result to the tool call requested by the LLM.

Without correctly associating the result with its tool call, the model/provider may reject the message sequence.

---

## 36. Limitations of the Basic Implementation

The lecture builds a basic working MCP Client.

Some improvements that would be needed for a more complete application include:

- Better error handling for failed tool calls.
- Proper handling of repeated tool-calling rounds.
- Authentication for protected remote servers.
- Persistent chat history.
- More robust server connection management.

The implementation shown in this README handles multiple tool calls in one model response, but it does not implement a full repeated tool-calling loop.

---

# PART 15: Key Takeaways

## 37. What We Learned

1. MCP Clients can be created independently of Claude Desktop.
2. LangChain MCP Adapters provide a simple way to integrate MCP Servers into LangChain applications.
3. `MultiServerMCPClient` allows us to configure multiple MCP Servers.
4. Local MCP Servers commonly use STDIO transport.
5. Remote MCP Servers can use Streamable HTTP.
6. `client.get_tools()` retrieves tools from connected servers.
7. `bind_tools()` makes MCP tools available to an LLM.
8. The LLM selects tools and generates tool-call requests.
9. The application is responsible for executing the selected tools.
10. `ToolMessage` is used to return tool results to the LLM.
11. A tool-call ID connects each tool result with the original request.
12. Normal conversations should be handled without forcing tool execution.
13. Multiple tool calls can be processed using a loop.
14. Third-party MCP Servers can also be integrated into our chatbot.
15. Streamlit can provide a GUI for the MCP Client.

---

## 38. Final Summary

In this lecture, we built our own MCP Client using **LangChain MCP Adapters**.

We started by connecting a Local Maths MCP Server and learning the complete tool-calling workflow.

Then we connected a Remote Expense Tracker MCP Server.

After that, we demonstrated how to connect a third-party Manim MCP Server.

Finally, we integrated the MCP Client logic into a Streamlit chatbot.

### Final Result

A custom AI chatbot capable of:

- Normal conversations.
- Mathematical calculations.
- Expense tracking.
- Animation generation.
- Communicating with multiple MCP Servers.

This completes the basic MCP learning journey covering:

**Why MCP → MCP Architecture → MCP Lifecycle → Local Servers → Remote Servers → Custom MCP Clients**

---

## 39. What's Next?

The instructor plans to return to the LangGraph playlist.

Advanced MCP topics mentioned for future coverage include:

- MCP Authentication.
- MCP Sampling.
- MCP Elicitation.
- Other advanced MCP concepts.

These topics will help extend the basic MCP Client and Server architecture into more advanced applications.
