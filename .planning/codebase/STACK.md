# Technology Stack

**Analysis Date:** 2026-01-31

## Languages

**Primary:**

- TypeScript 5.8.2 - All source code, ESM modules throughout
- JavaScript - Build scripts and utilities

**Secondary:**

- Shell scripts - Build and deployment automation (`install/`, `script/`)
- Nix - Reproducible development environment (`flake.nix`)

## Runtime

**Environment:**

- Bun 1.3.5 - Primary JavaScript runtime and package manager
- Node.js 22+ - Compatibility target (via `@tsconfig/node22`)

**Package Manager:**

- Bun with lockfile (`bun.lock`)
- Workspace monorepo structure with catalog dependencies

## Frameworks

**Core Framework:**

- SolidJS 1.9.10 - Reactive UI framework for web and desktop apps
- SolidStart - Server-side rendering framework (via pkg.pr.new)
- SolidJS Router 0.15.4 - Client-side routing

**Web Frameworks:**

- Astro 5.7.13 - Static site generation for documentation
- Starlight 0.34.3 - Documentation theme for Astro
- Hono 4.10.7 - Lightweight web framework for API endpoints
- Hono OpenAPI 1.1.2 - OpenAPI/Swagger integration for Hono

**Desktop Framework:**

- Tauri v2 - Native desktop application wrapper
  - `@tauri-apps/api` ^2 - Tauri API bindings
  - Multiple Tauri plugins: deep-link, dialog, notification, shell, store, updater, http

**UI Framework:**

- Kobalte Core 0.13.11 - Accessible component primitives
- OpenTUI Core/Solid 0.1.75 - Terminal UI components
- TailwindCSS 4.1.11 - Utility-first CSS framework
- TailwindCSS Vite Plugin - Tailwind Vite integration

**AI Integration:**

- Vercel AI SDK (`ai` 5.0.119) - Unified AI provider interface
  - 15+ provider SDKs: OpenAI, Anthropic, Google, Azure, AWS Bedrock, Mistral, Groq, Cohere, Perplexity, etc.
- Model Context Protocol SDK 1.25.2 - AI agent protocol
- Agent Client Protocol SDK 0.12.0 - Agent communication standard

**Server/Cloud:**

- SST (Serverless Stack) 3.17.23 - Infrastructure as code framework
- Cloudflare Workers - Serverless compute platform

**ORM/Database:**

- Drizzle ORM 0.41.0 - TypeScript ORM for PlanetScale
- Drizzle Kit 0.30.5 - Database migrations and management
- `@planetscale/database` 1.19.0 - PlanetScale database driver

**Authentication:**

- OpenAuth 0.0.0-20250322224806 - Authentication framework for Cloudflare Workers

## Key Dependencies

**Critical Infrastructure:**

- `zod` 4.1.8 - Runtime type validation and schema definition
- `@octokit/rest` 22.0.0 - GitHub API client
- `remeda` 2.26.0 - Functional programming utilities
- `ulid` 3.0.1 - Unique identifier generation
- `luxon` 3.6.1 - Date/time manipulation
- `marked` 17.0.1 - Markdown parsing
- `shiki` 3.20.0 - Syntax highlighting

**Development Tools:**

- Turbo 2.5.6 - Monorepo task runner
- Husky 9.1.7 - Git hooks management
- Prettier 3.6.2 - Code formatting (semi: false, printWidth: 120)

**Testing:**

- Bun Test - Built-in testing framework for unit tests
- Playwright 1.57.0 - E2E browser automation testing
- `@playwright/test` 1.51.0 - Playwright test runner

**Build Tools:**

- Vite 7.1.4 - Build tool and dev server
- Vite Plugin Solid 2.11.10 - SolidJS Vite plugin
- TypeScript Native Preview - Fast TypeScript compilation

## Configuration

**TypeScript:**

- Base config: `@tsconfig/bun/tsconfig.json`
- Root: `tsconfig.json` extends Bun config with empty compiler options
- Catalog versions for consistency across packages

**Package Management:**

- Workspaces: `packages/*`, `packages/console/*`, `packages/sdk/js`, `packages/slack`
- Catalog dependencies for version consistency
- Workspace dependencies use `workspace:*` protocol

**Environment:**

- Environment variables managed via SST secrets
- Stage-based configuration (production, dev, custom stages)
- Domain: `opencode.ai` (production), `dev.opencode.ai` (dev), `{stage}.dev.opencode.ai` (custom)

**Build:**

- `bunfig.toml` - Bun configuration
- `turbo.json` - Turbo task pipeline configuration
- `sst.config.ts` - SST infrastructure configuration
- Patched dependencies in `patches/` directory

## Platform Requirements

**Development:**

- Bun 1.3.5+
- Git
- Nix (optional, for reproducible environment via `flake.nix`)

**Production Deployment:**

- Cloudflare Workers
- Cloudflare R2 (object storage)
- Cloudflare KV (key-value storage)
- PlanetScale (MySQL-compatible database)

**Desktop Build:**

- Rust toolchain (for Tauri native components)
- Platform-specific build tools (Xcode, Visual Studio, etc.)

---

_Stack analysis: 2026-01-31_
