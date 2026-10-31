# MCP Lifecycle (Model Context Protocol)

## 1. What is MCP Lifecycle?

MCP Lifecycle describes the complete sequence of steps followed by an MCP Host, Client, and Server to establish, use, and terminate a connection during a session.

### What is a Session?

A Session is a continuous connection between an MCP Client and an MCP Server.

For example:
- Host: Claude Desktop
- Client: MCP Client running inside Claude Desktop
- Server: GitHub MCP Server

When Claude Desktop starts, it connects to the GitHub MCP Server.

As long as the connection remains active, both parties can communicate.

### Three Stages of MCP Lifecycle

MCP Lifecycle consists of three main phases:

1. **Initialization Phase** – Establishing the connection and negotiating capabilities.
2. **Operation Phase** – Discovering and using available tools, resources, and prompts.
3. **Shutdown Phase** – Terminating the connection between Client and Server.

---

# 2. Phase 1 – Initialization

Initialization is the first interaction between the MCP Client and MCP Server.

The main purpose is to establish communication and agree on how both parties will communicate.

Two important things happen during initialization:

1. Protocol Version Negotiation.
2. Capability Negotiation.

## Step 1: Client Sends Initialize Request

The client sends an `initialize` request to the server.

The request contains three important things:

### A. Protocol Version

The client tells the server which MCP protocol version it supports.

Example:

```json
{
  "protocolVersion": "2025-03-26"
}
```

**Why is this necessary?**

MCP is continuously evolving, and different clients and servers might support different versions.

Both parties need to agree on a compatible version before normal communication starts.

### B. Client Capabilities

The client tells the server which features it supports.

Three important client capabilities:

| Capability | Meaning |
|---|---|
| Roots | Allows the client to provide filesystem boundaries or directories that the server can work within. |
| Sampling | Allows the server to request AI/model assistance through the client. |
| Elicitation | Allows the server to request additional information from the user through the client. |

### C. Client Implementation Information

The client shares its identification information:

- Client name
- Client version

This helps the server identify which client is connecting.

## Step 2: Server Sends Initialize Response

After receiving the initialization request, the server responds.

The server also shares three things:

### A. Protocol Version

The server tells the client which MCP protocol version it supports.

### B. Server Capabilities

The server tells the client which features it provides.

| Capability | Meaning |
|---|---|
| Tools | Functions that perform actions. |
| Resources | Data or information that can be accessed. |
| Prompts | Reusable prompt templates. |

### C. Server Implementation Information

The server shares its name and version.

Example:

```json
{
  "name": "filesystem",
  "version": "20"
}
```

## Step 3: Client Sends Initialized Notification

After receiving the server's response, the client sends an initialized notification.

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/initialized"
}
```

**Important Points:**

- There is no `id` in this message.
- It is a notification, not a request.
- Notifications do not expect a response.

This message tells the server that initialization is complete and normal communication can begin.

### Complete Initialization Flow

```text
       CLIENT
          |
          |  initialize request
          |
          v
       SERVER
          |
          |  initialize response
          |
          v
       CLIENT
          |
          |  notifications/initialized
          |
          v
   INITIALIZATION COMPLETE
```

## Two Important Rules During Initialization

### Rule 1: Client Side

Before receiving the server's response to the `initialize` request, the client must not send other requests except `ping`.

For example, the client cannot call a tool before initialization is complete.

### Rule 2: Server Side

Before receiving the `notifications/initialized` notification, the server must not process normal requests.

The server can send certain messages, such as:
- Ping requests.
- Logging messages.

**Why are these rules important?**

Both parties must complete their handshake before starting normal communication.

---

# 3. Version Negotiation

Version Negotiation handles situations where the client and server support different MCP protocol versions.

Suppose:

- Client supports Version A.
- Server responds with Version B.

The client checks whether Version B is included in its supported protocol versions.

### Two Possible Cases

**Case 1: Compatible Version**

If the server's version is supported by the client:

- Client accepts the version.
- Client sends `notifications/initialized`.
- Communication continues.

**Case 2: Incompatible Version**

If the server's version is not supported:

- Client cannot establish a compatible session.
- Client disconnects.
- Normal communication does not continue.

### Version Negotiation Flow

```text
Client sends Version A
          |
          v
