# Model Context Protocol (MCP) - Complete Guide

## Table of Contents
1. [What is MCP?](#what-is-mcp)
2. [Core Architecture](#core-architecture)
3. [Protocol Specification](#protocol-specification)
4. [Transports](#transports)
5. [Tools, Resources & Prompts](#tools-resources--prompts)
6. [Authentication & Security](#authentication--security)
7. [Building Your First Server](#building-your-first-server)

---

## What is MCP?

**Model Context Protocol (MCP)** is an open standard created by Anthropic that enables AI applications to integrate with external tools, data sources, and services through a unified interface.

### Why MCP Matters

Before MCP, every AI application needed custom integrations for each tool or data source. MCP solves this by providing:

- **Universal Interface**: One protocol for all AI integrations (like HTTP for the web)
- **Secure Access**: Controlled, auditable access to external systems
- **Bidirectional Communication**: AI clients and servers communicate seamlessly
- **Reduced Fragmentation**: Standard protocol eliminates one-off integrations
- **Context Sharing**: Real-time data access across multiple sources

### Key Benefits

```
✓ Standardization    - Single protocol vs. custom integrations
✓ Security          - Isolated processes with clear boundaries
✓ Scalability       - Deploy multiple servers independently
✓ Real-time         - Push updates and subscriptions
✓ Composability     - Chain operations across servers
✓ Language Agnostic - Implement in any language
```

---

## Core Architecture

### Client-Server Model

```
┌─────────────────────┐         ┌─────────────────────┐
│   AI Application    │ ◄─────► │    MCP Server       │
│     (Client)        │   MCP   │   (Tools/Data)      │
└─────────────────────┘ Protocol└─────────────────────┘
         │                               │
         │                               │
    [Uses Tools]                  [Exposes Tools]
    [Reads Resources]             [Provides Resources]
    [Gets Prompts]                [Offers Prompts]
```

### Key Components

| Component | Description |
|-----------|-------------|
| **Client** | AI application that needs tools/data (e.g., Claude Desktop) |
| **Server** | Process that exposes tools, resources, and prompts |
| **Protocol** | JSON-RPC 2.0 based communication format |
| **Transport** | How messages are sent (stdio, HTTP, WebSocket, SSE) |

### Communication Flow

```
1. Client connects to Server
2. Initialize & negotiate capabilities
3. Discover tools/resources/prompts
4. Execute operations (call tools, read resources)
5. Process results and continue conversation
```

---

## Protocol Specification

MCP uses **JSON-RPC 2.0** for all communication.

### Message Format

#### Request
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list",
  "params": {}
}
```

#### Response
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [...]
  }
}
```

#### Error
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -32600,
    "message": "Invalid Request",
    "data": {}
  }
}
```

### Core Operations

| Method | Purpose | Direction |
|--------|---------|-----------|
| `initialize` | Handshake & capability negotiation | Client → Server |
| `ping` | Keep-alive check | Bidirectional |
| `tools/list` | List available tools | Client → Server |
| `tools/call` | Execute a tool | Client → Server |
| `resources/list` | List available resources | Client → Server |
| `resources/read` | Read resource content | Client → Server |
| `resources/subscribe` | Watch for changes | Client → Server |
| `prompts/list` | List prompt templates | Client → Server |
| `prompts/get` | Get specific prompt | Client → Server |
| `sampling/createMessage` | Request model inference | Server → Client |

### Initialization Sequence

```json
// 1. Client sends initialize request
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "sampling": {}
    },
    "clientInfo": {
      "name": "my-ai-app",
      "version": "1.0.0"
    }
  }
}

// 2. Server responds with capabilities
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2024-11-05",
    "capabilities": {
      "tools": {},
      "resources": {
        "subscribe": true
      },
      "prompts": {}
    },
    "serverInfo": {
      "name": "my-mcp-server",
      "version": "1.0.0"
    }
  }
}
```

### Error Codes

| Code | Meaning |
|------|---------|
| `-32700` | Parse error |
| `-32600` | Invalid Request |
| `-32601` | Method not found |
| `-32602` | Invalid params |
| `-32603` | Internal error |
| `-32000` | Server error (rate limit, auth, etc.) |

---

## Transports

MCP supports multiple transport mechanisms. Choose based on your deployment needs.

### 1. STDIO (Standard Input/Output)

**Best for**: Local integrations, simple deployments, development

```bash
# Server reads from stdin, writes to stdout
echo '{"jsonrpc":"2.0","id":1,"method":"initialize"}' | python server.py
```

**Characteristics**:
- No network overhead
- Simple debugging
- Process-based isolation
- Common for CLI tools

### 2. HTTP

**Best for**: REST-like integrations, web services

```http
POST /mcp HTTP/1.1
Host: api.example.com
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list"
}
```

**Characteristics**:
- Familiar to web developers
- Easy debugging with curl/Postman
- Stateless by default
- Load balancer compatible

### 3. Server-Sent Events (SSE)

**Best for**: Real-time updates, resource subscriptions

```
GET /mcp/sse HTTP/1.1
Host: api.example.com

