---
name: frontend-dev-guide
description: Use whenever the user wants to build, plan, or add the "frontend," "UI," "interface," "pages," "screens," or visual/interactive part of a web app or personal project, especially if they are not an experienced developer and don't yet know their requirements or tech stack. Also use whenever frontend-related code files are being created, to decide what technology to use, where files should live, and to prevent mixing incompatible or redundant technologies. Trigger even if the user just says something like "I want it to look like X" or "I want people to be able to click Y" without mentioning "frontend" by name.
---

# Frontend Development Guide (for new-developers building with AI)

This skill helps someone who is **not a professional developer** plan and build the frontend of a web app with Claude or Copilot, without ending up with a messy, duplicated, or over-engineered interface.

Your job when this skill is active: **think like a calm, experienced engineer keeping the project simple.** Ask before assuming. Pick one stack and stick to it. Never add a framework or library "just in case."

---

## 0. Where this file goes

This file, on its own, is just a readable reference — it only becomes a skill Claude or Copilot loads *automatically* when it sits in the exact folder and filename each tool expects.

**Claude (Claude Code / Claude Desktop)** — wants the file named `SKILL.md` inside its own folder:

```
your-project/
 └── .claude/
      └── skills/
           └── frontend-dev-guide/     ← folder name = the skill's name
                └── SKILL.md           ← rename this file to SKILL.md here
```

**GitHub Copilot** — wants the filename to end in `.instructions.md`, with a small frontmatter block telling it which files to apply to:

```
your-project/
 └── .github/
      └── instructions/
           └── frontend.instructions.md   ← rename this file, add:
                                             ---
                                             applyTo: "frontend/**"
                                             ---
```

**Just keeping it as `Frontend.md`?** That's completely fine too — Claude and Copilot can both still read it if you paste its contents into a chat, or reference it with `@Frontend.md`. It just won't auto-load on its own without the exact name/location above. Rename or copy it once you're ready to wire it in.

---

## 1. What is "the frontend," in plain terms?

The frontend is everything a person sees and interacts with directly in their browser:

- **Structure** — the content and layout (HTML)
- **Style** — colors, fonts, spacing, what makes it look good (CSS)
- **Interactivity** — what happens when you click, type, or scroll (JavaScript)

If the backend is the stockroom and register, the frontend is the storefront — the part customers actually walk through.

---

## 2. Pick ONE stack — do not mix

Before writing any code, choose a single approach based on how interactive the app really needs to be. Do not combine multiple frontend frameworks (e.g., never mix React and Vue in the same project).

| If the app needs... | Use this | Needs npm/Node.js? |
|---|---|---|
| A simple site with a few pages, minimal interactivity (a portfolio, a landing page, a simple form) | Plain **HTML + CSS + JavaScript** | No — no build tools needed at all |
| A single, richly interactive app (dashboards, live updates, lots of dynamic state) | **React** (most common, most AI/tooling support) | Yes |
| Similar to React but the user wants something simpler to read | **Vue** | Yes |
| A content-heavy site (blog, marketing site) that still wants some interactivity | **Astro** or a static site generator | Yes, but minimal |

**Default recommendation for beginners:** if the app is mostly a few pages with a form or two, start with plain HTML/CSS/JS — it's simpler to understand, has zero setup, and nothing to break. Only reach for React/Vue when the app genuinely needs to manage a lot of changing, interactive state (e.g., a live chat, a complex dashboard).

If the user hasn't chosen, ask what the app should feel like (a few static pages vs. an interactive tool) — that answer decides the stack.

---

## 3. Keep the build clean: the noise-limiting framework

This is the section that keeps the project from turning into a pile of half-used tools. It's written to work the same way inside **Claude** (Claude Code, Claude Desktop, claude.ai) and **Copilot** (Chat, Edit mode, Agent mode) — both tools generate code eagerly and will happily add a dependency, a config file, or a whole folder structure you never asked for unless something stops them. This section is that stop.

### 3.1 The governing principle

**Every file, package, or dependency that appears in the project must trace back to a specific thing the user asked for.** If you (the AI) can't point to the sentence that justified it, it doesn't belong yet. This applies equally to a single import line and to an entire new framework.

Treat it as a running budget, not a one-time check: revisit it every time you're about to run an install command, scaffold a folder, or generate more than one file at once.

### 3.2 The pre-generation checklist

Before writing or installing anything, silently run through this:

1. **Does this match the chosen approach's own ecosystem?** (Section 2's table is the source of truth — plain HTML/CSS/JS needs zero npm packages; React/Vue projects use npm, but only for what the app actually needs.)
2. **Does something like this already exist in the project?** If a button, card, or form pattern already exists, reuse or extend it — don't create a near-duplicate.
3. **Did the user ask for this specific capability?** State management, animation libraries, testing setups, custom design systems — all genuinely useful, all also the most common source of bulk code nobody asked for in a first build. Add them when requested or when the stated feature truly can't work without them, not by default.
4. **Is this the simplest tool that does the job?** Prefer plain CSS/JS or the framework's built-in state before reaching for a new library.
5. **Am I generating more files than the user can review in one pass?** If a request would create many pages/components at once, build and show the smallest working slice first, then continue — don't front-load the whole site.

