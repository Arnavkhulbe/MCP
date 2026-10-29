# MCP Architecture — Complete Notes

## 1. Introduction

This README summarizes the lecture on **Model Context Protocol (MCP)** architecture.

The lecture focuses on three broad topics:
1. **MCP architecture**
2. **The MCP data layer**
3. **The MCP transport layer**

The main goal is to understand how an AI application communicates with external servers, what servers can offer, how messages are structured, and how those messages travel between the client and server.

---

## 2. The Basic MCP Architecture

The simplest explanation begins with two main entities:
* **Host:** The AI application the user interacts with.
* **Server:** A component that provides access to particular tools or capabilities.

Examples of hosts mentioned in the lecture include **Claude Desktop**, **Cursor**, and custom-built AI chatbots.

Examples of servers include **GitHub**, **Slack**, and **Google Drive** servers.

A host may use an LLM such as a model from OpenAI, Anthropic, or Google. When the model needs information or an operation that is not available directly, the host can use an appropriate server.

### Simplified Architecture Flow

```text
User
  │
  ▼
Host / AI Application
  │
  ▼
LLM
  │
  │ (Needs an external capability)
  ▼
External Server
  │
  ▼
Result returned to the Host
  │
  ▼
LLM generates an answer
  │
  ▼
User sees the answer
```

> **Note:** This is a simplified view. In actual MCP architecture, the host does not communicate directly with the server; an **MCP client** coordinates communication between them.

---

## 3. Host, Client, and Server

### Host
The host is the primary AI application the user interacts with. Its responsibilities include:
* Receiving the user's prompt.
* Sending the relevant prompt and context to an LLM.
* Using the model's decision to determine whether a tool or external capability is needed.
* Coordinating the MCP client.
* Providing the final result back to the user.

### MCP Client
The MCP client is the component inside the host that directly communicates with an MCP server:
* Translates host requirements into MCP protocol requests.
* Sends requests in the standardized JSON-RPC format.
* Processes and handles the server's responses.

### MCP Server
An MCP server exposes capabilities that a host can use (e.g., GitHub, Slack, Google Drive, or databases):
* Handles the requested operation.
* Returns structured results to the client.

### Communication Path

```text
User
  │
  ▼
Host (AI Application)
  │
  ▼
MCP Client
  │
  │ (MCP Request)
  ▼
MCP Server
  │
  │ (Result)
  ▼
MCP Client
  │
  ▼
Host / LLM
  │
  ▼
User
```

> **Key takeaway:** The host communicates with a server strictly through an MCP client using standard MCP protocol messages.

---

## 4. Example: Finding Recent GitHub Commits

> **User Query:** *"Are there any new commits in my GitHub repository?"*

1. **Prompt reception:** The user sends the question to the host.
2. **LLM evaluation:** The host sends the prompt and context to the LLM.
3. **Intent detection:** The LLM determines that it requires GitHub repository information.
4. **Client routing:** The host coordinates with the relevant GitHub MCP client.
5. **Execution request:** The client sends an MCP-compatible request to the GitHub MCP server.
6. **Data retrieval:** The server checks the repository and returns commit information.
7. **Client handling:** The client returns the structured result back to the host.
8. **Synthesis:** The LLM uses the result to form an answer.
9. **Display:** The host displays the formatted response to the user.

---

## 5. One Client per Server Connection

The architecture establishes a **one-to-one** relationship between an MCP client and an MCP server. A host connecting to multiple servers maintains a distinct client instance for each:

```text
                 Host
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
     Client 1  Client 2  Client 3
        │         │         │
        ▼         ▼         ▼
     GitHub     Slack   Google Drive
     Server    Server      Server
```

### Architectural Benefits

* **Decoupling:** Each connection is isolated. A fault or failure on one integration does not cascade to other independent connections.
* **Parallel Execution:** Where supported, the host can dispatch concurrent queries across independent clients simultaneously.
* **Scalability:** Adding capabilities is modular—configure and instantiate a new client instance without rewriting core orchestration logic.

