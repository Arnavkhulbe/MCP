
# Remote MCP Server using FastMCP

## 1. Introduction

In the previous lecture, we learned how to create **Local MCP Servers** using FastMCP.

In this lecture, we will learn how to:

- Create a Remote MCP Server using FastMCP.
- Deploy the Remote MCP Server to the internet.
- Convert a Local Expense Tracker MCP Server into a Remote MCP Server.
- Deploy the Expense Tracker using FastMCP Cloud.
- Solve problems related to database access and concurrent users.
- Connect Remote MCP Servers to Claude Desktop.
- Use a Local Proxy Server to connect Remote MCP Servers.

---

## 2. Local MCP Server vs Remote MCP Server

### Local MCP Server

A Local MCP Server runs on the same machine as the MCP Host and MCP Client.

### Remote MCP Server

A Remote MCP Server runs on a different machine, usually on a server accessible through the internet.

### Comparison

| Local MCP Server | Remote MCP Server |
|---|---|
| Runs on the user's local machine. | Runs on a remote machine. |
| Communication happens on the same machine. | Communication happens over the internet or a network. |
| Generally faster communication. | Communication may be relatively slower because of network latency. |
| Usually designed for local usage. | Can serve multiple clients. |
| Uses local machine resources. | Can use more powerful server resources. |

### Advantages of Remote MCP Servers

1. **Multiple Clients:** A single Remote MCP Server can serve multiple clients.
2. **More Computing Power:** Remote servers can run on powerful machines and handle computationally intensive tasks.
3. **Enterprise Usage:** Enterprise MCP Servers are generally expected to use remote infrastructure.

### Disadvantages

- Network communication can introduce latency.
- Remote servers require deployment and infrastructure.
- Authentication and security become important when multiple users access the same server.

---

## 3. Roadmap of This Lecture

We will cover four major steps:

1. Create a simple Remote MCP Server.
2. Convert the Expense Tracker MCP Server into a Remote MCP Server.
3. Deploy the server using FastMCP Cloud.
4. Identify and solve problems in the deployed server.

---

# PART 1: Creating a Simple Remote MCP Server

## 4. Environment Setup

### Prerequisites

- Python
- UV package manager
- VS Code
- FastMCP
- MCP Inspector
- Git and GitHub
- FastMCP Cloud

### Step 1: Install UV

Run the following command in your terminal:

```bash
pip install uv
```

### Step 2: Create a Project Folder

Create a new folder:

```text
test_remote_server
```

Open this folder in VS Code.

### Step 3: Initialize the UV Project

Open the VS Code terminal and run:

```bash
uv init .
```

### Step 4: Install FastMCP

```bash
uv add fastmcp
```

---

## 5. Creating the Remote MCP Server

Create a file named:

```text
main.py
```

The server contains:

### Tools

1. `add_numbers` - Adds two given numbers.
2. `generate_random_number` - Generates a random number within a given range.

### Resource

1. `server_info` - Provides information about the MCP server.

### Important Difference Between Local and Remote Servers

In the previous local MCP server, we used:

```python
mcp.run()
```

By default, this starts the server using the **STDIO transport**.

For a Remote MCP Server, we use HTTP transport.

Example:

```python
mcp.run(
    transport="http",
    host="0.0.0.0",
    port=8000
)
```

### Understanding the Configuration

| Parameter | Meaning |
|---|---|
| `transport="http"` | Uses HTTP-based communication. |
| `host="0.0.0.0"` | Makes the server listen on all available network interfaces. |
| `port=8000` | Specifies the port on which the server runs. |

The lecture uses **Streamable HTTP** as the transport for the Remote MCP Server.

**Important:** The main tools and resources are written almost the same way as in a Local MCP Server. The major change is the transport configuration.

---

## 6. Running the Remote MCP Server

Run the server using:

```bash
uv run main.py
```

Alternatively, FastMCP provides a command to run the server:

```bash
uv run fastmcp run main.py --transport http --host 0.0.0.0 --port 8000
```

After execution, the server starts and displays its URL.

---

## 7. Testing with MCP Inspector

We should test the server before deploying it.

Open another terminal and execute:

```bash
uv run fastmcp dev main.py
```

This opens MCP Inspector.

### Testing Steps

1. Select the Streamable HTTP transport.
2. Click Connect.
3. Open the Resources section.
4. Check the `server_info` resource.
5. Open the Tools section.
6. Check the available tools.
7. Run `generate_random_number` with a minimum and maximum value.

Example:

```text
Minimum: 10
Maximum: 100
```

The tool generates a random number within the given range.

