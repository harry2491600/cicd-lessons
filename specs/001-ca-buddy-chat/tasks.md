---

description: "Ordered implementation tasks for CA Buddy tax guidance chat"
---

# Tasks: CA Buddy tax guidance chat

**Input**: Design documents from `specs/001-ca-buddy-chat/`

**Prerequisites**: [plan.md](./plan.md), [spec.md](./spec.md),
[research.md](./research.md), [data-model.md](./data-model.md),
[contracts/chat-service.md](./contracts/chat-service.md), and
[quickstart.md](./quickstart.md)

**Organization**: The constitution requires exactly five ordered tasks, with one
GitHub issue and one pull request for each task. The task phases below map those five
constitutional delivery units to the three P1 user stories. Story 1 is the MVP; Stories
2 and 3 build on the shared service, prompt, and conversation behavior.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: No task is marked `[P]`; the constitution requires the five delivery tasks to
  execute in order.
- **[Story]**: The story label maps a task in a user-story phase to the primary journey
  it completes or validates.
- Every checklist item includes concrete repository paths and is one of the five issue /
  pull-request delivery units.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish the frontend project, the accessible single-screen shell, and the
unit-test foundation required by the first delivery task.

- [X] T001 Initialize the Node.js 22.22.2 React/TypeScript/Vite project and accessible chat UI shell in `.nvmrc`, `package.json`, `index.html`, `src/main.tsx`, `src/App.tsx`, `src/components/ChatPanel.tsx`, `src/components/MessageList.tsx`, `src/components/MessageInput.tsx`, and `src/styles.css`; configure Vitest and Testing Library in `vitest.config.ts`, add initial shell behavior tests in `tests/unit/App.test.tsx`, and add the push/pull-request unit-test workflow in `.github/workflows/test.yml`.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish the provider-independent service boundary and deterministic test
implementation before production conversation behavior is connected.

**Checkpoint**: After T002, the UI can depend on a stable ChatService contract, the
production provider is configured without hard-coded credentials, and unit tests can
record requests or simulate errors without network access.

- [X] T002 Define the ChatService request/response types and fake and Gemini implementations in `src/services/chat-service.ts`, `src/services/fake-chat-service.ts`, and `src/services/gemini-chat-service.ts`; install and lock the required dependencies in `package.json` and `package-lock.json`, configure `VITE_GOOGLE_API_KEY` handling and build settings in `vite.config.ts` and `.env.example`, and verify the contract with deterministic tests in `tests/unit/chat-service.test.ts` using `gemini-2.5-flash`, a 512-token cap, immutable ordered history, and retryable error behavior.

---

## Phase 3: User Story 1 - Get a quick general answer (Priority: P1) MVP

**Goal**: A small-business owner can submit a supported GST, TDS, ITR-deadline, or
audit-basics question and receive a concise, plain-language answer in the visible chat.

**Independent Test**: With T001 and T002 complete, run the unit suite, open the first
screen, verify the required shell and disclaimer, submit a non-empty supported question,
and confirm that the user message and fake-service answer appear without an account.

**Implementation for User Story 1**

- [X] T003 [US1] Add the CA persona, supported-topic rules, consult-a-CA fallback, disclaimer text, and transient conversation behavior in `src/prompts/ca-system-prompt.ts`, `src/App.tsx`, `src/components/ChatPanel.tsx`, `src/components/MessageList.tsx`, and `src/components/MessageInput.tsx`; pass completed `Conversation.messages` as ordered history while sending the current question separately, preserve retryable errors, clear messages and context on New chat, and extend `tests/unit/App.test.tsx` and `tests/unit/chat-service.test.ts` for supported answers, scope limits, disclaimer visibility, empty input, context, reset, and no persistence.

**Checkpoint**: User Story 1 is a usable MVP and its shared implementation also exposes
the context and safety behavior needed by the later story phases.

---

## Phase 4: User Story 2 - Continue or reset the current conversation (Priority: P1)

**Goal**: A follow-up uses only the current exchange, and New chat removes all visible
messages and model context.

**Independent Test**: With T003 complete, run the Playwright suite with Gemini requests
intercepted, submit a first question and a context-dependent follow-up, inspect the
request history, select New chat, then confirm the next request has empty history and no
prior answer is restored after reload.

**Implementation for User Story 2**

- [X] T004 [US2] Add intercepted-provider browser coverage in `e2e/ca-buddy.spec.ts`, configure Chromium desktop and mobile projects in `playwright.config.ts`, and wire the Playwright job into `.github/workflows/test.yml`; assert the first-screen contract, supported response, ordered follow-up context, New chat reset, empty-input no-op, retryable provider error, no restored conversation, and the consult-a-CA response using `page.route()` fixtures without a live Gemini call.

**Checkpoint**: User Stories 1 and 2 are independently demonstrable in the browser, with
context and reset behavior verified at the external request boundary.

---

## Phase 5: User Story 3 - Recognize when professional help is needed (Priority: P1)

**Goal**: Unsupported and individualized requests are safely bounded, and the complete
feature is deployable only after all required tests pass.

**Independent Test**: With T004 complete, run the full local validation guide, submit an
unsupported topic and a business-specific filing decision, confirm both responses
recommend consulting a Chartered Accountant, and verify that a successful default-branch
workflow deploys only after unit and E2E jobs succeed.

**Implementation for User Story 3 and Delivery**

