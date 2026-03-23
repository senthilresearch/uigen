# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Initial setup (installs deps, generates Prisma client, runs migrations)
npm run setup

# Development
npm run dev          # Start dev server with Turbopack (port 3000 or next available)
npm run stop         # Kill the server on port 3001
npm run restart      # stop + dev
npm run dev:daemon   # Run dev server in background, output to logs.txt
npm run build        # Production build
npm start            # Start production server
npm run lint         # Run ESLint
npm run test         # Run all tests with Vitest

# Database
npm run db:reset     # Reset the SQLite database

# Run a single test file
npx vitest run src/path/to/file.test.ts
```

## Environment Variables

Configured in `.env`:

- `ANTHROPIC_API_KEY` — Optional. When set, uses real Claude (`claude-haiku-4-5`); without it, a `MockLanguageModel` returns static sample components.
- `JWT_SECRET` — Optional. Defaults to `"development-secret-key"` if unset (fine for local dev, set a real secret in production).

## Architecture

UIGen is a Next.js 15 app that uses Claude AI to generate React components with live preview. The core flow:

1. User sends a chat message → `ChatContext` calls `POST /api/chat`
2. API route reconstructs `VirtualFileSystem` from serialized state, then calls Claude via Vercel AI SDK (`streamText`)
3. Claude uses two tools to write code: `str_replace_editor` (create/edit files) and `file_manager` (manage files)
4. Tool calls stream back to the client; `FileSystemContext` handles them, updating the in-memory `VirtualFileSystem`
5. `PreviewFrame` renders `App.jsx` from the VirtualFileSystem inside an iframe using client-side Babel transpilation
6. On stream completion, authenticated users' projects are saved to SQLite via Prisma

### Virtual File System

`src/lib/file-system.ts` — A `VirtualFileSystem` class manages all generated files entirely in memory (no disk writes). It serializes/deserializes to JSON for persistence in the database and transmission via the chat API. The AI always maintains `/App.jsx` as the root entrypoint.

`serialize()` strips the `children: Map` to avoid JSON serialization issues; deserialization rebuilds the tree by sorting paths and creating parents first.

### AI Integration

- **Provider:** `src/lib/provider.ts` — Returns a real Claude model (`claude-haiku-4-5`) when `ANTHROPIC_API_KEY` is set, or a `MockLanguageModel` that generates sample components without an API key. The mock simulates streaming (15–30ms per char) and limits to 4 steps vs 40 for real Claude.
- **System Prompt:** `src/lib/prompts/generation.tsx` — Instructs Claude to use Tailwind CSS, keep `/App.jsx` as root, use `@/` import alias, and only operate through tool calls. Uses Anthropic ephemeral prompt caching (`cacheControl: { type: "ephemeral" }`) to reduce token costs.
- **Tools:** `src/lib/tools/str_replace.ts` (commands: `view`, `create`, `str_replace`, `insert`, `undo_edit`) and `src/lib/tools/file_manager.ts` (commands: `rename`, `delete`). Both are built dynamically with the current `VirtualFileSystem` instance injected. Note: `undo_edit` is listed in the schema but not implemented — it returns an error if called.
- **API Route:** `src/app/api/chat/route.ts` — Max 120s, 10,000 tokens, 40 steps (4 for mock).

### Preview System

`src/lib/transform/jsx-transformer.ts` — Transpiles files client-side using `@babel/standalone`. Local imports (`./`, `/`, `@/`) are converted to blob URLs; third-party packages are mapped to `esm.sh` CDN URLs. CSS imports are stripped and not rendered. Transformation errors are collected per file; errored files are skipped while others render normally.

The iframe uses `sandbox="allow-scripts allow-same-origin allow-forms"` — `allow-same-origin` is required for blob URL imports to work.

### State Management

Two React contexts wire everything together:
- **`FileSystemContext`** (`src/lib/contexts/file-system-context.tsx`) — Holds the `VirtualFileSystem` and processes incoming AI tool calls to mutate it.
- **`ChatContext`** (`src/lib/contexts/chat-context.tsx`) — Wraps Vercel AI SDK's `useChat`, serializes the file system into each API request, and tracks anonymous work.

### Authentication

JWT-based auth (`src/lib/auth.ts`) using the `jose` library with HS256. Tokens stored in HTTP-only cookies (7-day expiration, `SameSite: lax`, `secure` in production). Passwords hashed with bcrypt (10 rounds). Server Actions in `src/actions/index.ts` handle sign-up/sign-in/sign-out.

Anonymous users can generate components; their work is tracked in `sessionStorage` via `src/lib/anon-work-tracker.ts` and can be saved upon registration. `src/middleware.ts` enforces JWT verification on protected API routes (`/api/projects`, `/api/filesystem`).

### Database

Prisma with SQLite (`prisma/dev.db`). Two models: `User` (email + hashed password) and `Project` (chat history + serialized file system as JSON strings, belongs to User with cascade delete). Prisma client is generated to `src/generated/prisma/` (non-standard location configured in `schema.prisma`).

### Routing

- **`/` (home):** Authenticated users are redirected to their most recent project; if none exist, a new project is auto-created first. Unauthenticated users see the landing/chat page.
- **`/[projectId]`:** Loads a specific project for authenticated users; unauthenticated users are redirected to `/`.
- **`src/hooks/use-auth.ts`** — Post-sign-in hook that migrates anonymous work to an authenticated project, then redirects to it.

### UI Layout

`src/app/main-content.tsx` — Three-panel resizable layout: Chat (left, 35%) | Preview/Code tabs (right, 65%). The Code view splits further into FileTree (30%) + Monaco Editor (70%).

### Testing

Vitest with jsdom environment (`vitest.config.mts`). Tests are colocated in `__tests__/` subdirectories next to the source files they test. Path aliases resolved via `vite-tsconfig-paths`.

### Path Alias

`@/*` maps to `./src/*` throughout the codebase.

### Tailwind & Styling

Uses Tailwind CSS v4 with `@theme inline` syntax in `globals.css`. Colors use OKLch color space. Component library is shadcn/ui (style: "new-york", base color: "neutral") with Lucide icons.

### Node.js Compatibility

Build/dev scripts inject `NODE_OPTIONS="--require ./node-compat.cjs"` (see `node-compat.cjs`) to polyfill Web Storage globals that conflict with Node 25+ experimental APIs and break SSR.