If the tools and resources are visible and working, our Remote MCP Server is ready for deployment.

---

# PART 2: Deploying the Remote MCP Server

## 8. Introduction to FastMCP Cloud

FastMCP Cloud is a deployment service provided by the FastMCP ecosystem.

It allows developers to deploy Remote MCP Servers without manually managing all the server infrastructure.

According to the lecture, other deployment options include:

- AWS
- Render
- FastMCP Cloud

The lecture uses FastMCP Cloud for deployment.

Website:

https://fastmcp.cloud

---

## 9. Push the Project to GitHub

FastMCP Cloud allows us to deploy our own code from a GitHub repository.

### Step 1: Create a GitHub Repository

Create a new repository:

```text
test-remote-mcp-server
```

### Step 2: Initialize Git

Inside the project folder:

```bash
git init
```

### Step 3: Check Project Files

```bash
git status
```

### Step 4: Add Files

```bash
git add .
```

### Step 5: Commit Changes

```bash
git commit -m "Initial commit"
```

### Step 6: Add Remote Repository

```bash
git remote add origin YOUR_GITHUB_REPOSITORY_URL
```

### Step 7: Push to GitHub

```bash
git push origin main
```

Now our project is available on GitHub.

---

## 10. Deploying on FastMCP Cloud

Follow these steps:

1. Open FastMCP Cloud.
2. Create an account or sign in.
3. Select **Deploy from Your Own Code**.
4. Connect your GitHub account.
5. Select the repository `test-remote-mcp-server`.
6. Click Deploy.

### Deployment Configuration

During deployment, provide the following information:

| Setting | Value / Meaning |
|---|---|
| Server Name | Name used as part of the server URL. |
| Entry Point | `main.py` |
| Authentication | No authentication for this basic demonstration. |
| Discoverable | Optional setting to list the server on the FastMCP discovery page. |

Click Deploy.

FastMCP Cloud builds and deploys the server.

After deployment, it provides a **Remote MCP Server URL**.

This URL can be shared with other users so that they can connect to the server.

---

## 11. Connecting the Remote MCP Server to Claude Desktop

The lecture demonstrates connecting the deployed server through Claude Desktop.

### Method 1: Using a Custom Connector

Steps:

1. Open Claude Desktop.
2. Go to Settings.
3. Open Connectors.
4. Select Add Custom Connector.
5. Enter a connector name.
6. Paste the Remote MCP Server URL.
7. Confirm and add the connector.
8. Restart Claude Desktop if required.

After connecting, the server's tools become available inside Claude.

For example:

```text
Generate a random number between 11 and 20.
```

Claude can call the `generate_random_number` tool and return its result.

**Note from the lecture:** At the time of recording, the Add Custom Connector option was described as available to Pro users, not Free users. This availability may change over time.

---

# PART 3: Converting the Expense Tracker into a Remote MCP Server

## 12. Project Overview

In the previous lecture, we created an **Expense Tracker MCP Server**.

Now we will convert that Local MCP Server into a Remote MCP Server.

Instead of writing the entire application again, we reuse the existing Expense Tracker code.

### Existing Project Components

- `main.py` - Contains the MCP server and its tools.
- `categories.json` - Contains expense categories.
- `expenses.db` - SQLite database created by the application.

### Expense Tracker Tools

| Tool | Functionality |
|---|---|
| `add_expense` | Adds a new expense. |
| `list_expenses` | Retrieves expenses. |
| `summarize_expenses` | Provides expense summaries. |

### Resource

The server also exposes the categories information through a Resource.

---

## 13. Reusing the Existing Expense Tracker Code

Follow these steps:

1. Open the existing Expense Tracker project.
2. Copy its `main.py` code.
3. Replace the simple calculator server code with the Expense Tracker code.
4. Copy `categories.json` into the new project.
5. Keep the required dependencies in the project.
6. Run the application locally to verify it.

Run:

```bash
uv run main.py
```

This initializes the database if it does not already exist.

Now start MCP Inspector:

```bash
uv run fastmcp dev main.py
```

Connect to the server and check:

- `add_expense`
- `list_expenses`
- `summarize_expenses`
- Categories resource

The lecture encountered a date-input formatting issue while testing `list_expenses`. The tools and resources were discoverable, but the date arguments needed to be supplied in the expected format.

---

## 14. Deploying the Expense Tracker

After testing locally, push the updated project to GitHub.

```bash
git status
git add .
git commit -m "Add expense tracking code"
git push origin main
```

FastMCP Cloud detects the new GitHub commit and starts a new build.

Once the build succeeds, the updated application is deployed.

