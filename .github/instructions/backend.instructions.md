---
name: backend-dev-guide
description: Use whenever the user wants to build, plan, or add the "backend," "server," "API," "database," or "logic" side of a web app or personal project, especially if they are not an experienced developer and don't yet know their requirements or tech stack. Also use whenever backend-related code files are being created, to decide what technology to use, where files should live, and to prevent mixing incompatible or redundant technologies. Trigger even if the user just says something like "I want to build an app that does X" without mentioning "backend" by name.
---

# Backend Development Guide (for non-developers building with AI)

This skill helps someone who is **not a professional developer** plan and build the backend of a web app with Claude or Copilot, without ending up with a messy, duplicated, or mismatched tech stack.

Your job when this skill is active: **think like a calm, experienced engineer keeping the project simple.** Ask before assuming. Pick one stack and stick to it. Never add a tool "just in case."

---

## 0. Where this file goes

This file, on its own, is just a readable reference — it only becomes a skill Claude or Copilot loads *automatically* when it sits in the exact folder and filename each tool expects.

**Claude (Claude Code / Claude Desktop)** — wants the file named `SKILL.md` inside its own folder:

```
your-project/
 └── .claude/
      └── skills/
           └── backend-dev-guide/     ← folder name = the skill's name
                └── SKILL.md          ← rename this file to SKILL.md here
```

**GitHub Copilot** — wants the filename to end in `.instructions.md`, with a small frontmatter block telling it which files to apply to:

```
your-project/
 └── .github/
      └── instructions/
           └── backend.instructions.md   ← rename this file, add:
                                            ---
                                            applyTo: "backend/**"
                                            ---
```

**Just keeping it as `Backend.md`?** That's completely fine too — Claude and Copilot can both still read it if you paste its contents into a chat, or reference it with `@Backend.md`. It just won't auto-load on its own without the exact name/location above. Rename or copy it once you're ready to wire it in.

---

## 1. What is "the backend," in plain terms?

The backend is the part of an app nobody sees directly. It's everything that happens behind the scenes:

- **Logic** — the rules of the app (e.g., "a user can only book one appointment per day")
- **Data** — where information is stored and remembered (a database)
- **API** — the "waiter" that carries requests between the frontend (what people see) and the backend (where the work happens)

If the frontend is the storefront, the backend is the stockroom, the cash register, and the rulebook — combined.

---

## 2. Pick ONE stack — do not mix

Before writing any code, choose a **single language** and stick to its natural ecosystem. Do not combine backend ecosystems (e.g., never run a Python backend and a Node.js backend side by side for the same app).

| If the language is... | Use this framework (pick one) | Use this database (pick one, start simple) |
|---|---|---|
| **Python** | FastAPI (recommended for APIs) or Flask (simpler, smaller apps) | SQLite (simplest, file-based) → PostgreSQL (when it needs to scale) |
| **JavaScript / TypeScript** | Express (simple) or NestJS (larger apps) | SQLite/PostgreSQL via Prisma, or MongoDB if data is very flexible/unstructured |
| **Ruby** | Ruby on Rails | PostgreSQL |
| **Go** | Fiber or Gin | PostgreSQL |

**Default recommendation for beginners:** Python + FastAPI + SQLite. It's readable, well-documented, has excellent AI tooling support, and SQLite needs zero setup.

If the user hasn't chosen a language, ask. Don't guess and don't default silently — the right choice depends on what they already know and what the frontend will be built in.

---

## 3. Keep the build clean: the noise-limiting framework

This is the section that keeps the project from turning into a pile of half-used tools. It's written to work the same way inside **Claude** (Claude Code, Claude Desktop, claude.ai) and **Copilot** (Chat, Edit mode, Agent mode) — both tools generate code eagerly and will happily add a dependency, a config file, or a whole folder structure you never asked for unless something stops them. This section is that stop.

### 3.1 The governing principle

**Every file, package, or dependency that appears in the project must trace back to a specific thing the user asked for.** If you (the AI) can't point to the sentence that justified it, it doesn't belong yet. This applies equally to a single import line and to an entire new framework.

Treat it as a running budget, not a one-time check: revisit it every time you're about to run an install command, scaffold a folder, or generate more than one file at once.

### 3.2 The pre-generation checklist

Before writing or installing anything, silently run through this:

1. **Does this match the chosen language's own ecosystem?** (Section 2's table is the source of truth — a Python backend pulls from PyPI/pip, never npm; a Node backend pulls from npm, never pip.)
2. **Does something like this already exist in the project?** If yes, extend or reuse it — don't create a second version "to be safe."
3. **Did the user ask for this specific capability?** Testing, auth, deployment, logging, caching, background jobs — all genuinely useful, all also the most common source of bulk code nobody asked for in a first build. Add them when requested or when the stated feature truly can't work without them, not by default.
4. **Is this the simplest tool that does the job?** Prefer the standard library or the framework's built-in features over a new dependency when both would work.
5. **Am I generating more files than the user can review in one pass?** If a request would create many files at once, generate the smallest working slice first, show it, then continue — don't front-load the whole build.

