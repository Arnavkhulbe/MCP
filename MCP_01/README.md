Model Context Protocol (MCP) — Why It Was Created
Overview
This README explains the main ideas from the lecture about why MCP (Model Context Protocol) was created, the problems it addresses, how it relates to function calling, and why a shared protocol can make AI integrations easier to maintain.

The central idea is:
MCP provides a standardized way for compatible AI applications to communicate with external tools and data sources, so developers do not need to build every connection from scratch for every AI application.

Table of Contents
1. The Three Stages of AI Adoption
2. The Fragmentation Problem
3. What Is Context?
4. Why Manual Context Assembly Is Difficult
5. How Function Calling Helps
6. The Integration Problem
7. What Is MCP?
8. MCP Client and MCP Server
9. A Workflow Example
10. Function Calling vs. MCP
11. Benefits of MCP
12. Why MCP Adoption Can Grow
13. Key Takeaways
1. The Three Stages of AI Adoption
The lecture describes three broad stages in the adoption of ChatGPT and AI.
Stage 1: Pure Wonder
People experimented with ChatGPT by asking unusual questions, requesting poems, exploring scientific topics, and sharing interesting answers.

The main purpose was to discover what AI could do.
Stage 2: Professional Adoption
People began using AI for real work, such as:
- Summarizing documents.
- Debugging code.
- Planning curricula.
- Assisting with professional tasks.
They realized that AI could help improve productivity, not merely entertain or answer general questions.
Stage 3: The API Revolution
Developers began integrating language-model capabilities into their own applications through APIs. AI features started appearing in coding tools, office applications, search tools, and other software.

However, this introduced a new challenge: fragmentation.
2. The Fragmentation Problem
Fragmentation means that information and AI capabilities are distributed across separate systems that do not automatically share context.

A company might use:
- Jira for tasks and requirements.
- GitHub for source code.
- MySQL for database information.
- Google Drive for documents.
- Slack for team discussions.
Even when these services have AI features, one tool may not know what is happening in another.

For example, a coding assistant might have access to a codebase but not to the Slack discussion explaining why a particular design decision was made.

The goal is to make it possible for an AI assistant to retrieve relevant information from multiple systems rather than forcing a person to manually move information between them.
3. What Is Context?
Context is the information an AI model can use to generate a response.

Context can include:
- Previous messages in a conversation.
- Documents and instructions.
- Source code.
- Database information.
- Task requirements.
- Results returned by external tools.
Simple example
1. You ask, “What is quantum physics?”
2. The AI explains it.
3. You ask, “How difficult is it to learn?”
The AI can interpret “it” by using the earlier conversation as context.
Professional example
Suppose a developer must implement two-factor authentication. The information required for the task may be spread across several systems:
System	Relevant information
Jira	Feature requirements
GitHub	Existing application code
MySQL	Database schema
Google Drive	Security guidelines
Slack	Team discussions


The AI needs the relevant context to make a useful recommendation. The issue is not necessarily a lack of intelligence; the AI may simply lack access to the information required for the task.
4. Why Manual Context Assembly Is Difficult
Without integrations, a developer might need to:
1. Open Jira and copy the requirements.
2. Open GitHub and copy relevant code.
3. Retrieve the database schema.
4. Copy security guidelines from Google Drive.
5. Find relevant Slack discussions.
6. Paste the collected information into the AI chat.
This is manual context assembly.

The lecture calls developers “human APIs” in this situation because they manually transfer information between systems and the AI.

As projects and the number of tools grow, copying and pasting becomes slow, difficult to maintain, and hard to scale.

Ideally, the AI application should be able to request the relevant information from connected systems when needed.
5. How Function Calling Helps
Function calling allows a model to request that an application execute an external function.

An application provides the model with available function descriptions and the expected arguments. Based on the user's request, the model can select an appropriate function and request its execution.

For example:
load_file(filename)
If the user asks, “Read the contents of abc.txt,” the flow can be:
1. The user asks to read the file.
2. The model selects load_file and supplies the filename.
3. The application executes the function.
4. The function returns the file contents.
5. The model uses the result to answer the user.
The model requests the function call; the application executes it.

Functions can retrieve information from a database, search a repository, call an API, or perform other supported tasks. This lets AI applications interact with external systems instead of only generating text.
The limitation
Function calling does not automatically standardize how every AI application connects to every external service.

If each AI application implements its own integration for every service, duplicated code and maintenance work can grow quickly.

That leads to the integration problem.
6. The Integration Problem
Imagine a company has:
- 3 AI applications.
- 20 external services.
If each AI application needs a separate custom integration for every service, the illustrative number of integrations is:
\[
N \times M = 3 \times 20 = 60
\]
Each integration may need its own:
- Connection logic.
- Authentication handling.
- Data-format conversion.
- Error handling.
- Maintenance when an API changes.
For example, if a service changes its API, multiple separately implemented integrations may need to be updated.

This can increase development time, maintenance effort, and security-management complexity.

A better approach would let each service expose a standardized interface that multiple compatible AI applications can use.
7. What Is MCP?
MCP stands for Model Context Protocol.

It is a standardized protocol that enables compatible AI applications to communicate with external tools and data sources.

MCP defines how clients and servers exchange information and discover and invoke available capabilities.

