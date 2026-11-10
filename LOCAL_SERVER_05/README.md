
# Building a Local MCP Server Using FastMCP

## 📌 Introduction

In this lecture, we move from using existing MCP servers to **building our own MCP servers**.

Previously, we learned:

1. **Why MCP?** – Why do we need Model Context Protocol?
2. **What is MCP?**
   - MCP Architecture
   - MCP Lifecycle
3. **How to use MCP?**
   - Connecting existing MCP servers with Claude Desktop.
   - Using local and remote MCP servers.

Now, we are moving towards developing our own MCP servers.

### MCP Learning Roadmap

The instructor divides the learning process into three major parts:

| Part | Topic | Description |
|---|---|---|
| 1 | Why | Why do we need MCP? |
| 2 | What | MCP Architecture and Lifecycle |
| 3 | How | Building MCP servers and clients |

The practical implementation is further divided into:

- Building local MCP servers.
- Building remote MCP servers.
- Building our own MCP clients.

**This lecture focuses on building a Local MCP Server.**

---

# 1. Project Overview – AI Expense Tracker MCP Server

We are going to build an **Expense Tracker MCP Server**.

The idea is to manage expenses through natural-language conversations with an AI assistant such as Claude Desktop.

Instead of opening an expense-tracking application and filling out forms, we can simply write:

```text
I bought milk today for ₹20.
Add this expense to my database.
```

The AI assistant will understand the request and use our MCP server to store the expense in a database.

Later, we can ask questions such as:

```text
How much did I spend in the last 7 days?
```

```text
How much did I spend on entertainment this month?
```

```text
Show me all my expenses from last week.
```

The assistant will use the MCP tools to retrieve and analyze the stored data.

### Main Objective

Build an MCP server that allows an AI assistant to:

- Add expenses.
- List expenses.
- Summarize expenses.
- Analyze spending by category.
- Retrieve expenses within a date range.

### Why Use MCP for Expense Tracking?

Traditional expense-tracking applications generally require users to enter information through forms.

With an MCP server:

- We can enter expenses using natural language.
- The LLM can interpret dates and categories.
- Data is stored in a database.
- We can ask questions about our spending.
- The assistant can retrieve information using MCP tools.

---

# 2. Development Roadmap

The instructor follows an incremental development approach.

Instead of directly building the complete Expense Tracker, we first create a simple demo server.

### Development Stages

| Stage | What We Build |
|---|---|
| Stage 1 | Basic demo MCP server |
| Stage 2 | Add expense and list expenses |
| Stage 3 | Add date-range filtering |
| Stage 4 | Add expense summarization |
| Stage 5 | Add category resources |

The final result will be a functional local Expense Tracker MCP Server.

### Local vs Remote Server

In this lecture, our server runs on our own machine.

Communication happens through the **STDIO transport**.

In the next stage of the learning roadmap, the instructor plans to convert this local server into a remote server by hosting it on a remote service.

---

# 3. Understanding MCP SDK vs FastMCP

Before writing code, we need to understand an important distinction.

When searching for tutorials about building MCP servers in Python, we commonly encounter two names:

1. MCP SDK
2. FastMCP

Both are related, but they have different roles and development approaches.

## 3.1 What Is MCP SDK?

MCP is a protocol introduced by Anthropic.

A protocol defines a set of rules that different applications follow to communicate with each other.

If we implement all these rules manually, building MCP servers and clients becomes complicated.

To simplify this process, Anthropic introduced the official Python MCP SDK.

The SDK provides three important components:

| Component | Purpose |
|---|---|
| `mcp.server` | Build MCP servers |
| `mcp.client` | Build MCP clients |
| `mcp.cli` | Run and manage MCP-related commands |

Install the MCP SDK with:

```bash
pip install "mcp[cli]"
```

### Problem with the Initial MCP SDK Approach

The initial server implementation involved a lot of boilerplate code.

Even a simple operation, such as adding two numbers, required relatively more code.

Developers had to handle more of the underlying implementation details.

This made the development process less beginner-friendly.

---

## 3.2 What Is FastMCP?

To simplify MCP development, **Jeremiah Lowin**, the creator of Prefect, developed FastMCP as an abstraction over the MCP SDK.

