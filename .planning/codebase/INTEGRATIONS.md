# External Integrations

**Analysis Date:** 2026-01-31

## APIs & External Services

**AI/LLM Providers:**
The codebase integrates with 15+ AI providers through the Vercel AI SDK:

- **OpenAI** (`@ai-sdk/openai` 2.0.89) - GPT-4, GPT-3.5 models
- **Anthropic** (`@ai-sdk/anthropic` 2.0.57) - Claude models
- **Google** (`@ai-sdk/google` 2.0.52, `@ai-sdk/google-vertex` 3.0.97) - Gemini, PaLM models
- **Azure OpenAI** (`@ai-sdk/azure` 2.0.91) - Azure-hosted OpenAI models
- **AWS Bedrock** (`@ai-sdk/amazon-bedrock` 3.0.73) - AWS-managed models
- **Mistral** (`@ai-sdk/mistral` 2.0.27) - Mistral models
- **Groq** (`@ai-sdk/groq` 2.0.34) - Fast inference
- **Cohere** (`@ai-sdk/cohere` 2.0.22) - Cohere models
- **Perplexity** (`@ai-sdk/perplexity` 2.0.23) - Perplexity AI
- **Together AI** (`@ai-sdk/togetherai` 1.0.31) - Model aggregation
- **xAI** (`@ai-sdk/xai` 2.0.51) - Grok models
- **Cerebras** (`@ai-sdk/cerebras` 1.0.34) - Cerebras inference
- **DeepInfra** (`@ai-sdk/deepinfra` 1.0.31) - Model hosting
- **OpenRouter** (`@openrouter/ai-sdk-provider` 1.5.2) - Model routing
- **GitLab AI** (`@gitlab/gitlab-ai-provider` 3.3.1) - GitLab AI

**GitHub:**

- `@octokit/rest` 22.0.0 - GitHub REST API
- `@octokit/graphql` 9.0.2 - GitHub GraphQL API
- `@octokit/auth-app` 8.0.1 - GitHub App authentication
- `@actions/core` 1.11.1 - GitHub Actions toolkit
- `@actions/github` 6.0.1 - GitHub Actions helper
- GitHub App integration via `GITHUB_APP_ID` and `GITHUB_APP_PRIVATE_KEY` secrets

**Slack:**

- `@slack/bolt` ^3.17.1 - Slack bot framework
- Located in `packages/slack/src/index.ts`

**Payment Processing:**

- Stripe SDK 18.0.0 - Payment processing
- Webhook endpoint at `https://{domain}/stripe/webhook`
- Events: checkout sessions, charges, invoices, customers, subscriptions
- Product: "OpenCode Black" with tiered pricing ($20, $100, $200/month)
- Secrets: `STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`

**Email:**

- AWS SES (Simple Email Service) - Transactional email
- JSX Email (`@jsx-email/render` 1.1.1) - Email templating
- Secrets: `AWS_SES_ACCESS_KEY_ID`, `AWS_SES_SECRET_ACCESS_KEY`
- EmailOctopus - Newsletter/mailing list
  - Secret: `EMAILOCTOPUS_API_KEY`

**Monitoring/Observability:**

- Honeycomb - Observability platform (production only)
  - Secret: `HONEYCOMB_API_KEY`
  - Log processor worker: `packages/console/function/src/log-processor.ts`

**Discord:**

- Discord bot integration for support
- Secret: `DISCORD_SUPPORT_BOT_TOKEN`, `DISCORD_SUPPORT_CHANNEL_ID`

**Feishu (Lark):**

- Feishu/Lark app integration
- Secrets: `FEISHU_APP_ID`, `FEISHU_APP_SECRET`

## Data Storage

**Primary Database:**

- **PlanetScale** - MySQL-compatible serverless database
  - Organization: `anomalyco`
  - Database: `opencode`
  - Connection via `@planetscale/database` driver
  - Drizzle ORM for type-safe queries
  - Production branch: `production`
  - Development branches per stage

**Object Storage:**

- **Cloudflare R2** - S3-compatible object storage
  - Buckets: `Bucket` (API), `ZenData`, `ZenDataNew` (Console)

