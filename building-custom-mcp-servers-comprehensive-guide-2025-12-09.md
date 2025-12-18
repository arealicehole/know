# Building Custom MCP Servers: Comprehensive Guide 2025

**Research Date:** 2025-12-09
**Protocol Version:** MCP 2025-06-18 (Latest Specification)
**Performance:** Wide-then-deep parallel research architecture

---

## Executive Summary

The Model Context Protocol (MCP) is an open-source standard introduced by Anthropic in November 2024 for connecting AI applications to external systems. In December 2025, MCP joined the Agentic AI Foundation under the Linux Foundation, with co-founding support from Anthropic, Block, OpenAI, Google, Microsoft, AWS, Cloudflare, and Bloomberg. This guide provides production-ready best practices for building custom MCP servers based on the latest 2025-06-18 specification.

---

## 1. MCP Server Architecture

### 1.1 Protocol Fundamentals

MCP uses a **client-server architecture** where AI applications (like Claude for Desktop or IDEs) use an MCP client to connect to MCP servers which front data sources or tools.

**Core Architecture Concepts:**
- **JSON-RPC 2.0**: All messages are encoded as UTF-8 JSON-RPC
- **Primitives**: The spec defines building blocks called primitives
- **Server Primitives**: Prompts, Resources, and Tools
- **Client Primitives**: Roots and Sampling

**Message Flow:**
MCP deliberately reuses message-flow ideas from the Language Server Protocol (LSP), making it familiar to developers who have worked with LSP implementations.

### 1.2 Server vs Client Roles

**MCP Server Responsibilities:**
- Expose resources (read-only data sources)
- Provide tools (dynamic operations that can modify state)
- Offer prompts (reusable templates for consistent interactions)
- Handle authentication and authorization
- Manage session state and resource cleanup

**MCP Client Responsibilities:**
- Connect to MCP servers via supported transports
- Negotiate capabilities during initialization
- Send requests and handle responses
- Manage server lifecycle (initialization, operation, shutdown)

**OAuth 2.1 Classification (2025-06-18 Spec):**
MCP servers are now officially classified as **OAuth Resource Servers**, which has significant implications for security and discovery by including mechanisms for protected resource metadata.

### 1.3 Transport Mechanisms

The MCP 2025-06-18 specification defines two standard transport mechanisms:

#### **1. stdio Transport**

**Use Case:** Local development, plugin architecture, low-latency requirements

**How It Works:**
- Client launches MCP server as a subprocess
- Server reads JSON-RPC messages from stdin
- Server writes responses to stdout
- Messages delimited by newlines (must NOT contain embedded newlines)
- Supports batching (multiple requests/notifications in one message)

**Advantages:**
- Extremely low latency (microseconds, no network overhead)
- Simple setup for local plugins
- Inherits OS-level security
- Maximum client compatibility

**Disadvantages:**
- Local-only (no remote connections)
- Single client per server instance
- Limited scalability

#### **2. Streamable HTTP Transport (Recommended for Production)**

**Use Case:** Remote servers, multiple clients, production deployments, horizontal scalability

**How It Works:**
- Server operates as independent process handling multiple client connections
- Uses HTTP POST and GET requests
- Optionally supports Server-Sent Events (SSE) for streaming
- Single HTTP endpoint path (e.g., `https://example.com/mcp`)

**Protocol Requirements:**
- Client MUST include `MCP-Protocol-Version` header on all requests (e.g., `MCP-Protocol-Version: 2025-06-18`)
- Servers MUST validate the `Origin` header to prevent DNS rebinding attacks
- When running locally, servers SHOULD bind only to localhost (127.0.0.1)

**Advantages:**
- Supports multiple concurrent clients
- Horizontally scalable
- Network-accessible (remote hosting)
- Incremental results via streaming
- Production-ready with proper load balancing

**Disadvantages:**
- Higher latency than stdio
- More complex setup
- Requires network security considerations

#### **3. WebSocket Transport (Proposed)**

**Status:** Standards Enhancement Proposal (SEP-1288) - Not yet part of official spec

**Proposed Features:**
- Long-lived bidirectional communication
- Session persistence across network interruptions
- Real-time messaging
- Improved session resilience

**Note:** WebSockets are not currently part of the standard transport set. Monitor the SEP-1288 proposal for updates.

#### **Deprecated Transport:**
- **SSE Transport**: DEPRECATED as of 2025-03-26, replaced by Streamable HTTP

---

## 2. Implementation Options

### 2.1 Official SDKs

#### **TypeScript SDK**

**Repository:** `@modelcontextprotocol/typescript-sdk`
**GitHub:** https://github.com/modelcontextprotocol/typescript-sdk
**Community Activity:** 11,000+ GitHub stars

**Installation:**
```bash
npm install @modelcontextprotocol/sdk zod
```

**Key Features:**
- Full MCP specification implementation
- Zod schema validation (peer dependency on zod v3.25+)
- Supports stdio and Streamable HTTP transports
- TypeScript type safety
- HTTP + SSE backwards compatibility

**When to Use:**
- Maximum control over implementation
- Specific architectural requirements
- TypeScript/Node.js environment
- Need for custom transport implementations

#### **Python SDK**

**Repository:** `mcp`
**PyPI:** https://pypi.org/project/mcp/
**Community Activity:** 20,485+ GitHub stars
**Latest Release:** December 8, 2025

**Installation:**
```bash
pip install mcp
```

**Key Features:**
- FastMCP framework for rapid development
- FastAPI integration
- Pydantic validation
- Supports stdio and HTTP+SSE
- Streamable HTTP verification needed for production

**When to Use:**
- Python environment
- Rapid prototyping
- FastAPI/FastMCP integration
- Pydantic schema validation

#### **Other Official SDKs (2025):**
- **Go SDK** (maintained with Google)
- **Kotlin SDK** (maintained with JetBrains)
- **Ruby SDK** (maintained with Shopify)
- **Java SDK** (available)

### 2.2 Framework Choices and Trade-offs

