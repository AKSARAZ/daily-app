# AGENTS.md — Daily App

This file governs how AI agents operate within this project.
Read this file in full before planning or executing any task.

---

## Project Overview

**Name:** Daily App
**Type:** Personal productivity web application (local, single-user)
**Stack:** Next.js 16 · TypeScript · Tailwind CSS v4 · shadcn/ui (Radix, Nova preset) · Prisma 7 · SQLite
**Runtime:** Node.js 24 · npm 11
**Dev command:** `npm run dev` → runs at `http://localhost:3000`

### Features
1. **Task Management** — Daily priority-based to-do list with daily review ritual
2. **Finance Tracker** — Income/expense tracker with manual and recurring transactions

These two features are **intentionally separate** — no cross-feature data merging.

---

## Architecture

```
src/
├── app/                  # Next.js App Router (pages and API routes)
│   ├── layout.tsx        # Root layout with sidebar
│   ├── page.tsx          # Dashboard / home
│   ├── tasks/
│   │   └── page.tsx
│   └── finance/
│       └── page.tsx
│   └── api/
│       ├── tasks/
│       │   └── route.ts
│       └── finance/
│           └── route.ts
├── components/
│   ├── ui/               # shadcn/ui auto-generated components (DO NOT manually edit)
│   ├── tasks/            # Task-specific components
│   ├── finance/          # Finance-specific components
│   └── layout/           # Sidebar, navbar, shell components
├── lib/
│   ├── prisma.ts         # Prisma client singleton
│   └── utils.ts          # Shared utility functions
prisma/
├── schema.prisma         # Database schema
└── dev.db                # SQLite file (auto-generated, never commit)
```

---

## Database Schema

```prisma
model Task {
  id          String   @id @default(cuid())
  title       String
  priority    String   // "high" | "medium" | "low"
  status      String   // "todo" | "done"
  targetDate  DateTime
  tag         String?
  createdAt   DateTime @default(now())
}

model Transaction {
  id          String    @id @default(cuid())
  type        String    // "income" | "expense"
  amount      Float
  categoryId  String
  note        String?
  date        DateTime
  isRecurring Boolean   @default(false)
  createdAt   DateTime  @default(now())
  category    Category  @relation(fields: [categoryId], references: [id])
}

model Category {
  id           String        @id @default(cuid())
  name         String
  type         String        // "income" | "expense"
  transactions Transaction[]
}

model RecurringTemplate {
  id          String  @id @default(cuid())
  name        String
  amount      Float
  categoryId  String
  dayOfMonth  Int
}
```

---

## Design System

### Theme
- **Mode:** Light and dark mode supported via `next-themes`. Default to system preference.
- **Font:** Geist (already configured by Nova preset)
- **Style:** Modern clean — subtle cards, soft shadows, clear hierarchy, generous whitespace
- **Primary color:** To be defined in `globals.css` CSS variables. Use a neutral-leaning slate/indigo palette.

### Icons
- **NEVER use emoji as icons anywhere in the codebase.**
- Always use SVG-based icons from `lucide-react` (already included in Nova preset).
- Import icons individually: `import { CheckCircle } from "lucide-react"`

### Animations
- Use **subtle, purposeful transitions only** — no flashy or distracting animations.
- Hover states: `transition-colors duration-150` or `transition-all duration-200`
- Page/component entry: use Tailwind's `animate-in fade-in` from `tw-animate-css`
- Interactive elements (buttons, cards): subtle scale or shadow shift on hover
- Never animate layout shifts or cause CLS (Cumulative Layout Shift)

### Layout
- **Sidebar navigation** on the left — fixed, not collapsible for now
- Main content area on the right
- Sidebar contains: app name/logo, nav links (Dashboard, Tasks, Finance), theme toggle
- Responsive behavior: sidebar collapses to icon-only on smaller screens (future consideration)

---

## Coding Standards

### General
- All code in **TypeScript** — no `any` types unless absolutely unavoidable, and must be commented why
- Use `async/await` — no `.then()` chaining
- All API routes use Next.js Route Handlers (`route.ts`), not Pages Router
- Keep components small and single-responsibility
- Co-locate component-specific types within the component file unless shared

### Naming
- Components: `PascalCase`
- Files: `kebab-case`
- Variables/functions: `camelCase`
- Database fields: `camelCase` in Prisma schema, snake_case not allowed

### Prisma
- Always use the singleton client from `lib/prisma.ts`
- **Never run `prisma migrate reset` without explicit user approval**
- **Never run `prisma db push --force-reset` without explicit user approval**
- Migration command: `npx prisma migrate dev --name <descriptive-name>`
- After schema changes, always run `npx prisma generate`

### API Routes
- Return consistent JSON shape: `{ data, error, message }`
- Use proper HTTP status codes
- Validate inputs before hitting the database

---

## Agent Workflow Protocol

### Before Starting Any Task
1. Read this file in full
2. Read the relevant source files before editing them
3. State your implementation plan as an **artifact (plan)** before writing any code
4. Wait for confirmation if the task involves: database migrations, deleting files, or installing new packages

### Task Execution Order
For any feature implementation, always follow this order:
1. Schema changes (if needed) → migrate → generate
2. API route
3. Server-side logic / data fetching
4. UI components
5. Integration and wiring
6. Browser verification

### Artifact Policy
- **Always create a Walkthrough artifact** before making final commits on any non-trivial change
- **Always create an Implementation Plan artifact** for tasks that touch more than one file
- Log any unexpected behavior encountered during execution in a **Diagnostic artifact**

### Browser Verification
- After any UI change, verify in browser at `http://localhost:3000`
- Check both **light mode and dark mode**
- Check that no emoji appear anywhere in the UI
- Check that transitions/animations are present and subtle

### Prohibited Without Explicit Approval
- Running database migrations
- Installing new npm packages
- Deleting any file
- Modifying `prisma/schema.prisma` without stating the change in a plan first
- Modifying `components/ui/` (shadcn auto-generated files)

---

## Environment

- `.env` contains `DATABASE_URL` — never commit this file
- `.env` is already in `.gitignore` — verify before any push
- SQLite database file is at `prisma/dev.db` — never commit this file, add to `.gitignore` if not present

---

## What This App Is Not

- Not a multi-user app — no authentication, no sessions, no roles
- Not a cloud app — no external database, no deployment target
- Not a mobile app — desktop browser only for now
- The two features (Tasks and Finance) are intentionally decoupled — do not create shared state or data dependencies between them