# FastMCP - Building MCP Servers & Clients

## Table of Contents
1. [Introduction](#introduction)
2. [Installation](#installation)
3. [Quick Start](#quick-start)
4. [Building Servers](#building-servers)
5. [Building Clients](#building-clients)
6. [Tools](#tools)
7. [Resources](#resources)
8. [Prompts](#prompts)
9. [Authentication](#authentication)
10. [Deployment](#deployment)
11. [Advanced Patterns](#advanced-patterns)

---

## Introduction

**FastMCP** is a Python framework that simplifies building Model Context Protocol (MCP) servers and clients. It handles the protocol complexity so you can focus on implementing your tools and resources.

### Why FastMCP?

```
✓ Pythonic API          - Decorators and type hints
✓ Multiple Transports   - STDIO, HTTP, SSE, WebSocket
✓ Built-in Auth         - API keys, JWT, OAuth 2.0
✓ Production Ready      - Middleware, rate limiting, logging
✓ Client & Server       - Complete implementation
✓ Composable            - Mount multiple servers
```

### Core Concepts

- **Decorators**: `@mcp.tool`, `@mcp.resource`, `@mcp.prompt`
- **Context**: Access request context, logging, progress
- **Composition**: Mount and import servers
- **Transports**: Flexible connection options

---

## Installation

```bash
# Install FastMCP
pip install fastmcp

# For development
pip install fastmcp[dev]

# For specific features
pip install fastmcp[jwt]      # JWT authentication
pip install fastmcp[oauth]    # OAuth support
```

### Requirements

- Python 3.10 or higher
- Type hints support
- Async/await support

---

## Quick Start

### Create a Simple Server

```python
# server.py
from fastmcp import FastMCP

# Initialize server
mcp = FastMCP("My First Server")

# Define a tool
@mcp.tool
def greet(name: str) -> str:
    """Greet a user by name."""
    return f"Hello, {name}!"

# Define a resource
@mcp.resource("config://settings")
def get_settings() -> dict:
    """Get application settings."""
    return {"theme": "dark", "version": "1.0"}

# Run the server
if __name__ == "__main__":
    mcp.run()  # Defaults to STDIO transport
```

### Run the Server

```bash
# STDIO transport (default)
python server.py

# HTTP transport
python server.py --transport http --port 8000

# Using the CLI
fastmcp run server.py
```

### Create a Simple Client

```python
# client.py
import asyncio
from fastmcp import Client

async def main():
    # Connect to server
    client = Client("http://localhost:8000/mcp")

    async with client:
        # List available tools
        tools = await client.list_tools()
        print(f"Available tools: {[t.name for t in tools]}")

        # Call a tool
        result = await client.call_tool("greet", {"name": "Alice"})
        print(result)

        # Read a resource
        content = await client.read_resource("config://settings")
        print(content[0].text)

asyncio.run(main())
```

---

## Building Servers

### Server Initialization

```python
from fastmcp import FastMCP

# Basic server
mcp = FastMCP("MyServer")

# With configuration
mcp = FastMCP(
    name="ConfiguredServer",
    include_tags={"public", "v1"},      # Only expose tagged components
    exclude_tags={"internal", "beta"},   # Hide these components
    on_duplicate_tools="error",          # How to handle duplicates
    on_duplicate_resources="warn",
    include_fastmcp_meta=True            # Include FastMCP metadata
)
```

### Server Configuration Options

| Parameter | Type | Description |
|-----------|------|-------------|
| `name` | str | Server name (required) |
| `include_tags` | set[str] | Only expose components with these tags |
| `exclude_tags` | set[str] | Hide components with these tags |
| `on_duplicate_tools` | str | `"error"`, `"warn"`, or `"replace"` |
| `on_duplicate_resources` | str | Same as tools |
| `on_duplicate_prompts` | str | Same as tools |
| `include_fastmcp_meta` | bool | Include framework metadata |

### Running Servers

#### Method 1: Direct Run

```python
if __name__ == "__main__":
    # STDIO (default)
    mcp.run()

    # HTTP
    mcp.run(transport="http", port=8000)

    # Server-Sent Events
    mcp.run(transport="sse", port=8000)
```

#### Method 2: Async Run

```python
import asyncio

async def main():
    await mcp.run_async(transport="http", port=8000)

asyncio.run(main())
```

#### Method 3: CLI

```bash
# Run directly
fastmcp run server.py

# Specify entrypoint
fastmcp run server.py:mcp

# With factory function
fastmcp run server.py:create_server

# With options
fastmcp run server.py --transport http --port 8000
```

### Factory Functions

For setup code that runs before server starts:

```python
async def create_server() -> FastMCP:
    """Factory function for server creation."""
    mcp = FastMCP("MyServer")

    # Setup code runs before server starts
    @mcp.tool
    def add(x: int, y: int) -> int:
        return x + y

    # Configuration or initialization
    tool = await mcp.get_tool("add")
    # ... configure tool ...

    return mcp
```

```bash
fastmcp run server.py:create_server
```

---

## Building Clients

### Client Initialization

```python
from fastmcp import Client

# From URL
client = Client("http://localhost:8000/mcp")

# From local script
client = Client("./my_server.py")

# From in-memory server (testing)
from fastmcp import FastMCP
server = FastMCP("TestServer")
client = Client(server)

# From multi-server config
config = {
    "mcpServers": {
        "weather": {"url": "https://weather-api.example.com/mcp"},
        "calendar": {"command": "python", "args": ["./calendar.py"]}
    }
}
client = Client(config)
```

### Basic Client Operations

```python
async with client:
    # Ping server
    await client.ping()

    # List operations
    tools = await client.list_tools()
    resources = await client.list_resources()
    prompts = await client.list_prompts()
    templates = await client.list_resource_templates()

    # Call a tool
    result = await client.call_tool("tool_name", {"param": "value"})

    # Read a resource
    contents = await client.read_resource("resource://example")

    # Get a prompt
    prompt = await client.get_prompt("prompt_name", {"arg": "value"})
```

### Multi-Server Client

```python
config = {
    "mcpServers": {
        "weather": {"url": "https://weather.example.com/mcp"},
        "assistant": {"command": "python", "args": ["./assistant.py"]}
    }
}

client = Client(config)

async with client:
    # Tools are namespaced by server
    weather = await client.call_tool("weather_get_forecast", {"city": "NYC"})
    answer = await client.call_tool("assistant_ask", {"question": "What?"})

    # Resources use prefixed URIs
    icons = await client.read_resource("weather://weather/icons/sunny")
```

### Raw MCP Protocol Access

For complete control, use `*_mcp` methods:

```python
async with client:
    # Returns raw MCP protocol objects
    tools_result = await client.list_tools_mcp()
    # tools_result -> mcp.types.ListToolsResult

    call_result = await client.call_tool_mcp("my_tool", {"param": "value"})
    # call_result -> mcp.types.CallToolResult

    # Check for errors
    if call_result.isError:
        print(f"Tool failed: {call_result.content}")
```

---

## Tools

Tools are functions that AI can call. They're the primary way to give AI capabilities.

### Basic Tool Definition

```python
@mcp.tool
def add(a: int, b: int) -> int:
    """Add two numbers together."""
    return a + b
```

FastMCP automatically generates JSON schema from type hints.

### Advanced Tool Examples

#### With Optional Parameters

```python
@mcp.tool
def search(
    query: str,
    max_results: int = 10,
    filter: str | None = None
) -> list[dict]:
    """Search with optional parameters."""
    results = perform_search(query, max_results)

    if filter:
        results = [r for r in results if filter in r["category"]]

    return results
```

#### With Complex Types

```python
from typing import Literal

@mcp.tool
def process_data(
    data: list[dict],
    operation: Literal["filter", "sort", "group"],
    field: str
) -> list[dict]:
    """Process data with specified operation."""
    if operation == "filter":
        return [d for d in data if field in d]
    elif operation == "sort":
        return sorted(data, key=lambda x: x.get(field, ""))
    elif operation == "group":
        # ... grouping logic
        pass
```

#### With Context

```python
from fastmcp import Context

@mcp.tool
async def long_operation(
    data: list[str],
    ctx: Context  # Context parameter (name doesn't matter)
) -> dict:
    """Process data with progress reporting."""
    total = len(data)

    for i, item in enumerate(data):
        # Report progress
        await ctx.report_progress(i, total)

        # Log information
        ctx.info(f"Processing item {i+1}/{total}")

        # Process item
        process_item(item)

    return {"processed": total}
```

### Tool Configuration

```python
@mcp.tool(
    name="custom_name",           # Override function name
    description="Custom desc",    # Override docstring
    tags={"v1", "public"},        # Add tags for filtering
    enabled=True                  # Enable/disable tool
)
def my_tool(param: str) -> str:
    return param
```

### Error Handling in Tools

```python
@mcp.tool
def divide(a: float, b: float) -> float:
    """Divide two numbers."""
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b

# FastMCP handles exceptions and returns proper error responses
```

### Accessing Server Resources from Tools

```python
@mcp.tool
async def get_user_data(user_id: str, ctx: Context) -> dict:
    """Get user data from resource."""
    # Read resource from within tool
    content = await ctx.read_resource(f"user://{user_id}")

    return {"user_id": user_id, "data": content[0].text}
```

---

## Resources

Resources provide data that AI can access. They're like files or API endpoints.

### Static Resources

```python
@mcp.resource("config://settings")
def get_settings() -> dict:
    """Application settings."""
    return {"theme": "dark", "version": "1.0"}

@mcp.resource("file:///data/users.json")
def get_users() -> list[dict]:
    """User database."""
    with open("/data/users.json") as f:
        return json.load(f)
```

### Dynamic Resource Templates

Use `{parameter}` syntax for dynamic resources:

```python
@mcp.resource("user://{user_id}")
def get_user(user_id: str) -> dict:
    """Get specific user data."""
    return database.get_user(user_id)

@mcp.resource("weather://{city}/forecast")
def get_weather(city: str) -> dict:
    """Get weather forecast for a city."""
    return fetch_weather(city)
```

### Resource Configuration

```python
@mcp.resource(
    uri="data://example",
    name="Example Resource",
    description="Detailed description",
    mime_type="application/json",     # Content type
    tags={"public"},                  # Tags for filtering
    enabled=True                      # Enable/disable
)
def my_resource() -> dict:
    return {"data": "example"}
```

### Resource Types

FastMCP supports different content types:

```python
# JSON data
@mcp.resource("data://json")
def json_data() -> dict:
    return {"key": "value"}

# Text content
@mcp.resource("text://content")
def text_data() -> str:
    return "Plain text content"

# Binary data
@mcp.resource("file://image.png")
def image_data() -> bytes:
    with open("image.png", "rb") as f:
        return f.read()
```

### Subscribable Resources

For resources that can change:

```python
# Resources support subscriptions automatically
@mcp.resource("live://stock_price")
def get_stock_price() -> dict:
    return {"symbol": "AAPL", "price": get_current_price()}

# Clients can subscribe to receive updates
# Server should notify clients when resource changes
```

---

## Prompts

Prompts are reusable templates with parameters.

### Basic Prompt

```python
@mcp.prompt
def code_review() -> str:
    """System prompt for code review."""
    return "You are an expert code reviewer. Review code for quality and best practices."
```

### Parameterized Prompts

```python
@mcp.prompt
def analyze_sentiment(text: str) -> dict:
    """Analyze sentiment of text."""
    return {
        "messages": [
            {
                "role": "system",
                "content": "You are a sentiment analysis expert. Respond with: positive, negative, or neutral."
            },
            {
                "role": "user",
                "content": f"Analyze the sentiment of: {text}"
            }
        ]
    }
```

### Multi-Turn Prompts

```python
@mcp.prompt
def debug_session(
    error_message: str,
    code_snippet: str,
    language: str
) -> dict:
    """Debug session prompt."""
    return {
        "messages": [
            {
                "role": "system",
                "content": f"You are a {language} debugging expert."
            },
            {
                "role": "user",
                "content": f"I'm getting this error:\n{error_message}"
            },
            {
                "role": "user",
                "content": f"Here's the code:\n```{language}\n{code_snippet}\n```"
            },
            {
                "role": "user",
                "content": "What's wrong and how do I fix it?"
            }
        ]
    }
```

### Prompt Configuration

```python
@mcp.prompt(
    name="custom_prompt",
    description="Custom description",
    tags={"v1"},
    enabled=True
)
def my_prompt(param: str) -> dict:
    return {"messages": [...]}
```

---

## Authentication

FastMCP supports multiple authentication methods.

### API Key (Bearer Token)

```python
from fastmcp.server.auth import TokenVerifier

# Simple token validation
mcp = FastMCP(
    name="SecureServer",
    auth=TokenVerifier(valid_tokens=["secret-key-1", "secret-key-2"])
)

@mcp.tool
def protected_tool() -> str:
    """This tool requires authentication."""
    return "Secret data"
```

**Client usage**:
```bash
curl -H "Authorization: Bearer secret-key-1" \
     http://localhost:8000/mcp
```

### JWT Authentication

```python
from fastmcp.server.auth import JWTVerifier

mcp = FastMCP(
    name="JWTServer",
    auth=JWTVerifier(
        secret="your-jwt-secret",
        algorithm="HS256",
        audience="your-api",
        issuer="your-auth-service"
    )
)

@mcp.tool
def jwt_protected() -> dict:
    """Requires valid JWT."""
    return {"data": "protected"}
```

**Generate JWT** (example):
```python
import jwt
import datetime

token = jwt.encode({
    "sub": "user123",
    "aud": "your-api",
    "iss": "your-auth-service",
    "exp": datetime.datetime.utcnow() + datetime.timedelta(hours=1)
}, "your-jwt-secret", algorithm="HS256")
```

### Custom Token Verification

```python
from fastmcp.server.auth import TokenVerifier

class CustomTokenVerifier(TokenVerifier):
    async def verify_token(self, token: str) -> dict:
        """Custom token verification logic."""
        # Check against database
        user = await database.verify_token(token)

        if not user:
            raise ValueError("Invalid token")

        return {"user_id": user.id, "scopes": user.scopes}

mcp = FastMCP(
    name="CustomAuthServer",
    auth=CustomTokenVerifier()
)
```

### OAuth 2.0

```python
from fastmcp.server.auth.providers import OAuthProvider

class MyOAuthProvider(OAuthProvider):
    async def get_user(self, user_id: str):
        """Retrieve user information."""
        return await database.get_user(user_id)

    async def get_client(self, client_id: str):
        """Retrieve client information."""
        return await database.get_client(client_id)

    async def register_client(self, client_info):
        """Register new OAuth client."""
        await database.save_client(client_info)

mcp = FastMCP(
    name="OAuthServer",
    auth=MyOAuthProvider(
        user_store=user_database,
        client_store=client_database
    )
)
```

### Access User Context in Tools

```python
from fastmcp import Context

@mcp.tool
async def get_my_data(ctx: Context) -> dict:
    """Get authenticated user's data."""
    # Access user info from auth context
    user_id = ctx.request_context.meta.get("user_id")

    return await database.get_user_data(user_id)
```

---

## Deployment

### Configuration File

Create `fastmcp.json`:

```json
{
  "$schema": "https://gofastmcp.com/public/schemas/fastmcp.json/v1.json",
  "source": {
    "type": "filesystem",
    "path": "server.py",
    "entrypoint": "mcp"
  },
  "environment": {
    "type": "uv",
    "python": ">=3.10",
    "dependencies": ["pandas", "numpy", "requests"]
  },
  "deployment": {
    "transport": "http",
    "port": 8000,
    "log_level": "INFO"
  }
}
```

### Docker Deployment

#### Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy server code
COPY server.py .

# Run server
CMD ["python", "server.py"]
```

#### docker-compose.yml

```yaml
version: '3.8'

services:
  mcp-server:
    build: .
    ports:
      - "8000:8000"
    environment:
      - LOG_LEVEL=INFO
      - API_KEY=${API_KEY}
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
```

### Production Server

```python
from fastmcp import FastMCP
from fastmcp.server.middleware import Middleware, MiddlewareContext
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Rate limiting middleware
class RateLimitMiddleware(Middleware):
    def __init__(self, requests_per_minute: int = 60):
        self.limit = requests_per_minute
        self.requests = {}

    async def on_request(self, context: MiddlewareContext, call_next):
        client_id = context.request_context.meta.get("user_id", "default")

        # Check rate limit
        # ... (implementation)

        return await call_next(context)

# Initialize server
mcp = FastMCP(
    name="ProductionServer",
    auth=TokenVerifier(valid_tokens=[os.getenv("API_KEY")])
)

# Add middleware
mcp.add_middleware(RateLimitMiddleware(requests_per_minute=100))

# Define tools
@mcp.tool
def production_tool() -> str:
    logger.info("Tool called")
    return "Production data"

# Run with HTTP
if __name__ == "__main__":
    mcp.run(
        transport="http",
        port=int(os.getenv("PORT", 8000)),
        host="0.0.0.0"
    )
```

### Health Check Endpoint

```python
from starlette.responses import JSONResponse

@mcp.custom_route("/health", methods=["GET"])
async def health_check(request):
    """Health check for load balancers."""
    return JSONResponse({"status": "healthy"})

http_app = mcp.http_app()
```

### Environment Variables

```bash
# .env file
API_KEY=your-secret-key
DATABASE_URL=postgresql://user:pass@host:5432/db
LOG_LEVEL=INFO
PORT=8000
RATE_LIMIT=100
```

Load in server:
```python
from dotenv import load_dotenv
load_dotenv()

api_key = os.getenv("API_KEY")
```

---

## Advanced Patterns

### Server Composition

Mount multiple servers together:

```python
from fastmcp import FastMCP

# Create component servers
weather_server = FastMCP("WeatherService")

@weather_server.tool
def get_forecast(city: str) -> dict:
    return {"city": city, "temp": 72}

calendar_server = FastMCP("CalendarService")

@calendar_server.tool
def add_event(title: str, date: str) -> dict:
    return {"event": title, "date": date}

# Create main server
main_server = FastMCP("MainApp")

# Mount subservers with prefixes
main_server.mount(weather_server, prefix="weather")
main_server.mount(calendar_server, prefix="calendar")

# Now available:
# - weather_get_forecast
# - calendar_add_event
```

### Static Import

For static composition:

```python
import asyncio

async def setup():
    # Import subserver (static snapshot)
    await main_server.import_server(weather_server, prefix="weather")

asyncio.run(setup())
main_server.run()
```

### Middleware

Custom middleware for cross-cutting concerns:

```python
from fastmcp.server.middleware import Middleware, MiddlewareContext
import time

class TimingMiddleware(Middleware):
    async def on_request(self, context: MiddlewareContext, call_next):
        start = time.time()

        try:
            result = await call_next(context)
            duration = time.time() - start
            print(f"Request took {duration:.2f}s")
            return result
        except Exception as e:
            duration = time.time() - start
            print(f"Request failed after {duration:.2f}s: {e}")
            raise

# Add middleware
mcp.add_middleware(TimingMiddleware())
```

### Context State Management

Share data across request lifecycle:

```python
@mcp.tool
async def first_tool(ctx: Context) -> str:
    # Store data in context state
    ctx.state["user_data"] = {"id": "123", "name": "Alice"}
    return "Data stored"

@mcp.tool
async def second_tool(ctx: Context) -> dict:
    # Access data from context state
    user_data = ctx.state.get("user_data", {})
    return {"message": f"Hello {user_data.get('name')}"}
```

### Tag-Based Filtering

Control what gets exposed:

```python
# Server with tag filtering
prod_server = FastMCP(
    name="ProdServer",
    include_tags={"production"},      # Only show production tools
    exclude_tags={"internal", "beta"} # Hide these
)

@prod_server.tool(tags={"production"})
def prod_tool() -> str:
    return "Production ready"

@prod_server.tool(tags={"beta"})
def beta_tool() -> str:
    return "Beta feature"  # Won't be exposed

# Recursive filtering with mounted servers
api_server = FastMCP(name="API")

@api_server.tool(tags={"production"})
def api_prod_tool() -> str:
    return "API production"

@api_server.tool(tags={"development"})
def api_dev_tool() -> str:
    return "API development"

# Mount with parent filtering
prod_server.mount(api_server, prefix="api")
# Only api_prod_tool will be available
```

### Proxy Pattern

Create unified proxy for multiple servers:

```python
config = {
    "mcpServers": {
        "weather": {"url": "https://weather-api.example.com/mcp"},
        "calendar": {"url": "https://calendar-api.example.com/mcp"}
    }
}

# Create unified proxy
proxy = FastMCP.as_proxy(config, name="Unified Proxy")

# All tools/resources accessible with prefixes
# - weather_get_forecast
# - calendar_add_event
```

### FastAPI Integration

Convert FastAPI app to MCP server:

```python
from fastapi import FastAPI
from fastmcp import FastMCP

app = FastAPI()

@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"id": user_id, "name": "Alice"}

@app.post("/users")
def create_user(name: str, email: str):
    return {"id": 123, "name": name, "email": email}

# Convert FastAPI to MCP
mcp = FastMCP.from_fastapi(app)

# Now available as MCP tools:
# - get_user (with user_id parameter)
# - create_user (with name, email parameters)
```

### Progress Reporting

Report progress for long operations:

```python
from fastmcp import Context

@mcp.tool
async def process_files(files: list[str], ctx: Context) -> dict:
    """Process multiple files with progress."""
    total = len(files)

    for i, file in enumerate(files):
        # Report progress (current, total)
        await ctx.report_progress(i, total)

        # Process file
        process_file(file)

        # Log progress
        ctx.info(f"Processed {i+1}/{total} files")

    return {"processed": total, "status": "complete"}
```

### MCP Mixin Pattern

Use mixins for reusable components:

```python
from fastmcp.contrib.mcp_mixin import MCPMixin, mcp_tool, mcp_resource

class WeatherComponent(MCPMixin):
    def __init__(self, api_key: str):
        self.api_key = api_key

    @mcp_tool(name="get_weather", description="Get weather")
    def get_weather_data(self, city: str) -> dict:
        return fetch_weather(city, self.api_key)

    @mcp_resource(uri="weather://config")
    def weather_config(self) -> dict:
        return {"api_key": self.api_key[:4] + "..."}

# Register with server
mcp = FastMCP("MyServer")
weather = WeatherComponent(api_key="secret")
weather.register_all(mcp, prefix="weather")
```

---

## Best Practices

### 1. Type Hints

Always use type hints - FastMCP generates schemas from them:

```python
@mcp.tool
def good_tool(name: str, count: int = 5) -> list[str]:
    """Good: Full type hints."""
    return [name] * count

@mcp.tool
def bad_tool(name, count=5):  # Bad: No type hints
    """FastMCP can't generate proper schema."""
    return [name] * count
```

### 2. Documentation

Use docstrings - they become tool descriptions:

```python
@mcp.tool
def search(query: str, limit: int = 10) -> list[dict]:
    """
    Search for items matching the query.

    Args:
        query: Search query string
        limit: Maximum number of results to return

    Returns:
        List of matching items with metadata
    """
    return perform_search(query, limit)
```

### 3. Error Handling

Handle errors gracefully:

```python
@mcp.tool
def safe_divide(a: float, b: float) -> float:
    """Divide two numbers safely."""
    if b == 0:
        raise ValueError("Cannot divide by zero")

    if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
        raise TypeError("Arguments must be numbers")

    return a / b
```

### 4. Resource Naming

Use consistent URI schemes:

```python
# Good: Consistent scheme
@mcp.resource("user://123")
@mcp.resource("user://456")

# Good: Hierarchical
@mcp.resource("data://users/active")
@mcp.resource("data://users/inactive")

# Bad: Inconsistent
@mcp.resource("user123")
@mcp.resource("data/user456")
```

### 5. Testing

Write tests for your tools:

```python
import pytest
from fastmcp import FastMCP, Client

@pytest.mark.asyncio
async def test_tool():
    mcp = FastMCP("TestServer")

    @mcp.tool
    def add(a: int, b: int) -> int:
        return a + b

    client = Client(mcp)

    async with client:
        result = await client.call_tool("add", {"a": 2, "b": 3})
        assert result == 5
```

---

## Summary

FastMCP simplifies MCP server and client development:

**Key Features**:
- Pythonic decorators (`@mcp.tool`, `@mcp.resource`, `@mcp.prompt`)
- Multiple transports (STDIO, HTTP, SSE, WebSocket)
- Built-in authentication (API keys, JWT, OAuth)
- Middleware support for cross-cutting concerns
- Server composition and mounting
- Type-safe with full type hint support

**Quick Reference**:
```python
from fastmcp import FastMCP, Context, Client

# Server
mcp = FastMCP("MyServer")

@mcp.tool
def my_tool(param: str) -> str:
    return f"Result: {param}"

@mcp.resource("data://example")
def my_resource() -> dict:
    return {"data": "value"}

mcp.run(transport="http", port=8000)

# Client
client = Client("http://localhost:8000/mcp")
async with client:
    result = await client.call_tool("my_tool", {"param": "test"})
```

**Next Steps**:
1. Build your first server with tools and resources
2. Add authentication for production
3. Implement middleware for logging/monitoring
4. Deploy with Docker
5. Build a client to interact with your server

For more details, see the [MCP Guide](./MCP_GUIDE.md) for protocol specifics.