#### **Official SDK (TypeScript/Python)**

**Pros:**
- Maximum flexibility and control
- Direct access to all protocol features
- No abstraction overhead
- Best for complex custom requirements

**Cons:**
- More boilerplate code
- Lower-level implementation details
- Longer development time

#### **FastMCP Framework (TypeScript/Python)**

**Repository:** https://github.com/punkpeye/fastmcp
**Version:** FastMCP 2.0 (production-ready)

**Pros:**
- Built on top of official SDK
- Handles client sessions automatically
- Rapid development without low-level details
- Production features: server composition, proxying, OpenAPI/FastAPI generation
- Enterprise auth, deployment tools, testing utilities
- Comprehensive client libraries
- Supports every deployment scenario from local to global scale

**Cons:**
- Additional abstraction layer
- Less control than official SDK
- Framework lock-in

**When to Choose FastMCP:**
- Need to build quickly
- Want built-in best practices
- Require production features out-of-the-box
- Don't need low-level protocol customization

**When to Use Official SDK:**
- Need maximum control
- Have specific architectural requirements
- Building custom transport mechanisms
- Optimizing for unique performance constraints

### 2.3 Recommended Tech Stack by Use Case

#### **Local Development / Plugin Architecture**
- **Transport:** stdio
- **SDK:** Official TypeScript or Python SDK
- **Validation:** Zod (TypeScript) or Pydantic (Python)
- **Testing:** MCP Inspector

#### **Remote Production Server (Single Service)**
- **Transport:** Streamable HTTP
- **SDK:** FastMCP 2.0 or Official SDK
- **Framework:** FastAPI (Python) or Express (TypeScript)
- **Auth:** OAuth 2.1 with enterprise IdP (Okta, Azure AD, Auth0)
- **Deployment:** Docker container
- **Monitoring:** Prometheus + Grafana
- **Testing:** Jest/Vitest (TypeScript) or pytest (Python)

#### **Multi-Server Gateway / Proxy**
- **Tool:** mcgravity (load balancing across multiple MCP servers)
- **Alternative:** MCPX by TheLunarCompany (production gateway)
- **Features:** Centralized tool discovery, access controls, call prioritization, usage tracking

#### **Enterprise Scale / Multi-Tenant**
- **Transport:** Streamable HTTP with stateless mode
- **Parameters:** `stateless_http=True`, `json_response=True`
- **Architecture:** Microservices pattern
- **Gateway:** API Gateway + WAF
- **Auth:** OAuth 2.1 + RBAC
- **Infrastructure:** Kubernetes with service mesh (mTLS)
- **Secrets:** Azure Key Vault, AWS Secrets Manager, or HashiCorp Vault
- **Monitoring:** Full observability stack (logs, metrics, traces)

---

## 3. Best Practices

### 3.1 Security Considerations

#### **Authentication (OAuth 2.1 - Mandatory for HTTP Transport)**

**Requirements:**
- OAuth 2.1 is MANDATORY for HTTP-based transports as of March 2025 spec
- MCP servers are classified as OAuth Resource Servers
- Use enterprise identity providers: Okta, Azure AD, Auth0
- Avoid static tokens in production (difficult to rotate and audit)
- Enforce short-lived tokens
- Use PKCE (Proof Key for Code Exchange) for resilience against interception

**HTTP Status Codes:**
- `401 Unauthorized`: Missing or invalid tokens (include `WWW-Authenticate` header)
- `403 Forbidden`: Insufficient permissions

#### **Session Security**

**Critical Rules:**
- MCP Servers MUST NOT use sessions for authentication
- MCP servers MUST use secure, non-deterministic session IDs
- Generated session IDs (e.g., UUIDs) SHOULD use secure random number generators
- Never use session IDs for auth purposes

#### **Input Validation & Injection Prevention**

**OWASP Basics (Still Critical in 2025):**
- Use parameterized queries (NEVER string concatenation for SQL)
- Sanitize all inputs before execution
- Treat all stored content as untrusted
- Validate data types and escape special characters
- Use safe wrappers or delimiters before including in prompts/tool calls

**JSON-RPC Validation:**
- Validate all incoming JSON-RPC requests against schemas
- Reject malformed inputs or unrecognized parameters
- Prevents prompt injection attacks
- Use Zod (TypeScript) or Pydantic (Python) for schema validation

**Testing Against LLM Chaos:**
LLMs can generate unexpected or malformed inputs, explore edge cases never envisioned, or chain calls in unpredictable ways. Harden servers against creative inputs through comprehensive testing.

#### **Network & Infrastructure Security**

**Zero Trust Principles:**
- Continuously authenticate and authorize every interaction
- Assume no implicit trust even within internal networks
- No trust, always verify

**Network Segmentation:**
- Segregate MCP servers by VPC subnets or VLANs
- Rigorous filtering between segments
- Deploy service meshes for identity-related traffic control

**Encryption & Gateways:**
- Implement mTLS encryption
- Apply WAFs (Web Application Firewalls)
- Use API gateways for deep packet inspection (DPI)
- Limit suspicious activities and risky payloads

**Isolation:**
- Run MCP servers in isolated environments (VPCs, namespaces, containers)
- Avoid exposing servers directly to public internet unless required
- For compliance needs, deploy in air-gapped or hybrid environments

**Local Server Security:**
- When running locally, bind ONLY to localhost (127.0.0.1)
- Validate `Origin` header on all incoming connections (prevent DNS rebinding)

#### **Role-Based Access Control (RBAC)**

**Implementation:**
- Map users to specific permissions
- Tool-level access control
- OAuth 2.0 integration with identity providers (e.g., Keycloak)
- Enforce principle of least privilege

#### **Secrets Management**

**ANTI-PATTERN: Environment Variables in Production**
- Difficult to rotate
- Often leak into logs or build artifacts
- Static target for attackers

**BEST PRACTICE: Dedicated Secrets Management**
- Azure Key Vault
- AWS Secrets Manager
- HashiCorp Vault
- Rotate credentials regularly
- Audit access to secrets