FastMCP makes building MCP servers easier and more concise.

Instead of writing a lot of boilerplate code, we can:

1. Create a FastMCP server instance.
2. Write normal Python functions.
3. Add decorators to expose those functions as MCP tools.
4. Run the server.

### Example: MCP SDK vs FastMCP

Conceptually, both approaches can create the same functionality.

However, FastMCP provides a simpler developer experience.

| MCP SDK | FastMCP |
|---|---|
| Lower-level implementation | Higher-level abstraction |
| More boilerplate in the initial approach | Less boilerplate |
| More implementation details | Simpler development |
| Official MCP SDK | Developer-friendly framework |

## 3.3 Evolution of FastMCP

The instructor explains the evolution of the MCP Python ecosystem chronologically.

### Stage 1: MCP Protocol

Anthropic introduced MCP.

Developers wanted to create their own MCP clients and servers but found implementing the protocol manually complicated.

### Stage 2: MCP SDK

Anthropic introduced the official Python SDK.

It simplified the implementation, but the initial server-development approach was still relatively verbose.

### Stage 3: FastMCP 1.0

Jeremiah Lowin created FastMCP as an abstraction over the MCP SDK.

It made server development simpler and more beginner-friendly.

FastMCP became popular because developers could create MCP servers with less code.

### Stage 4: FastMCP Became Part of the SDK Ecosystem

FastMCP 1.0 was adopted into the MCP SDK's server-development experience.

This made the simpler approach available through the SDK ecosystem.

### Stage 5: FastMCP 2.0

FastMCP later became an independent library again, with its own development direction and additional features.

The instructor distinguishes between:

- Installing the MCP SDK, which includes the SDK's FastMCP-based server approach.
- Installing the independent FastMCP 2.x library.

For this lecture, the instructor chooses **FastMCP 2.x**.

Install it using:

```bash
pip install fastmcp
```

Or, as used in this project:

```bash
uv add fastmcp
```

**Important:** The version history described here follows the lecture. For current installation and compatibility details, check the official documentation.

---

## 3.4 FastMCP and FastAPI Connection

The instructor also introduces an interesting relationship between FastMCP and FastAPI.

FastMCP follows a developer-friendly design philosophy similar to FastAPI.

Both focus on simplifying development through abstractions and convenient interfaces.

The lecture demonstrates that FastMCP can work with existing FastAPI applications.

This is useful when a company already has a backend API and wants to expose its functionality through MCP.

We will understand this in Section 11.

---

# 4. Setting Up the Development Environment

We will use:

- Python
- UV
- FastMCP
- VS Code
- Claude Desktop
- SQLite
- MCP Inspector

## 4.1 What Is UV?

UV is a Python package and project manager.

It is used to manage Python dependencies and project environments.

The instructor uses UV because FastMCP recommends it for project management.

Install UV:

```bash
pip install uv
```

## 4.2 Create the Project Folder

Create a new folder on your Desktop:

```text
expense-tracker-mcp-server
```

Open this folder in VS Code.

## 4.3 Initialize the UV Project

Open the terminal inside VS Code.

Run:

```bash
uv init .
```

The command initializes the current directory as a UV project.

It generates the initial project files.

A typical project structure will look like:

```text
expense-tracker-mcp-server/
│
├── main.py
├── pyproject.toml
├── README.md
└── .python-version
```

## 4.4 Install FastMCP

Run:

```bash
uv add fastmcp
```

UV installs FastMCP and its dependencies into the project.

## 4.5 Verify FastMCP Installation

Run:

```bash
uv run fastmcp version
```

The command displays information such as:

- FastMCP version.
- MCP SDK version.
- Python version.
- Operating system.
- Installation path.

The instructor's demonstration shows FastMCP 2.11.

Your installed version may be different.

---

# 5. Building Our First Demo MCP Server

Before building the Expense Tracker, we will create a basic MCP server.

The purpose is to understand the complete development workflow.

### Features of the Demo Server

We will create two tools:

1. `roll_dice`
2. `add_numbers`

## 5.1 Create a FastMCP Server

First, import FastMCP:

```python
from fastmcp import FastMCP
```