Server responds with Version B
          |
          v
Does Client support Version B?
          |
     +----+----+
     |         |
    YES        NO
     |         |
     v         v
  Accept    Disconnect
  Version
     |
     v
 Continue Communication
```

**Remember:** Different versions do not automatically mean connection failure. Compatibility is what matters.

---

# 4. Capability Negotiation

Capability Negotiation means the client and server exchange information about which MCP features they support.

This helps both parties understand what they can request from each other during the session.

There are two main categories of capabilities.

## A. Client Capabilities

The lecture discusses three major client capabilities.

### 1. Roots

Roots allow the client to provide filesystem boundaries or directories to the server.

Example:

```text
my_project/
    |
    |-- main.py
    |-- server.py
    |-- README.md
```

The client can provide the project directory as a root.

This tells the filesystem server which directory it is allowed to work within.

### 2. Sampling

Sampling allows the server to request AI/model assistance through the client.

MCP is a bidirectional protocol.

Communication is not restricted to:

```text
Client → Server
```

The server can also request assistance from the client.

**Example:**

1. User asks the server to summarize thousands of documents.
2. Server needs AI assistance.
3. Server requests model assistance through the client.
4. Client uses its AI model to generate the summary.
5. Client returns the result to the server.

Flow:

```text
       MCP SERVER
            |
            | Sampling Request
            v
       MCP CLIENT
            |
            | Uses AI Model
            v
       AI MODEL
            |
            | Generated Result
            v
       MCP CLIENT
            |
            | Returns Result
            v
       MCP SERVER
```

### 3. Elicitation

Elicitation allows the server to request additional information from the user through the client.

**Example:**

User asks:

"Fetch all repositories from my GitHub account."

But the server needs additional information, such as an API key.

The server can request the missing information through the client.

The client can then ask the user to provide it.

**Difference:**

- Sampling = Request AI assistance.
- Elicitation = Request additional user information.

## B. Server Capabilities

The lecture discusses four major server capabilities.

| Capability | Meaning |
|---|---|
| Tools | Functions that perform actions. |
| Resources | Data or information that can be accessed. |
| Prompts | Reusable prompt templates. |
| Logging | Server sends logging messages to the client. |

### Logging

Logging allows the server to send logging messages to the client.

Example of a long-running task:

1. Form filled.
2. Payment processed.
3. Confirmation received.

The server can send logging messages to communicate these events.

This helps the client understand what is happening during execution.

## C. Sub-Capabilities

Two additional capabilities discussed in the lecture are:

### 1. List Changed

Suppose a server initially has five tools.

Later, a new tool is added.

The server can send a notification to the client informing it that the available tool list has changed.

The same concept applies to resources and prompts where supported.

### 2. Subscribe

Suppose you are connected to a GitHub MCP Server.

You subscribe to a particular repository's README resource.

Later, when that resource changes, the server can notify the client.

### Difference Between List Changed and Subscribe

| List Changed | Subscribe |
|---|---|
| Notifies about changes in available lists. | Notifies about changes to a subscribed resource. |
| Example: A new tool is added. | Example: A README resource changes. |

---

# 5. Phase 2 – Operation

During the Operation Phase, the client and server exchange messages according to the negotiated protocol version and capabilities.

Two important rules:

1. Both parties must respect the negotiated protocol version.
2. Only successfully negotiated capabilities can be used.

The Operation Phase is divided into two parts.

## Part A: Capability Discovery

During initialization, the server tells the client which categories of capabilities it supports.

For example:

```text
I support:
- Tools
- Resources
- Prompts
```

But the client still doesn't know exactly which tools, resources, or prompts are available.

This is where Capability Discovery comes in.

The client sends specific JSON-RPC requests to discover the available items.

### Important Discovery Methods

| What Client Wants | Method |
|---|---|
| List available tools | `tools/list` |
| List available resources | `resources/list` |
| List available prompts | `prompts/list` |

The server responds with the available items, descriptions, and schemas.

### Example Tool List Response

```json
{
  "tools": [
    {
      "name": "read_file",
      "description": "Read the contents of a file"
    },
    {
      "name": "write_file",
      "description": "Write content to a file"
    }
  ]
}
```

Now the client knows which tools are available and what arguments they require.

### Practical Demonstration: Claude Desktop

When Claude Desktop starts, the following communication happens:

1. Client sends `initialize`.
2. Server sends its initialization response.
3. Client sends `notifications/initialized`.
4. Client requests `tools/list`.
5. Client requests `prompts/list`.
6. Client requests `resources/list`.

The filesystem server supports Tools but does not support Prompts or Resources.

Therefore:

```text
tools/list       → Successful Response
prompts/list     → Method Not Found
resources/list   → Method Not Found
```

**Important:** A server returning `Method Not Found` for an unsupported method does not mean the entire connection has failed.

## Part B: Tool Calling

After capability discovery, the client knows which tools are available.

Now it can call the required tool according to the user's request.

### Example: Reading a File

User asks Claude:

```text
Tell me what's written in hello.py on my Desktop.
```

### Step 1: User Sends Request

The user asks Claude to read a file.

### Step 2: Client Understands the Request

Claude understands that it needs to read a file.

It checks the available tools and identifies `read_file`.

### Step 3: Client Calls the Tool

The client sends a `tools/call` request.

Example:

```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "method": "tools/call",
  "params": {
    "name": "read_file",
    "arguments": {
      "path": "hello.py"
    }
  }
}
```

### Step 4: Server Executes the Tool

The filesystem server reads the requested file.

### Step 5: Server Returns the Result

The server sends the file content back to the client.

### Step 6: Client Displays the Result

Claude presents the file content to the user.

### Complete Tool Calling Flow

```text
USER
 |
 | Requests file content
 v
