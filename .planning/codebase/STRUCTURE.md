# Codebase Structure

**Analysis Date:** 2026-01-31

## Directory Layout

```
[project-root]/
├── packages/
│   ├── opencode/          # Core CLI and server
│   │   ├── src/
│   │   │   ├── cli/       # CLI commands and UI
│   │   │   ├── server/    # HTTP server and routes
│   │   │   ├── tool/      # AI tool implementations
│   │   │   ├── agent/     # Agent configurations
│   │   │   ├── session/   # Session management
│   │   │   ├── project/   # Project/instance management
│   │   │   ├── provider/  # LLM provider integrations
│   │   │   ├── storage/   # Persistence layer
│   │   │   ├── config/    # Configuration management
│   │   │   ├── auth/      # Authentication
│   │   │   ├── permission/# Permission system
│   │   │   ├── lsp/       # Language server protocol
│   │   │   ├── mcp/       # Model Context Protocol
│   │   │   ├── bus/       # Event bus
│   │   │   ├── plugin/    # Plugin system
│   │   │   └── util/      # Utilities
│   │   └── script/        # Build/dev scripts
│   ├── app/               # Web UI (SolidJS)
│   │   ├── src/
│   │   │   ├── components/# React/Solid components
│   │   │   ├── context/   # SolidJS context providers
│   │   │   ├── pages/     # Page components
│   │   │   └── utils/     # App utilities
│   │   └── addons/        # App addons
│   ├── ui/                # UI component library
│   │   └── src/
│   │       ├── components/# Reusable UI components
│   │       ├── context/   # UI contexts
│   │       └── theme/     # Theme/styling
│   ├── sdk/js/            # JavaScript SDK
│   │   └── src/
│   │       ├── gen/       # Generated OpenAPI client
│   │       └── v2/        # V2 API client
│   ├── plugin/            # Plugin API types
│   ├── script/            # Script utilities
│   ├── function/          # Cloud function API
│   ├── desktop/           # Desktop app shell
│   ├── enterprise/        # Enterprise features
│   ├── util/              # Shared utilities
│   ├── web/               # Marketing website
│   ├── slack/             # Slack integration
│   └── console/           # Console infrastructure
│       ├── app/
│       ├── core/
│       ├── function/
│       ├── mail/
│       └── resource/
├── sdks/
│   └── vscode/            # VSCode extension
├── script/                # Root build scripts
├── infra/                 # SST infrastructure
├── github/                # GitHub integrations
├── themes/                # Editor themes
├── patches/               # Package patches
└── .opencode/             # Self-hosted opencode config
```

## Directory Purposes

**packages/opencode/src/cli/:**

- Purpose: Command-line interface implementation
- Contains: Command definitions, argument parsing, terminal UI components
- Key files: `packages/opencode/src/index.ts`, `packages/opencode/src/cli/cmd/*.ts`

**packages/opencode/src/server/:**

- Purpose: HTTP API server
- Contains: Hono routes, middleware, SSE streaming
- Key files: `packages/opencode/src/server/server.ts`, `packages/opencode/src/server/routes/*.ts`

**packages/opencode/src/tool/:**

- Purpose: AI tool implementations
- Contains: Tool definitions, file operations, shell execution
- Key files: `packages/opencode/src/tool/tool.ts`, `packages/opencode/src/tool/read.ts`, `packages/opencode/src/tool/edit.ts`, `packages/opencode/src/tool/bash.ts`

**packages/opencode/src/agent/:**

- Purpose: Agent configurations and orchestration
- Contains: Agent definitions, prompts, modes
- Key files: `packages/opencode/src/agent/agent.ts`

**packages/opencode/src/session/:**

- Purpose: Session management
- Contains: Message types, conversation handling
- Key files: `packages/opencode/src/session/`

**packages/opencode/src/project/:**

- Purpose: Project workspace management
- Contains: Instance management, worktree handling, VCS
- Key files: `packages/opencode/src/project/instance.ts`, `packages/opencode/src/project/project.ts`

**packages/opencode/src/provider/:**

- Purpose: LLM provider integrations
- Contains: OpenAI, Anthropic, and compatible providers
- Key files: `packages/opencode/src/provider/provider.ts`

**packages/app/src/:**

- Purpose: Web UI application
- Contains: SolidJS components, pages, contexts
- Key files: `packages/app/src/app.tsx`, `packages/app/src/entry.tsx`

**packages/ui/src/:**