Create a server instance:

```python
mcp = FastMCP("Demo Server")
```

Here:

- `FastMCP` is the class used to create an MCP server.
- `mcp` is our server instance.
- `"Demo Server"` is the server name.

## 5.2 Create the Roll Dice Tool

Create a normal Python function that generates random dice values.

Conceptually:

```python
@mcp.tool()
def roll_dice(number: int) -> list[int]:
    """
    Roll the specified number of dice.
    """
```

The important concept is the decorator:

```python
@mcp.tool()
```

This decorator exposes the Python function as an MCP tool.

The AI client can discover and call this tool.

## 5.3 Create the Add Numbers Tool

Now create another function:

```python
@mcp.tool()
def add_numbers(a: int, b: int) -> int:
    """
    Add two numbers.
    """
```

This function takes two numbers and returns their sum.

Again, the decorator:

```python
@mcp.tool()
```

converts the Python function into an MCP tool.

## 5.4 Run the Server

At the end of the file, use:

```python
if __name__ == "__main__":
    mcp.run()
```

This starts the MCP server.

### Complete Conceptual Structure

```text
main.py
   |
   ├── Import FastMCP
   |
   ├── Create FastMCP instance
   |
   ├── Define roll_dice()
   |       |
   |       └── @mcp.tool()
   |
   ├── Define add_numbers()
   |       |
   |       └── @mcp.tool()
   |
   └── Run MCP server
```

**Important:** The code snippets above show the main structure and concepts from the lecture. The complete implementation also needs the appropriate Python imports and function bodies.

---

# 6. Testing the Server Using MCP Inspector

Before connecting our server to Claude Desktop, we should test it.

For this, we use **MCP Inspector**.

## 6.1 What Is MCP Inspector?

MCP Inspector is a debugging and testing tool for MCP servers.

It allows us to:

- Connect to an MCP server.
- Inspect server capabilities.
- Discover available tools.
- Test tool calls.
- View JSON-RPC communication.
- Inspect resources and prompts.

It is similar in purpose to how Postman helps developers test APIs.

## 6.2 Start MCP Inspector

Run:

```bash
uv run fastmcp dev main.py
```

This starts the development environment and launches MCP Inspector.

## 6.3 Connect to the Server

The instructor uses the STDIO transport.

Select the appropriate transport and click **Connect**.

If the connection succeeds, our MCP server is running correctly.

## 6.4 Explore MCP Primitives

MCP Inspector provides sections for:

- Resources
- Prompts
- Tools

Our demo server currently has only tools.

Click the Tools section.

## 6.5 List Available Tools

Click:

```text
List Tools
```

Behind the scenes, the client sends a JSON-RPC request:

```json
{
  "jsonrpc": "2.0",
  "method": "tools/list",
  "id": 1
}
```

The server responds with the available tools.

We should see:

```text
roll_dice
add_numbers
```

## 6.6 Test the Add Numbers Tool

Select `add_numbers`.

Provide:

```json
{
  "a": 5,
  "b": 6
}
```

Click Run Tool.

Expected result:

```text
11
```

## 6.7 Understand JSON-RPC Communication

MCP uses JSON-RPC messages for communication.

When we list tools, the method is:

```text
tools/list
```

When we execute a tool, the method is:

```text
tools/call
```

MCP Inspector displays the communication history.

This allows us to understand what messages are exchanged between the client and server.

**Why use MCP Inspector?**

It is better to test and debug a server independently before integrating it with Claude Desktop.

---

# 7. Running and Integrating the Demo Server with Claude Desktop

After testing, we can run the server and connect it to Claude Desktop.

## 7.1 Run the Server

Run:

```bash
uv run fastmcp run main.py
```

This starts the MCP server.

The server runs locally on our machine.

Since we are using STDIO transport, the MCP client communicates with the server through standard input and output.

## 7.2 Install the Server in Claude Desktop

FastMCP provides a CLI command to install the server into supported clients.

Run:

```bash
uv run fastmcp install claude-desktop main.py
```

This adds the server configuration to Claude Desktop.

## 7.3 Fix the UV Path Issue

In the lecture, Claude Desktop initially fails to connect to the server.

