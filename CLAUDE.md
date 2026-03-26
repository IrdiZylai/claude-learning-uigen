# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UIGen is an AI-powered React component generator with live preview. Users describe a component, Claude AI generates it using a virtual file system, and a real-time iframe preview renders it.

## Commands

```bash
npm run setup     # First-time setup: install deps + prisma generate + db migrate
npm run dev       # Dev server with Turbopack at http://localhost:3000
npm run build     # Production build
npm run start     # Run production server
npm test          # Run Vitest tests
npm run lint      # Run ESLint
npm run db:reset  # Destructively reset SQLite database
```

## Architecture

### Virtual File System (`src/lib/file-system.ts`)
All generated code lives in an in-memory virtual FS — no disk writes. Files are serialized to JSON and stored in SQLite. The AI entrypoint is always `/App.jsx`. All paths use `/` separators.

### AI Integration (`src/app/api/chat/route.ts`)
Uses Vercel AI SDK with streaming. Two modes:
- **Real**: Anthropic Claude (requires `ANTHROPIC_API_KEY` in `.env`)
- **Mock**: `MockLanguageModel` is used automatically when no API key is present — returns static components (Counter, ContactForm, Card), useful for development

Claude uses two tools defined in `src/lib/tools/`:
- `str_replace_editor` — view, create, str_replace, insert in files
- `file_manager` — rename or delete files

The agentic loop runs up to 40 steps (4 for mock), with Claude making tool calls and iterating on results.

### State Management
Two React Contexts in `src/lib/contexts/`:
- **ChatContext** (`chat-context.tsx`) — messages, input state, chat submission via Vercel AI SDK
- **FileSystemContext** (`file-system-context.tsx`) — virtual FS state, file operations, project hydration from DB

### Component Preview (`src/components/preview/PreviewFrame.tsx`)
Renders in an isolated iframe. Babel Standalone transpiles JSX at runtime; Tailwind CSS is loaded via CDN. No SSR.

### Auth (`src/lib/auth.ts`)
JWT sessions (HS256, 7-day). httpOnly + sameSite=lax cookies. bcrypt for passwords. Anonymous users are supported — projects can exist without a user.

### Data Persistence
Projects store `messages` (JSON-serialized chat history) and `data` (JSON-serialized virtual FS) in SQLite via Prisma. Schema at `prisma/schema.prisma`; Prisma client generated to `src/generated/prisma`.

## Key File Locations

| Purpose | Path |
|---|---|
| AI chat endpoint | `src/app/api/chat/route.ts` |
| Main UI layout (resizable panels) | `src/app/main-content.tsx` |
| Virtual file system | `src/lib/file-system.ts` |
| Language model selection | `src/lib/provider.ts` |
| AI system prompt | `src/lib/prompts/generation.tsx` |
| Server actions (auth, projects) | `src/actions/` |

## Conventions

- `"use client"` directive required for all interactive components
- Internal imports use `@/` alias (maps to `src/`)
- Server actions in `src/actions/` return `{ success, error? }` or data
- Tests colocated in `__tests__/` subdirectories next to source files
- shadcn/ui (new-york style, Tailwind v4) for all UI components
