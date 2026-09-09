# Feature Specification: CA Buddy tax guidance chat

**Feature Branch**: `001-ca-buddy-chat`

**Created**: 2026-09-09

**Status**: Draft

**Input**: User description: "CA Buddy PRD from prd.md, governed by constitution.md"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Get a quick general answer (Priority: P1)

An Indian small-business owner opens CA Buddy, asks an everyday question about GST,
TDS, ITR deadlines, or audit basics, and receives a concise explanation in plain
language.

**Why this priority**: A useful first explanation is the core value of CA Buddy and
must work before any follow-up or safety workflow is considered.

**Independent Test**: Open the chat, submit one supported question, and confirm that the
question and a readable general answer appear without requiring an account.

**Acceptance Scenarios**:

1. **Given** the first screen is open, **When** the user submits a non-empty supported
   question, **Then** the question appears in the chat and a concise answer appears
   afterward.
2. **Given** the first screen is open, **When** the user inspects the page before
   asking a question, **Then** the CA Buddy header, chat panel, message input, New chat
   action, and one-line disclaimer are visible.

---

### User Story 2 - Continue or reset the current conversation (Priority: P1)

The owner asks a follow-up that depends on the preceding exchange and receives an
answer that uses the current conversation. The owner can start over with New chat and
remove the prior visible exchange and context.

**Why this priority**: Follow-up questions make the first explanation practical, while
an explicit reset preserves the user's control over a conversation that is not saved.

**Independent Test**: Submit a question, submit a follow-up referring to it, verify the
follow-up answer reflects the earlier exchange, select New chat, and verify that the
old messages no longer appear or influence a new question.

**Acceptance Scenarios**:

1. **Given** the chat contains a previous question and answer, **When** the user asks a
   follow-up that refers to that exchange, **Then** the response uses the current
   conversation context.
2. **Given** the chat contains messages, **When** the user selects New chat, **Then**
   the visible messages and conversation context are cleared and the input is ready
   for a new exchange.
3. **Given** the user has started a new chat, **When** the user asks a question, **Then**
   the response does not rely on messages from the prior chat.

---

### User Story 3 - Recognize when professional help is needed (Priority: P1)

The owner asks for a personalized filing decision or a topic outside CA Buddy's
supported scope and receives a clear explanation of the limit together with a
recommendation to consult a Chartered Accountant.

**Why this priority**: Tax guidance can cause harm when general information is treated
as individualized advice, so the boundary must be visible in the main workflow.

**Independent Test**: Submit one unsupported question and one individualized question,
then confirm that each response recommends consulting a Chartered Accountant without
pretending to make the decision for the user.

**Acceptance Scenarios**:

1. **Given** the chat is ready for a question, **When** the user asks about an
   unsupported topic, **Then** the response states that CA Buddy is outside that
   topic's scope and recommends consulting a Chartered Accountant.
2. **Given** the chat is ready for a question, **When** the user asks for a decision
   specific to their business or filing situation, **Then** the response explains the
   limit and recommends consulting a Chartered Accountant.
3. **Given** the user is chatting, **When** the user views any state of the screen,
   **Then** the one-line disclaimer remains visible.

### Edge Cases

- When the input is empty or contains only whitespace, the chat MUST NOT add a message
  or request an answer.
- When the answer service is unavailable or returns an error, the user MUST see a
  clear, non-technical failure message and be able to retry without losing the visible
  conversation.
- When a question mixes a supported topic with a request for individualized judgment,
  the response MUST provide only general information and recommend consulting a
  Chartered Accountant for the individualized part.
- When the user starts New chat, no earlier message or context may reappear in the new
  exchange.
- When the user leaves or reloads the experience, no previous conversation may be
  restored.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The first screen MUST present exactly the CA Buddy header, one chat
  panel, one message input, one New chat action, and one visible one-line disclaimer.
- **FR-002**: The product MUST add each submitted non-empty user question to the chat
  and display the corresponding answer.
- **FR-003**: Each submitted question MUST be answered using the current conversation
  context when prior messages exist.
- **FR-004**: The product MUST provide concise plain-language general information for
  everyday Indian GST, TDS, ITR deadlines, and audit-basics questions.
- **FR-005**: The product MUST identify unsupported topics and requests for
  individualized professional judgment, explain the limit, and recommend consulting a
  Chartered Accountant.
- **FR-006**: The one-line disclaimer MUST remain visible while the user chats.
- **FR-007**: Selecting New chat MUST clear all visible messages and conversation
  context, and the product MUST NOT retain conversation history after the current
  session.
- **FR-008**: The product MUST NOT require the user to provide a service credential or
  expose a provider credential in the user interface.
- **FR-009**: The product MUST remain limited to a single-screen chat experience and
  MUST exclude login, settings, saved history, tax filing or submission, and topics
  outside the supported scope.

### Key Entities *(include if feature involves data)*

- **Conversation**: The current, unsaved exchange between the owner and CA Buddy; it
  contains ordered messages and is cleared by New chat.
- **Message**: One user question or CA Buddy answer in the current conversation; it has
  readable text and a role that distinguishes who sent it.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In the acceptance walkthrough, 100% of reviewers can identify the CA
  Buddy header, chat panel, input, New chat action, and disclaimer on the first screen
  without signing in.
- **SC-002**: Under normal service availability, at least 95% of supported questions
  display a readable answer within 10 seconds of submission.
- **SC-003**: 100% of acceptance tests for a contextual follow-up show that the answer
  uses the current exchange, and 100% of New chat tests show no prior messages or
  context.
- **SC-004**: 100% of acceptance tests for unsupported and individualized questions
  contain a clear recommendation to consult a Chartered Accountant and no
  individualized filing decision presented as fact.
- **SC-005**: At least 4 of 5 representative small-business owners complete the first
  question and follow-up flow without moderator assistance and describe the answer as
  understandable.
- **SC-006**: 100% of privacy checks confirm that a conversation is not restored after
  the user leaves or reloads the experience.

## Assumptions

- Users have internet access and the answer service is available during the session.
- The first release is intended for general information, not tax filing, submission,
  representation, or individualized professional advice.
- Users do not need an account, and the product does not retain conversation history
  after the current session.
- The supported scope and the consult-a-CA boundary are more important than answering
  every possible tax question.
- Response-time measurement excludes outages or failures in the external answer
  service.
- Delivery follows the constitution's five ordered tasks, with automated checks required
  before deployment.