The reason is that the configuration contains:

```text
uv
```

but Claude Desktop cannot correctly locate the executable.

### Solution

Find the absolute path of UV.

On Windows:

```bash
where uv
```

On macOS/Linux:

```bash
which uv
```

Copy the absolute path.

Open Claude Desktop:

```text
Settings
   |
   └── Developer
         |
         └── Edit Config
```

Replace the command:

```text
uv
```

with the actual absolute executable path.

Save the configuration.

Completely close Claude Desktop and restart it.

## 7.4 Test the Tools in Claude Desktop

After restarting, open the available MCP tools.

We should see:

```text
Demo Server
   |
   ├── roll_dice
   |
   └── add_numbers
```

Try these prompts:

```text
Roll a dice.
```

```text
Roll two dice.
```

```text
Add 234 and 567.
```

Claude Desktop calls the appropriate MCP tools and returns the results.

**We have successfully built and integrated our first local MCP server!**

---

# 8. Building the Expense Tracker MCP Server

Now that we understand the complete workflow, we can replace our demo server with the Expense Tracker.

### Main Features

Our Expense Tracker will have three main features:

| Tool | Functionality |
|---|---|
| `add_expense` | Add a new expense |
| `list_expenses` | Retrieve expenses |
| `summarize_expenses` | Summarize spending |

We will use SQLite as our database.

## 8.1 Why SQLite?

In production applications, we might use databases such as:

- MySQL
- PostgreSQL
- Oracle

However, our main objective is learning MCP development.

Therefore, we use SQLite because:

- It is easy to set up.
- It does not require a separate database server.
- It stores data in a local database file.
- It works well for this learning project.

Our database file will be:

```text
expenses.db
```

It will be stored inside the project folder.

## 8.2 Database Schema

We create a table named:

```text
expenses
```

The table contains the following columns:

| Column | Purpose |
|---|---|
| `id` | Unique ID for every transaction |
| `date` | Date of the transaction |
| `amount` | Expense amount |
| `category` | Main expense category |
| `subcategory` | More specific expense type |
| `note` | Additional description |

### Example Database Entry

| id | date | amount | category | subcategory | note |
|---|---|---:|---|---|---|
| 1 | 2026-11-01 | 500 | Transportation | Cab | Cab ride to Delhi |
| 2 | 2026-11-02 | 500 | Groceries | Food | Grocery purchase |

*Example data for understanding the schema.*

## 8.3 Initialize the Database

We create a function responsible for initializing the database.

Its responsibilities:

1. Connect to SQLite.
2. Check whether the `expenses` table exists.
3. Create the table if it does not exist.
4. Define the required columns.

Conceptually:

```sql
CREATE TABLE IF NOT EXISTS expenses (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    date TEXT,
    amount REAL,
    category TEXT,
    subcategory TEXT,
    note TEXT
);
```

The instructor uses Python's SQLite functionality to execute SQL queries.

---

# 9. Iteration 1 – Add and List Expenses

In the first version, we implement two tools.

## 9.1 Tool 1: add_expense

The purpose of this tool is to add a new expense to the database.

It accepts:

- Date
- Amount
- Category
- Subcategory
- Note

### Basic Working

1. Receive expense details.
2. Connect to the SQLite database.
3. Execute an SQL INSERT query.
4. Store the expense.
5. Return a confirmation.

Example natural-language prompt:

```text
Add an expense for a cab ride to Delhi.
The fare was ₹500 yesterday.
```

Claude interprets the request and calls `add_expense`.

The expense is stored in the database.

### Tool Description

```text
Add a new expense entry to the database.
```

A clear tool description helps the LLM understand when to use the tool.

## 9.2 Tool 2: list_expenses

The purpose of this tool is to retrieve expense entries from the database.

In the first version, it retrieves all expenses.

Conceptually, the SQL query is:

```sql
SELECT *
FROM expenses
ORDER BY date ASC;
```

The tool:

1. Connects to the database.
2. Executes the SQL query.
3. Retrieves expense records.
4. Returns the results.

### Tool Description

```text
List all expense entries from the database.
```

### Testing in Claude Desktop

Example prompts:

```text
Add an expense for groceries yesterday for ₹500.
```

```text
Show me all my expenses.
```

Claude calls the corresponding tools.

We can also open the `expenses.db` file and verify that the records have actually been stored.

### Limitation of Version 1

The first version retrieves all expenses together.

However, users generally want to retrieve expenses within a specific date range.

We solve this in the next iteration.

---

# 10. Iteration 2 – Add Date-Range Filtering

We improve the `list_expenses` tool.

Instead of retrieving all expenses, it will now accept two parameters:

- `start_date`
- `end_date`

### New Functionality

The tool retrieves expenses between the given dates.

We use an SQL `WHERE` clause with `BETWEEN`.

Example:

```sql
SELECT *
FROM expenses
WHERE date BETWEEN ? AND ?
ORDER BY date ASC;
```

The `?` placeholders are used for parameterized query values.

### Example Prompts

```text
Show me all expenses from the first week of September.
```

```text
List my expenses from last week.
```

Claude understands the date range and passes the appropriate parameters to the tool.

### What Happens?

1. User provides a natural-language date range.
2. Claude interprets the start date.
3. Claude interprets the end date.
4. Claude calls `list_expenses`.
5. The server executes the SQL query.
6. Matching expenses are returned.

We can also ask:

```text
List my expenses from last week in tabular format.
```

Claude can present the returned records in a table.

---

# 11. Iteration 3 – Summarize Expenses by Category

Now we add the third tool:

```text
summarize_expenses
```

The objective is to calculate total spending within a date range.

We also want to filter by category.

### Function Parameters

The tool accepts:

1. `start_date`
2. `end_date`
3. `category` (optional)

### Two Possible Cases

**Case 1: Category is provided**

Calculate total spending for that category within the given date range.

Example:

```text
How much did I spend on Education in the last 10 days?
```

**Case 2: Category is not provided**

Calculate total spending across all categories within the given date range.

Example:

```text
How much did I spend in the last 7 days?
```

## 11.1 SQL Query

The base query:

```sql
SELECT category, SUM(amount) AS total_amount
FROM expenses
WHERE date BETWEEN ? AND ?
```

If the category is provided, append:

```sql
AND category = ?
```

Then group the results:

```sql
GROUP BY category
ORDER BY category ASC;
```

### Understanding the Query

| SQL Clause | Purpose |
|---|---|
| `SELECT` | Selects the category and total amount |
| `SUM(amount)` | Calculates total spending |
| `FROM expenses` | Reads from the expenses table |
| `WHERE` | Filters records |
| `BETWEEN` | Filters by date range |
| `AND category = ?` | Applies optional category filtering |
| `GROUP BY` | Groups records by category |
| `ORDER BY` | Sorts the result |

### Example Prompt

```text
What was my total expense on Education in the last 10 days?
```

Claude identifies:

- Start date
- End date
- Category = Education

It calls `summarize_expenses`.

The server executes the SQL query and returns the result.

---

# 12. Iteration 4 – Add Categories as an MCP Resource

Now we address an important problem.

## 12.1 The Problem: Inconsistent Categories

When adding expenses, the LLM automatically determines the category.

For example, it might classify an expense as:

```text
Education
```

But in another request, it might generate:

```text
education
```

Or:

```text
Upskilling
```

These inconsistent values can cause problems during analysis.

For example, spending may be divided across multiple category names even though the expenses belong to the same category.

### Another Problem: Missing Subcategories

The database contains a `subcategory` column.

However, Claude sometimes ignores it while adding expenses.

We need a way to provide a consistent list of available categories and subcategories.

## 12.2 Solution: Use an MCP Resource

We create a JSON file containing all available categories and their subcategories.

Example structure:

```json
{
  "Transportation": [
    "Cab",
    "Bus",
    "Train",
    "Fuel"
  ],
  "Education": [
    "Online Courses",
    "Books",
    "Subscriptions"
  ],
  "Entertainment": [
    "Movies",
    "Games",
    "Subscriptions"
  ],
  "Groceries": [
    "Milk",
    "Vegetables",
    "Food"
  ],
  "Health": [
    "Medicines",
    "Doctor",
    "Fitness"
  ]
}
```

