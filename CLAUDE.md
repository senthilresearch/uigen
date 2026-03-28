# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**UIGen** is an AI-powered React component generator with live preview. Users describe components in natural language; Claude generates, previews, and iteratively refines them in real-time. Supports both authenticated accounts (project persistence) and anonymous sessions.

## Code Style

- Only comment complex code — avoid obvious or redundant comments.

## Commands

```bash
# Initial setup (install deps + generate Prisma client + run migrations)
npm run setup

# Development server (Turbopack)
npm run dev

# Production build
npm run build

# Linting
npm run lint

# Run all tests
npm test

# Reset database
npm run db:reset
```

To run a single test file:
```bash
npx vitest run src/path/to/__tests__/file.test.ts
```

## Environment Variables

```
ANTHROPIC_API_KEY=your-key   # If absent, mock provider generates static components
JWT_SECRET=your-secret       # Defaults to "development-secret-key"
```

## Architecture

### Key Technologies

- **Next.js 15** (App Router) + **React 19** + **TypeScript**
- **Prisma** + **SQLite** for persistence
- **Vercel AI SDK** + **@ai-sdk/anthropic** for streaming Claude responses
- **Babel Standalone** for client-side JSX transpilation
- **Monaco Editor** for code display
- **Vitest** + **jsdom** + **@testing-library/react** for tests

### Core Data Flow

1. User sends message → `POST /api/chat` → Claude streams response with tool calls
2. Claude uses two tools: `str_replace_editor` (create/view/edit files) and `file_manager` (rename/delete)
3. Tool calls update the **VirtualFileSystem** (in-memory, serialized as JSON in the DB)
4. `PreviewFrame` transpiles JSX via Babel and renders in a sandboxed iframe
5. State persisted to DB (`project.data` = serialized VFS, `project.messages` = chat history)

### State Management

- **`ChatContext`** (`src/lib/contexts/chat-context.tsx`): wraps AI SDK's `useChat`, manages message history
- **`FileSystemContext`** (`src/lib/contexts/file-system-context.tsx`): owns VirtualFileSystem instance, handles tool call execution
- No external state library — React context + server actions

### AI Tool System

Claude is given two tools (defined in `src/lib/tools/`):
- `str_replace_editor`: `view`, `create`, `str_replace`, `insert` operations on virtual files
- `file_manager`: `rename`, `delete` operations

The system prompt (`src/lib/prompts/generation.tsx`) instructs Claude to generate React components as separate files in the virtual FS. Agentic loop max: 40 steps (4 for mock).

### Authentication

- JWT sessions (7-day expiration) stored in HTTP-only cookies via `jose`
- `src/lib/auth.ts` — session creation/verification
- `src/middleware.ts` — protects API routes
- Passwords hashed with bcrypt (10 rounds)
- Anonymous users: localStorage session tracking via `src/lib/anon-work-tracker.ts`

### Virtual File System

`src/lib/file-system.ts` — `VirtualFileSystem` class with create/read/update/delete/rename, path normalization, serialization. Persisted as JSON in `project.data`.

### Database Schema

> Always reference `prisma/schema.prisma` for the authoritative database schema before making assumptions about models, fields, or relations.

```prisma
model User  { id, email (unique), password, projects[] }
model Project { id, name, userId (nullable), messages (JSON), data (JSON) }
```

### Key Files

| File | Purpose |
|------|---------|
| `src/app/api/chat/route.ts` | Claude streaming endpoint |
| `src/lib/file-system.ts` | VirtualFileSystem implementation |
| `src/lib/contexts/file-system-context.tsx` | VFS state + tool execution |
| `src/lib/contexts/chat-context.tsx` | Chat state |
| `src/lib/provider.ts` | Real/mock Claude model init |
| `src/components/preview/PreviewFrame.tsx` | JSX preview renderer |
| `src/lib/prompts/generation.tsx` | System prompt for generation |
| `src/lib/tools/str-replace.ts` | Tool definitions |