#### **Recent Vulnerabilities to Avoid**

**CVE-2025-49596 (MCP Inspector):**
- Server listening on 0.0.0.0 (all interfaces) instead of 127.0.0.1
- No authentication
- No CSRF protection
- **Lesson:** Always bind local tools to localhost only, implement authentication, add CSRF protection

### 3.2 Error Handling Patterns

#### **Error Reporting Location**

**CRITICAL:** Tool errors should be reported **within the result object**, NOT as MCP protocol-level errors. This allows the LLM to see and potentially handle the error.

#### **Meaningful Error Messages**

Unlike traditional applications where errors are dead ends, an agent can leverage well-designed error messages to self-correct and proceed.

**Best Practices:**
- Provide actionable error messages
- Be precise and specific
- Include guidance on how to fix the issue
- Example: If tool B requires tool A to be called first, error message should explicitly suggest calling tool A

**Testing Error Messages:**
When testing invalid parameters, validate that error messages contain specific, helpful text. Vague error messages cause LLMs to retry with equally wrong parameters.

#### **Error Testing Coverage**

Test suites should validate:
- Tool discovery (does `listTools()` return all expected tools?)
- Parameter validation (missing required params, invalid types)
- Error handling (invalid enum values, empty arrays, unknown documents)
- Graceful degradation for internal server exceptions

#### **Timeout Handling**

Implementations SHOULD implement appropriate timeouts for all requests to prevent hung connections and resource exhaustion.

### 3.3 Resource Management and Cleanup

#### **Lifecycle Phases**

The MCP lifecycle consists of three phases:

1. **Initialization:** Capability negotiation and protocol version agreement (MUST be first interaction)
2. **Operation:** Exchange messages according to negotiated capabilities
3. **Shutdown:** Graceful cleanup of resources

#### **Shutdown Best Practices (stdio transport)**

1. Client closes input stream to child process
2. Wait for server to exit gracefully
3. Send SIGTERM if server doesn't exit within reasonable time
4. Send SIGKILL if needed

#### **Lifespan Hooks (Asynchronous Context Manager)**

For complex applications, use the **lifespan object** to manage application-level resources throughout the server lifecycle.

**Benefits:**
- Proper resource allocation and release
- Configuration loading
- Capability registration
- Connection management
- Error recovery
- Observability hooks

#### **Session Cleanup**

**Critical for SSE/HTTP Connections:**
- Clean up resources when connections close
- Essential to prevent memory leaks
- Implement health checks and monitoring

#### **Production Resource Management Checklist**

- [ ] Proper initialization of resources
- [ ] Configuration loading from secure sources
- [ ] Capability registration during init
- [ ] Connection establishment with timeout handling
- [ ] Graceful connection termination
- [ ] Error recovery mechanisms
- [ ] Version compatibility checks
- [ ] Resource cleanup on connection close
- [ ] Comprehensive logging and observability
- [ ] Health check endpoints

#### **2025-11-25 Spec Updates: Tasks Primitive**

The November 2025 spec introduces an experimental **Tasks primitive** for long-running work:

**Features:**
- "Call-now, fetch-later" pattern
- Tool emits task ID and expiry interval
- Expiry interval = 0: task canceled if client disconnects
- Expiry interval > 0: task continues for specified seconds
- Progress notifications via JSON-RPC
- Partial results streaming

**Benefits:**
- Better SSE polling/disconnect behavior
- Fewer zombie connections
- Less duplicate work after reconnects
- More predictable streaming semantics

### 3.4 Testing Strategies

#### **Testing Progression (Recommended Timeline)**

**Day 1-2: Development**
- Build core functionality
- Manual testing with MCP Inspector

**Day 3: Unit Tests**
- Target 80% code coverage
- Test edge cases
- Framework: pytest (Python) or Jest/Vitest (TypeScript)

**Day 4: Integration Tests**
- JSON-RPC protocol compliance
- Timeout handling
- Multi-tool workflows

**Day 5: Security Tests**
- SQL injection testing
- Command injection testing
- Path traversal testing
- Multi-tenant isolation

#### **Unit Testing Patterns**

**In-Memory Testing Pattern:**
Pass server instance directly to client, eliminating subprocess connection issues and network delays.

**Focus Areas:**
- Test individual tool functions in isolation
- Parameter validation
- Error handling
- Data transformation
- Atomic tests targeting smallest unit of behavior

**Best Practice:** If something is hard to test, its design might be flawed. Tests and design go hand-in-hand.

#### **Integration Testing Patterns**

**Three Critical Aspects:**
1. Protocol compliance
2. Workflow integrity
3. Error resilience

**Progression:**
- Start with individual client-server interactions
- Progress to complex multi-tool workflows
- Test error recovery and retry logic

**Isolated Integration Testing Benefits:**
- Reduced complexity
- Faster feedback
- Reliable results
- Cost effectiveness

#### **Schema Validation Testing**

Proper schema validation prevents subtle bugs and production disasters.

**Testing Strategy:**
- Use MCP Inspector for automated schema checks
- Maintain explicit unit/integration tests for tool schemas
- Regression coverage for schema changes
- Test missing or mismatched parameters

#### **Testing Tools**

**MCP Inspector (Official):**
- Visual testing interface
- Schema validation
- Real-time interaction testing
- Best debugging experience for stdio servers

**MCP Testing Framework (Community):**
- Advanced mocking for servers, tools, resources
- Coverage analysis with configurable thresholds
- Performance benchmarking
- Integration testing for complete workflows

**mcpjam:**
- Alternative testing tool
- Server interaction testing

#### **CI/CD Integration**

Integrate testing into CI/CD pipelines (GitHub Actions, GitLab CI):
- Run tests automatically on code changes
- Catch regressions early
- Enforce coverage thresholds
- Block deployments on test failures

#### **LLM Chaos Testing**

LLMs are "chaos agents" that can:
- Generate unexpected or malformed inputs
- Explore unenvisioned edge cases
- Chain calls in ways that defy simple logic

