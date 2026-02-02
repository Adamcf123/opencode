# Coding Conventions

**Analysis Date:** 2026-01-31

## Naming Patterns

**Files:**

- Lowercase with hyphens for multi-word files (e.g., `message-v2.ts`, `apply-patch.ts`)
- Test files: `[name].test.ts` alongside source or in `test/` directory
- Config files: Use `.config.ts` suffix (e.g., `vite.config.ts`)

**Functions:**

- camelCase for functions and methods
- Single word preferred when possible (e.g., `fn`, `iife`, `lazy`)
- Verb-noun pattern for actions (e.g., `formatDuration`, `filterCompacted`)

**Variables:**

- camelCase for variables
- Single word preferred (e.g., `input`, `result`, `config`)
- Boolean flags use positive naming (e.g., `loaded`, `active`)

**Types:**

- PascalCase for interfaces, types, and namespaces
- Namespace pattern for domain entities (e.g., `MessageV2`, `Tool`, `Log`)
- Type suffix pattern: `Info` for entity types (e.g., `Tool.Info`, `MessageV2.Info`)

**Constants:**

- Upper camel case for enum-like objects (e.g., `Level`, `Status`)

## Code Style

**Formatting:**

- Prettier configuration in root `package.json`:
  - `semi: false` - No semicolons
  - `printWidth: 120` - 120 character line width

**Linting:**

- ESLint via `/home/adam/projects/opencode/sdks/vscode/eslint.config.mjs`:
  - `@typescript-eslint/naming-convention` - Import naming rules
  - `curly: warn` - Require curly braces
  - `eqeqeq: warn` - Require strict equality
  - `no-throw-literal: warn` - No throwing literals
  - `semi: warn` - Semicolon consistency

**Import Organization:**

1. External dependencies (e.g., `zod`, `ulid`, `ai`)
2. Workspace packages (e.g., `@opencode-ai/util/error`)
3. Internal absolute imports (e.g., `@/bus`, `@/config`)
4. Internal relative imports (e.g., `../util/log`)
5. Type-only imports marked with `type` keyword

**Path Aliases:**

- `@/*` - Maps to `./src/*` in `packages/opencode`
- `@tui/*` - Maps to `./src/cli/cmd/tui/*`
- Workspace packages use full package name: `@opencode-ai/sdk`

## Error Handling

**Patterns:**

1. **NamedError pattern** - Typed errors with Zod schemas:

```typescript
export const OutputLengthError = NamedError.create("MessageOutputLengthError", z.object({}))
export const AbortedError = NamedError.create("MessageAbortedError", z.object({ message: z.string() }))
```

2. **fn() wrapper** - Input validation with Zod:

```typescript
export const get = fn(z.object({ sessionID: z.string(), messageID: z.string() }), async (input) => {
  /* implementation */
})
```

3. **Error cause propagation** - Preserve error chains:

```typescript
return new MessageV2.AuthError({ providerID: ctx.providerID, message: e.message }, { cause: e }).toObject()
```

4. **Result pattern** - Use Zod schema validation before execution

## Logging

**Framework:** Custom `Log` namespace in `/home/adam/projects/opencode/packages/opencode/src/util/log.ts`

**Patterns:**

```typescript
const log = Log.create({ service: "my-service" })
log.info("message", { extra: "data" })
log.error("failed", { error })

// Timed operations
using timer = log.time("operation")
```

**Levels:** DEBUG, INFO, WARN, ERROR (configurable per environment)

## Comments

**When to Comment:**

- JSDoc/TSDoc for exported APIs
- Inline comments for complex logic or workarounds
- `@deprecated` tags for deprecated code
- `@ts-expect-error` with explanation for intentional type overrides

**Style Guide:**

- Comments above the code they describe
- Use `//` for single-line comments
- Avoid `/* */` block comments

## Function Design

**Size:** Keep functions under 50 lines; large files should be split

**Parameters:**

- Prefer single object parameter for multiple arguments
- Use Zod schemas for validation
- Destructuring discouraged - preserve context with `obj.property`

**Return Values:**

- Explicit return types on exported functions
- Use `Result` pattern or typed errors for error cases

**Control Flow:**

- Avoid `let` statements - prefer `const` with ternary
- Avoid `else` statements - use early returns
- Prefer early returns over nested conditionals

```typescript
// Good
const foo = condition ? 1 : 2

// Bad
let foo
if (condition) foo = 1
else foo = 2
```

```typescript
// Good
function foo() {
  if (condition) return 1
  return 2
}

// Bad
function foo() {
  if (condition) return 1
  else return 2
}
```

## Module Design

**Exports:**

- Named exports preferred over default exports
- Namespace pattern for domain modules:

```typescript
export namespace Tool {
  export interface Info {
    /* ... */
  }
  export function define() {
    /* ... */
  }
}
```

**Barrel Files:** Not heavily used; prefer direct imports

**File Structure:**

- Namespace-based organization (e.g., `Tool.define()`, `Session.create()`)
- Feature-based directory structure
- Co-locate related types and implementations

## TypeScript Conventions

**Type Inference:**

- Rely on type inference when possible
- Avoid explicit type annotations unless needed for exports or clarity
- Use `satisfies` operator for type checking without widening

**Zod Integration:**

- All runtime validation uses Zod schemas
- Infer types from schemas: `z.infer<typeof Schema>`
- Add `.meta()` for OpenAPI/JSON Schema generation

**Configuration:**

- Base: `@tsconfig/bun/tsconfig.json`
- JSX: `preserve` with `@opentui/solid` import source
- Path mapping: `@/*` → `./src/*`

## Architecture Patterns

**Dependency Injection:**

```typescript
// Use App.provide() or Instance.provide() for DI context
await Instance.provide({
  directory: tmp.path,
  fn: async () => {
    // Code with DI context
  },
})
```

**Namespace Pattern:**

```typescript
export namespace Domain {
  // Types
  export interface Info {}

  // Functions
  export const get = fn(schema, async (input) => {})

  // Events
  export const Event = {
    Updated: BusEvent.define("domain.updated", schema),
  }
}
```

**Event Bus:**

```typescript
// Define events with Zod schemas
export const Event = BusEvent.define(
  "event.name",
  z.object({
    /* schema */
  }),
)
```

---

_Convention analysis: 2026-01-31_