*This is an illustrative structure. Use the actual categories defined in your project.*

## 12.3 What Is an MCP Resource?

An MCP Resource allows a server to expose data or contextual information to an MCP client.

Unlike tools, which are designed to perform operations, resources provide information that the client can read.

In our project, the resource provides the available expense categories.

## 12.4 Create a Resource Using FastMCP

FastMCP provides the decorator:

```python
@mcp.resource("...")
```

We use it to expose a resource.

Conceptually:

```python
@mcp.resource("...")
def categories():
    # Open the categories JSON file
    # Read its contents
    # Return the category data
```

The function:

1. Opens the JSON file.
2. Reads its content.
3. Returns the category information.

We also define the correct file path in the project.

## 12.5 How Does It Work?

The resource becomes available through the MCP server.

In Claude Desktop:

1. Open the resource section.
2. Find the Expense Tracker resource.
3. Select the categories resource.
4. Read its contents.

Claude now has access to the predefined categories and subcategories.

### Example Prompt

```text
Add a new expense:
Cab ride to airport last Wednesday for ₹700.

Pick the category and subcategory from the
provided categories resource.
```

Now Claude can use the predefined category information rather than inventing category names.

### Benefits

- Consistent categories.
- Consistent subcategories.
- Better database organization.
- More reliable expense analysis.
- Reduced inconsistency in LLM-generated values.

**Important:** The resource provides category information to the client. The application should still validate incoming category values before inserting them into the database.

---

# 13. Final Expense Tracker Architecture

After completing the iterations, our local MCP server has three tools and one resource.

## MCP Tools

| Tool | Purpose |
|---|---|
| `add_expense` | Insert a new expense into SQLite |
| `list_expenses` | Retrieve expenses within a date range |
| `summarize_expenses` | Calculate spending totals |

## MCP Resource

| Resource | Purpose |
|---|---|
| Categories JSON | Provides valid categories and subcategories |

### Architecture Diagram

```text
                   USER
                     |
                     |
             Natural Language
                     |
                     v
              CLAUDE DESKTOP
                     |
                     |
                MCP CLIENT
                     |
                     |
              STDIO TRANSPORT
                     |
                     v
          EXPENSE TRACKER MCP SERVER
                     |
          +----------+----------+
          |          |          |
          v          v          v
       add_       list_      summarize_
      expense    expenses    expenses
          |          |          |
          +----------+----------+
                     |
                     v
               SQLITE DATABASE
                 expenses.db

                     |
                     v
                MCP RESOURCE
                     |
                     v
               categories.json
```

### Complete Project Structure

A conceptual project structure:

```text
expense-tracker-mcp-server/
│
├── main.py
│
├── test.py
│
├── expenses.db
│
├── categories.json
│
├── pyproject.toml
│
├── uv.lock
│
├── .python-version
│
└── README.md
```

- `main.py` – Main MCP server and its tools/resources.
- `test.py` – Temporary testing file used during development.
- `expenses.db` – SQLite database containing expense records.
- `categories.json` – Available expense categories and subcategories.
- `pyproject.toml` – Project configuration and dependencies.
- `uv.lock` – Locked dependency versions.

---

# 14. Additional Features to Implement

The instructor recommends extending the project after completing the lecture.

Possible features include:

| Feature | Description |
|---|---|
| Edit Expense | Modify an existing expense |
| Delete Expense | Delete an expense |
| Add Credit | Record incoming money |
| Budget Management | Set spending limits |
| Monthly Reports | Generate monthly expense reports |
| Category Analysis | Compare spending across categories |

These features can be implemented by adding more MCP tools.

---

# 15. FastMCP and FastAPI Integration

At the end of the lecture, the instructor explains an important feature: **FastMCP can integrate with an existing FastAPI application.**

## 15.1 Understanding the Problem

Imagine a company has built an Expense Tracker application.

The company already has:

- A website.
- An Android application.
- An iPhone application.
- A FastAPI backend.
- A database.

The existing FastAPI backend provides APIs used by all these applications.

### Existing Architecture

```text
              WEBSITE
                 |
                 |
            ANDROID APP
                 |
                 |
             IPHONE APP
                 |
                 v
            FASTAPI BACKEND
                 |
                 v
              DATABASE
```