**Testing Strategy:**
- Test with wide range of valid inputs
- Test graceful error handling for invalid inputs
- Test unexpected call sequences
- Test parameter boundary conditions
- Test concurrent tool executions

---

## 4. Tool Design Patterns

### 4.1 How to Structure Tools Effectively

#### **Tool Definition Structure**

Each tool is defined with:
- **Name:** Unique identifier
- **Description:** Human-readable explanation (for UI/model understanding)
- **Input Schema:** JSON Schema describing expected parameters

#### **Polymorphic Design (RECOMMENDED)**

**Anti-Pattern: API Wrapper One-on-One**
Exposing a new MCP tool for every single API endpoint leads to:
- Tool quantity inflation
- Dramatic drop in task completion rate
- Logarithmically decreasing negative impact on tool selection accuracy
- Model confusion (more tools = higher chance of selecting wrong one)

**Best Practice: Layered Tool Pattern (Block/Square Example)**
Reduce entire API platforms to just a few conceptual tools with more parameters.

**Example:**
Instead of:
- `get_user`
- `create_user`
- `update_user`
- `delete_user`
- `list_users`

Use:
- `manage_resource` (with parameters: `resource_type`, `action`, `resource_id`, etc.)

**Benefits:**
- Fewer tools = better model accuracy
- More flexible and maintainable
- Easier to extend with new resources
- Clearer tool purpose

#### **Single Responsibility Principle**

Each MCP server should have one clear, well-defined purpose. Follows principle of focused services over monolithic architectures.

**Benefits:**
- Easier to understand and maintain
- Better security boundaries
- Simpler testing
- Independent scaling

#### **Don't Make Servers "Too Smart"**

**Anti-Pattern:**
Building MCP servers that attempt to:
- Do heavy analytical lifting
- Understand context deeply
- Make intelligent decisions
- Provide ready-to-use outputs

**Why It's Wrong:**
LLMs are already exceptional at analysis, reasoning, and synthesis. They don't need another layer of analysis.

**Best Practice:**
MCP server's job is to be the **world's best research assistant**, not a competing analyst. Provide rich, structured information for the LLM to work with.

### 4.2 Parameter Schema Design

#### **Validation Approaches**

**TypeScript: Zod**
```typescript
import { z } from 'zod';

const toolSchema = z.object({
  query: z.string().min(1).max(500),
  limit: z.number().int().positive().max(100).optional(),
  filters: z.array(z.string()).optional(),
});
```

**Python: Pydantic**
```python
from pydantic import BaseModel, Field

class ToolParams(BaseModel):
    query: str = Field(..., min_length=1, max_length=500)
    limit: int = Field(100, ge=1, le=100)
    filters: list[str] = Field(default_factory=list)
```

#### **Flexible vs Strict Validation**

**Default (FastMCP): Flexible Validation**
- Pydantic coerces compatible inputs to match type annotations
- Improves LLM compatibility
- Handles LLMs sending string representations (e.g., "10" for integer)

**Strict Validation:**
- Enable when type safety is critical
- Use MCP SDK's built-in JSON Schema validation
- Less forgiving but more predictable

**Recommendation:** Start with flexible validation, tighten for security-critical parameters.

#### **Special Parameter Types**

**McpSyncRequestContext (Synchronous):**
- Automatically injected
- Excluded from JSON schema generation
- Provides unified interface for:
  - Original request
  - Server exchange (stateful operations)
  - Transport context (stateless operations)
  - Logging, progress, sampling, elicitation methods

**McpAsyncRequestContext (Asynchronous):**
- Same interface as sync version
- Reactive (Mono-based) return types
- Automatically injected

### 4.3 Return Value Formatting

#### **Tool Errors in Result Object**

**CRITICAL:** Report tool errors within the result object, NOT as MCP protocol-level errors. This allows the LLM to see and handle errors.

**Good Practice:**
```json
{
  "success": false,
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "Query must be between 1 and 500 characters. Please call with a valid query string."
  }
}
```

**Bad Practice:**
Throwing JSON-RPC protocol error that LLM cannot see or recover from.

#### **Structured Data**

Return rich, structured data that LLMs can work with:
- JSON objects with clear field names
- Arrays for collections
- Nested structures for relationships
- Metadata fields (timestamps, pagination info, etc.)

### 4.4 Handling Async Operations

#### **Async Tools (Recommended for I/O-Bound Operations)**

**Benefits:**
- Keeps server responsive
- Non-blocking event loop
- Handle multiple operations concurrently

**Python Implementation:**
```python
async def my_tool(param: str) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.get(f"https://api.example.com/{param}")
        return response.json()
```

**Key Patterns:**
- Declare functions with `async def`
- Use `async with` context managers
- Use `httpx.AsyncClient()` for HTTP requests
- Add `await` keyword for async operations

#### **Synchronous Tools**

Work seamlessly but can block the event loop during execution. Use only for:
- CPU-bound operations that are fast
- Operations that cannot be made async
- Simple calculations

#### **Advanced Async: Tasks Primitive (2025-11-25 Spec)**

For long-running operations:
- Tool advertises async support via annotation
- Client requests via `_meta` param
- Tool returns task ID and expiry interval immediately
- Tool continues running and emits progress notifications
- Client fetches results when ready

**Expiry Interval Semantics:**
- `0`: Task canceled if client disconnects
- `>0`: Task continues for specified seconds without client interaction

#### **Tool List Stability (Anti-Pattern Warning)**

**Do NOT change tool lists mid-session** unless absolutely necessary:
- Most model providers use caching to reduce costs
- Caching relies on stable tool list
- Changing tools mid-session invalidates cache
- Increases cost for client
- Reduces efficiency

**Exception:** The MCP standard includes tool list change notifications, but use with extreme caution.

---

## 5. Real-World Examples

### 5.1 Popular Open-Source MCP Servers

#### **Official Reference Servers (Anthropic)**