CLIENT
 |
 | Selects read_file tool
 v
CLIENT → SERVER
 |
 | tools/call + arguments
 v
SERVER
 |
 | Executes read_file function
 v
SERVER → CLIENT
 |
 | Returns file content
 v
CLIENT
 |
 | Displays result
 v
USER
```

### Permissions

During the practical demonstration, Claude asks for permission before accessing directories or reading files.

This means the host/client can enforce user approval before allowing particular tool operations.

The server having a tool does not automatically mean every operation must be approved without user interaction.

---

# 6. Phase 3 – Shutdown

Shutdown means terminating the session between the MCP Client and MCP Server.

There are two possible reasons:

1. Client shuts down.
2. Server shuts down.

Normally, the client initiates shutdown.

**Important:** Shutdown does not require an additional JSON-RPC shutdown request.

Unlike initialization and operation, the transport layer handles connection termination.

There are two transport scenarios.

## A. STDIO Transport – Local Server

STDIO is generally used when the MCP Server runs locally.

In STDIO:

- Client starts the local MCP Server as a subprocess.
- Client controls the server's input and output streams.
- Communication happens through standard input and standard output.

### Client-Initiated Shutdown

Suppose Claude Desktop is connected to a local filesystem server.

When Claude Desktop closes, the following sequence happens:

**Step 1: Client Closes STDIN**

The client closes the server's input stream.

This tells the server that no more input will arrive.

**Step 2: Client Waits**

The client waits for the server process to exit.

**Step 3: SIGTERM**

If the server does not exit, the client can ask the operating system to send `SIGTERM`.

SIGTERM means a termination request.

It gives the process an opportunity to shut down gracefully.

**Step 4: SIGKILL**

If the server still does not exit, the client can use `SIGKILL`.

SIGKILL forcefully terminates the process.

### Difference Between SIGTERM and SIGKILL

| SIGTERM | SIGKILL |
|---|---|
| Requests termination. | Forces termination. |
| Process can handle it. | Process cannot ignore or handle it. |
| Graceful shutdown opportunity. | Forceful shutdown. |

## B. HTTP Transport – Remote Server

HTTP is used when the MCP Server runs remotely.

### Client-Initiated Shutdown

The client closes its HTTP connection with the server.

### Server-Initiated Shutdown

The server closes its HTTP connection.

In this situation, the client should be prepared to handle an unexpected dropped connection.

For example:

- Stop or clean up ongoing work.
- Handle incomplete operations.
- Attempt reconnection when appropriate.

### STDIO vs HTTP Shutdown

| STDIO | HTTP |
|---|---|
| Used for local servers. | Used for remote servers. |
| Client closes input stream. | Client closes HTTP connection. |
| Can use SIGTERM and SIGKILL. | Connection is terminated. |

### Practical Demonstration

The teacher closes Claude Desktop.

The terminal displays:

```text
Server transport closed
```

Notice that no additional JSON-RPC shutdown message is exchanged.

---

# 7. Special Cases in MCP Lifecycle

Apart from the normal lifecycle, there are four important special cases:

1. Ping
2. Error Handling
3. Timeout
4. Progress Notifications

## A. Ping

Ping is a lightweight request-response method.

Its purpose is to check whether the other side is still alive and whether the connection is responsive.

### Example

Suppose:

- Client and Server are connected.
- A long-running task is executing.
- No messages have been exchanged for some time.

The client can send a Ping request to check whether the server is still active.

### Ping Request

```json
{
  "jsonrpc": "2.0",
  "id": 10,
  "method": "ping"
}
```

### Ping Response

```json
{
  "jsonrpc": "2.0",
  "id": 10,
  "result": {}
}
```

### Important Points

- Request has ID 10.
- Response also has ID 10.
- Result is empty.
- Both Client and Server can send Ping requests.

### When is Ping Useful?

**1. During Initialization**

To check whether the other side is responsive.

**2. During Long-Running Tasks**

To periodically check whether the connection is still active.

Periodic communication can also help prevent proxies or firewalls from treating an otherwise idle connection as inactive.

**Remember:** Ping does not perform an actual task. It simply checks responsiveness.

---

## B. Error Handling

Not every request will succeed.

Sometimes something goes wrong during communication.

MCP uses JSON-RPC's standard error structure.

### Common Error Scenarios

| Scenario | Explanation |
|---|---|
| Protocol Mismatch | Client and server cannot agree on a compatible protocol version. |
| Method Not Found | Client calls a method that the server doesn't support. |
| Invalid Arguments | Client sends incorrect tool arguments. |
| Internal Server Error | Server encounters a problem while processing a request. |
| Timeout | Server takes longer than the allowed time. |
| Invalid JSON-RPC Message | Message has incorrect structure or syntax. |

### JSON-RPC Error Object

A typical error response looks like this:

```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "error": {
    "code": -32601,
    "message": "Method not found",
    "data": {}
  }
}
```

### Three Important Fields

| Field | Meaning |
|---|---|
| `code` | Identifies the type of error. |
| `message` | Explains what went wrong. |
| `data` | Optional additional debugging information. |

### Common JSON-RPC Error Codes

| Error Code | Meaning |
|---|---|
| -32700 | Parse Error |
| -32600 | Invalid Request |
| -32601 | Method Not Found |
| -32602 | Invalid Parameters |
| -32603 | Internal Error |

Additional server-specific errors can include:

- Authentication failures.
- Rate limits.
- Quota errors.
- Internal server issues.

**Remember:** You don't need to memorize every error code. Understand what each error means.

---

## C. Timeout

Timeout means:

**Don't wait forever for a server response.**

Suppose:

- Client sends a request.
- Server takes too long to respond.
- Client has configured a timeout of 30 seconds.

If the server doesn't respond within 30 seconds, the timeout is triggered.

### Why Do We Need Timeout?

Three important reasons:

1. Prevent users from waiting indefinitely.
2. Release resources being used by an unfinished request.
3. Inform the user that something went wrong.

### How Does Timeout Work?

1. Client configures a 30-second timeout.
2. Client sends a request to the server.
3. Server takes more than 30 seconds.
4. Client triggers the timeout.
5. Client sends a cancellation notification.
6. Server receives the cancellation and stops processing if it can.

### Cancellation Notification

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/cancelled",
  "params": {
    "requestId": 7,
    "reason": "Timeout exceeded"
  }
}
```