The same backend serves all three platforms.

Now the company wants to make its Expense Tracker available through AI applications such as Claude Desktop.

It needs an MCP server.

### Traditional Approach

The company might have to rewrite the existing functionality using FastMCP.

This could duplicate a lot of existing business logic.

## 15.2 FastMCP Solution

FastMCP provides an integration mechanism for exposing an existing FastAPI application through MCP.

The lecture demonstrates:

```python
from fastmcp import FastMCP
from main import app
```

Here:

- `FastMCP` is imported from FastMCP.
- `app` is the existing FastAPI application.

Create an MCP server using the FastAPI application:

```python
mcp = FastMCP.from_fastapi(
    app=app,
    name="Expense Tracker"
)
```

The exact API and supported behavior depend on the FastMCP version.

The important idea is that FastMCP can expose functionality from an existing FastAPI application through MCP.

## 15.3 Existing FastAPI Endpoints

The instructor's demonstration contains three endpoints:

| Endpoint | Purpose |
|---|---|
| `POST /expenses` | Add an expense |
| `GET /expenses` | Retrieve expenses |
| `GET /expense-summary` | Get expense summary |

These endpoints already implement the Expense Tracker functionality.

Instead of rewriting the entire backend, FastMCP can expose the existing API functionality through MCP.

## 15.4 Testing with MCP Inspector

Run the FastMCP development command for the MCP server file.

Example:

```bash
uv run fastmcp dev server.py
```

Connect through MCP Inspector.

The inspector discovers tools corresponding to the supported FastAPI endpoints.

The instructor demonstrates that the expense-adding functionality is available through the MCP interface.

## 15.5 Integrating with Claude Desktop

The instructor uses the FastMCP installation command:

```bash
uv run fastmcp install claude-desktop server.py
```

This installs the MCP server configuration in Claude Desktop.

**Important:** The instructor also mentions removing the older demo server configuration to avoid conflicts.

### Main Benefit

We do not necessarily need to rewrite an entire existing backend to make its functionality available through MCP.

FastMCP can help bridge an existing FastAPI application and MCP.

---

# 16. Important Concepts to Remember

| Concept | Meaning |
|---|---|
| MCP | Protocol for communication between AI applications and external tools |
| MCP SDK | Official Python SDK for MCP development |
| FastMCP | Developer-friendly framework for creating MCP servers |
| UV | Python package and project manager |
| MCP Inspector | Tool for testing and debugging MCP servers |
| STDIO | Transport used for local client-server communication |
| SQLite | Local database used to store expenses |
| MCP Tool | Allows an AI client to request an operation |
| MCP Resource | Exposes data or contextual information |
| FastAPI | Python framework used to build APIs |

---

# 17. Final Takeaways

In this lecture, we learned how to build our own Local MCP Server using FastMCP.

### What We Built

1. A basic demo MCP server.
2. A `roll_dice` tool.
3. An `add_numbers` tool.
4. An Expense Tracker MCP server.
5. An `add_expense` tool.
6. A `list_expenses` tool.
7. A `summarize_expenses` tool.
8. A categories JSON resource.

### What We Learned

- How to install and use FastMCP.
- How to initialize a Python project using UV.
- How to create MCP tools using decorators.
- How to run a local MCP server.
- How to test a server using MCP Inspector.
- How to connect a server with Claude Desktop.
- How to use SQLite with an MCP server.
- How to expose structured data through MCP Resources.
- How FastMCP can integrate with existing FastAPI applications.

### Final Understanding

The main idea of this project is:

**We can expose our Python functions and existing application functionality through MCP so that AI assistants can discover and use them.**

Instead of manually interacting with an application, users can communicate through natural language while the AI assistant uses MCP tools to perform operations.

---

## 🚀 What's Next?

In the next lecture, we will take our local Expense Tracker MCP Server and convert it into a remote MCP server.

We will learn how to:

1. Improve the local server.
2. Make it suitable for remote access.
3. Host it on a remote service.
4. Allow clients from different machines to access it.

This will take us one step closer to building production-oriented MCP applications.
