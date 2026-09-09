# Data Model: CA Buddy tax guidance chat

## Conversation

Represents the one unsaved exchange visible in the current browser session.

| Field | Type | Rules |
| --- | --- | --- |
| `messages` | ordered list of Message | Starts empty; preserves display order; contains completed user and assistant messages only |
| `status` | `idle`, `submitting`, or `error` | `submitting` disables duplicate submission; `error` keeps the visible exchange available for retry |
| `errorMessage` | optional string | User-readable and non-technical; not sent to the model as conversation context |

### Conversation invariants

- The conversation exists only in transient browser memory.
- A submitted user message is displayed immediately, but it is not added to the
  service history until its assistant response succeeds.
- A completed exchange consists of a user Message followed by an assistant Message.
- New chat replaces the conversation with an empty idle Conversation.
- No field is written to browser storage, a URL, a database, or a server-side session.

## Message

Represents one completed entry in the current exchange.

| Field | Type | Rules |
| --- | --- | --- |
| `id` | transient string | Unique within the current conversation; never used for persistence |
| `role` | `user` or `assistant` | Identifies the sender and controls presentation |
| `text` | string | Trimmed, non-empty readable content |

### Message validation

- Whitespace-only user input is ignored and creates no Message.
- User text is preserved as entered after trimming outer whitespace.
- Assistant text is displayed as returned by ChatService after trimming; an empty
  provider response is treated as an error.
- Messages are never edited after insertion; a retry creates a new response for the
  same visible user question only after the failed attempt is handled.

## State transitions

| Current state | Event | Result |
| --- | --- | --- |
| Empty idle | Submit non-empty question | Show the user question; enter submitting with empty completed history |
| Completed conversation idle | Submit non-empty question | Show the user question; send all prior completed messages as history; enter submitting |
| Submitting | Provider succeeds with non-empty text | Append assistant Message; return to idle |
| Submitting | Provider fails or returns empty text | Keep visible user question; expose errorMessage; return to error |
| Error | Retry the same question | Reuse the prior completed history without duplicating the failed question; enter submitting |
| Any state | New chat | Replace conversation with empty idle state |
| Any idle or error state | Submit empty or whitespace-only input | No state change and no service call |

## Context handoff

ChatService receives the current completed message list in order. The current user
question is a separate request field. This ensures that a follow-up includes all prior
completed exchanges and that a retry or New chat cannot accidentally duplicate or reuse
stale context.