---

## 6. MCP Server Primitives

A server exposes three main primitives:
1. **Tools**
2. **Resources**
3. **Prompts**

### 6.1 Tools
Actions or executable operations that an AI application requests the server to perform dynamically.
* *Examples:* List GitHub issues, push commits, search Google Drive files, run a database query.
* *Best used for:* Real-time, mutable, or computational operations.

### 6.2 Resources
Structured, read-only data sources or documents exposed by a server.
* *Examples:* Reading a `README.md` file, fetching a database schema, reading local log files.
* *Best used for:* Context enrichment and reference material.

### 6.3 Prompts
Predefined prompt templates and instructions provided by the server to guide the LLM toward structured outputs.

#### Example: Issue Generation

Without server prompt guidance, an LLM might generate:
```text
Title: Login bug
Body: The login button doesn't work.
```

With an `issue_report` prompt template loaded from the server:
```markdown
Title: Login button does not respond

Steps to reproduce:
1. Open the login page.
2. Enter valid credentials.
3. Click the Login button.

Expected behavior:
The user should be logged in.

Actual behavior:
Nothing happens after clicking the button.

Environment:
Chrome on macOS.
```

### Primitive Summary

| Primitive | Main Purpose | Example |
| :--- | :--- | :--- |
| **Tools** | Perform an action or operation | Create a GitHub issue |
| **Resources** | Provide data to read | Read a repository `README.md` |
| **Prompts** | Provide reusable prompt templates | Generate a structured bug report |

---

## 7. Standard Operations for Working with Primitives

MCP defines standard JSON-RPC methods for capability discovery and execution.

### 7.1 Tool Methods
* `tools/list`: Discovers available tools, including their names, descriptions, and JSON input schemas.
* `tools/call`: Executes a tool with provided arguments.

### 7.2 Resource Methods
* `resources/list`: Discovers available resources.
* `resources/read`: Reads the content of a specific resource via its URI.
* `resources/subscribe`: Requests update notifications for a supported resource.
* `resources/unsubscribe`: Terminates an active resource subscription.

### 7.3 Prompt Methods
* `prompts/list`: Lists prompt templates provided by the server.
* `prompts/get`: Retrieves a specific prompt template, optionally populated with arguments.

### Methods at a Glance

| Primitive | Method | Purpose |
| :--- | :--- | :--- |
| **Tools** | `tools/list` | Discover available tools |
| **Tools** | `tools/call` | Execute a tool |
| **Resources** | `resources/list` | Discover available resources |
| **Resources** | `resources/read` | Read a resource |
| **Resources** | `resources/subscribe` | Subscribe to supported resource changes |
| **Resources** | `resources/unsubscribe` | Stop a resource subscription |
| **Prompts** | `prompts/list` | Discover prompt templates |
| **Prompts** | `prompts/get` | Retrieve a prompt template |

---

## 8. The MCP Data Layer

The **data layer** defines message structure, rules, errors, and validation for client-server communication. It represents the language and grammar of MCP and is built upon **JSON-RPC 2.0**.

---

## 9. What Is JSON?

**JSON (JavaScript Object Notation)** is an open standard text format used to represent structured data using key-value pairs and arrays.

```json
{
  "name": "Arnav",
  "language": "Python",
  "active": true
}
```

---

## 10. What Is RPC?

**RPC (Remote Procedure Call)** allows a computer program to cause a subroutine or procedure to execute in another address space or machine without the programmer explicitly coding the low-level network details.

```text
Client asks: Run add(2, 3)
         │
         ▼
Remote server executes the method
         │
         ▼
Server returns: 5
```

---

## 11. What Is JSON-RPC 2.0?

JSON-RPC 2.0 combines RPC concepts with JSON serialization, providing standard formats for requests, responses, notifications, and errors.