### Important Points

- There is no `id` because this is a notification.
- `requestId` identifies which earlier request needs to be cancelled.
- Cancellation tells the server to stop processing the request.

**Important:** Sending cancellation does not guarantee that every operation can be immediately stopped.

---

## D. Progress Notifications

Sometimes the server receives a long-running task.

The user would like to know how much work has been completed.

For this purpose, MCP supports Progress Notifications.

### Example

Suppose the user asks:

```text
Scan all files in my repository and identify security vulnerabilities.
```

Assume the repository contains 1,000 files.

The user would like to know how much work has been completed.

### How Does It Work?

**Step 1: Client Sends a Request**

The client sends a request to scan the repository.

It also includes a progress token in the request metadata.

Example:

```json
{
  "jsonrpc": "2.0",
  "id": 7,
  "method": "tools/call",
  "params": {
    "name": "search_code",
    "_meta": {
      "progressToken": "token-7"
    }
  }
}
```

**Step 2: Server Starts Processing**

The server begins scanning the files.

**Step 3: Server Sends Progress Notifications**

The server periodically sends progress updates.

Example:

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/progress",
  "params": {
    "progressToken": "token-7",
    "progress": 60,
    "total": 100,
    "message": "Scanning 600 of 1000 files"
  }
}
```

**Step 4: Client Displays Progress**

The host displays the progress information to the user.

For example:

```text
Scanning Repository

