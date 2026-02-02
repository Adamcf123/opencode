# Architecture

**Analysis Date:** 2026-01-31

## Pattern Overview

**Overall:** Modular Monolith with Plugin Architecture

**Key Characteristics:**

- **Namespace-based organization**: Code organized using TypeScript namespaces (e.g., `Tool.define()`, `Session.create()`)
- **Context-based dependency injection**: Uses `Context.create()` pattern for request-scoped dependencies
- **Event-driven communication**: Global event bus (`Bus`, `GlobalBus`) for decoupled component communication
- **Functional tool system**: Tools defined as pure functions with Zod schemas for validation
- **Code generation**: SDK auto-generated from OpenAPI specs via `hono-openapi`

## Layers

**CLI Layer:**

- Purpose: Command-line interface and user interaction
- Location: `packages/opencode/src/cli/`
- Contains: Command definitions, argument parsing, terminal UI
- Depends on: Server layer, Tool layer, Agent layer
- Used by: End users via terminal

**Server Layer:**

- Purpose: HTTP API server and WebSocket handling
- Location: `packages/opencode/src/server/`
- Contains: Hono routes, middleware, SSE streaming, OpenAPI generation
- Depends on: Tool layer, Session layer, Instance layer
- Used by: CLI, Web app, Desktop app, SDK clients

**Tool Layer:**

- Purpose: AI tool implementations (file operations, shell, search)
- Location: `packages/opencode/src/tool/`
- Contains: Tool definitions with Zod schemas, execution logic
- Depends on: Permission layer, Project layer
- Used by: Agent layer, Server layer

**Agent Layer:**

- Purpose: AI agent configuration and orchestration
- Location: `packages/opencode/src/agent/`
- Contains: Agent definitions, prompt templates, mode configurations
- Depends on: Provider layer, Permission layer
- Used by: Session layer

**Session Layer:**

- Purpose: Conversation management and message handling
- Location: `packages/opencode/src/session/`
- Contains: Message types, conversation state, prompt building
- Depends on: Agent layer, Tool layer, Storage layer
- Used by: Server layer

**Storage Layer:**

- Purpose: Persistent storage for sessions, config, and state
- Location: `packages/opencode/src/storage/`
- Contains: File-based storage with migrations
- Depends on: Filesystem utilities
- Used by: Session layer, Config layer

**Provider Layer:**

- Purpose: LLM provider abstraction (OpenAI, Anthropic, etc.)
- Location: `packages/opencode/src/provider/`
- Contains: Provider implementations, model configurations, transforms
- Depends on: AI SDK
- Used by: Agent layer

**Project Layer:**

- Purpose: Project/workspace management and isolation
- Location: `packages/opencode/src/project/`
- Contains: Instance management, worktree handling, VCS integration
- Depends on: Storage layer
- Used by: Server layer, Tool layer

**UI Layer (App):**

- Purpose: Web-based user interface
- Location: `packages/app/src/`
- Contains: SolidJS components, contexts, pages
- Depends on: SDK, UI component library
- Used by: End users via browser

**SDK Layer:**

- Purpose: Auto-generated API client
- Location: `packages/sdk/js/src/`
- Contains: Generated TypeScript client from OpenAPI spec
- Depends on: Server OpenAPI spec
- Used by: App, External integrations

## Data Flow

**Request Flow (CLI → Server → Tool → LLM):**

1. CLI command invoked via `packages/opencode/src/index.ts`
2. Yargs parses arguments, dispatches to command handler
3. Server routes (`packages/opencode/src/server/routes/`) handle HTTP requests
4. Instance context created via `Instance.provide()` for request isolation
5. Tools executed through `Tool.execute()` with Zod validation
6. Agent processes tool results via LLM provider
7. Session state persisted to Storage
8. Events broadcast via GlobalBus for real-time updates

**Event Flow:**