### Request Format
```json
{
  "jsonrpc": "2.0",
  "method": "add",
  "params": {
    "a": 2,
    "b": 3
  },
  "id": 1
}
```

* `jsonrpc`: Specification version (`"2.0"`).
* `method`: Name of the procedure to invoke.
* `params`: Input payload / parameters.
* `id`: Correlation identifier to match responses to requests.

### Successful Response Format
```json
{
  "jsonrpc": "2.0",
  "result": 5,
  "id": 1
}
```

### Error Response Format
```json
{
  "jsonrpc": "2.0",
  "error": {
    "code": -32601,
    "message": "Method not found"
  },
  "id": 1
}
```

---

## 12. JSON-RPC Messages Used in MCP

### 12.1 Discovering Tools (`tools/list`)

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list"
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [
      {
        "name": "list_issues",
        "description": "List issues in a repository"
      },
      {
        "name": "list_pull_requests",
        "description": "List pull requests in a repository"
      }
    ]
  }
}
```

### 12.2 Calling a Tool (`tools/call`)

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "list_issues",
    "arguments": {
      "owner": "example-owner",
      "repo": "example-repo"
    }
  }
}
```

**Response:**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Issue #1: Login button is not responding"
      }
    ]
  }
}
```

### 12.3 Listing and Reading Resources

**Request:**
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "resources/list"
}
```

Specific files or data can subsequently be accessed via `resources/read` using the assigned URI.

### 12.4 Batch Requests Note
Standard JSON-RPC 2.0 specifies that multiple operations can be sent as an array batch. **Note:** MCP does not implement standard JSON-RPC batching.

### 12.5 Notifications
Notifications omit the `"id"` property and do not expect an operational response:

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/initialized"
}
```

* **Request:** Contains an `id`, requires a matching response.
* **Notification:** Contains no `id`, unidirectional one-way event.

---

## 13. Why Does MCP Use JSON-RPC 2.0?

1. **Lightweight Message Structure:** Easily serialized, human-readable, and straightforward to inspect and debug.
2. **Bidirectional Capability:** Enables both client-to-server and server-to-client notifications or operations.
3. **Transport Independence:** Decouples message syntax from network transmission mechanisms.
4. **Standardized Notification Model:** Native support for one-way event broadcasting without response overhead.

### Comparison: REST vs. JSON-RPC

| Aspect | REST-Style APIs | JSON-RPC |
| :--- | :--- | :--- |
| **Main Model** | Resources and HTTP verbs (`GET`, `POST`, `DELETE`) | Explicit named methods (`tools/list`) |
| **Request Format** | Varied per API / endpoint path | Uniform JSON-RPC schema |
| **Correlation** | Dependent on HTTP session / header state | Explicit request `id` tracking |
| **Notifications** | Requires Webhooks, SSE, or WebSockets | Built-in notification construct |
| **Transport** | Strongly coupled to HTTP | Transport-agnostic |

---

## 14. The MCP Transport Layer

The **transport layer** governs how serialized JSON-RPC messages move between client and server.

```text
Data Layer      → What does the message look like? (JSON-RPC)
Transport Layer → How does the message get there?  (stdio, HTTP)
```

### 14.1 Local MCP Servers (`stdio`)

Local servers execute directly on the host machine. The primary transport mechanism is **`stdio` (Standard Input / Output)**.

* The host starts the MCP server as a sub-process.
* Requests travel to the server via **`stdin`**.
* Responses travel back to the client via **`stdout`**.
* Diagnostic logs are sent via **`stderr`** to prevent corrupting the JSON stream.

```text
Local Machine
┌───────────────────────────────────────┐
│ Host Application                      │
│   │                                   │
│   ▼                                   │
│ MCP Client                            │
│   │                                   │
│   │ stdin / stdout (stdio)            │
│   ▼                                   │
│ Local MCP Server                      │
│   │                                   │
│   ▼                                   │
│ Local Filesystem / Local Environment  │
└───────────────────────────────────────┘
```

### 14.2 Remote MCP Servers (Streamable HTTP)

Remote servers run on separate network machines. 

* The current MCP specification designates **Streamable HTTP** as the standard remote transport.
* Replaces legacy HTTP + Server-Sent Events (SSE) models with modern bidirectional HTTP streaming capabilities.

```text
Host Computer                     Remote Server
┌────────────────────┐            ┌────────────────────┐
│ Host Application   │            │ MCP Server         │
│   │                │   HTTP     │   │                │
│   ▼                ├───────────►│   ▼                │
│ MCP Client         │            │ External Service / │
│                    │◄───────────┤ API                │
└────────────────────┘            └────────────────────┘
```

### Local vs. Remote Servers

| Aspect | Local Server | Remote Server |
| :--- | :--- | :--- |
| **Location** | Host machine | Remote network server |
| **Transport** | `stdio` | Streamable HTTP |
| **Communication** | Process pipes (`stdin` / `stdout`) | Network endpoints |
| **Example** | Local filesystem server | GitHub / Slack cloud server |
| **Key Concerns** | Local file / process permissions | Network latency, TLS, authentication |

---

## 15. Data Layer vs. Transport Layer

| Layer | Responsibility | Key Question |
| :--- | :--- | :--- |
| **Data Layer** | Defines message schemas, methods, errors, and validation | *What does the message look like?* |
| **Transport Layer** | Manages physical/logical transmission between peers | *How does the message get there?* |

---

## 16. Complete MCP Architecture Summary

```text
                         User
                          │
                          ▼
                 ┌──────────────────┐
                 │ Host Application │
                 │     + LLM        │
                 └────────┬─────────┘
                          │
              ┌───────────┴───────────┐
              │                       │
              ▼                       ▼
       ┌─────────────┐         ┌─────────────┐
       │ MCP Client 1│         │ MCP Client 2│
       └──────┬──────┘         └──────┬──────┘
              │                       │
              │ JSON-RPC messages     │ JSON-RPC messages
              │ via stdio             │ via Streamable HTTP
              ▼                       ▼
       ┌─────────────┐         ┌─────────────┐
       │ MCP Server 1│         │ MCP Server 2│
       └──────┬──────┘         └──────┬──────┘
              │                       │
              ▼                       ▼
          Filesystem                GitHub