data: {"jsonrpc":"2.0","method":"resources/updated"}
```

**Characteristics**:
- Server pushes updates to client
- Efficient for many resources
- Browser-compatible
- One-way communication (server → client)

### 4. WebSocket

**Best for**: Bidirectional real-time communication

```javascript
ws://mcp-server:8000/connect

// Send
{"jsonrpc": "2.0", "id": 1, "method": "tools/list"}

// Receive
{"jsonrpc": "2.0", "id": 1, "result": {...}}
```

**Characteristics**:
- Full-duplex communication
- Low latency
- Suitable for interactive tools
- Persistent connection

### Transport Comparison

| Feature | STDIO | HTTP | SSE | WebSocket |
|---------|-------|------|-----|-----------|
| Real-time | ✓ | ✗ | ✓ | ✓ |
| Bidirectional | ✓ | ✗ | Partial | ✓ |
| Network | ✗ | ✓ | ✓ | ✓ |
| Complexity | Low | Low | Medium | High |
| Scalability | Low | High | High | Medium |

---

## Tools, Resources & Prompts

The three core primitives of MCP.

### Tools

**Purpose**: Expose functions that AI can call

**Definition**:
```json
{
  "name": "search_web",
  "description": "Search the web for information",
  "inputSchema": {
    "type": "object",
    "properties": {
      "query": {
        "type": "string",
        "description": "Search query"
      },
      "max_results": {
        "type": "number",
        "description": "Maximum number of results",
        "default": 10
      }
    },
    "required": ["query"]
  }
}
```

**Calling a Tool**:
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "search_web",
    "arguments": {
      "query": "Model Context Protocol",
      "max_results": 5
    }
  }
}
```

**Tool Response**:
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Found 5 results for 'Model Context Protocol':\n1. ..."
      }
    ]
  }
}
```

**When to Use Tools**:
- Actions with side effects (write, update, delete)
- Function execution
- API calls
- Data transformations
- Complex operations

### Resources

**Purpose**: Provide access to data and information

**Definition**:
```json
{
  "uri": "file:///config/settings.json",
  "name": "Application Settings",
  "description": "Current application configuration",
  "mimeType": "application/json"
}
```

**Reading a Resource**:
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "resources/read",
  "params": {
    "uri": "file:///config/settings.json"
  }
}
```

**Resource Response**:
```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "contents": [
      {
        "uri": "file:///config/settings.json",
        "mimeType": "application/json",
        "text": "{\"theme\": \"dark\", \"version\": \"1.0\"}"
      }
    ]
  }
}
```

**Resource URIs**:
```
file:///path/to/file.txt        - File system
http://api.example.com/data     - HTTP endpoint
memory://user_preferences        - In-memory data
database://users/123             - Database record
weather://london/current         - Custom protocol
```

**Subscribing to Resources**:
```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "resources/subscribe",
  "params": {
    "uri": "memory://user_preferences"
  }
}

// Server pushes updates when resource changes
{
  "jsonrpc": "2.0",
  "method": "notifications/resources/updated",
  "params": {
    "uri": "memory://user_preferences"
  }
}
```

**When to Use Resources**:
- Read-only or read-mostly data
- Files, documents, configurations
- Database records
- API responses
- Real-time data streams (with subscriptions)

### Prompts

**Purpose**: Reusable prompt templates and system instructions

**Definition**:
```json
{
  "name": "code_review",
  "description": "Review code for quality and best practices",
  "arguments": [
    {
      "name": "code",
      "description": "Code to review",
      "required": true
    },
    {
      "name": "language",
      "description": "Programming language",
      "required": true
    }
  ]
}
```

**Getting a Prompt**:
```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "method": "prompts/get",
  "params": {
    "name": "code_review",
    "arguments": {
      "code": "def hello(): print('hi')",
      "language": "python"
    }
  }
}
```

**Prompt Response**:
```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "result": {
    "messages": [
      {
        "role": "system",
        "content": {
          "type": "text",
          "text": "You are an expert code reviewer. Review the following Python code for quality, best practices, and potential issues."
        }
      },
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "Please review this code:\n\ndef hello(): print('hi')"
        }
      }
    ]
  }
}
```

**When to Use Prompts**:
- Consistent prompt engineering
- Role-specific instructions
- Complex multi-turn prompts
- Parameterized system messages
- Reusable templates

### Comparison Table