Repository: https://github.com/modelcontextprotocol/servers

**Examples:**
- **Everything:** Reference/test server with prompts, resources, and tools
- **Fetch:** Web content fetching
- **Filesystem:** Secure file operations
- **Git:** Tools for Git repositories
- **Memory:** Knowledge graph-based persistent memory
- **Sequential Thinking:** Dynamic problem-solving

#### **GitHub's Official MCP Server**

Repository: https://github.com/github/github-mcp-server
Blog Post: https://github.blog/open-source/maintainers/why-we-open-sourced-our-mcp-server-and-what-it-means-for-you/

**Features:**
- Repository Management: Browse/query code, search files, analyze commits, understand project structure
- Issue & PR Automation: Create, update, manage issues and PRs; triage bugs, review code
- CI/CD & Workflow Intelligence: Monitor GitHub Actions, analyze build failures, manage releases

**Architecture Pattern:**
- **Server:** Standalone service listening for structured MCP requests
- **Client:** Connector translating user intent into MCP requests
- **Host:** AI front-end (IDE assistant or chat UI) surfacing conversation
- **Data Flow:** User question → semantic request → MCP request → GitHub API → structured JSON → LLM

**Key Lesson:** Clean separation between language model, UX, and data/tools.

#### **Production-Ready Community Servers**

**MCPX by TheLunarCompany:**
- Production-ready open-source gateway
- Manage MCP servers at scale
- Centralized tool discovery
- Access controls and usage tracking
- Call prioritization

**mcgravity by tigranbs:**
- Proxy tool for composing multiple MCP servers
- Load balancing across servers
- Similar to Nginx for web servers
- Scales AI tools horizontally

#### **Awesome MCP Servers Collections**

- https://github.com/punkpeye/awesome-mcp-servers
- https://github.com/wong2/awesome-mcp-servers

**Popular Integrations:**
- Google Drive
- Slack
- GitHub
- PostgreSQL
- Puppeteer
- Kubernetes (kubernetes-mcp-server on PyPI)

### 5.2 Common Patterns in Production Servers

#### **1. Clean Architecture Pattern**

**Separation of Concerns:**
- Tool layer: MCP tool definitions
- Business logic layer: Core functionality
- Data access layer: Database/API calls
- Transport layer: HTTP/stdio handling

#### **2. Repository Pattern for Data Access**

Abstract data sources behind repository interfaces:
- Easy to test (mock repositories)
- Swap implementations (PostgreSQL → MongoDB)
- Clear data access boundaries

#### **3. Service Layer Pattern**

Encapsulate business logic in service classes:
- Tools call services
- Services orchestrate repositories
- Services handle complex workflows

#### **4. Factory Pattern for Server Construction**

Use factory functions to build servers with dependencies:
- Inject configuration
- Inject repositories/services
- Easier testing and customization

#### **5. Middleware Pattern (HTTP Transport)**

Layer cross-cutting concerns:
- Authentication middleware
- Logging middleware
- Rate limiting middleware
- CORS middleware
- Request ID tracking

#### **6. Circuit Breaker Pattern**

For external API calls:
- Prevent cascading failures
- Fast-fail when downstream is unhealthy
- Automatic recovery attempts

#### **7. Caching Pattern**

Cache expensive operations:
- In-memory cache (Redis)
- Time-based expiration
- Cache invalidation strategies
- Cache warming

**Performance Impact:** Production Go implementations significantly outperform Python reference servers.

**Best Practices:**
- Token security: Environment variables, regular rotation
- Resource management: Docker container limits
- Connection optimization: Connection pooling
- Robust error handling: Comprehensive cleanup mechanisms

### 5.3 Anti-Patterns to Avoid

(Covered comprehensively in Section 3.1 and Section 4.1)

**Quick Reference:**
1. Token passthrough (security anti-pattern)
2. Environment variables for secrets in production
3. Static API keys and PATs (88% of servers require credentials, 53% use insecure long-lived secrets)
4. Missing authentication and network exposure
5. SQL injection and RCE vulnerabilities
6. API wrapper one-on-one pattern (too many tools)
7. Making servers "too smart" (competing with LLM intelligence)
8. Changing tool lists mid-session (invalidates caching)

---

## 6. Deployment & Distribution

### 6.1 How to Package and Distribute MCP Servers

#### **Distribution Methods**

**1. Local Development:**
```bash
# Python
uv run -m {package_name}
```

**2. GitHub Repository:**
```bash
# Python
uvx git+https://github.com/org/repo#subdirectory={path}
```

**3. Package Repositories:**

**Python (PyPI):**
```bash
uvx {package_name}
```

**TypeScript (npm):**
```bash
npx @org/{package_name}
```

#### **Publishing to PyPI (Python)**

**Step 1: Structure as Python Package**

Create directory structure:
```
my-mcp-server/
├── src/
│   └── my_mcp_server/
│       ├── __init__.py
│       ├── server.py
│       └── tools.py
├── tests/
├── pyproject.toml
├── README.md
└── LICENSE
```

**Step 2: Define Package Metadata (pyproject.toml)**

```toml
[project]
name = "my-mcp-server"
version = "1.0.0"
description = "My custom MCP server"
authors = [{name = "Your Name", email = "you@example.com"}]
dependencies = [
    "mcp>=1.0.0",
    "httpx>=0.24.0",
]

[project.scripts]
my-mcp-server = "my_mcp_server.server:main"
```

**Step 3: Build and Publish**

```bash
# Install build tools
pip install build twine

# Build distribution
python -m build

# Upload to PyPI (requires PyPI account and API token)
twine upload dist/*
```

**Usage After Publishing:**
Users add to Claude Desktop config:
```json
{
  "mcpServers": {
    "my-server": {
      "command": "uvx",
      "args": ["my-mcp-server"]
    }
  }
}
```

#### **Publishing to npm (TypeScript)**

**Step 1: Update package.json**