Restart Claude Desktop and check whether the Expense Tracker tools are available.

Example request:

```text
Can you tell me about all of my expenses in September?
```

The server can retrieve the expenses using its database tools.

---

# PART 4: Solving the Database Read-Only Problem

## 15. Problem: SQLite Database in Read-Only Mode

After deploying the Expense Tracker, the lecture encountered an important problem.

When trying to add an expense, the server returned an error similar to:

```text
Unable to add the expense because the database is set to read-only mode.
```

### Why Did This Happen?

The SQLite database was being accessed in a way that did not allow writing in the deployed environment.

The solution demonstrated in the lecture was to change the database initialization and storage path so that the application creates and uses its database in a writable directory.

### Solution

The instructor used an updated code snippet that:

1. Creates a directory for storing the database.
2. Uses the database from that directory.
3. Allows the deployed application to write to the database.

The exact implementation is in the lecture's updated source code.

After updating the code, push the changes:

```bash
git status
git add .
git commit -m "Fix database read-only issue"
git push origin main
```

FastMCP Cloud detects the new commit and deploys the updated version.

### Testing Again

After restarting Claude Desktop, try adding an expense:

```text
Add an expense:
Trip to railway station
Date: 15 September
Category: Cab Ride
Amount: 200
```

Then retrieve the expenses:

```text
Show me all of my expenses in September.
```

Now the application can read and write expenses.

---

# PART 5: Making the MCP Server Asynchronous

## 16. Problem: Synchronous Server Architecture

The next problem identified in the lecture is that the server is synchronous.

### What Does Synchronous Mean?

In the current implementation, tools and database operations execute synchronously.

This means that an operation can block the execution flow while it is running.

### Example

Suppose multiple users are accessing the server:

1. User 1 calls `add_expense`.
2. The tool performs a database operation.
3. Other requests may have to wait while the blocking operation executes.
4. Additional users may experience delays.

This is not ideal for a Remote MCP Server that needs to handle multiple users.

### Solution: Asynchronous Programming

Python provides the `async` and `await` features to write asynchronous code.

The lecture introduces two changes:

1. Convert the MCP tools into asynchronous functions.
2. Convert database operations into asynchronous operations.

---

## 17. Using aiosqlite

Instead of using the standard SQLite library directly, the lecture introduces:

**aiosqlite**

It helps perform SQLite database operations asynchronously.

### Install aiosqlite

```bash
uv add aiosqlite
```

### Important Changes

| Before | After |
|---|---|
| Standard SQLite library | aiosqlite |
| Synchronous functions | Asynchronous functions |
| Blocking database operations | Asynchronous database operations |
| Regular function definitions | `async def` functions |
| Regular database calls | Calls using `await` |

### Example of Asynchronous Python

```python
async def add_expense():
    # Perform asynchronous operations
    pass
```

The `async` keyword defines an asynchronous function.

The `await` keyword is used when waiting for an asynchronous operation to complete.

### Updating the MCP Tools

The tools are also converted into asynchronous functions.

Conceptually:

```python
@mcp.tool()
async def add_expense():
    # Asynchronous database operation
    pass
```

The same approach is applied to the other tools and database functions.

### Benefits

- The server can handle concurrent requests more effectively.
- An operation waiting for database I/O does not need to block the entire execution flow.
- Multiple users can make requests without requiring every request to wait for a previous blocking operation.

**Important:** Asynchronous programming does not automatically make every operation faster. Its main benefit here is improving how the server handles concurrent I/O operations.

---

## 18. Deploying the Asynchronous Version

After updating the code:

```bash
uv add aiosqlite
```

Push the changes to GitHub:

```bash
git status
git add .
git commit -m "Add async feature"
git push origin main
```

FastMCP Cloud detects the new commit and starts another deployment.

After deployment:

1. Restart Claude Desktop.
2. Connect to the server again.
3. Try adding an expense.
4. Retrieve the expense list.

The lecture verifies that the updated asynchronous implementation continues to work.

However, because the instructor is testing alone, the actual performance improvement under multiple concurrent users is not quantified.

---

# PART 6: Connecting Remote MCP Servers on the Free Plan

## 19. Problem: Add Custom Connector Not Available

The lecture explains that the direct Custom Connector method was not available to users on Claude's Free plan at the time of recording.

FastMCP provides another approach using a **Proxy Server**.

### What Is a Proxy Server?

A proxy server acts as an intermediary between two systems.

In this architecture:

- Claude Desktop connects to a Local Proxy MCP Server.
- The Local Proxy MCP Server connects to the Remote MCP Server.
- The Remote MCP Server sends the response back through the proxy.
- The proxy returns the response to Claude Desktop.