**Key-Value Storage:**

- **Cloudflare KV** - Distributed key-value store
  - `AuthStorage` - Authentication session storage
  - `GatewayKv` - Gateway rate limiting/caching

**Caching:**

- Cloudflare Workers Cache API - Edge caching
- KV storage for persistent caching needs

## Authentication & Identity

**Auth Provider:**

- OpenAuth (custom implementation on Cloudflare Workers)
- Location: `packages/console/function/src/auth.ts`
- Domain: `auth.{domain}`

**OAuth Providers:**

- GitHub OAuth - `GITHUB_CLIENT_ID_CONSOLE`, `GITHUB_CLIENT_SECRET_CONSOLE`
- Google OAuth - `GOOGLE_CLIENT_ID`

**Authentication Storage:**

- Cloudflare KV for session storage
- JWT tokens via `jose` library (6.0.11)

## Monitoring & Observability

**Error Tracking:**

- Cloudflare Workers built-in logging (Logpush enabled)
- Tail consumers for log processing (production)

**Logs:**

- Cloudflare Logpush - Automatic log forwarding
- Custom log processor worker (production/staging)
- Honeycomb integration for advanced observability

**Performance:**

- Cloudflare Workers Analytics
- Smart placement mode for Workers

## CI/CD & Deployment

**Infrastructure as Code:**

- SST (Serverless Stack) 3.17.23
- Cloudflare provider
- Stripe provider
- PlanetScale provider

**Hosting:**

- **Cloudflare Workers** - Serverless compute (API, Auth, Gateway)
- **Cloudflare Pages** - Static site hosting (Documentation)
- **Custom domains:**
  - `opencode.ai` / `opncd.ai` (production)
  - `dev.opencode.ai` / `dev.opncd.ai` (development)
  - `{stage}.dev.opencode.ai` (custom stages)

**CI Pipeline:**

- GitHub Actions (`.github/workflows/`)
- Automated testing with Playwright
- Release automation via scripts in `script/`

## Environment Configuration

**Required Environment Variables:**

**Infrastructure:**

- `STRIPE_SECRET_KEY` - Stripe API access
- `CLOUDFLARE_API_TOKEN` - Cloudflare API access
- `CLOUDFLARE_DEFAULT_ACCOUNT_ID` - Cloudflare account

**Authentication:**

- `GITHUB_APP_ID`, `GITHUB_APP_PRIVATE_KEY` - GitHub App
- `GITHUB_CLIENT_ID_CONSOLE`, `GITHUB_CLIENT_SECRET_CONSOLE` - GitHub OAuth
- `GOOGLE_CLIENT_ID` - Google OAuth
- `ZEN_SESSION_SECRET` - Session encryption

**Third-Party Services:**

- `AWS_SES_ACCESS_KEY_ID`, `AWS_SES_SECRET_ACCESS_KEY` - AWS SES
- `HONEYCOMB_API_KEY` - Observability (production)
- `DISCORD_SUPPORT_BOT_TOKEN`, `DISCORD_SUPPORT_CHANNEL_ID` - Discord
- `FEISHU_APP_ID`, `FEISHU_APP_SECRET` - Feishu
- `EMAILOCTOPUS_API_KEY` - Email marketing

**Admin:**

- `ADMIN_SECRET` - Admin authentication

**Secrets Management:**

- SST Secrets for encrypted environment variables
- Per-stage secret configuration
- Secret rotation via SST

## Webhooks & Callbacks

**Incoming:**

- `POST /stripe/webhook` - Stripe payment events
  - Checkout sessions (completed, failed, expired)
  - Subscription events (created, updated, deleted, paused, resumed)
  - Invoice events (payment succeeded, failed)
  - Customer events

**Outgoing:**

- GitHub API calls for repository operations
- Discord API for support notifications
- Feishu API for enterprise integrations
- AWS SES for email delivery
- Slack API for bot interactions

**Protocols:**

- Model Context Protocol (MCP) - AI agent communication
- Agent Client Protocol (ACP) - Agent-to-agent communication
- WebSocket for real-time features (via `@solid-primitives/websocket`)

---

_Integration audit: 2026-01-31_
