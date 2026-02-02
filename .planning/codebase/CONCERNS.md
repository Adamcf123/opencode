# Codebase Concerns

**Analysis Date:** 2026-01-31

## Tech Debt

### Large Component Files

**Session and Layout Components:**

- Issue: `packages/app/src/pages/session.tsx` (3,054 lines) and `packages/app/src/pages/layout.tsx` (2,901 lines) are monolithic files containing UI logic, state management, and business logic
- Files: `packages/app/src/pages/session.tsx`, `packages/app/src/pages/layout.tsx`
- Impact: Difficult to maintain, test, and reason about; changes in one area risk breaking unrelated functionality
- Fix approach: Extract smaller components and custom hooks; separate business logic from presentation

**Generated SDK Files:**

- Issue: Auto-generated SDK files are extremely large and committed to the repository
- Files: `packages/sdk/js/src/v2/gen/types.gen.ts` (4,983 lines), `packages/sdk/js/src/gen/types.gen.ts` (3,904 lines)
- Impact: Repository bloat; merge conflicts on regeneration; review noise
- Fix approach: Add to `.gitignore` and generate during build; or use a separate generated package

**LSP Server:**

- Issue: `packages/opencode/src/lsp/server.ts` (2,046 lines) handles multiple LSP capabilities in one file
- Files: `packages/opencode/src/lsp/server.ts`
- Impact: Hard to test individual LSP features; violates single responsibility principle
- Fix approach: Split into feature-specific modules (hover, completion, diagnostics, etc.)

### Duplicated Code

**SDK Generation:**

- Issue: Two versions of the SDK (v1 and v2) with nearly identical structure
- Files: `packages/sdk/js/src/gen/*`, `packages/sdk/js/src/v2/gen/*`
- Impact: Double maintenance burden; inconsistencies between versions
- Fix approach: Deprecate v1 and migrate all consumers to v2; or extract shared core

**Workarounds for Known Issues:**

- Issue: Multiple `setTimeout` workarounds for focus/input issues without proper fixes
- Files: `packages/opencode/src/cli/cmd/tui/component/prompt/index.tsx:99`, `packages/opencode/src/cli/cmd/tui/component/prompt/index.tsx:930-943`
- Impact: Technical debt accumulation; fragile UI behavior
- Fix approach: Address root cause in the UI framework or component lifecycle

### Suppressed Type Checking

**@ts-ignore/@ts-expect-error Usage:**

- Issue: Multiple locations suppress TypeScript errors instead of fixing them
- Files:
  - `packages/sdk/js/src/client.ts:11` - `// @ts-ignore`
  - `packages/sdk/js/src/v2/client.ts:11` - `// @ts-ignore`
  - `packages/app/src/components/session/session-sortable-tab.tsx:33` - `// @ts-ignore`
  - `packages/app/src/pages/layout.tsx:2108,2315` - `// @ts-ignore`
  - `packages/ui/src/components/message-part.tsx:595,638,935,944` - `// @ts-expect-error`
- Impact: Type safety gaps; potential runtime errors
- Fix approach: Fix underlying type issues or use proper type assertions with comments

## Security Considerations

### Path Traversal Vulnerabilities

**Filesystem Access:**

- Risk: Path traversal via symlinks and cross-drive paths on Windows
- Files: `packages/opencode/src/file/index.ts:284-285`, `packages/opencode/src/file/index.ts:344-345`
- Current mitigation: Lexical path containment check (`Filesystem.contains`)
- Recommendations:
  - Implement realpath canonicalization before containment checks
  - Add symlink resolution and validation
  - Test on Windows for cross-drive path bypasses

### Environment Variable Exposure

**Missing Validation:**

- Risk: Environment variables used without validation may be undefined or malformed
- Files:
  - `packages/enterprise/src/core/storage.ts:67-87` - Storage credentials with `!` assertions
  - `packages/slack/src/index.ts:5-14` - Slack tokens logged to console
- Current mitigation: TypeScript non-null assertions (`!`)
- Recommendations:
  - Add runtime validation with descriptive error messages
  - Remove credential presence logging (even if masked)
  - Use a configuration schema (Zod) to validate at startup

### Server Security

**Unsecured Server Warning:**

- Risk: Server starts without password protection in development
- Files: `packages/opencode/src/cli/cmd/web.ts:37`
- Current mitigation: Console warning only
- Recommendations: Require explicit `--insecure` flag to start without auth

## Performance Bottlenecks

### Large File Processing

**Icon Component:**

- Problem: `packages/web/src/components/icons/index.tsx` (4,454 lines) likely contains all icon definitions inline
- Files: `packages/web/src/components/icons/index.tsx`
- Cause: Bundle size impact; slow initial render
- Improvement path: Implement code-splitting or lazy loading for icons

### Synchronous Operations

**File System Operations:**

- Problem: Synchronous file reads in hot paths
- Files: `packages/opencode/src/file/index.ts` - Various `Bun.file()` and `fs` operations
- Cause: Blocking event loop during file I/O
- Improvement path: Use streaming for large files; add caching layer

### Memory Leaks

**MCP Client Management:**

- Problem: Comments indicate memory leak prevention is needed
- Files: `packages/opencode/src/mcp/index.ts:276,543`
- Cause: Client objects not properly cleaned up
- Current mitigation: Manual cleanup calls
- Recommendations: Implement proper resource disposal with `using` pattern or finalizers