| Feature | Tools | Resources | Prompts |
|---------|-------|-----------|---------|
| Purpose | Execute functions | Provide data | Offer instructions |
| Side Effects | Yes | No | No |
| Subscriptions | No | Yes | No |
| Parameterized | Yes | No | Yes |
| Examples | API calls, calculations | Files, configs | System messages |

---

## Authentication & Security

Security is critical for production MCP deployments.

### Authentication Methods

#### 1. API Key (Bearer Token)

```http
POST /mcp HTTP/1.1
Authorization: Bearer sk-xxxxxxxxxxxx
Content-Type: application/json

{"jsonrpc": "2.0", "method": "tools/list"}
```

**Implementation**:
```python
# Server validates token
def validate_token(token: str) -> bool:
    return token in valid_api_keys
```

#### 2. OAuth 2.0

**Flow**:
```
1. Client requests authorization
2. User grants permission
3. Client receives access token
4. Token included in MCP requests
5. Token refreshed as needed
```

**Use Case**: When MCP server needs to access user's external services (Google Drive, GitHub, etc.)

#### 3. JWT (JSON Web Tokens)

```python
import jwt

# Server validates JWT
def verify_jwt(token: str) -> dict:
    try:
        payload = jwt.decode(
            token,
            secret_key,
            algorithms=["HS256"],
            audience="my-mcp-server"
        )
        return payload
    except jwt.InvalidTokenError:
        raise AuthenticationError("Invalid token")
```

**Token includes**:
- User identity
- Permissions/scopes
- Expiration time
- Issuer information

#### 4. mTLS (Mutual TLS)

**Use Case**: Enterprise deployments requiring certificate-based authentication

```
Client Certificate ←→ Server Certificate
       ↓
  Both parties verified
```

### Security Best Practices

#### Input Validation

```python
# Always validate tool parameters
def search_tool(query: str, max_results: int) -> dict:
    # Validate types
    if not isinstance(query, str):
        raise ValueError("query must be a string")

    # Validate ranges
    if max_results < 1 or max_results > 100:
        raise ValueError("max_results must be between 1 and 100")

    # Sanitize input
    query = sanitize(query)

    # Execute
    return perform_search(query, max_results)
```

#### Rate Limiting

```python
# Prevent abuse
rate_limits = {
    "default": 60,  # 60 requests per minute
    "premium": 600  # 600 requests per minute
}

def check_rate_limit(user_id: str, tier: str) -> bool:
    limit = rate_limits.get(tier, 60)
    current = get_request_count(user_id, window="1m")

    if current >= limit:
        raise RateLimitError(f"Rate limit exceeded: {limit}/min")

    increment_request_count(user_id)
```

#### Resource Access Control

```python
# Per-resource permissions
def read_resource(uri: str, user_id: str) -> dict:
    # Check permissions
    if not has_access(user_id, uri):
        raise PermissionError(f"No access to {uri}")

    # Read resource
    return load_resource(uri)
```

#### Sandboxing

```yaml
# Docker container for isolation
services:
  mcp-server:
    image: mcp-server:latest
    security_opt:
      - no-new-privileges:true
    read_only: true
    cap_drop:
      - ALL
    resources:
      limits:
        cpus: '0.5'
        memory: 512M
```

#### Audit Logging

```python
# Log all operations
def log_tool_call(user_id: str, tool_name: str, args: dict):
    logger.info({
        "event": "tool_call",
        "user_id": user_id,
        "tool": tool_name,
        "arguments": sanitize_for_logs(args),
        "timestamp": datetime.utcnow().isoformat()
    })
```

### Security Checklist

```
✓ Validate all inputs against schemas
✓ Implement authentication for all endpoints
✓ Use TLS for HTTP/WebSocket transports
✓ Apply rate limiting per user/key
✓ Sandbox server processes
✓ Log all operations for audit
✓ Never expose internal secrets
✓ Implement per-resource ACLs
✓ Set operation timeouts
✓ Handle errors gracefully (no info leakage)
```

---

## Building Your First Server

Let's build a simple MCP server with tools and resources.

### Minimal Server (Python)