Instead of every AI application implementing a completely different communication interface for each service, compatible clients can connect to servers that expose capabilities through MCP.

MCP does not automatically give an AI access to every system. The relevant server must be available and configured, and the user or application must have the required permissions.
8. MCP Client and MCP Server
An MCP setup commonly includes two main components.
MCP Client
The client is the application that connects to MCP servers.

Examples can include compatible AI assistants, coding agents, and AI applications built by developers.

The client handles MCP communication and makes the server's available capabilities accessible to the AI application.
MCP Server
The server exposes capabilities from an external system through MCP.

For example, a GitHub MCP server might expose tools for supported operations such as searching repositories, retrieving files, or inspecting issues.

The server handles communication with the underlying service and returns permitted results.
MCP Protocol
The protocol defines how the client and server communicate, including how tools are discovered and how requests and results are exchanged.

Conceptually:
                 AI Application
                  (MCP Client)
                       |
                       | MCP protocol
                       |
          +------------+------------+
          |            |            |
      MCP Server   MCP Server   MCP Server
        GitHub       Google       Slack
                     Drive
A client can connect to multiple servers, depending on what the application supports and how it is configured.
9. A Workflow Example
Suppose a developer asks:
Help me implement two-factor authentication.

With suitable MCP servers configured and authorized, the workflow could be:
1. The AI requests the Jira ticket to understand the requirements.
2. It retrieves relevant code from GitHub.
3. It requests the necessary database schema information.
4. It retrieves applicable security guidelines.
5. It searches relevant team discussions in Slack.
6. The AI uses the returned information to help plan or implement the feature.
Each server communicates with its underlying service and returns the permitted results.

The developer does not need to manually copy and paste all the information if the connected tools can retrieve it.

Important: MCP does not mean the AI automatically sees all information in these systems. The client and servers must be configured, tools must be available, and access must be authorized. The AI may also need to make multiple tool calls to gather enough context.
10. Function Calling vs. MCP
Function calling	MCP
Lets a model request that a function be executed.	Standardizes communication between clients and external servers.
The application implements or arranges function execution.	An MCP server exposes capabilities through a common protocol.
Integrations can be custom-built for each application.	Compatible clients can reuse an MCP server's interface.
Does not, by itself, standardize external connections.	Defines standardized discovery and communication between client and server.


How they work together
MCP does not replace function calling. They operate at different levels and can work together.

An AI application may use function calling or a similar tool-selection mechanism to decide when to use a tool. MCP provides the communication mechanism for discovering and invoking tools exposed by an MCP server.

The lecture simplifies the difference by suggesting that the client has nothing to do and the server does all the work. In practice, the client still needs to support MCP, establish connections, process requests and results, and manage configuration and permissions. MCP standardizes communication; it does not eliminate all client-side responsibilities.
11. Benefits of MCP
1. Less duplicated integration code
A compatible MCP server can be used by multiple compatible clients, reducing the need to implement the same service integration separately in each AI application.
2. Easier maintenance
Service integration logic can be maintained at the server rather than independently duplicated across clients. API changes may still require server updates, but clients can often continue using the same MCP interface.
3. Reduced development effort
Developers can reuse existing MCP servers instead of building every integration from scratch.
4. More consistent integration management
Connections and access policies can be managed more consistently. However, MCP does not automatically guarantee security. Servers still need appropriate authentication, authorization, secret management, and access controls.
5. Potential reduction in integration count
Using the earlier example of 3 clients and 20 services:
- Without a shared protocol, a simple model gives \(3 \times 20 = 60\) custom integrations.
- With reusable client and server integrations, an illustrative model gives \(3 + 20 = 23\) integration components.
This is a simplified comparison, not a guarantee that every real-world deployment will have exactly this number. The actual architecture and implementation requirements determine the work involved.
12. Why MCP Adoption Can Grow
The lecture describes a potential network effect:
1. More AI applications support MCP.
2. More service providers create MCP servers.
3. Those servers become useful to more compatible AI applications.
4. Developers have more reasons to adopt MCP because reusable integrations are available.
5. The ecosystem becomes more valuable as adoption grows.
A shared protocol can reduce the need for every client and service to create a unique integration.

The lecture predicts that MCP could become an industry standard. This is the lecturer's expectation rather than a guaranteed outcome.
13. Key Takeaways
- Context: Information an AI model can use to generate a response.
- Fragmentation: Relevant information is spread across separate systems.
- Context assembly: Gathering the information the AI needs.
- Function calling: Lets a model request execution of application functions.
- Integration problem: Custom connections can be duplicated and costly to maintain.
- MCP: A standard protocol for communication between compatible AI clients and servers.
- MCP client: The application connecting to MCP servers.
- MCP server: Exposes tools or other capabilities from an external system.
- Main benefit: Reusable, standardized integrations can reduce duplicated work.
The complete flow
Fragmented information
        |
        v
Manual context assembly
        |
        v
Function calling and external tools
        |
        v
Duplicated integration work
        |
        v
MCP standardizes client-server communication
        |
        v
Reusable connections to external capabilities
In one sentence: MCP helps compatible AI applications connect to external tools and data sources through a common protocol, making integrations more reusable and reducing the need for every AI application to implement every connection independently.