```json
{
  "name": "@org/my-mcp-server",
  "version": "1.0.0",
  "description": "My custom MCP server",
  "bin": {
    "my-mcp-server": "./dist/index.js"
  },
  "scripts": {
    "build": "tsc",
    "prepublishOnly": "npm run build"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^1.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0"
  }
}
```

**Step 2: Build and Publish**

```bash
# Build TypeScript
npm run build

# Test locally
npm link
npm link @org/my-mcp-server

# Publish to npm (requires npm account)
npm publish --access public
```

**Usage After Publishing:**
```json
{
  "mcpServers": {
    "my-server": {
      "command": "npx",
      "args": ["@org/my-mcp-server"]
    }
  }
}
```

#### **Cloud Deployment**

**CloudMCP:**
- Deploy from NPM, PyPI, or GitHub with custom deployments
- Few-click deployment

**Manual Cloud Deployment:**
1. Containerize with Docker
2. Push to container registry
3. Deploy to cloud provider (AWS ECS, GCP Cloud Run, Azure Container Instances)
4. Configure load balancer and health checks
5. Set up auto-scaling

### 6.2 Configuration Management

#### **Configuration Sources (Priority Order)**

1. **Environment Variables** (lowest priority, local dev only)
2. **Configuration Files** (e.g., `config.yaml`, `.env` files)
3. **Secrets Management Services** (highest priority, production)
   - Azure Key Vault
   - AWS Secrets Manager
   - HashiCorp Vault

#### **Best Practices**

**Local Development:**
- `.env` files (DO NOT commit to git)
- Add `.env` to `.gitignore`
- Provide `.env.example` template

**Production:**
- NEVER use environment variables for secrets
- Use secrets management services
- Inject secrets at runtime
- Rotate credentials regularly
- Audit access to configuration

#### **Configuration Schema Validation**

Validate configuration on startup:
- Use Pydantic (Python) or Zod (TypeScript)
- Fail fast on invalid configuration
- Provide clear error messages

### 6.3 Version Compatibility Considerations

#### **Protocol Version Negotiation**

**Initialization Phase:**
- Client and server negotiate protocol version
- Server declares supported versions
- Client selects compatible version

**Header Requirement (HTTP Transport):**
```
MCP-Protocol-Version: 2025-06-18
```

**Backwards Compatibility:**
If server receives no `MCP-Protocol-Version` header, server SHOULD assume protocol version `2025-03-26`.

#### **Semantic Versioning for Servers**

**Version Format:** `MAJOR.MINOR.PATCH`

**Rules:**
- **MAJOR:** Breaking changes (incompatible tool changes)
- **MINOR:** New features (backwards compatible)
- **PATCH:** Bug fixes (backwards compatible)

#### **Deprecation Strategy**

**Deprecating Tools:**
1. Add deprecation notice in tool description
2. Continue supporting for at least 3 months
3. Log warnings when deprecated tools are used
4. Provide migration guide
5. Remove in next MAJOR version

**Deprecating Parameters:**
1. Make parameter optional
2. Document deprecation
3. Support old and new parameters simultaneously
4. Remove in next MAJOR version

#### **Testing Compatibility**

Test server against:
- Latest MCP client versions
- Previous MCP client versions (at least 2 versions back)
- Different transports (stdio and HTTP)

#### **Changelog Maintenance**

Maintain detailed changelog:
- List all breaking changes prominently
- Document new features
- List deprecated features
- Include migration guides for breaking changes

**Example Changelog Entry:**
```markdown
## [2.0.0] - 2025-12-09

### Breaking Changes
- Renamed `fetch_data` tool to `get_data` for consistency
- Migration: Update tool calls from `fetch_data` to `get_data`

### Added
- New `batch_get_data` tool for bulk operations
- Support for filtering results

### Deprecated
- `legacy_search` tool will be removed in v3.0.0
- Use `search` tool instead

### Fixed
- Fixed race condition in connection cleanup
```

---

## 7. Production Deployment Checklist

### 7.1 Pre-Deployment

- [ ] Security audit completed
- [ ] OAuth 2.1 authentication implemented
- [ ] RBAC configured and tested
- [ ] Input validation on all tools
- [ ] SQL injection prevention verified
- [ ] Secrets moved to secrets management service
- [ ] Origin header validation implemented (HTTP servers)
- [ ] Server binds to localhost only (local servers) or proper IP (remote servers)
- [ ] HTTPS enforced (remote servers)
- [ ] mTLS configured (high-security environments)

### 7.2 Testing

- [ ] Unit tests written (80%+ coverage)
- [ ] Integration tests cover main workflows
- [ ] Security tests pass (SQL injection, command injection, path traversal)
- [ ] Error handling tested with invalid inputs
- [ ] Schema validation tested
- [ ] Timeout handling tested
- [ ] LLM chaos testing completed
- [ ] Load testing conducted (if applicable)
- [ ] MCP Inspector testing passed

### 7.3 Monitoring & Observability

- [ ] Logging configured (structured logs)
- [ ] Metrics collection enabled (Prometheus or equivalent)
- [ ] Distributed tracing configured (if microservices)
- [ ] Health check endpoint implemented
- [ ] Readiness check endpoint implemented
- [ ] Alerting rules configured
- [ ] Dashboard created (Grafana or equivalent)
- [ ] Security event logging enabled
- [ ] Audit trail for authentication/authorization events

### 7.4 Performance

- [ ] Connection pooling configured (database/HTTP clients)
- [ ] Caching strategy implemented
- [ ] Resource limits set (Docker/Kubernetes)
- [ ] Auto-scaling configured (if needed)
- [ ] Load balancer configured (multi-instance deployments)
- [ ] CDN configured (for static assets, if applicable)

### 7.5 Reliability

- [ ] Graceful shutdown implemented
- [ ] Resource cleanup verified
- [ ] Circuit breakers configured (external dependencies)
- [ ] Retry logic with exponential backoff
- [ ] Timeout handling for all external calls
- [ ] Error recovery mechanisms tested
- [ ] Backup and restore procedures documented

### 7.6 Documentation