```python
#!/usr/bin/env python3
import json
import sys
from typing import Any

class MCPServer:
    def __init__(self, name: str):
        self.name = name
        self.tools = {}
        self.resources = {}

    def add_tool(self, name: str, func, schema: dict):
        self.tools[name] = {"func": func, "schema": schema}

    def add_resource(self, uri: str, content: str, mime_type: str):
        self.resources[uri] = {"content": content, "mime_type": mime_type}

    def handle_request(self, request: dict) -> dict:
        method = request.get("method")
        params = request.get("params", {})
        req_id = request.get("id")

        if method == "initialize":
            return {
                "jsonrpc": "2.0",
                "id": req_id,
                "result": {
                    "protocolVersion": "2024-11-05",
                    "serverInfo": {"name": self.name, "version": "1.0.0"},
                    "capabilities": {
                        "tools": {},
                        "resources": {}
                    }
                }
            }

        elif method == "tools/list":
            tools = [
                {
                    "name": name,
                    "description": tool["schema"].get("description", ""),
                    "inputSchema": tool["schema"]
                }
                for name, tool in self.tools.items()
            ]
            return {"jsonrpc": "2.0", "id": req_id, "result": {"tools": tools}}

        elif method == "tools/call":
            tool_name = params["name"]
            arguments = params.get("arguments", {})

            if tool_name not in self.tools:
                return {
                    "jsonrpc": "2.0",
                    "id": req_id,
                    "error": {"code": -32601, "message": "Tool not found"}
                }

            tool = self.tools[tool_name]
            result = tool["func"](**arguments)

            return {
                "jsonrpc": "2.0",
                "id": req_id,
                "result": {
                    "content": [{"type": "text", "text": str(result)}]
                }
            }

        elif method == "resources/list":
            resources = [
                {
                    "uri": uri,
                    "name": uri.split("/")[-1],
                    "mimeType": res["mime_type"]
                }
                for uri, res in self.resources.items()
            ]
            return {"jsonrpc": "2.0", "id": req_id, "result": {"resources": resources}}

        elif method == "resources/read":
            uri = params["uri"]
            if uri not in self.resources:
                return {
                    "jsonrpc": "2.0",
                    "id": req_id,
                    "error": {"code": -32601, "message": "Resource not found"}
                }

            resource = self.resources[uri]
            return {
                "jsonrpc": "2.0",
                "id": req_id,
                "result": {
                    "contents": [{
                        "uri": uri,
                        "mimeType": resource["mime_type"],
                        "text": resource["content"]
                    }]
                }
            }

        else:
            return {
                "jsonrpc": "2.0",
                "id": req_id,
                "error": {"code": -32601, "message": "Method not found"}
            }

    def run_stdio(self):
        """Run server using STDIO transport"""
        for line in sys.stdin:
            try:
                request = json.loads(line)
                response = self.handle_request(request)
                print(json.dumps(response), flush=True)
            except Exception as e:
                error_response = {
                    "jsonrpc": "2.0",
                    "id": None,
                    "error": {"code": -32603, "message": str(e)}
                }
                print(json.dumps(error_response), flush=True)

# Create server
server = MCPServer("demo-server")

# Add a tool
def add_numbers(a: int, b: int) -> int:
    return a + b

server.add_tool(
    "add",
    add_numbers,
    {
        "type": "object",
        "properties": {
            "a": {"type": "number", "description": "First number"},
            "b": {"type": "number", "description": "Second number"}
        },
        "required": ["a", "b"]
    }
)

# Add a resource
server.add_resource(
    "config://settings",
    json.dumps({"theme": "dark", "version": "1.0"}),
    "application/json"
)

# Run server
if __name__ == "__main__":
    server.run_stdio()
```

### Testing Your Server

```bash
# Send initialize request
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","clientInfo":{"name":"test","version":"1.0"}}}' | python server.py

# List tools
echo '{"jsonrpc":"2.0","id":2,"method":"tools/list"}' | python server.py

# Call a tool
echo '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"add","arguments":{"a":5,"b":3}}}' | python server.py

# List resources
echo '{"jsonrpc":"2.0","id":4,"method":"resources/list"}' | python server.py

# Read resource
echo '{"jsonrpc":"2.0","id":5,"method":"resources/read","params":{"uri":"config://settings"}}' | python server.py
```

### Next Steps

1. **Use a Framework**: Production servers should use frameworks like FastMCP (Python) or MCP SDK (TypeScript)
2. **Add Authentication**: Implement token validation
3. **Implement Rate Limiting**: Prevent abuse
4. **Add Error Handling**: Graceful failure handling
5. **Deploy with Docker**: Containerize for production
6. **Monitor & Log**: Track usage and errors
7. **Write Tests**: Comprehensive test coverage

---

## Additional Resources

- **Official Spec**: [Model Context Protocol Specification](https://spec.modelcontextprotocol.io/)
- **FastMCP**: Python framework for building servers
- **MCP TypeScript SDK**: Official TypeScript implementation
- **Claude Desktop**: Reference client implementation

---

## Summary

MCP provides a standardized way for AI applications to integrate with external tools and data:

- **Protocol**: JSON-RPC 2.0 based communication
- **Transports**: STDIO, HTTP, SSE, WebSocket
- **Primitives**: Tools (functions), Resources (data), Prompts (templates)
- **Security**: Authentication, rate limiting, sandboxing
- **Scalability**: Multiple servers, composable architecture

By adopting MCP, you gain a universal integration layer that works across AI platforms and eliminates the need for custom integrations.