### 3.3 Universal noise patterns to refuse by default

These apply across every stack choice, not just the "no npm for plain HTML" case:

- **Cross-ecosystem contamination** — build tooling (npm, Vite, Webpack) appearing in a project that never chose a framework requiring it
- **Parallel implementations** — a second UI framework, a second styling system (e.g., Tailwind layered on top of hand-written CSS), or a second component library installed alongside one already doing the job
- **Premature infrastructure** — full design systems, component libraries, or automated testing setups scaffolded before a single page is working and reviewed
- **Speculative scaffolding** — empty component/page files created "for later" with no real content yet
- **Unrequested cross-cutting concerns** — analytics scripts, animation libraries, or global state managers added "just in case it's needed," without being asked
- **Copy-pasted UI** — the same button, card, or form markup repeated across multiple files instead of built once and reused

### 3.4 Using this inside Claude vs. Copilot

- **In Claude Code / Claude Desktop:** when Claude proposes running `npm install` or creating a new top-level config file, pause and ask it to state in one line which requirement that install serves. If it can't, don't run it. This SKILL.md, once placed per Section 0, gets pulled in automatically and Claude should apply this checklist without being asked each time.
- **In Copilot (Chat, Edit, or Agent mode):** review Copilot's proposed diff before accepting — look specifically for new files outside `frontend/`, new entries in `package.json`, or a second copy of a component that already exists. Copilot's `applyTo` scoping (Section 0) keeps this file's rules attached specifically to files under `frontend/**`, so it stays in context while you're working there.
- **In both tools:** if you notice the same noise pattern being suggested more than once, that's a sign to state the constraint explicitly in your prompt (e.g., "plain HTML/CSS/JS only, no build tools") rather than relying on the AI to infer it every time.

**One approach. One styling method. One set of dependencies. Nothing extra.** If you're about to add a library and can't point to the specific feature the user asked for that needs it, don't add it.

---

## 4. Figure out requirements first — don't assume

Most non-developers starting a project don't know what "requirements" means yet, and that's normal. Before writing code, get plain-language answers to:

1. **What should it look like?** (simple/clean, colorful, professional, playful — references help: "like Notion," "like a magazine")
2. **What pages or screens does it need?** (a home page, a form, a dashboard — list them by name)
3. **What should happen when someone interacts with it?** (click a button → what happens; submit a form → what happens)
4. **Does it need to remember anything between visits?** (this signals it needs to talk to a backend — see the backend-dev-guide skill)

Turn these answers into a short plan (a list of pages/screens and what each does) before generating code. This prevents bulk, unfocused code and keeps the build matched to what was actually asked for.

---

## 5. Folder rule — keep frontend and backend separate

All frontend code lives in a top-level `frontend/` folder. Any time you create, generate, or move a frontend file (pages, components, styles, client-side scripts), it belongs in `frontend/`, never mixed into `backend/` or the project root.

```
project-root/
├── frontend/
│   ├── index.html        (or App.jsx/App.vue for framework projects)
│   ├── styles/
│   ├── components/        (only if using a framework — reusable UI pieces)
│   └── package.json       (only if the stack actually needs npm)
└── backend/
    └── ...                 (see the backend-dev-guide skill)
```

If a file is clearly frontend (renders UI, handles a click, defines a page layout), move or create it under `frontend/` automatically — don't ask permission for this specific housekeeping step, just do it and mention it happened.

---

## 6. Best practices to follow automatically

- **Reuse, don't repeat.** If the same button, card, or form shape appears more than once, make it one reusable component/snippet instead of copy-pasting.
- **Keep it responsive.** Layouts should work on both a phone-sized and a laptop-sized screen by default.
- **Keep it accessible.** Buttons and links should be real `<button>`/`<a>` elements (not styled `<div>`s), images need alt text, and text needs enough color contrast to read easily.
- **Small, working steps.** Build and show one working page before generating the next five — don't generate the whole site in one shot before anything has been seen or tested.
- **Explain the "why."** When you generate frontend code for a non-developer, briefly say what each major piece does in plain language — they're learning, not just receiving code.
- **Match effort to need.** A simple form doesn't need animation libraries, custom fonts pulled from five sources, or a component for every possible edge case.

---

## 7. How the frontend talks to the backend

The frontend never touches a database directly. When it needs to save or fetch information, it sends a request to the backend's **API** (e.g., "get all tasks," "add a new task") and displays whatever comes back.

When planning, briefly list what data the frontend needs from the backend and when (on page load? after a button click?) — this keeps the frontend build focused on exactly what's needed, nothing more.

See the companion **backend-dev-guide** skill for the other half of this picture.
