# MCP Lifecycle

[YouTube Video Link](https://www.youtube.com/watch?v=sBHeMcxupmE&list=PLKnIA16_Rmva_oZ9F4ayUu9qcWgF7Fyc0&index=4)

## Table of Contents

- [What is the MCP Lifecycle?](#what-is-the-mcp-lifecycle)
  - [What is a Session?](#what-is-a-session)
- [1. The Initialization Phase (The "Handshake")](#1-the-initialization-phase-the-handshake)
  - [Step 1: Client Sends initialize Request](#step-1-client-sends-initialize-request)
  - [Step 2: Server Responds](#step-2-server-responds)
  - [Step 3: Client Sends initialized Notification](#step-3-client-sends-initialized-notification)
  - [Important Rules for Initialization](#️-important-rules-for-initialization)
- [2. The Operation Phase](#2-the-operation-phase)
  - [Part A: Capability Discovery (Automatic)](#part-a-capability-discovery-automatic)
  - [Part B: Tool Calling (User-Driven)](#part-b-tool-calling-user-driven)
- [3. The Shutdown Phase](#3-the-shutdown-phase)
- [Special Lifecycle Scenarios](#special-lifecycle-scenarios)
  - [1. Pings](#1-pings)
  - [2. Error Handling](#2-error-handling)
  - [3. Timeouts & Cancellation](#3-timeouts--cancellation)
  - [4. Progress Notifications](#4-progress-notifications)

---

## What is the MCP Lifecycle?

The MCP Lifecycle describes the complete sequence of steps that govern how a Host (Client) and a Server **establish, use, and end** a connection during a session.

### What is a Session?

A session is one continuous connection between a client and a server. For example, when you open Claude Desktop (Host/Client), it automatically connects to an installed GitHub MCP Server in the background. As long as Claude remains open, this connection stays alive. The lifecycle defines exactly what happens during this entire period.

The lifecycle is divided into three distinct phases:

1. **Initialization Phase**
2. **Operation Phase**
3. **Shutdown Phase**

---

## 1. The Initialization Phase (The "Handshake")

This phase marks the very first interaction between the Client and the Server. Its primary goals are **checking version compatibility** and **negotiating capabilities** (essentially, both parties telling each other what they can do).

The initialization process happens in **three strict steps:**

### Step 1: Client Sends `initialize` Request

The client reaches out first, providing three pieces of information:

- **Protocol Version:** The version of MCP the client is running.
- **Capabilities:** What the client can offer the server (e.g., `roots` allows the server to access the host's file directories).
- **Implementation Info:** The client's name and version (e.g., Claude Desktop v1.0).

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "roots": {}
    },
    "clientInfo": {
      "name": "Claude Desktop",
      "version": "1.0.0"
    }
  }
}
```

### Step 2: Server Responds

The server replies with its own details:

- **Protocol Version:** To confirm compatibility.
- **Capabilities:** What it can do (e.g., it supports `tools` and `resources`).
- **Implementation Info:** The server's name and version (e.g., File System Server v2.0).

### Step 3: Client Sends `initialized` Notification

Once the client verifies the server's response, it sends an `initialized` notification (a fire-and-forget message with no `id`). This officially finalizes the connection.

### ⚠️ Important Rules for Initialization

- **Strict Order:** Until Step 3 is completed, neither party can send any other commands (like asking for a list of tools). Sending a premature command will crash the system.
- **Version Negotiation:** If the server's protocol version does not match any versions supported by the client, the client will immediately drop the connection, and the session ends right there.

---

## 2. The Operation Phase

Once initialization is complete, the system enters the Operation phase. This phase is governed entirely by the **capabilities negotiated in Step 1**. It is broken down into two main parts:

### Part A: Capability Discovery (Automatic)

Immediately after sending the `initialized` notification, the client automatically asks the server for the exact details of its capabilities.

1. The client sends a `tools/list` request.
2. The server responds with a detailed list of all available functions (e.g., `read_file`, `create_directory`), including the required inputs/schemas for each.
3. The Host stores this list so the LLM knows what tools are available to answer user prompts.

### Part B: Tool Calling (User-Driven)

When a user asks a question (e.g., *"What is written in the hello.py file on my desktop?"*), the client executes a specific tool.

1. The client sends a `tools/call` request specifying the tool name (`read_file`) and the arguments (the path to `hello.py`).
2. The server processes this and returns the content of the file.

---

## 3. The Shutdown Phase

The Shutdown phase **gracefully terminates** the session. Typically, the Client initiates this (e.g., when the user closes Claude Desktop).

> **Crucial Note:** Unlike the first two phases, **no JSON-RPC messages are exchanged** during shutdown. The shutdown is handled entirely by the underlying Transport Layer.

| Server Type | Shutdown Mechanism |
|---|---|
| **Local Servers (STDIO)** | Client closes `stdin`. If server doesn't shut down → `SIGTERM`. If server still refuses → `SIGKILL`. |
| **Remote Servers (HTTP)** | Client closes the active HTTP POST connection. If the server drops unexpectedly, the client must handle the error and attempt to reconnect. |

---

## Special Lifecycle Scenarios

The standard lifecycle is linear, but specific situations require special handling mechanisms.

### 1. Pings

A Ping is a lightweight JSON-RPC method (`"method": "ping"`) used to check if the other side of the connection is still alive.

> **Why use it?** If a server is running a task that takes 20 minutes, the OS, firewalls, or proxies might assume the connection is dead due to inactivity and drop it. Sending periodic pings keeps the connection "warm" and active.

### 2. Error Handling

MCP inherits its error handling directly from standard JSON-RPC 2.0. If something goes wrong, the system returns a standard **Error Object** containing an error code and a message.

- **Example Scenario:** The client asks for a list of Prompts, but the server only supports Tools.
- **Example Error:** The server returns code `-32601` with the message `"Method not found"`.

### 3. Timeouts & Cancellation

If a client sends a request but the server takes too long to respond (e.g., the server is overloaded), the client will trigger a **timeout**.

1. The client sets a threshold (e.g., 30 seconds).
2. If 30 seconds pass with no response, the client sends a `"notifications/cancelled"` message to the server.
3. The server stops processing the request, freeing up CPU and memory, and the client alerts the user that the request timed out.

### 4. Progress Notifications

For long-running tasks, staring at a loading screen is a bad user experience. MCP allows servers to send **periodic progress updates**.

1. When the client makes a request (e.g., *"Scan my entire code base for security vulnerabilities"*), it includes a `progressToken` (e.g., `token: 7`).
2. As the server works, it sends fire-and-forget notifications back to the client referencing `token: 7`.
3. **Example Notification:** `"progress": 60, "total": 100, "message": "Searching 600 out of 1000 files."`
4. The client uses this data to display a **real-time progress bar** to the user.
