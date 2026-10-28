
# MCP (Model Context Protocol)

## 1. What Is MCP?

**MCP stands for Model Context Protocol.** It is an open protocol that standardizes how AI applications communicate with external tools, data sources, and services.

An AI application may need information from repositories, files, databases, project-management systems, or messaging platforms. MCP provides a common way for the application to discover and use capabilities exposed by MCP servers.

In simple terms:

> MCP standardizes how AI applications connect to external tools and data.

MCP does not automatically give an AI access to every service. The relevant servers must be configured, and access must be authorized.

---

## 2. Why Do We Need MCP?

AI applications often need to work with external systems. Without a shared protocol, developers may have to build and maintain custom integrations between each application and each service.

For example, different AI applications may need different services:

- A general assistant may need Slack, Google Drive, and Jira.
- A coding assistant may need GitHub and Jira.
- An analytics assistant may need a database and Google Drive.

Not every application needs every service. Custom integrations can be duplicated and become difficult to maintain.

MCP provides a standard interface so compatible AI applications can reuse integrations built for MCP servers.

### Without MCP

- Each AI application may need custom code for each external service.
- Developers may need to handle different request and response formats.
- Integrations can be duplicated across applications.
- Maintaining many custom connections can become complicated.

### With MCP

- MCP defines a common communication protocol between clients and servers.
- Servers expose capabilities in a standardized way.
- Compatible clients can discover and invoke available tools.
- Integrations can be reused, subject to configuration, compatibility, and permissions.

**Important:** MCP does not mean every AI application must connect to every MCP server. Each application connects only to the services it needs and is authorized to use.

---

## 3. Main Components of MCP

MCP commonly involves three main components: the host application, an MCP client, and one or more MCP servers.

### 3.1 MCP Host

The **host** is the AI application that the user interacts with. It may contain an LLM, conversation logic, and a user interface.

Examples include compatible AI assistants, coding agents, and AI applications built by developers.

### 3.2 MCP Client

The **MCP client** is the component inside the host application that communicates with an MCP server.

It handles MCP protocol communication, including discovering available capabilities and sending tool requests. The host application remains responsible for deciding how to use returned results.

### 3.3 MCP Server

An **MCP server** exposes capabilities from an external system through MCP.

MCP servers can expose:

- **Tools:** Operations an AI application can request, such as searching a repository.
- **Resources:** Data or content that a client can retrieve.
- **Prompts:** Reusable prompt templates that a client can discover or use.

A server may communicate with an underlying API, database, local files, or another system to fulfill a request.

---

## 4. MCP Architecture

A simplified architecture looks like this:

```text
                 AI Application (Host)
                          |
                     MCP Client
                          |
                   MCP Protocol
                          |
             +------------+------------+
             |            |            |
        MCP Server   MCP Server   MCP Server
             |            |            |
          GitHub      Google Drive    Slack
```

Each server exposes its own capabilities. A client can connect to multiple servers when the host application supports and is configured for those connections.

The MCP protocol standardizes communication between the client and server. It does not replace the APIs or internal systems that a server may use behind the scenes.

---

## 5. How MCP Works: Step by Step

Suppose a developer asks an AI assistant:

> Find the README file in my GitHub repository.

A possible workflow is:

1. **Connection:** The host application connects its MCP client to a configured GitHub MCP server.
2. **Discovery:** The client discovers the tools that the server makes available.
3. **Model decision:** The application provides relevant tool information to the LLM. The model decides whether a tool call is appropriate.
4. **Tool request:** The application asks the MCP client to invoke the selected tool with the required arguments.
5. **Protocol message:** The client sends a structured request to the server using MCP.
6. **Execution:** The server performs the operation, perhaps by calling the GitHub API.
7. **Tool result:** The server returns the result through the MCP client.
8. **Final response:** The host application supplies the result to the model, which can formulate an answer for the user.

The exact sequence depends on the host application and the tool workflow. A task may require more than one tool call.

---

## 6. Technical Details: JSON-RPC and Transports

MCP uses **JSON-RPC 2.0** messages for protocol communication. These messages represent requests, results, and errors in a structured format.

A simplified JSON-RPC request might look like this:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list",
  "params": {}
}
```

This example illustrates the general shape of a request to list tools. A real MCP exchange also follows the protocol's initialization and capability-negotiation requirements.

MCP supports transports including:

- **stdio:** The client communicates with a server through standard input and standard output. This is commonly used when a client launches a local server process.
- **Streamable HTTP:** The client communicates with a server over HTTP, with streaming support as defined by the protocol.

The appropriate transport depends on how the server is hosted and how the client is configured.

---

## 7. Example: An MCP Tool in Python

The following example uses the Python MCP SDK's `FastMCP` interface to register a tool.

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("GitHub Tools")

@mcp.tool()
def search_repository(query: str) -> str:
    """Search for files in a repository."""
    # Add the actual repository-search implementation here.
    return f"Search requested for: {query}"

if __name__ == "__main__":
    mcp.run()
```

### What Does This Code Do?

- `FastMCP("GitHub Tools")` creates an MCP server instance.
- `@mcp.tool()` registers the decorated function as a tool.
- `search_repository()` defines the tool's name, input, and returned value.
- `mcp.run()` starts the server using the SDK's configured default transport.

**Note:** This is a minimal illustrative example. It does not connect to GitHub or perform a real repository search. That functionality must be implemented, and the server must be configured and launched by an MCP-compatible client.

