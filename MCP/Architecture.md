# MCP Architecture Guide

[https://www.youtube.com/watch?v=nQa31xdXbGk&list=PLKnIA16_Rmva_oZ9F4ayUu9qcWgF7Fyc0&index=3](https://www.youtube.com/watch?v=nQa31xdXbGk&list=PLKnIA16_Rmva_oZ9F4ayUu9qcWgF7Fyc0&index=3)

## 1. The Core Entities: Host, Client, and Server

The foundation of MCP involves three main components communicating with each other:

- **Host:** The AI chatbot or application the user interacts with (e.g., Claude Desktop, Cursor IDE, or a custom LangGraph bot). The host connects to an LLM (OpenAI, Anthropic, Gemini).
- **Server:** A specialized external tool or service with the capability to execute specific tasks (e.g., a GitHub server, a Slack server, a Google Drive server).
- **Client:** The helper entity living inside the host that translates high-level AI requests into the specific MCP language the server understands.

### The Phone and SIM Analogy

Think of the **Host** as a **Mobile Phone**, the **Server** as the **Network Provider** (Airtel, Jio, Vodafone), and the **Client** as the **SIM Card**. The phone cannot talk to the network directly; it needs the SIM card. Furthermore, the relationship is 1:1 — if you want to connect to three different networks (GitHub, Slack, Drive), your phone needs three different SIM cards (three separate MCP clients).

### End-to-End Execution Example

1. User asks the Host: *"Are there any new commits on the GitHub repo?"*
2. The Host passes this to the LLM. The LLM realizes it doesn't have this live data but knows a GitHub server is available.
3. The Host generates a high-level request and passes it to the Client.
4. The Client translates this into an MCP-compatible request and sends it to the GitHub Server.
5. The GitHub Server fetches the latest commits and sends a structured MCP response back to the Client.
6. The Client translates this back for the Host, the LLM synthesizes the answer, and it is displayed to the user.

---

## 2. Benefits of the Client-Server Architecture

- **Decoupling (Separation of Concerns):** The communication pipeline for GitHub is completely isolated from Slack or Google Drive. If the GitHub connection fails, it does not crash the Slack or Drive integrations, providing system safety.
- **Scalability & Parallelism:** A single host can connect to an infinite number of servers simply by adding more clients. If a user asks a complex prompt requiring data from both Slack and GitHub, the two dedicated clients can fetch this data in parallel, drastically speeding up execution.

---

## 3. MCP Primitives

Primitives are the specific offerings a server provides to a host. There are three types:

### Tools (Dynamic Actions)

Actions the AI can ask the server to perform.

- **GitHub Server Examples:** Get the current commit count, list active issues, or count the number of repositories.
- **Google Drive Server Examples:** Search for a specific file/folder, or create a new file.
- **Standard Operations:** `tools/list`, `tools/call`

### Resources (Static Data)

Structured, static data sources the AI can read.

- **Examples:** Fetching a repository's `README.md` file, or pulling a static database schema.
- **Standard Operations:** `resources/list`, `resources/read`, `resources/subscribe`, `resources/unsubscribe`

### Prompts (Behavior Shaping)

Pre-defined templates offered by the server to guide how the AI formats its requests.

- **The "Login Bug" Example:** A user tells the AI, *"Create an issue for a bug, the login button doesn't work."* Without a prompt primitive, the AI might send a vague, poorly formatted issue to GitHub. However, if the server provides an **"Issue Report Prompt"** (instructing the AI to always include a Title, Steps to Reproduce, Expected Behavior, Actual Behavior, and Environment), the AI will automatically format the user's vague request into a professional, highly detailed GitHub issue before sending it to the server.
- **Standard Operations:** `prompts/list`, `prompts/get`

---

## 4. The Data Layer (JSON-RPC 2.0)

The Data Layer is the standard language and grammar used for all Client-Server communication. MCP uses **JSON-RPC 2.0** (JavaScript Object Notation Remote Procedure Call).

### What is RPC?

Remote Procedure Call allows a program to execute a function on another computer as if it were local. Instead of running `add(2, 3)` locally, you send a payload to a remote machine asking it to run `add` with parameters `2` and `3`.

### JSON-RPC Examples in MCP

#### 1. Basic Request & Response (Tool Discovery)

When the host first connects, it asks the server what tools are available.

```json
// Client Request
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list"
}

// Server Response
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": ["list_issues", "list_pulls"]
  }
}
```

#### 2. Error Handling

If the client calls `list_issues` but only provides the owner name and forgets the repository name:

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "error": {
    "code": -32601,
    "message": "Missing required field: repo"
  }
}
```

#### 3. Notifications

Notifications are "fire-and-forget" messages. If a client uses `resources/subscribe` to watch a file, the server will send a notification when it changes. Notice there is no `id`, because no response is expected back.

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/resources/updated",
  "params": {
    "uri": "file://path/to/document",
    "updated_by": "UserX"
  }
}
```

### Why JSON-RPC Instead of REST APIs?

| Feature | Benefit |
|---|---|
| **Lightweight** | No bulky HTTP headers or metadata; just raw JSON — fast and easy to debug. |
| **Bidirectional** | REST is strictly Client-to-Server. JSON-RPC allows servers to initiate requests to the client (vital for advanced AI agent workflows). |
| **Transport Agnostic** | Works over HTTP, STDIO, WebSockets, or custom transports. |
| **Batching** | Multiple requests (e.g., fetch issues AND fetch PRs) can be sent in a single array payload. |
| **Notifications** | Natively supports sending messages that do not require an acknowledgment. |

---

## 5. The Transport Layer

The Transport Layer is the physical mechanism that moves the JSON-RPC messages between the client and server. The method depends entirely on the server type.

| Server Type | Location | Transport | How It Works |
|---|---|---|---|
| **Local Servers** | Same computer as the Host | STDIO (Standard Input/Output) | The Host launches the server as a sub-process (parent-child relationship). JSON-RPC messages are piped through the system's standard input/output streams. |
| **Remote Servers** | Hosted externally on a network or the internet | HTTP + SSE (Server-Sent Events) | The Host sends JSON-RPC payloads inside HTTP POST requests. Responses stream back using SSE. |

### Transport Layer Examples

#### Local Server Demo (File System & STDIO)

**Action:** Asking Claude Desktop, *"Is there a Python file on my desktop?"*

**Mechanism:** Claude uses the local File System Server. Because it is local, it uses STDIO.

> **The Python Analogy:** When you run `python3 hello.py` in a terminal, the terminal (Host) launches the script (Server). When the script prompts you for your name, you type `"Nitesh"` (Standard Input). The script processes this and prints `"Hello Nitesh"` (Standard Output). MCP local servers do this exact same process invisibly using JSON-RPC strings.

**Benefits of STDIO:** Blazing fast, highly secure (no open network ports), and natively supported by all programming languages.

#### Remote Server Demo (GitHub & SSE)

**Action:** Asking Claude, *"List my top 5 most starred repositories."*

**Mechanism:** Claude uses the remote GitHub MCP Server via HTTP POST.

> **Why SSE?** Server-Sent Events allow the server to stream data back incrementally over a single open connection. If an AI agent is performing a long-running, multi-step task on a remote server, SSE allows the user to see live, chunk-by-chunk updates rather than waiting 30 seconds for a single massive response block.

### The Brilliance of the Architecture

Because the **Data Layer** (JSON-RPC) is entirely separate from the **Transport Layer**, the underlying code logic remains identical. The exact same JSON-RPC text payload works flawlessly whether it is being piped locally through STDIO or sent across the world via HTTP.
