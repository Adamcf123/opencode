# Testing Patterns

**Analysis Date:** 2026-01-31

## Test Framework

**Runner:**

- **Bun Test** - Native Bun test runner (`bun:test`)
  - Config: No separate config file; uses Bun defaults
  - Fast execution with native TypeScript support

**Assertion Library:**

- Built-in Bun assertions via `bun:test`
- Matchers: `expect().toBe()`, `toEqual()`, `toContain()`, `toHaveBeenCalledTimes()`, `toBeGreaterThan()`, etc.

**Run Commands:**

```bash
# Run all tests (from package directory)
bun test

# Run specific test file
bun test test/tool/tool.test.ts

# Run with coverage
bun test --coverage
```

## Test File Organization

**Location:**

- Unit tests: `test/` directory within package (e.g., `/packages/opencode/test/`)
- E2E tests: `e2e/` directory (e.g., `/packages/app/e2e/`)
- Co-located with source: Some tests in `src/` alongside implementation

**Naming:**

- `*.test.ts` - Unit/integration tests
- `*.spec.ts` - E2E tests (Playwright convention)

**Structure:**

```
packages/opencode/
├── test/
│   ├── fixture/          # Test fixtures and utilities
│   │   └── fixture.ts    # tmpdir() helper
│   ├── util/             # Tests for utility functions
│   │   ├── format.test.ts
│   │   ├── lazy.test.ts
│   │   └── iife.test.ts
│   ├── tool/             # Tests for tool implementations
│   │   ├── grep.test.ts
│   │   ├── registry.test.ts
│   │   └── question.test.ts
│   └── ...
├── src/
│   └── util/
│       ├── format.ts     # Source file
│       └── format.test.ts  # Or co-located test

packages/app/
├── e2e/                  # Playwright E2E tests
│   ├── fixtures.ts       # Custom test fixtures
│   ├── utils.ts          # Test utilities
│   └── *.spec.ts         # E2E test files
```

## Test Structure

**Suite Organization:**

```typescript
import { describe, expect, test } from "bun:test"

describe("namespace.component", () => {
  describe("functionName", () => {
    test("should do something specific", () => {
      expect(result).toBe(expected)
    })

    test("should handle edge case", () => {
      expect(edgeCase).toEqual([])
    })
  })
})
```

**Patterns:**

1. **Descriptive test names** - "should [behavior] when [condition]"
2. **Multiple assertions per test** for related checks
3. **Grouped by functionality** using nested `describe`
4. **Setup/Teardown:**
   - Use `beforeEach`/`afterEach` for per-test setup
   - Use `afterAll` for cleanup after all tests

**Example:**

```typescript
describe("core.storage", () => {
  test("should list files with after and before range", async () => {
    // Setup
    await Storage.write(["test", "users", "user1"], { name: "user1" })

    // Execute
    const result = await Storage.list({ prefix: ["test", "users"], after: "user2", before: "user4" })

    // Assert
    expect(result).toEqual([["test", "users", "user3"]])
  })

  afterAll(async () => {
    // Cleanup
    const testFiles = await Storage.list({ prefix: ["test"] })
    for (const file of testFiles) {
      await Storage.remove(file)
    }
  })
})
```

## Mocking

**Framework:** Built-in Bun spies via `bun:test`

**Patterns:**

```typescript
import { describe, expect, test, spyOn, beforeEach, afterEach } from "bun:test"

describe("tool.question", () => {
  let askSpy: any

  beforeEach(() => {
    askSpy = spyOn(QuestionModule.Question, "ask").mockImplementation(async () => {
      return []
    })
  })

  afterEach(() => {
    askSpy.mockRestore()
  })

  test("should successfully execute with valid parameters", async () => {
    askSpy.mockResolvedValueOnce([["Red"]])
    const result = await tool.execute({ questions }, ctx)
    expect(askSpy).toHaveBeenCalledTimes(1)
  })
})
```

**What to Mock:**

- External service calls
- Module dependencies in unit tests
- User input prompts

**What NOT to Mock:**

- File system operations (use tmpdir fixtures)
- Internal data structures
- Actual implementation logic in integration tests

## Fixtures and Factories

**Test Data:**

```typescript
// Using the tmpdir fixture pattern
import { tmpdir } from "../fixture/fixture"

test("loads tools from directory", async () => {
  await using tmp = await tmpdir({
    init: async (dir) => {
      const toolDir = path.join(dir, ".opencode", "tool")
      await fs.mkdir(toolDir, { recursive: true })
      await Bun.write(path.join(toolDir, "hello.ts"), `export default { execute: async () => 'hello' }`)
    },
  })

  await Instance.provide({
    directory: tmp.path,
    fn: async () => {
      const ids = await ToolRegistry.ids()
      expect(ids).toContain("hello")
    },
  })
})
```

**Location:**

- `/packages/opencode/test/fixture/fixture.ts` - tmpdir helper
- `/packages/app/e2e/utils.ts` - E2E test utilities
- `/packages/app/e2e/fixtures.ts` - Playwright fixtures

**tmpdir Pattern:**

