# Implementation Plan: CA Buddy tax guidance chat

**Branch**: `001-ca-buddy-chat` | **Date**: 2026-09-09 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `specs/001-ca-buddy-chat/spec.md`

## Summary

Build a single-screen assistant for Indian small-business owners seeking a fast,
plain-language first explanation of everyday GST, TDS, ITR deadlines, and audit basics.
The browser keeps the ordered message list as transient conversation context, routes
questions through a replaceable ChatService contract, and gives a consult-a-CA fallback
for unsupported or individualized requests. The delivery design uses a fake model for
unit tests, intercepted provider requests for end-to-end tests, and a deployment gate
that runs only after both suites pass.

## Technical Context

**Language/Version**: TypeScript 5.x with Node.js 22.22.2 and npm 10.9.7; exact
dependency versions are pinned in `package-lock.json`

**Primary Dependencies**: React, Vite, `@langchain/google-genai`, `@langchain/core`,
Vitest, Testing Library, `jsdom`, and Playwright Test; compatible versions are pinned
in `package-lock.json`

**Storage**: None; the current message list is in-memory browser state and no history
is persisted

**Testing**: Vitest and Testing Library unit tests with a fake ChatService; Playwright
Test end-to-end tests with Gemini requests intercepted; Chromium desktop and mobile
viewport projects; browser binaries are pinned by the Playwright package lock

**Target Platform**: Modern browsers covered by the current Vite Baseline Widely
Available production target, served as a static site from GitHub Pages

**Project Type**: Frontend-only single-page web application

**Performance Goals**: At least 95% of supported questions display a readable answer
within 10 seconds under normal provider availability; the initial screen must be
usable within the first viewport

**Constraints**: One screen and one active conversation; no login, backend, server,
database, browser persistence, tax filing, or submission; direct provider access uses
`VITE_GOOGLE_API_KEY`; model output is capped at 512 tokens; deployment is gated by
unit and end-to-end tests

**Scale/Scope**: One transient conversation per browser session, one user-facing screen,
four supported topic areas, Chromium desktop/mobile acceptance coverage, and five ordered
delivery tasks; no server-side scale target

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- I. Defined Scope and Plain-Language Guidance: PASS. The spec limits answers to GST,
  TDS, ITR deadlines, and audit basics, and requires a consult-a-CA fallback for
  unsupported or individualized questions.
- II. Conversation-Local Privacy: PASS. Messages are transient, follow-ups use the
  current exchange, New chat clears context, and no history is retained.
- III. Contract-Separated Model Access: PASS. The design uses a ChatService boundary,
  an approved Gemini implementation, a fake test implementation, the approved model,
  the 512-token cap, and `VITE_GOOGLE_API_KEY` without hard-coded credentials.
- IV. Testable Delivery: PASS. Unit tests cover UI and service contracts; intercepted
  Playwright tests cover browser workflows; deployment waits for both suites.
- V. Single-Screen Simplicity: PASS. The chosen structure is a frontend-only Vite
  application with the required header, chat panel, input, New chat action, and
  disclaimer, without excluded product surfaces.
- Workflow and quality gates: PASS. The plan follows exactly five ordered tasks, one
  issue and one pull request per task, with deployment last and gated on all tests.

No constitution violations require a complexity exception.

### Phase 1 Re-check

- I. Defined Scope and Plain-Language Guidance: PASS. The data model and service
  contract preserve the supported-topic boundary and consult-a-CA fallback.
- II. Conversation-Local Privacy: PASS. The data model has no persistence field or
  storage adapter; reset returns the conversation to an empty state.
- III. Contract-Separated Model Access: PASS. The service contract isolates the UI from
  the mandated Gemini implementation and accepts ordered current-session context.
- IV. Testable Delivery: PASS. Contract examples and quickstart scenarios cover fake
  unit calls, intercepted provider calls, context, errors, reset, and deployment gates.
- V. Single-Screen Simplicity: PASS. The structure contains one frontend and no backend,
  database, login, settings, or saved-history surface.
- Workflow and quality gates: PASS. The quickstart and CI design keep deployment behind
  both required test jobs.

Phase 1 introduces no new violation or complexity exception.

## Project Structure

### Documentation (this feature)

```text
specs/001-ca-buddy-chat/
├── plan.md              # This implementation plan
├── research.md          # Phase 0 decisions and resolved unknowns
├── data-model.md        # Phase 1 transient conversation model
├── quickstart.md        # Phase 1 local and CI validation guide
├── contracts/
│   └── chat-service.md  # ChatService request/response contract
└── tasks.md             # Created later by /speckit-tasks
```

### Source Code (repository root)

```text
.nvmrc
package.json
package-lock.json
index.html
vite.config.ts
vitest.config.ts
playwright.config.ts

src/
├── App.tsx
├── main.tsx
├── styles.css
├── components/
│   ├── ChatPanel.tsx
│   ├── MessageList.tsx
│   └── MessageInput.tsx
├── prompts/
│   └── ca-system-prompt.ts
└── services/
    ├── chat-service.ts
    ├── fake-chat-service.ts
    └── gemini-chat-service.ts

tests/
└── unit/
    ├── App.test.tsx
    └── chat-service.test.ts

e2e/
└── ca-buddy.spec.ts

.github/
└── workflows/
    ├── test.yml
    └── deploy.yml
```

**Structure Decision**: Use one frontend project at the repository root. Keep UI
components, the prompt, and service implementations under `src/`; place focused unit
tests under `tests/unit/` and browser acceptance coverage under `e2e/`. There is no
backend, API server, database, or separate frontend/backend split. The exact prompt
path is retained from the constitution. Use one npm lockfile and one Playwright config
to keep local and CI behavior aligned.

## Complexity Tracking

No entries. The design uses one application, one transient conversation model, and one
provider boundary, so no constitutional exception or additional architectural layer is
needed.