- Purpose: Shared UI component library
- Contains: Reusable SolidJS components
- Key files: `packages/ui/src/components/*.tsx`

**packages/sdk/js/src/:**

- Purpose: Auto-generated API client
- Contains: Generated from OpenAPI spec
- Key files: `packages/sdk/js/src/client.ts`, `packages/sdk/js/src/gen/`

**packages/plugin/src/:**

- Purpose: Plugin API type definitions
- Contains: Plugin interfaces, hooks
- Key files: `packages/plugin/src/index.ts`

**packages/util/src/:**

- Purpose: Shared utility functions
- Contains: Array, path, encoding utilities
- Key files: `packages/util/src/*.ts`

**packages/console/:**

- Purpose: Cloud console infrastructure
- Contains: SST resources, functions, email templates
- Key files: `packages/console/*/`

## Key File Locations

**Entry Points:**

- CLI: `packages/opencode/src/index.ts`
- Server: `packages/opencode/src/cli/cmd/serve.ts`
- App: `packages/app/src/entry.tsx`
- SDK: `packages/sdk/js/src/client.ts`
- Desktop: `packages/desktop/src/index.tsx`
- Enterprise: `packages/enterprise/src/entry-client.tsx`

**Configuration:**

- Root package: `package.json`
- Workspace config: `turbo.json`
- TypeScript: `tsconfig.json`
- Bun config: `bunfig.toml`
- SST: `sst.config.ts`

**Core Logic:**

- Server: `packages/opencode/src/server/server.ts`
- Tool base: `packages/opencode/src/tool/tool.ts`
- Agent base: `packages/opencode/src/agent/agent.ts`
- Instance: `packages/opencode/src/project/instance.ts`
- Config: `packages/opencode/src/config/config.ts`
- Storage: `packages/opencode/src/storage/storage.ts`

**Testing:**

- Test files: `packages/*/test/**/*.ts`
- Example: `packages/enterprise/test/core/`

## Naming Conventions

**Files:**

- PascalCase for component files: `Button.tsx`, `Dialog.tsx`
- camelCase for utility files: `log.ts`, `filesystem.ts`
- lowercase for directories: `tool/`, `server/`, `agent/`
- `.txt` for prompt templates: `bash.txt`, `generate.txt`

**Directories:**

- Plural for collections: `tools/`, `routes/`, `components/`
- Singular for concepts: `tool/`, `agent/`, `server/`

**TypeScript Namespaces:**

- PascalCase namespace names: `Tool`, `Session`, `Agent`, `Config`
- Static methods on namespaces: `Tool.define()`, `Session.create()`
- No class instantiation, use factory functions

## Where to Add New Code

**New Tool:**

- Implementation: `packages/opencode/src/tool/{tool-name}.ts`
- Add to registry in: `packages/opencode/src/tool/registry.ts`
- Tests: `packages/opencode/test/tool/{tool-name}.test.ts`

**New CLI Command:**

- Implementation: `packages/opencode/src/cli/cmd/{command}.ts`
- Register in: `packages/opencode/src/index.ts`
- Follow pattern from: `packages/opencode/src/cli/cmd/run.ts`

**New Server Route:**

- Add to: `packages/opencode/src/server/routes/{area}.ts`
- Or extend existing route file
- Regenerate SDK: `./script/generate.ts`

**New UI Component:**

- Implementation: `packages/ui/src/components/{Component}.tsx`
- Styles: `packages/ui/src/components/{Component}.css`
- Use in app: `packages/app/src/components/`

**New Agent Mode:**

- Add to: `packages/opencode/src/agent/agent.ts`
- Define in `Agent` namespace state
- Add prompt templates as `.txt` files

**New Provider:**

- Add to: `packages/opencode/src/provider/`
- Follow pattern from existing providers
- Register in: `packages/opencode/src/provider/provider.ts`

**Utilities:**

- Shared helpers: `packages/util/src/{utility}.ts`
- Package-specific: `packages/opencode/src/util/{utility}.ts`

## Special Directories

**packages/sdk/js/src/gen/:**

- Purpose: Auto-generated from OpenAPI spec
- Generated: Yes (via `./script/generate.ts`)
- Committed: Yes (for dependency management)
- Do not edit manually

**patches/:**

- Purpose: Package patches for dependencies
- Files: `patches/*.patch`
- Applied by: Bun package manager

**packages/opencode/src/\*/:**

- Purpose: Core namespaces
- Pattern: Each major feature has its own directory
- Entry: Usually `index.ts` or main file

---

_Structure analysis: 2026-01-31_