### Architecture

```text
+----------------------+
|    Claude Desktop    |
+----------------------+
           |
           | STDIO
           v
+----------------------+
|  Local Proxy Server  |
|      (FastMCP)       |
+----------------------+
           |
           | HTTP
           v
+----------------------+
|   Remote MCP Server  |
|   (FastMCP Cloud)    |
+----------------------+
           |
           v
+----------------------+
|    Expense Tracker   |
|       Database       |
+----------------------+
```

The Local Proxy Server allows Claude Desktop to communicate with the Remote MCP Server through a local STDIO connection.

---

## 20. Creating a Local Proxy Server

Create a separate folder:

```text
proxy_server
```

Initialize the project:

```bash
uv init .
```

Install FastMCP:

```bash
uv add fastmcp
```

Create:

```text
main.py
```

### Proxy Server Code Concept

The lecture uses the FastMCP proxy functionality.

```python
from fastmcp import FastMCP

mcp = FastMCP.as_proxy(
    "YOUR_REMOTE_MCP_SERVER_URL",
    name="Expense Tracker Proxy"
)

if __name__ == "__main__":
    mcp.run()
```

Replace:

```text
YOUR_REMOTE_MCP_SERVER_URL
```

with the URL of your deployed Remote MCP Server.

The proxy uses STDIO when running through:

```python
mcp.run()
```

This is a Local MCP Server that forwards requests to the Remote MCP Server.

---

## 21. Testing the Proxy Server

Run:

```bash
uv run fastmcp dev main.py
```

MCP Inspector opens.

Select STDIO as the transport and connect.

Now check the tools.

You should be able to discover the Remote MCP Server's tools through the proxy, such as:

- `add_expense`
- `list_expenses`
- `summarize_expenses`

This confirms that the proxy is communicating with the remote server.

---

## 22. Installing the Proxy in Claude Desktop

Run:

```bash
uv run fastmcp install claude-desktop main.py
```

This installs the local proxy configuration in Claude Desktop.

### Important: UV Path Configuration

The lecture encounters an issue where Claude Desktop cannot start the proxy server.

The reason is that the configuration file contains the UV command but does not contain the complete path to the UV executable.

### Solution

Find the UV executable path.

On Windows:

```bash
where uv
```

Copy the complete executable path.

Then open the Claude Desktop configuration file and replace the UV command with its full path.

Restart Claude Desktop.

Now the proxy server should start correctly.

### Testing

Try these requests:

```text
List all September expenses.
```

```text
Add an expense for groceries worth 500.
```

The proxy forwards the requests to the Remote MCP Server and returns the responses.

### Limitation

Communication through a proxy introduces an additional network and forwarding step, so it may be slower than a direct connection.

---

# PART 7: Identifying the Major Security Problem

## 23. Problem: Shared Database Across Multiple Users

The lecture identifies another major logical flaw in the Expense Tracker.

Suppose 10 users are accessing the same Remote MCP Server.

All users are adding expenses to the same central database.

But the database table does not contain a `user_id` column.

### What Is the Problem?

Consider this situation:

- User A adds an expense of ₹500.
- User B adds an expense of ₹1,000.
- User C asks to see their September expenses.

Because the database does not identify which user created each expense, the server cannot distinguish between the users' records.

As a result, one user may receive expenses belonging to other users.

This is a serious data-isolation and privacy problem.

---

## 24. Solution: Add User ID to the Database

One part of the solution is to add a `user_id` column to the expenses table.

Example:

| id | user_id | amount | category | date |
|---|---|---|---|---|
| 1 | 101 | 500 | Grocery | 2025-09-10 |
| 2 | 102 | 1000 | Food | 2025-09-12 |
| 3 | 101 | 200 | Travel | 2025-09-15 |

Now each expense can be associated with a particular user.

When retrieving expenses, the application should filter records using the authenticated user's ID.

Conceptually:

```sql
SELECT *
FROM expenses
WHERE user_id = ?
AND date BETWEEN ? AND ?;
```

### But There Is Another Problem: Authentication

Adding a `user_id` column is not enough.

The application must know who the actual user is.

In the current implementation:

- Users directly interact with the LLM.
- There is no login system.
- There is no authentication mechanism.
- The server has no trusted way to verify a user's identity.

A user could claim to have another user's ID.

The server must not blindly trust an ID supplied by the user or the LLM.

### What Is Required?

A proper Remote MCP Server needs an authentication mechanism.

The server should:

1. Authenticate the user.
2. Verify the user's identity.
3. Associate requests with the authenticated user.
4. Retrieve only that user's expenses.
5. Prevent access to other users' records.

**Important:** The lecture does not implement this authentication solution. It identifies authentication as a topic for upcoming lectures.

---

## 25. Why the Current Expense Tracker Is Not Ready for Public Use

The lecture identifies the following limitations:

| Problem | Explanation |
|---|---|
| Shared Database | Multiple users access the same database. |
| No User ID | Expenses are not associated with individual users. |
| No Authentication | The server cannot verify the identity of users. |
| Data Privacy | Users may see other users' expenses. |
| Blocking Operations | The original synchronous implementation does not handle concurrent I/O as effectively. |

A Local MCP Server used by one person may not face the same multi-user data-isolation problem.

However, a publicly accessible Remote MCP Server needs proper authentication and user-level data isolation.

---

# PART 8: Complete Architecture

## 26. Final Remote MCP Server Architecture

```text
                  USER
                    |
                    v
             CLAUDE DESKTOP
                    |
                    v
              MCP CLIENT
                    |
                    v
          REMOTE MCP CONNECTION
                    |
                    v
           FASTMCP CLOUD SERVER
                    |
                    v
             FASTMCP SERVER
                    |
          +---------+---------+
          |         |         |
          v         v         v
       ADD       LIST      SUMMARIZE
      EXPENSE   EXPENSES   EXPENSES
          |         |         |
          +---------+---------+
                    |
                    v
             ASYNC DATABASE
                (aiosqlite)
                    |
                    v
               SQLITE DB
```

### Proxy Architecture

For users who cannot directly add a custom remote connector:

```text
Claude Desktop
      |
      | STDIO
      v
Local Proxy MCP Server
      |
      | HTTP
      v
Remote MCP Server
      |
      v
Expense Tracker Database
```

---

## 27. Important Commands Summary

### Project Setup

```bash
pip install uv
uv init .
uv add fastmcp
```

### Run Remote MCP Server

```bash
uv run main.py
```

### Run MCP Inspector

```bash
uv run fastmcp dev main.py
```

### Run FastMCP Server Using HTTP

```bash
uv run fastmcp run main.py --transport http --host 0.0.0.0 --port 8000
```

### Install aiosqlite

```bash
uv add aiosqlite
```

### Install Local Proxy in Claude Desktop

```bash
uv run fastmcp install claude-desktop main.py
```

### Git Commands

```bash
git status
git add .
git commit -m "Your commit message"
git push origin main
```

---

## 28. Important Concepts Covered

| Concept | Meaning |
|---|---|
| Local MCP Server | MCP Server running on the same machine as the client/host. |
| Remote MCP Server | MCP Server running on a remote machine. |
| STDIO | Transport commonly used for local MCP communication. |
| HTTP | Network-based transport used for remote communication. |
| Streamable HTTP | HTTP transport used by the Remote MCP Server in this lecture. |
| FastMCP Cloud | Service used to deploy the Remote MCP Server. |
| MCP Inspector | Tool used to test MCP servers. |
| Proxy Server | An intermediary that forwards communication between systems. |
| Synchronous Programming | Operations execute in a blocking manner. |
| Asynchronous Programming | Allows execution to continue while waiting for asynchronous operations. |
| aiosqlite | Library used for asynchronous SQLite operations. |
| Authentication | Process of verifying a user's identity. |
| User ID | Identifier used to associate data with a particular user. |

---

## 29. Key Takeaways

1. A Remote MCP Server runs on a different machine and can serve multiple clients.
2. FastMCP makes it possible to create Remote MCP Servers with relatively small changes to the server code.
3. The important transport change in this lecture is from STDIO to Streamable HTTP.
4. FastMCP Cloud can deploy an MCP Server using a GitHub repository.
5. An existing Local MCP Server can be converted into a Remote MCP Server by updating its transport and deployment configuration.
6. SQLite database storage must be configured correctly in the deployed environment to support writing.
7. `async` and `await` can help the server handle concurrent I/O operations.
8. `aiosqlite` provides asynchronous SQLite database operations.
9. A Local Proxy MCP Server can forward requests to a Remote MCP Server.
10. A shared database without user-level isolation creates a serious privacy problem.
11. Authentication is essential before exposing a multi-user Expense Tracker as a public service.

---

## 30. What's Next?

In the next lecture, the instructor plans to cover:

- Building a custom MCP Client.
- Connecting to MCP Servers without depending on Claude Desktop.
- Authentication in MCP.
- MCP Sessions.
- Sampling.
- Elicitation.

These concepts will help us build more advanced and secure MCP applications.
