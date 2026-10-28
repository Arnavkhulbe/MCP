
# MCP (Model Context Protocol) — Why Do We Need MCP?

## 1. Introduction

MCP stands for **Model Context Protocol**.

MCP addresses an important problem in the AI ecosystem: how can different AI applications connect to external tools and services without requiring developers to build and maintain separate custom integrations for every connection?

To understand why MCP became important, we need to understand how AI applications evolved, why context is important, how function calling helped, and why the integration problem still remained.

This README follows the concepts explained in the lecture.

---

## 2. Three Stages of AI Adoption

The tutor explains the adoption of ChatGPT and AI through three stages.

### Stage 1: Pure Wonder

When ChatGPT became available, people were amazed by its ability to understand and respond to natural language.

People experimented with different questions, explored its capabilities, and shared interesting responses on social media.

At this stage, people were mainly exploring what AI could do.

### Stage 2: Professional Adoption

People gradually realized that ChatGPT could help with professional work.

Examples included:

- Summarizing long documents.
- Debugging code.
- Planning curricula.
- Helping with research and writing.
- Improving productivity.

AI started becoming more than a chatbot for entertainment. It became a potential work partner.

### Stage 3: The API Revolution

AI capabilities began appearing in other software applications through model APIs and integrations.

Existing applications started incorporating AI features, while new AI-powered tools also emerged.

AI was no longer restricted to a single chatbot. It became available across multiple applications and workflows.

### The Problem That Emerged

Although AI became more accessible, information and work became distributed across different AI applications and software services.

This led to a problem called **fragmentation**.

---

## 3. The Problem of Fragmentation

Fragmentation means that information and AI capabilities are spread across different applications that do not automatically share the necessary context.

For example:

- Notion may contain project information.
- Slack may contain team discussions.
- VS Code may contain source code.
- GitHub may contain repositories.
- Google Drive may contain documents.

An AI assistant inside one application may not automatically know what is happening in another application.

As a result, users may need to move between multiple applications to collect the information required to complete a task.

The original vision was to have a unified AI assistant that could understand the relevant work and help solve problems end to end.

However, information was scattered across multiple systems.

**This leads to the next major problem: context assembly.**

---

## 4. What Is Context?

**Context is the information an LLM uses to generate a response.**

It can include:

- Conversation history.
- User instructions.
- External documents.
- Source code.
- Database information.
- Information retrieved from tools.
- Other relevant information supplied to the model.

The quality and relevance of the available context can affect how effectively an AI system helps with a task.

### A Simple Example

Suppose you are discussing quantum physics with an AI assistant.

You ask:

1. What is quantum physics?
2. Where can I learn it?
3. Which books are useful?
4. How difficult is this topic?

The assistant can use the earlier conversation to understand what you mean by "this topic."

In this example, the conversation history provides context.

However, professional tasks often require information from many different systems, making context assembly much more complicated.

---

## 5. The Context Assembly Problem

Consider a software developer who needs to implement two-factor authentication in a product.

The information required for the task may be distributed across several systems:

| System | Information Required |
|---|---|
| Jira | Task requirements and assigned ticket |
| GitHub | Current source code and repository information |
| MySQL | Existing database schema |
| Google Drive | Security guidelines and documentation |
| Slack | Relevant team discussions |

The developer needs to combine this information to understand the task properly.

### Without Automated Tool Access

The developer may need to:

1. Open Jira and copy the ticket details.
2. Open GitHub and retrieve relevant source code.
3. Inspect the database schema.
4. Find the security documentation in Google Drive.
5. Search Slack for relevant discussions.
6. Manually provide all this information to the AI assistant.

Only after collecting and supplying the context can the developer ask the AI to help implement the feature.

This is called **context assembly**.

### Why Is This a Problem?

- It consumes time before the actual task begins.
- Information is scattered across different systems.
- The user must repeatedly copy and paste information.
- Important details may be missed.
- The process becomes harder as projects and systems grow.
- Manually collecting all relevant context does not scale well.

The tutor describes developers in this situation as becoming **human APIs** because they manually move information between systems and the AI.

### The Desired Solution