Progress: 60%

600 / 1000 files scanned
```

### Benefits of Progress Notifications

- User knows how much work has been completed.
- User gets feedback during long-running tasks.
- User has a better understanding of the remaining work.

---

# 8. Complete MCP Lifecycle – Summary

The complete MCP Lifecycle consists of three major phases.

## Phase 1: Initialization

1. Client sends `initialize`.
2. Server responds with protocol version and capabilities.
3. Client sends `notifications/initialized`.
4. Connection is established.

## Phase 2: Operation

### Part A: Capability Discovery

Client discovers available:

- `tools/list`
- `resources/list`
- `prompts/list`

### Part B: Actual Operations

1. Client selects the required tool.
2. Client sends `tools/call`.
3. Server executes the tool.
4. Server returns the result.
5. Client displays the result to the user.

## Phase 3: Shutdown

1. Client or server terminates the connection.
2. Transport layer closes the connection.
3. Session ends.

### Special Cases

| Special Case | Purpose |
|---|---|
| Ping | Check connection responsiveness. |
| Error Handling | Handle failed requests. |
| Timeout | Stop waiting indefinitely. |
| Progress Notifications | Show task progress. |

---

# 9. Important Concepts to Remember

| Concept | Explanation |
|---|---|
| Session | Continuous connection between client and server. |
| Initialization | First handshake between client and server. |
| Version Negotiation | Agreeing on a compatible MCP version. |
| Capability Negotiation | Exchanging supported features. |
| Capability Discovery | Finding exact available tools, resources, and prompts. |
| Tool Calling | Executing a specific server tool. |
| Shutdown | Terminating the session. |
| Ping | Checking whether the other side is responsive. |
| Error Handling | Handling unsuccessful requests. |
| Timeout | Cancelling a request that takes too long. |
| Progress Notification | Reporting how much work has been completed. |

---

# 10. Final Takeaway

MCP Lifecycle explains how an MCP Client and Server communicate throughout a session.

The three main phases are:

- **Initialization:** Establish connection and negotiate capabilities.
- **Operation:** Discover and execute available tools and access supported resources.
- **Shutdown:** Terminate the connection.

Understanding these concepts is important before building your own MCP Clients and MCP Servers.