- [X] T005 [US3] Add the test-gated GitHub Pages deployment in `.github/workflows/deploy.yml`, set the Vite project-site asset base and production build behavior in `vite.config.ts`, document the restricted `VITE_GOOGLE_API_KEY` and Pages setup in `README.md`, and run the complete acceptance and security checks from `specs/001-ca-buddy-chat/quickstart.md`; make deployment depend on the successful unit and E2E jobs, use the `github-pages` environment and Pages artifact actions, and ensure no credential or conversation history is committed or persisted.

**Checkpoint**: All three user stories are covered, the production site can be built and
published through the required gate, and the five constitutional delivery tasks are
complete.

---

## Final Phase: Polish & Cross-Cutting Concerns

The constitution permits no sixth checklist task. Cross-cutting work is therefore
included in T001-T005 and must be reviewed before each corresponding pull request:
accessibility and first-viewport layout in T001, secret handling and provider errors in
T002-T003, intercepted browser behavior in T004, and deployment permissions and artifact
integrity in T005.

---

## Dependencies & Execution Order

### Phase Dependencies

- **T001 / Phase 1 Setup**: Starts immediately; creates the project shell, scripts, unit
  test configuration, and initial CI workflow.
- **T002 / Phase 2 Foundational**: Depends on T001; creates the service boundary and
  implementations that all user stories use.
- **T003 / Phase 3 User Story 1**: Depends on T002; connects the shell to ChatService,
  adds the persona and shared conversation state, and delivers the MVP.
- **T004 / Phase 4 User Story 2**: Depends on T003; verifies the browser workflow and
  current-session context at the provider request boundary.
- **T005 / Phase 5 User Story 3**: Depends on T004; adds the final deployment gate and
  confirms safety and acceptance behavior before publication.

### User Story Dependencies

- **User Story 1 (P1)**: Requires T001 for the shell, T002 for ChatService, and T003 for
  the supported-answer flow. It is the suggested MVP boundary.
- **User Story 2 (P1)**: Requires the shared conversation behavior delivered in T003 and
  the intercepted browser checks delivered in T004.
- **User Story 3 (P1)**: Requires the persona and safety rules in T003, the browser
  assertions in T004, and the gated release validation in T005.

### Within Each User Story

- Tests and fixtures are prepared before the behavior they validate within the same
  delivery task.
- The data model rules in `data-model.md` and the ChatService contract in
  `contracts/chat-service.md` are the source of truth for implementation decisions.
- A task is complete only when its listed unit, browser, or workflow checks pass and the
  next ordered issue / pull-request delivery unit can start.

### Parallel Opportunities

The five checklist tasks are intentionally sequential because the constitution requires
one ordered issue and pull request per task. Within a task, contributors can parallelize
non-overlapping work after the shared contract is agreed:

- **T001**: `src/components/MessageList.tsx`, `src/components/MessageInput.tsx`, and
  `src/styles.css` can be drafted in parallel with the initial `tests/unit/App.test.tsx`
  shell assertions.
- **T002**: `src/services/fake-chat-service.ts` and `tests/unit/chat-service.test.ts` can
  be developed in parallel with dependency and environment configuration, then checked
  against `src/services/chat-service.ts`.
- **T003**: Prompt authoring in `src/prompts/ca-system-prompt.ts` and conversation-state
  tests in `tests/unit/App.test.tsx` can proceed in parallel before integration.
- **T004**: Route fixtures and request-body assertions in `e2e/ca-buddy.spec.ts` can be
  prepared in parallel with `playwright.config.ts`, provided both use the same provider
  URL pattern.
- **T005**: Pages workflow authoring in `.github/workflows/deploy.yml` and deployment
  documentation in `README.md` can proceed in parallel after the final build base is
  agreed in `vite.config.ts`.

No cross-task parallel execution is allowed; each task must be merged and its checks
must pass before the next task's issue and pull request begin.

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete T001: project shell, accessible first screen, unit tests, and unit-test CI.
2. Complete T002: ChatService contract, fake service, and production Gemini service.
3. Complete T003: CA persona, supported-answer behavior, disclaimer, and conversation
   state.
4. Stop and validate User Story 1 with the unit suite and the first acceptance scenario.
5. Demonstrate the MVP before proceeding to browser coverage and deployment.

### Incremental Delivery

1. T001 establishes the shell and test foundation.
2. T002 establishes provider-independent model access.
3. T003 delivers the usable supported-answer MVP and shared safety/context behavior.
4. T004 adds browser-level evidence for context, reset, scope, errors, and privacy.
5. T005 adds the production build and Pages deployment gate.

Each stage preserves the preceding behavior and is delivered as one issue and one
pull-request in the required order.

### Story Test Criteria

- **US1**: The header, chat panel, input, New chat control, disclaimer, submitted user
  message, and supported assistant answer are visible and usable on the first screen.
- **US2**: A follow-up request contains the completed prior exchange; New chat clears
  visible messages and sends empty history; reload restores no prior conversation.
- **US3**: Unsupported and individualized questions receive a clear consult-a-CA
  recommendation; no response presents personalized filing judgment as fact; deployment
  cannot run unless unit and E2E tests pass.

## Notes

- Exactly five checklist tasks are present: T001 through T005.
- The five task descriptions correspond one-to-one with the PRD's five ordered delivery
  tasks and constitution governance requirement.
- Every task has the required checkbox, sequential ID, applicable story label, and exact
  file paths.
- The task list does not create `tasks.md` recursively or add a sixth polish task.