```

---

## 17. Key Terms

| Term | Meaning |
| :--- | :--- |
| **Host** | The primary AI application the user interacts with (e.g., Claude Desktop, Cursor). |
| **MCP Client** | Component coordinating JSON-RPC communication to a single server instance. |
| **MCP Server** | Process exposing specialized tools, resources, or prompts. |
| **Primitive** | Core server capability type: Tools, Resources, or Prompts. |
| **Tool** | Server action that an LLM can invoke to execute tasks or query data dynamically. |
| **Resource** | Read-only structured data accessible by URI. |
| **Prompt** | Reusable prompt templates used to structure LLM responses. |
| **Data Layer** | Protocol specification defining message formatting (JSON-RPC 2.0). |
| **Transport Layer** | Underlying pipeline moving bytes between client and server (`stdio` / HTTP). |
| **stdio** | Standard input/output pipe used for local sub-process execution. |
| **Streamable HTTP** | Modern standard transport for remote MCP servers. |

---

## 18. Final Takeaways

1. **Host:** Coordinates user prompts, models, and tool delegations.
2. **Client:** Handles one-to-one communication with a designated server.
3. **Server:** Exposes capabilities to the host ecosystem.
4. **Decoupled Architecture:** One client instance per server ensures fault isolation and modular scaling.
5. **Three Primitives:** Tools (actions), Resources (data), and Prompts (templates).
6. **Core Methods:** `tools/list`, `tools/call`, `resources/list`, `resources/read`, `prompts/list`, `prompts/get`.
7. **Grammar & Pipe Separation:** JSON-RPC 2.0 defines the message structure; `stdio` and Streamable HTTP define how the messages travel.
