<!--
Sync Impact Report
- Version change: placeholder scaffold (no prior version) -> 1.0.0
- Modified principles:
	- template principle 1 -> I. Defined Scope and Plain-Language Guidance
	- template principle 2 -> II. Conversation-Local Privacy
	- template principle 3 -> III. Contract-Separated Model Access
	- template principle 4 -> IV. Testable Delivery
	- template principle 5 -> V. Single-Screen Simplicity
- Added sections: Product and Technical Constraints; Development Workflow and Quality Gates
- Removed sections: none
- Follow-up TODOs: confirm the original ratification date recorded as TODO(RATIFICATION_DATE)
-->

# CA Buddy Constitution

## Core Principles

### I. Defined Scope and Plain-Language Guidance
CA Buddy MUST answer only everyday Indian GST, TDS, ITR deadlines, and audit-basics
questions. Responses MUST use concise plain language, state material assumptions when
needed, and MUST NOT present general information as individualized tax or legal advice.
Questions outside this scope or requiring a personalized filing decision MUST receive a
clear recommendation to consult a Chartered Accountant. Rationale: a narrow, explicit
scope keeps the first explanation useful without implying professional representation.

### II. Conversation-Local Privacy
The visible message list MUST be the only conversation memory. Follow-up requests MUST
include the current conversation context, and New chat MUST clear both the visible
messages and the model context. The application MUST NOT save conversation history,
credentials, or personal data in a database, server, or browser storage. Rationale:
small-business owners need a useful current exchange without creating an account or a
retained record.

### III. Contract-Separated Model Access
The React UI MUST depend on a ChatService interface rather than a provider-specific
client. The production implementation MUST call Google Gemini through LangChain.js
using @langchain/google-genai, model gemini-2.5-flash, and a maximum of 512 output
tokens. Tests MUST be able to replace the production service with a fake implementation.
The browser MUST read the key from VITE_GOOGLE_API_KEY, and no API key may be
hard-coded. Rationale: the interface isolates UI behavior, makes tests deterministic,
and keeps provider configuration explicit.

### IV. Testable Delivery
Every user-visible behavior and service contract change MUST have an automated check at
the smallest useful level. Unit tests MUST use Vitest and Testing Library with the model
faked. End-to-end tests MUST use Playwright and intercept Gemini requests. A change is
not ready for deployment until the required checks pass. Rationale: deterministic unit
tests protect interaction behavior, while intercepted end-to-end tests verify the full
browser workflow without depending on a live model or network response.

### V. Single-Screen Simplicity
CA Buddy MUST remain a frontend-only React, TypeScript, and Vite application with one
screen containing the header, one chat panel, one message input, a New chat control,
and a visible one-line disclaimer. Login, settings, saved history, a backend, a server,
a database, tax filing or submission, and unrelated topics are outside the product
boundary. Rationale: the product is a fast first explanation, so every added surface
must be justified against that purpose.

## Product and Technical Constraints

- The system prompt MUST live in `src/prompts/ca-system-prompt.ts` and define the CA
	persona, supported scope, plain-language behavior, consult-a-CA fallback, and
	disclaimer.
- The disclaimer MUST remain visible while the user chats: "CA Buddy provides general
	information, not professional tax advice; consult a Chartered Accountant for
	decisions specific to your business."
- Local development MUST read `VITE_GOOGLE_API_KEY` from local environment
	configuration. CI MUST provide the same variable through a GitHub Actions secret.
	Secrets MUST NOT be committed to the repository.
- The only permitted model provider and model are Google Gemini through LangChain.js
	and `gemini-2.5-flash`, respectively, unless this constitution is amended first.

## Development Workflow and Quality Gates

- Work MUST be delivered as exactly five ordered tasks, with one GitHub issue and one
	pull request for each task:
	1. Chat UI shell, unit tests, and the unit-test CI workflow.
	2. ChatService interface, LangChain and Gemini implementation, and a fake service.
	3. CA persona prompt, scope rules, consult-a-CA fallback, disclaimer, and
		 conversation memory.
	4. Playwright end-to-end tests with intercepted Gemini requests, wired into CI.
	5. GitHub Pages deployment gated on all tests passing.
- Push and pull-request workflows MUST run the unit and end-to-end test suites. The
	deployment job MUST run only after those required checks pass.
- Each pull request MUST verify scope handling, the consult-a-CA fallback, follow-up
	context, New chat reset behavior, disclaimer visibility, and absence of persisted
	history when those behaviors are affected.
- Acceptance review MUST cover an in-scope GST, TDS, ITR deadline, or audit-basics
	question; a contextual follow-up; an out-of-scope or individualized question; and a
	New chat reset.

## Governance

This constitution is the highest-priority project guidance. Every issue and pull request
MUST identify the principles and quality gates it affects. Conflicts MUST be resolved
in favor of this document until an amendment is approved. Complexity beyond the defined
scope MUST be documented and justified in the relevant pull request.

Amendments MUST be proposed in a pull request that includes the reason for the change,
the affected principles or sections, an updated Sync Impact Report, and any migration
or follow-up work. The amendment MUST be reviewed before merge, and the Last Amended
date MUST be updated when it changes. The Ratified date records the original adoption
date and MUST NOT change for ordinary amendments.

Versioning follows semantic versioning: MAJOR denotes a backward-incompatible removal
or redefinition of a principle; MINOR denotes a new principle or a material expansion
of a section; PATCH denotes clarification, wording, or other non-semantic refinement.
Every compliance review MUST check the scope boundary, privacy rules, model contract,
automated tests, secret handling, and deployment gate.

**Version**: 1.0.0 | **Ratified**: TODO(RATIFICATION_DATE): confirm original adoption date | **Last Amended**: 2026-09-09