### 3.3 Universal noise patterns to refuse by default

These apply across every language, not just the Python/npm case:

- **Cross-ecosystem contamination** — any package manager, config file, or lockfile that belongs to a different language than the one chosen (npm files in a Python project, a `requirements.txt` in a Node project, etc.)
- **Parallel implementations** — a second framework, second ORM, second HTTP client, or second config system installed alongside one that's already doing the job
- **Premature infrastructure** — Docker, CI/CD pipelines, cloud-deploy configs, message queues, caching layers, or microservice splits before there's a single working feature to deploy
- **Speculative scaffolding** — empty folders/files created "for later" (`tests/`, `migrations/`, `admin/`) with no real content yet
- **Unrequested cross-cutting concerns** — authentication, rate limiting, logging frameworks, or monitoring added without being asked, "just in case it's needed"
- **Copy-pasted logic** — the same validation, query, or route handler duplicated across files instead of written once and reused

### 3.4 Using this inside Claude vs. Copilot

- **In Claude Code / Claude Desktop:** when Claude proposes running `pip install`, `npm install`, or creating a new top-level file, pause and ask it to state in one line which requirement that install serves. If it can't, don't run it. This SKILL.md, once placed per Section 0, gets pulled in automatically and Claude should apply this checklist without being asked each time.
- **In Copilot (Chat, Edit, or Agent mode):** review Copilot's proposed diff before accepting — look specifically for new files outside `backend/`, new entries in the dependency file, or a second copy of something that already exists. Copilot's `applyTo` scoping (Section 0) keeps this file's rules attached specifically to files under `backend/**`, so it stays in context while you're working there.
- **In both tools:** if you notice the same noise pattern being suggested more than once, that's a sign to state the constraint explicitly in your prompt (e.g., "Python + FastAPI + SQLite only, no other packages") rather than relying on the AI to infer it every time.

**One language. One framework. One database. Nothing extra.** If you're about to add a tool and can't point to the specific feature the user asked for that needs it, don't add it.

---

## 4. Figure out requirements first — don't assume

Most non-developers starting a project don't know what "requirements" means yet, and that's normal. Before writing code, get plain-language answers to:

1. **What should this app actually do?** (in one or two sentences, no jargon)
2. **What information does it need to remember?** (this becomes your database tables — e.g., "users," "orders," "posts")
3. **Who uses it, and how?** (one person, many people, does it need logins?)
4. **Does it need to talk to anything else?** (payment processor, email, another API)

Turn these answers into a short plan (a few bullet points is enough) before generating code. This prevents the AI from guessing and building the wrong thing, and prevents bulk code no one asked for.

---

## 5. Folder rule — keep backend and frontend separate

All backend code lives in a top-level `backend/` folder. Any time you create, generate, or move a backend file (server code, routes, database models, config for the backend), it belongs in `backend/`, never mixed into `frontend/` or the project root.

```
project-root/
├── backend/
│   ├── main.py          (or app.js, etc. — the entry point)
│   ├── models/          (database tables/structures)
│   ├── routes/           (API endpoints)
│   └── requirements.txt  (or package.json — only the backend's own dependency file)
└── frontend/
    └── ...               (see the frontend-dev-guide skill)
```

If a file is clearly backend logic (handles data, talks to a database, defines an API route), move or create it under `backend/` automatically — don't ask permission for this specific housekeeping step, just do it and mention it happened.

---

## 6. Best practices to follow automatically

- **One entry point.** A single clear file (e.g., `main.py`) that starts the server, not several competing ones.
- **Secrets stay out of code.** API keys, passwords, and database URLs go in a `.env` file, never hard-coded in source files.
- **Name things plainly.** Route and table names should describe what they do (`/users`, `/orders`) not `/data1`, `/temp`.
- **Small, working steps.** Build one working endpoint (e.g., "save a new user") before adding the next feature — don't generate the whole API surface in one shot before anything has been tested.
- **Explain the "why."** When you generate backend code for a non-developer, briefly say what each major piece does in plain language — they're learning, not just receiving code.
- **Match the database to the need.** Don't reach for PostgreSQL or MongoDB for a small personal project that SQLite handles fine. Simpler is better until there's a real reason to scale up.

---

## 7. How the backend talks to the frontend

The backend exposes an **API** — a set of URLs the frontend can send requests to (e.g., `GET /tasks` to fetch a to-do list, `POST /tasks` to add one). The frontend never touches the database directly; it only talks to the API.

When planning, briefly list the API endpoints the frontend will need (in plain language: "a way to add a task, a way to see all tasks, a way to mark one done") — this list becomes the contract between the two sides and keeps both builds focused on exactly what's needed, nothing more.

See the companion **frontend-dev-guide** skill for the other half of this picture.