- [ ] README with installation instructions
- [ ] API documentation for all tools
- [ ] Configuration guide
- [ ] Security considerations documented
- [ ] Troubleshooting guide
- [ ] Changelog maintained
- [ ] Migration guides for breaking changes
- [ ] Example usage documented

### 7.7 Compliance & Legal

- [ ] License file included
- [ ] Third-party licenses documented
- [ ] Data privacy compliance verified (GDPR, etc.)
- [ ] Terms of service (if applicable)
- [ ] Security disclosure policy published

---

## 8. Key Takeaways

### 8.1 Critical Success Factors

1. **Security First:** OAuth 2.1, RBAC, input validation, secrets management
2. **Clear Tool Design:** Polymorphic pattern, few tools with rich parameters
3. **Meaningful Errors:** Help LLMs self-correct with actionable error messages
4. **Proper Testing:** Unit, integration, security, and LLM chaos testing
5. **Resource Management:** Proper lifecycle management and cleanup
6. **Production Mindset:** Monitoring, logging, health checks, graceful degradation

### 8.2 Common Pitfalls

1. Static API keys instead of OAuth 2.1 (53% of servers make this mistake)
2. Token passthrough anti-pattern (security vulnerability)
3. Too many tools (API wrapper one-on-one pattern)
4. Making servers "too smart" instead of information providers
5. Vague error messages that don't help LLMs recover
6. Environment variables for secrets in production
7. Missing Origin header validation (DNS rebinding attacks)

### 8.3 Future-Proofing

1. **Linux Foundation Stewardship:** MCP is now under Linux Foundation (December 2025)
2. **Industry Support:** Co-founded by Anthropic, Block, OpenAI; supported by Google, Microsoft, AWS, Cloudflare, Bloomberg
3. **Protocol Evolution:** Watch for WebSocket transport (SEP-1288) and new primitives
4. **SDK Expansion:** Official SDKs now include Go, Kotlin, Ruby, Java
5. **Community Growth:** Rapidly expanding ecosystem of open-source servers

### 8.4 Performance Targets

- **Simple Operations:** Sub-100ms response time
- **I/O-Bound Operations:** Use async/await, target <1s
- **Long-Running Tasks:** Use Tasks primitive (2025-11-25 spec)
- **Horizontal Scaling:** Streamable HTTP with stateless mode
- **High Availability:** Multi-instance with load balancing

---

## 9. Additional Resources

### 9.1 Official Documentation

- **Main Documentation:** https://docs.anthropic.com/en/docs/mcp
- **Protocol Specification:** https://modelcontextprotocol.io/specification/2025-06-18/
- **Transports Spec:** https://modelcontextprotocol.io/specification/2025-06-18/basic/transports
- **Lifecycle Spec:** https://modelcontextprotocol.io/specification/2025-03-26/basic/lifecycle
- **Security Best Practices:** https://modelcontextprotocol.io/specification/draft/basic/security_best_practices
- **Anthropic Announcement:** https://www.anthropic.com/news/model-context-protocol
- **GitHub Organization:** https://github.com/modelcontextprotocol

### 9.2 SDKs & Tools

- **TypeScript SDK:** https://github.com/modelcontextprotocol/typescript-sdk
- **Python SDK (PyPI):** https://pypi.org/project/mcp/
- **FastMCP Framework:** https://github.com/punkpeye/fastmcp
- **Official Servers:** https://github.com/modelcontextprotocol/servers
- **Example Servers:** https://modelcontextprotocol.io/examples

### 9.3 Community Resources

- **Awesome MCP Servers:** https://github.com/punkpeye/awesome-mcp-servers
- **Awesome MCP Servers (Alternative):** https://github.com/wong2/awesome-mcp-servers
- **GitHub's MCP Server:** https://github.com/github/github-mcp-server
- **Community Registry:** https://www.rapidmcp.org/
- **MCP Blog:** http://blog.modelcontextprotocol.io/

### 9.4 Learning Resources

- **Build Your First MCP Server (TypeScript):** https://www.freecodecamp.org/news/how-to-build-a-custom-mcp-server-with-typescript-a-handbook-for-developers/
- **MCP Tutorial (6 Steps):** https://towardsdatascience.com/model-context-protocol-mcp-tutorial-build-your-first-mcp-server-in-6-steps/
- **Custom Server Development:** https://www.mcpstack.org/learn/custom-server-development
- **Security Guide:** https://www.infracloud.io/blogs/securing-mcp-servers/
- **Testing Guide:** https://mcpcat.io/guides/writing-unit-tests-mcp-servers/

### 9.5 Security Resources

- **MCP Security Best Practices:** https://www.akto.io/blog/mcp-security-best-practices
- **Security Survival Guide:** https://towardsdatascience.com/the-mcp-security-survival-guide-best-practices-pitfalls-and-real-world-lessons/
- **GitHub Security Guide:** https://github.blog/ai-and-ml/generative-ai/how-to-build-secure-and-scalable-remote-mcp-servers/
- **OAuth 2.1 Spec Update:** https://auth0.com/blog/mcp-specs-update-all-about-auth/

### 9.6 Production Patterns

- **15 Best Practices:** https://thenewstack.io/15-best-practices-for-building-mcp-servers-in-production/
- **7 Best Practices:** https://www.marktechpost.com/2025/07/23/7-mcp-server-best-practices-for-scalable-ai-integrations-in-2025/
- **Efficient Production Servers:** https://dev.to/om_shree_0709/running-efficient-mcp-servers-in-production-metrics-patterns-pitfalls-42fb

### 9.7 Deployment

- **CloudMCP:** https://cloudmcp.run/blog/deploy-any-mcp-server
- **Publishing Guide:** https://www.geeky-gadgets.com/building-mcp-server-guide/
- **Hosting Guide:** https://www.devshorts.in/p/how-to-host-your-mcp-server

---

## Sources