---

## 8. A Multi-Tool Workflow

Suppose a developer asks:

> Help me implement two-factor authentication in this project.

With suitable MCP servers configured and authorized, an AI workflow could be:

1. Retrieve the relevant Jira ticket to understand the requirements.
2. Retrieve relevant code from GitHub.
3. Request the necessary database schema information.
4. Retrieve applicable security guidelines.
5. Search relevant team discussions in Slack.
6. Use the returned information to help plan or implement the feature.

Each server communicates with its underlying service and returns the permitted results.

The AI does not automatically have access to these systems. The relevant servers and tools must be available, configured, and authorized.

---

## 9. Function Calling vs. MCP

Function calling and MCP solve different problems. They can be used together.

| Aspect | Function Calling | MCP |
|---|---|---|
| Main purpose | Lets a model request that an application execute a function or tool. | Standardizes communication between AI applications and external tool/data servers. |
| Main role | Tool selection and invocation flow within an application. | Common protocol for discovering and communicating with server capabilities. |
| Execution | The application typically executes the requested function or arranges its execution. | An MCP server handles the requested capability and may call an underlying service or API. |
| Integration | Function definitions and execution logic may be custom to an application. | Compatible clients can reuse servers that expose capabilities through MCP. |
| Relationship | Can be used to decide when a tool should be called. | Can provide the standardized connection used to access the tool. |

### How They Work Together

An AI application may use function calling or a similar tool-selection mechanism to decide when to use a tool. If the tool is provided by an MCP server, the application can invoke it through its MCP client.

**MCP does not replace function calling.** Function calling and MCP can operate at different layers and work together.

---

## 10. How MCP Helps Multiple AI Applications

Imagine there are three AI applications and several external services.

A common misunderstanding is that every AI application must connect to every available tool. That is not required.

| AI Application | Services It May Need |
|---|---|
| General assistant | Slack, Google Drive, Jira |
| Coding assistant | GitHub, Jira |
| Analytics assistant | Database, Google Drive |

Each application connects only to the MCP servers relevant to its work.

Without a common protocol, developers may need custom integrations for each application-service combination. MCP can reduce duplicated integration work by standardizing how compatible clients communicate with servers.

The often-used comparison between `N × M` custom integrations and a more reusable MCP architecture is a simplified illustration, not a guarantee that every real deployment has exactly `N + M` integrations. The actual architecture depends on which services each application needs and which integrations already exist.

---

## 11. Benefits of MCP

### Standardization

MCP defines a common way for compatible clients and servers to communicate.

### Reusability

An MCP server can expose capabilities that multiple compatible applications may use, provided they are configured and authorized.

### Modularity

Applications and service integrations can be developed and maintained as separate components.

### Tool Discovery

Clients can discover the capabilities a server exposes through the protocol.

### Reduced Duplicated Integration Work

Developers may avoid rebuilding the same service integration separately for every compatible application.

### Flexible Connectivity

An application can connect to multiple MCP servers when needed, rather than requiring one server to expose every possible service.

---

## 12. Limitations and Security Considerations

MCP is a communication protocol, not a guarantee of access, correctness, or security.

- **Permissions still matter:** The application and server must be configured to access the relevant tools and data.
- **Access should be limited:** Give servers and credentials only the permissions they need.
- **Tool calls can have side effects:** Operations that modify files, create issues, or change data require appropriate safeguards.
- **Server implementations vary:** A server's available tools and behavior depend on its implementation.
- **The host still has responsibilities:** It must handle model decisions, tool requests, returned results, failures, and user interaction.
- **MCP does not eliminate APIs:** An MCP server may use an external service's API internally.
- **MCP does not guarantee correct answers:** The model can misinterpret returned information or use tools incorrectly.

Only connect to MCP servers you trust, and review their permissions and behavior before granting access.

---

## 13. Key Terms

| Term | Meaning |
|---|---|
| MCP | Model Context Protocol, a standard for communication between AI applications and external capabilities. |
| Host | The AI application that manages the overall interaction. |
| Client | The component that communicates with an MCP server. |
| Server | The component that exposes tools, resources, and/or prompts through MCP. |
| Tool | An operation a client can discover and request. |
| Resource | Data or content made available through an MCP server. |
| Prompt | A reusable prompt template exposed by a server. |
| JSON-RPC | A structured remote procedure call protocol used for MCP messages. |
| Transport | The mechanism used to carry protocol messages, such as stdio or Streamable HTTP. |
| API | An interface that allows software to interact with a service. |

---

## 14. Interview-Ready Definition

**MCP (Model Context Protocol) is an open protocol that standardizes how AI applications discover and interact with external tools and data sources.** It uses a client-server architecture and structured messages, including JSON-RPC, over supported transports such as stdio and Streamable HTTP.

MCP can work alongside LLM function calling: function calling helps an application decide when to invoke a tool, while MCP standardizes communication with servers that expose tools and other capabilities.

---

## 15. Final Takeaways

- MCP stands for **Model Context Protocol**.
- It standardizes communication between AI applications and external tool/data servers.
- The main components are the host application, MCP client, and MCP server.
- MCP servers can expose tools, resources, and prompts.
- MCP uses JSON-RPC messages and supports transports such as stdio and Streamable HTTP.
- MCP can reduce duplicated integration work across compatible applications.
- Not every AI application needs every MCP server.
- MCP does not automatically grant access to data or replace security, permissions, APIs, or function calling.