Ideally, an AI application should be able to retrieve the necessary information from connected systems when needed, rather than relying on the user to manually assemble everything.

This motivates the use of tools.

---

## 6. Function Calling: The First Step Toward Automated Context Retrieval

Function calling allows an LLM-based application to request the execution of external functions.

Instead of only generating text, a model can determine that a task requires a tool and request that the application invoke it.

### How Function Calling Works

Suppose an application provides a function named `load_file`.

The application also provides a description explaining that the function retrieves the contents of a file.

When the user asks:

> Read the contents of `abc.txt`.

The workflow can be:

1. The model receives the user request and available function descriptions.
2. It determines that `load_file` is appropriate.
3. It requests a call with the required filename argument.
4. The application executes the function.
5. The function returns the file contents.
6. The application provides the result to the model.
7. The model uses the result to answer the user.

Function calling allows applications to connect AI models to operations such as:

- Retrieving files.
- Querying databases.
- Searching repositories.
- Accessing weather information.
- Performing web searches.

### Why Was This Important?

The AI application could retrieve external information through tools instead of relying entirely on information manually supplied by the user.

This helped automate context assembly.

---

## 7. How Function Calling Helped With Context Assembly

Consider the same two-factor authentication task.

Suppose the AI application has tools for accessing Jira, GitHub, MySQL, Google Drive, and Slack.

The developer could ask the assistant to retrieve:

1. The Jira ticket requirements.
2. The latest relevant code from GitHub.
3. The database schema.
4. The security guidelines.
5. The relevant Slack discussions.

The tools can retrieve the information, allowing the application to provide it to the model.

The developer no longer needs to manually copy and paste every piece of information if the required integrations are available.

This is a major improvement.

However, **function calling does not automatically solve the integration problem**.

---

## 8. The Integration Problem

Function calling requires an application to have access to appropriate functions and a mechanism for executing them.

In a custom integration, developers may need to implement the logic that communicates with each external service.

This can include:

- Authentication.
- API requests.
- Input and output formatting.
- Error handling.
- Rate-limit handling.
- Maintaining compatibility with external APIs.

The tutor explains how this becomes difficult when an organization uses multiple AI applications and multiple external services.

### Example: Three AI Applications and Twenty Services

Suppose a company has three AI applications:

1. A general-purpose assistant.
2. A coding agent.
3. An analytics agent.

The company also uses twenty external services.

If every AI application needs a custom integration with every service, the potential number of application-service integrations is:

\[
N \times M
\]

Where:

- \(N\) = number of AI applications.
- \(M\) = number of external services.

In this example:

\[
3 \times 20 = 60
\]

That could mean sixty custom integrations in an architecture where every application needs every service.

**Important:** This is a simplified worst-case illustration. In practice, each application only needs the services relevant to its tasks, so the actual number may be lower.

### Problems With Many Custom Integrations

#### 1. Development Complexity

Each integration may require its own implementation, API handling, authentication, and error handling.

#### 2. Maintenance Overhead

If an external API changes, the integrations that depend on it may need updates.

#### 3. Security Management

Credentials and permissions may be distributed across multiple custom implementations, making consistent management more difficult.

#### 4. Cost and Time

Developing and maintaining many integrations requires engineering effort and resources.

The original goal was to make AI applications more useful and improve productivity. Maintaining a large number of custom integrations can create another significant engineering burden.

### What Do We Want Instead?

Ideally, developers should be able to build an integration for a service once and reuse it across compatible AI applications.

This is the integration problem that MCP aims to address.

---

## 9. What Is MCP?

**Model Context Protocol (MCP) is a standard protocol for communication between AI applications and external tool or data servers.**

The tutor explains MCP through two main entities:

- **Client:** The AI application that wants to use external capabilities.
- **Server:** The component that exposes those capabilities through MCP.

The client and server communicate using the MCP protocol.

### MCP Client

The client is the component in an AI application that communicates with MCP servers.

The host application may be ChatGPT, Claude, Cursor, Perplexity, or a custom AI application, provided it supports the relevant MCP connections.

### MCP Server

An MCP server exposes tools and other capabilities from an external system through a standardized interface.

For example, a server may expose capabilities related to:

- GitHub.
- Google Drive.
- Slack.
- Databases.
- Other APIs and internal services.

The server handles the implementation required to provide its capabilities, including communication with the underlying service.

### MCP Protocol

The MCP protocol defines how the client and server communicate, including how capabilities are discovered and how requests and results are exchanged.

The basic idea is to standardize the connection rather than requiring every AI application to invent its own integration method for every service.

---

## 10. MCP Architecture

A simplified architecture looks like this:

```text
                  AI Application
                  (MCP Host)
                       |
                       v
                   MCP Client
                       |
                 MCP Protocol
                       |
          +------------+------------+
          |            |            |
          v            v            v
      MCP Server   MCP Server   MCP Server
          |            |            |
          v            v            v
       GitHub      Google Drive    Slack
```

A single AI application can connect to multiple MCP servers.

Each server exposes the capabilities that it supports and that the application is authorized to use.

The client and server communicate through the MCP protocol, while the server may use the underlying service's API to perform the requested operation.

---

## 11. Function Calling vs. MCP

Function calling and MCP are related, but they solve different problems.

| Aspect | Function Calling | MCP |
|---|---|---|
| Main purpose | Allows a model to request that an application execute a function or tool. | Standardizes communication between AI applications and external tool/data servers. |
| Integration approach | Tool definitions and execution logic may be custom to an application. | Compatible clients can communicate with MCP servers through a common protocol. |
| Tool execution | The application executes the requested function or arranges its execution. | The MCP server handles the requested capability and may call an underlying API. |
| Reusability | Custom integrations may need to be repeated across applications. | A compatible MCP server can be reused by multiple compatible clients. |
| Relationship | Helps the application decide when and how to request a tool call. | Provides a standardized way to discover and communicate with server capabilities. |

### How They Work Together

MCP does not replace function calling.

An AI application may use function calling or another tool-selection mechanism to decide which tool is appropriate.

If the tool is exposed by an MCP server, the application can invoke it through its MCP client.

The two mechanisms can operate at different layers of the same workflow.

---

## 12. What Changes When We Use MCP?

Consider the example of connecting an AI application to GitHub.

### With a Custom Integration

A developer may need to write application-specific code to:

- Call the GitHub API.
- Handle authentication.
- Format requests and responses.
- Handle errors.
- Convert the returned data into a format the application can use.

If another AI application needs the same capabilities, its developers may need to build another integration.

### With an MCP Server

The GitHub-related capabilities can be exposed through an MCP server.

An MCP-compatible client can connect to that server, discover its available tools, and request their execution.

The server handles the implementation needed to perform the operation and returns the result through MCP.

The AI application still needs to support MCP, configure the connection, handle the tool results, and enforce appropriate permissions.

**The main benefit is a reusable, standardized integration interface.**

---

## 13. Benefits of MCP

### 1. Reduced Integration Duplication

Instead of building a separate custom integration for every AI application and service combination, compatible applications can reuse MCP servers.

### 2. Reduced Maintenance Work

The implementation for an external service can be maintained in the relevant server rather than duplicated across multiple AI applications.

Client and server implementations still need to be maintained, but the protocol can reduce duplicated integration work.

### 3. Reduced Development Time and Cost

Developers can connect compatible applications to existing MCP servers rather than implementing every integration from scratch.

The actual time and cost savings depend on the services, available servers, and deployment requirements.

### 4. More Centralized Integration Logic

An MCP server can centralize service-specific logic, such as API communication, authentication handling, request formatting, and error handling.

This can make integrations easier to manage.

### 5. Reusable Tool Access

Multiple compatible AI applications can use the same MCP server when their connections and permissions allow it.

### 6. Easier Ecosystem Connectivity

An AI application that supports MCP can potentially connect to many existing MCP servers without needing a completely separate custom protocol implementation for every service.

**Important:** MCP does not automatically guarantee better security. Authentication, authorization, credential storage, and server permissions must still be implemented and managed correctly.

---

## 14. Understanding the Integration Formula

The tutor contrasts a duplicated integration model with a more reusable MCP architecture.

### Without a Shared Protocol