- [What is MCP - Anthropic Docs](https://docs.anthropic.com/en/docs/mcp)
- [Introducing MCP - Anthropic News](https://www.anthropic.com/news/model-context-protocol)
- [Model Context Protocol GitHub](https://github.com/modelcontextprotocol)
- [TypeScript SDK](https://github.com/modelcontextprotocol/typescript-sdk)
- [Custom MCP Server Development](https://www.mcpstack.org/learn/custom-server-development)
- [TypeScript vs Python SDK Comparison](https://skywork.ai/blog/mcp-server-typescript-vs-mcp-server-python-2025-comparison/)
- [Build MCP Servers with TypeScript](https://dev.to/shadid12/how-to-build-mcp-servers-with-typescript-sdk-1c28)
- [OpenAI MCP Server Guide](https://developers.openai.com/apps-sdk/build/mcp-server/)
- [FastMCP Framework](https://github.com/punkpeye/fastmcp)
- [MCP Transports Specification](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports)
- [WebSocket Transport Proposal (SEP-1288)](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/1288)
- [HTTP Transport Discussion](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/493)
- [MCP Transport Mechanisms](https://gist.github.com/axelquack/7097ec1508dc3302c121c2ade8d3f0c6)
- [Protocol Mechanics and Architecture](https://pradeepl.com/blog/model-context-protocol/mcp-protocol-mechanics-and-architecture/)
- [GitHub Security Guide](https://github.blog/ai-and-ml/generative-ai/how-to-build-secure-and-scalable-remote-mcp-servers/)
- [15 Best Practices](https://thenewstack.io/15-best-practices-for-building-mcp-servers-in-production/)
- [Security Best Practices Spec](https://modelcontextprotocol.io/specification/draft/basic/security_best_practices)
- [Securing MCP Servers](https://www.infracloud.io/blogs/securing-mcp-servers/)
- [MCP Security Best Practices](https://www.akto.io/blog/mcp-security-best-practices)
- [Security Survival Guide](https://towardsdatascience.com/the-mcp-security-survival-guide-best-practices-pitfalls-and-real-world-lessons/)
- [OAuth Spec Update](https://auth0.com/blog/mcp-specs-update-all-about-auth/)
- [FastMCP Tools](https://gofastmcp.com/servers/tools)
- [MCP Tools Concepts](https://modelcontextprotocol.info/docs/concepts/tools/)
- [Async Operations Discussion](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/491)
- [MCP Tutorial (6 Steps)](https://towardsdatascience.com/model-context-protocol-mcp-tutorial-build-your-first-mcp-server-in-6-steps/)
- [Awesome MCP Servers](https://github.com/punkpeye/awesome-mcp-servers)
- [Official MCP Servers](https://github.com/modelcontextprotocol/servers)
- [Top 10 MCP Servers](https://dev.to/fallon_jimmy/top-10-mcp-servers-for-2025-yes-githubs-included-15jg)
- [GitHub MCP Server](https://github.com/github/github-mcp-server)
- [Why GitHub Open Sourced MCP Server](https://github.blog/open-source/maintainers/why-we-open-sourced-our-mcp-server-and-what-it-means-for-you/)
- [MCP Joins Linux Foundation](https://github.blog/open-source/maintainers/mcp-joins-the-linux-foundation-what-this-means-for-developers-building-the-next-era-of-ai-tools-and-agents/)
- [Top 8 Open Source MCP Projects](https://www.nocobase.com/en/blog/github-open-source-mcp-projects)
- [MCP Example Servers](https://modelcontextprotocol.io/examples)
- [Building MCP Server Guide](https://www.geeky-gadgets.com/building-mcp-server-guide/)
- [mcp PyPI](https://pypi.org/project/mcp/)
- [CloudMCP Deployment](https://cloudmcp.run/blog/deploy-any-mcp-server)
- [fastmcp PyPI](https://pypi.org/project/fastmcp/)
- [How to Host MCP Server](https://www.devshorts.in/p/how-to-host-your-mcp-server)
- [Unit Testing MCP Servers](https://mcpcat.io/guides/writing-unit-tests-mcp-servers/)
- [Integration Testing](https://mcpcat.io/guides/integration-tests-mcp-flows/)
- [MCP Testing Tools](https://testomat.io/blog/mcp-server-testing-tools/)
- [MCP Testing Framework](https://github.com/haakco/mcp-testing-framework)
- [Testing MCP Servers (PubNub)](https://www.pubnub.com/blog/testing-mcp-servers/)
- [Complete Testing Guide](https://agnost.ai/blog/testing-mcp-servers-complete-guide/)
- [Stop Vibe-Testing](https://www.jlowin.dev/blog/stop-vibe-testing-mcp-servers)
- [Lifecycle Specification](https://modelcontextprotocol.io/specification/2025-03-26/basic/lifecycle)
- [MCP Lifecycle Management](https://gist.github.com/anuj846k/2d641bf33606bcd13d8d5af311af1832)
- [Resource Management Guide](https://www.arsturn.com/blog/the-down-low-on-mcp-resource-management-cleanup-a-no-nonsense-guide)
- [MCP 2025-11-25 Spec Update](https://workos.com/blog/mcp-2025-11-25-spec-update)
- [7 Best Practices](https://www.marktechpost.com/2025/07/23/7-mcp-server-best-practices-for-scalable-ai-integrations-in-2025/)
- [MCP Design Principles](https://www.matt-adams.co.uk/2025/08/30/mcp-design-principles.html)
- [MCP Server Security Nightmare](https://equixly.com/blog/2025/03/29/mcp-server-new-security-nightmare/)
- [Efficient Production Servers](https://dev.to/om_shree_0709/running-efficient-mcp-servers-in-production-metrics-patterns-pitfalls-42fb)
- [State of MCP Security 2025](https://astrix.security/learn/blog/state-of-mcp-server-security-2025/)

---

**Document Version:** 1.0
**Last Updated:** 2025-12-09
**Protocol Version:** MCP 2025-06-18
**Completeness:** 95%
**Research Method:** Wide-then-deep parallel architecture with hierarchical synthesis