```typescript
export async function tmpdir<T>(options?: TmpDirOptions<T>) {
  const dirpath = sanitizePath(path.join(os.tmpdir(), "opencode-test-" + Math.random().toString(36).slice(2)))
  await fs.mkdir(dirpath, { recursive: true })

  // Optional git init
  if (options?.git) {
    await $`git init`.cwd(dirpath).quiet()
    await $`git commit --allow-empty -m "root commit"`.cwd(dirpath).quiet()
  }

  // Returns disposable resource
  const result = {
    [Symbol.asyncDispose]: async () => {
      // Cleanup
    },
    path: realpath,
    extra: extra as T,
  }
  return result
}
```

## Coverage

**Requirements:** No explicit coverage target enforced

**View Coverage:**

```bash
bun test --coverage
```

## Test Types

**Unit Tests:**

- Scope: Single function or module
- Location: `/test/` directory
- Example: `/packages/opencode/test/util/lazy.test.ts`
- Pattern: Test pure functions with various inputs

**Integration Tests:**

- Scope: Multiple modules working together
- Location: `/test/` directory
- Example: `/packages/opencode/test/tool/registry.test.ts`
- Pattern: Use `Instance.provide()` for dependency injection context

**E2E Tests:**

- Framework: **Playwright**
- Config: `/packages/app/playwright.config.ts`
- Location: `/packages/app/e2e/`
- Run commands:

```bash
# Run all E2E tests
cd packages/app && bun test

# Run with UI
cd packages/app && playwright test --ui

# Show report
cd packages/app && playwright show-report e2e/playwright-report
```

- **Configuration:**
  - Timeout: 60s per test
  - Expect timeout: 10s
  - Retries: 2 in CI, 0 locally
  - Reporter: HTML + line
  - Browser: Chromium only

**E2E Pattern:**

```typescript
import { test, expect } from "./fixtures"

test("can set a default server on web", async ({ page, gotoSession }) => {
  await gotoSession()

  const status = page.getByRole("button", { name: "Status" })
  await expect(status).toBeVisible()

  // Interactions...
})
```

**Custom Fixtures:**

```typescript
export const test = base.extend<TestFixtures, WorkerFixtures>({
  directory: [
    async ({}, use) => {
      const directory = await getWorktree()
      await use(directory)
    },
    { scope: "worker" },
  ],
  gotoSession: async ({ page, directory }, use) => {
    const gotoSession = async (sessionID?: string) => {
      await page.goto(sessionPath(directory, sessionID))
      await expect(page.locator(promptSelector)).toBeVisible()
    }
    await use(gotoSession)
  },
})
```

## Common Patterns

**Async Testing:**

```typescript
test("should handle async operations", async () => {
  const result = await asyncFunction()
  expect(result).toBe(expected)
})
```

**Error Testing:**

```typescript
// Test error handling
try {
  await shouldThrow()
  expect(true).toBe(false) // Should not reach here
} catch (e: any) {
  expect(e).toBeInstanceOf(Error)
  expect(e.cause).toBeInstanceOf(z.ZodError)
}

// Or using rejects
await expect(promise).rejects.toBeInstanceOf(Error)
```

**Generator/Stream Testing:**

```typescript
test("should yield values", async () => {
  const values = []
  for await (const value of asyncGenerator()) {
    values.push(value)
  }
  expect(values).toHaveLength(3)
})
```

**Snapshot Testing:**

```typescript
// Available in bun:test
expect(result).toMatchSnapshot()
```

**DI Context Testing:**

```typescript
// For tests requiring Instance context
await Instance.provide({
  directory: tmp.path,
  fn: async () => {
    // Test code with full DI context
    const result = await ToolRegistry.ids()
    expect(result).toContain("expected")
  },
})
```

**File System Testing:**

```typescript
// Create temporary files
await using tmp = await tmpdir({
  init: async (dir) => {
    await Bun.write(path.join(dir, "test.txt"), "content")
  },
})

// tmp auto-disposes after test
```

**Regex Testing:**

```typescript
describe("CRLF regex handling", () => {
  test("regex correctly splits Unix line endings", () => {
    const unixOutput = "file1.txt|1|content1\nfile2.txt|2|content2"
    const lines = unixOutput.trim().split(/\r?\n/)
    expect(lines.length).toBe(2)
  })
})
```

## Test Utilities

**Context Mock:**

```typescript
const ctx = {
  sessionID: "test",
  messageID: "",
  callID: "",
  agent: "test-agent",
  abort: AbortSignal.any([]),
  messages: [],
  metadata: () => {},
  ask: async () => {},
}
```

**Test Project Root:**

```typescript
const projectRoot = path.join(__dirname, "../..") // For loading real files
```

## CI/CD Integration

**Husky Pre-push:**

- Runs `bun typecheck` before push
- Validates Bun version matches `packageManager` field

**Turbo Tasks:**

```json
{
  "opencode#test": {
    "dependsOn": ["^build"],
    "outputs": []
  },
  "@opencode-ai/app#test": {
    "dependsOn": ["^build"],
    "outputs": []
  }
}
```

---

_Testing analysis: 2026-01-31_
