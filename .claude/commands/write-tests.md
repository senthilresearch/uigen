Write tests for the specified file or feature.

The argument is the file or feature to test (e.g. `src/lib/auth.ts` or "the chat API route").

Do the following steps in order:

1. Read the target file(s) to understand the code being tested
2. Check for any existing tests in a `__tests__/` subdirectory next to the source file
3. Identify what to test: exported functions, edge cases, error paths, and key behaviors
4. Write tests in a `__tests__/` directory colocated with the source file, named `<filename>.test.ts`
5. Follow the existing test style in the repo (Vitest, jsdom or node environment as appropriate)
6. Run the new test file with `npx vitest run <path/to/file.test.ts>` and fix any failures
7. Report which cases were covered and any gaps worth noting

## Testing Conventions

- **Framework:** Vitest (`describe`, `test`, `expect`, `vi`)
- **Location:** `src/path/to/__tests__/filename.test.ts`
- **Environment:** Add `// @vitest-environment node` for server-side code, default (jsdom) for components
- **Mocking:** Use `vi.mock(...)` at the top of the file for module-level mocks
- **Server-only modules:** Mock with `vi.mock("server-only", () => ({}))`
- **Next.js headers:** Mock `next/headers` cookies/headers as needed

## What to Cover

- Happy path: normal inputs produce expected outputs
- Edge cases: empty input, boundary values, nulls
- Error paths: thrown errors, rejected promises, invalid args
- Auth/permissions: authenticated vs unauthenticated behavior where relevant
- Focus on testing behaviour and public API's rather than implementation details

## Run Commands

```bash
# Run a single test file
npx vitest run src/path/to/__tests__/file.test.ts

# Run all tests
npm run test

# Run in watch mode
npx vitest src/path/to/__tests__/file.test.ts
```