## Fragile Areas

### Error Handling

**Silent Failures:**

- Issue: Many operations catch errors and do nothing
- Files:
  - `packages/opencode/src/file/index.ts:344` - `catch(() => {})` on file operations
  - `packages/opencode/src/auth/index.ts:46` - Empty catch with fallback to empty object
  - `packages/opencode/src/config/config.ts:220,224` - Silent failures on git operations
  - `packages/app/src/utils/speech.ts:105,288,303,317` - Empty catch blocks
  - `packages/ui/src/theme/context.tsx:41,65` - Silent theme loading failures
- Why fragile: Errors swallowed make debugging impossible; partial failures go unnoticed
- Safe modification: Add structured logging at minimum; propagate errors where possible
- Test coverage: Many of these paths are not tested for failure scenarios

### Race Conditions

**MCP Browser Opening:**

- Issue: Comment indicates race condition prevention needed
- Files: `packages/opencode/src/mcp/index.ts:802`
- Why fragile: Timing-dependent behavior may fail on slower systems
- Safe modification: Use proper synchronization primitives (semaphores, locks)
- Test coverage: Race conditions notoriously difficult to test

**File Operations:**

- Issue: Concurrent file reads/writes without locking
- Files: `packages/opencode/src/file/index.ts`
- Why fragile: Corruption possible with parallel session operations
- Safe modification: Implement file-level locking or use atomic operations

### State Management

**Global State:**

- Issue: Heavy use of global state in `Instance` and `Global` namespaces
- Files: `packages/opencode/src/global/index.ts`, `packages/opencode/src/project/instance.ts`
- Why fragile: Side effects are hard to track; test isolation difficult
- Safe modification: Inject dependencies explicitly; use context pattern

## Missing Critical Features

### Permission Persistence

**Unsaved Permission Rules:**

- Issue: Permission ruleset not saved to disk
- Files: `packages/opencode/src/permission/next.ts:223`
- Problem: Users must re-approve permissions on restart
- Blocks: Seamless user experience across sessions

### Centralized Tool Logic

**Scattered Tool Invocation:**

- Issue: Tool invocation logic duplicated across codebase
- Files: `packages/opencode/src/session/prompt.ts:318`
- Problem: Inconsistent behavior; harder to add cross-cutting concerns (logging, metrics)
- Blocks: Adding features like tool-level rate limiting or audit logging

## Test Coverage Gaps

### Untested Error Paths

**Silent Catch Blocks:**

- What's not tested: Error handling in file operations, network requests, external tool calls
- Files: `packages/opencode/src/file/index.ts`, `packages/opencode/src/tool/*.ts`
- Risk: Error handling code may be broken; edge cases unknown
- Priority: High

### Provider Transform Logic

**Model Provider Transformations:**

- What's not tested: Provider-specific parameter transformations
- Files: `packages/opencode/src/provider/transform.ts`
- Risk: Breaking changes to provider APIs cause silent failures
- Priority: Medium

### UI Components

**Complex UI Logic:**

- What's not tested: Session page, layout component, terminal component interactions
- Files: `packages/app/src/pages/session.tsx`, `packages/app/src/pages/layout.tsx`, `packages/app/src/components/terminal.tsx`
- Risk: UI regressions on refactoring
- Priority: Medium

## Dependencies at Risk

### Alpha/Beta Dependencies

**Nitro Framework:**

- Risk: Using alpha version (`3.0.1-alpha.1`)
- Impact: API instability; potential breaking changes
- Migration plan: Monitor for stable release; pin to specific alpha if needed

**@typescript/native-preview:**

- Risk: Preview/experimental package
- Impact: Unstable APIs; may be deprecated
- Migration plan: Track stable release; abstract behind internal API

### Catalog Dependencies

**Workspace Catalog Pattern:**

- Risk: Heavy use of `catalog:` protocol for dependencies
- Impact: Version conflicts between packages; hard to reason about versions
- Migration plan: Document catalog versions; add CI check for version consistency

## Code Quality Issues

### Console Logging in Production

**Debug Logs:**

- Issue: `console.log` and `console.error` statements in production code
- Files:
  - `packages/function/src/api.ts:34,75,183,190,230` - Debug logging
  - `packages/enterprise/src/core/share.ts:88,93,99` - Compaction logging
  - `packages/web/src/components/Share.tsx:72,104,112,117` - WebSocket logging
  - `packages/script/src/index.ts:59` - Script output
- Impact: Log spam in production; potential information disclosure
- Fix approach: Use structured logger with level filtering; remove or guard debug logs

### Type Safety Issues

**Loose Typing:**

- Issue: Use of `any` and `unknown` without proper guards
- Files:
  - `packages/function/src/api.ts:58,93,168,196,363` - `any` usage
  - `packages/sdk/js/src/client.ts:10-11` - `any` in custom fetch
- Impact: Runtime errors not caught by TypeScript
- Fix approach: Replace with proper types or use `unknown` with type guards

### Generated Code Quality

**SDK Client Error Handling:**

- Issue: TODO comments indicate incomplete error handling
- Files: `packages/sdk/js/src/v2/gen/client/client.gen.ts:226`, `packages/sdk/js/src/gen/client/client.gen.ts:172`
- Impact: Error responses may not be properly typed or handled
- Fix approach: Complete error type generation; handle all HTTP error codes

---

_Concerns audit: 2026-01-31_