1. Component emits event via `GlobalBus.emit()`
2. SSE endpoint (`/event`) streams events to connected clients
3. Web UI receives events and updates SolidJS stores
4. Terminal UI receives events for real-time output

**State Management:**

- **Instance-scoped state**: `Instance.state()` for per-directory state
- **Global state**: `Global` namespace for application-wide constants
- **Storage**: File-based JSON storage with automatic migrations

## Key Abstractions

**Tool:**

- Purpose: AI-callable operations with type-safe inputs
- Examples: `packages/opencode/src/tool/read.ts`, `packages/opencode/src/tool/bash.ts`, `packages/opencode/src/tool/edit.ts`
- Pattern: `Tool.define(id, initFn)` returns `{ description, parameters, execute }`
- Validation: Zod schemas for runtime type checking

**Agent:**

- Purpose: Configurable AI personas with different capabilities
- Examples: `packages/opencode/src/agent/agent.ts`
- Pattern: Agent.Info schema defines behavior, permissions, and prompts
- Modes: "primary", "subagent", "all" for different use cases

**Instance:**

- Purpose: Request-scoped context for project isolation
- Examples: `packages/opencode/src/project/instance.ts`
- Pattern: `Instance.provide({ directory, fn })` runs code in isolated context
- Contains: directory, worktree, project info

**Permission:**

- Purpose: Fine-grained access control for tools
- Examples: `packages/opencode/src/permission/next.ts`
- Pattern: Ruleset-based with glob patterns for file paths
- Levels: "allow", "deny", "ask"

**Session:**

- Purpose: Conversation container with message history
- Examples: `packages/opencode/src/session/`
- Pattern: MessageV2 format with parts (text, tool-call, tool-result)
- Storage: File-based in `~/.opencode/state/`

**Provider:**

- Purpose: Abstract LLM provider interface
- Examples: `packages/opencode/src/provider/provider.ts`
- Pattern: Provider.Info with model selection and auth
- Supported: OpenAI, Anthropic, and OpenAI-compatible APIs

## Entry Points

**CLI Entry:**

- Location: `packages/opencode/src/index.ts`
- Triggers: Command line execution
- Responsibilities: Parse args, initialize logging, dispatch commands

**Server Entry:**

- Location: `packages/opencode/src/cli/cmd/serve.ts`
- Triggers: `opencode serve` command
- Responsibilities: Start HTTP server, initialize routes

**App Entry:**

- Location: `packages/app/src/entry.tsx`
- Triggers: Browser page load
- Responsibilities: Initialize SolidJS app, set up providers

**SDK Entry:**

- Location: `packages/sdk/js/src/client.ts`
- Triggers: Imported by consumers
- Responsibilities: Create API client with fetch configuration

## Error Handling

**Strategy:** NamedError pattern with structured error objects

**Patterns:**

- All errors extend `NamedError` with `.toObject()` serialization
- HTTP status codes mapped from error types in server middleware
- Tool validation errors formatted with custom messages
- Fail-loud approach: errors thrown at tool layer, handled at entry layer

**Error Types:**

- `Storage.NotFoundError` → 404
- `Provider.ModelNotFoundError` → 400
- Worktree errors → 400
- Unknown errors → 500 with stack trace

## Cross-Cutting Concerns

**Logging:**

- Approach: Structured logging with `Log.create({ service })`
- Levels: DEBUG, INFO, WARN, ERROR
- Output: File-based with optional stderr printing

**Validation:**

- Approach: Zod schemas for all external inputs
- Location: Tool parameters, API routes, config loading
- Pattern: Parse at boundaries, assume valid internally

**Authentication:**

- Approach: Provider-based auth with multiple methods (OAuth, API key)
- Location: `packages/opencode/src/auth/`
- Storage: OS-specific secure storage

**Configuration:**

- Approach: Layered config with precedence (remote < global < project < env)
- Files: `opencode.jsonc`, `opencode.json`
- Location: `packages/opencode/src/config/config.ts`

---

_Architecture analysis: 2026-01-31_