If every one of \(N\) AI applications requires a custom integration with every one of \(M\) services, the potential number of integrations is:

\[
N \times M
\]

### With Reusable MCP Servers

The lecture describes the architecture using the simplified comparison:

\[
N + M
\]

Here:

- \(N\) represents the AI clients that support MCP.
- \(M\) represents the service integrations exposed through MCP servers.

For example, three AI applications and twenty services could be represented as:

\[
3 + 20 = 23
\]

This illustrates how standardization can reduce duplicated integration work.

**This formula is a conceptual model, not a universal count of actual components or connections.** Real systems vary depending on which services each client needs, which integrations already exist, and how the infrastructure is deployed.

---

## 15. Why Is MCP Becoming Popular?

The tutor explains MCP adoption through a network-effect argument.

### AI Applications Adopt MCP

When more AI applications support MCP, users and developers gain more ways to connect those applications to compatible MCP servers.

### Service Providers Build MCP Servers

When service providers make their capabilities available through MCP, more compatible AI applications can connect to them without requiring each client to implement a separate custom integration.

### More Servers Increase Ecosystem Value

As the number of useful MCP servers grows, MCP becomes more valuable to developers building new AI applications.

### More Compatible Clients Increase Demand

As more AI applications adopt MCP, service providers have a stronger incentive to expose their capabilities through MCP servers.

### The Network Effect

The relationship can be summarized as:

```text
More MCP-Compatible AI Applications
                 |
                 v
          More Demand for
             MCP Servers
                 |
                 v
      More Available MCP Servers
                 |
                 v
     More Value for AI Developers
                 |
                 v
      More MCP Adoption
```

This creates a reinforcing cycle in which more compatible clients and servers can increase the usefulness of the ecosystem.

The lecture presents this network effect as a reason MCP could become an important industry standard.

---

## 16. Important Limitations

MCP standardizes communication, but it does not eliminate every integration or security challenge.

- An AI application must support MCP and be configured to connect to a server.
- An MCP server must implement the required capabilities.
- Access to external systems must be authorized.
- Servers may require credentials or other configuration.
- Tool results must still be handled by the AI application.
- The model may need multiple tool calls to gather sufficient context.
- MCP does not guarantee that an AI model will always use tools correctly.
- The client and server implementations still require maintenance.
- Not every AI application needs every available MCP server.

MCP reduces the need for duplicated custom integrations, but it does not make software integration completely effortless.

---

## 17. Key Terms

| Term | Meaning |
|---|---|
| LLM | Large Language Model, a model that processes and generates language. |
| Context | Information an LLM uses to generate a response. |
| Context Assembly | Collecting relevant information from different sources and supplying it to the AI. |
| Function Calling | A mechanism that allows a model to request execution of a function or tool. |
| Tool | An operation an AI application can request. |
| API | An interface through which software communicates with a service. |
| MCP | Model Context Protocol, a standardized communication protocol for AI applications and external capabilities. |
| MCP Client | The component that communicates with MCP servers. |
| MCP Server | The component that exposes tools and other capabilities through MCP. |
| Integration | The connection and implementation that allows two systems to work together. |
| Network Effect | A situation where an ecosystem becomes more useful as participation and compatible connections grow. |

---

## 18. Final Summary

The lecture explains why MCP became important by following the evolution of AI applications.

First, ChatGPT made AI accessible to a broad audience. AI capabilities then spread into professional workflows and other software applications.

However, this created fragmentation: information was distributed across many different systems.

The context assembly problem emerged because users had to collect and manually provide information from those systems to their AI assistants.

Function calling helped automate context retrieval by allowing AI applications to invoke tools. But when multiple AI applications needed access to many services, custom integrations created development, maintenance, security-management, time, and cost challenges.

**MCP addresses this integration problem by standardizing communication between AI applications and external tool/data servers.**

Its key benefits are:

- Standardized communication.
- Reusable integrations.
- Reduced duplicated development work.
- Easier connectivity across compatible AI applications and services.
- An ecosystem that can grow through network effects.

The central idea is simple:

> Instead of every AI application building its own custom integration with every service, MCP provides a common protocol through which compatible clients can connect to servers that expose the required capabilities.
