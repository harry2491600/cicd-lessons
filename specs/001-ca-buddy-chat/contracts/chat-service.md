# Contract: ChatService

## Purpose

ChatService is the boundary between the single-screen UI and the answer provider. The
UI depends only on this contract, allowing unit tests to use a deterministic fake and
the browser build to use the constitution-approved Gemini implementation.

## Types

```ts
type ChatRole = "user" | "assistant";

type ChatMessage = {
  role: ChatRole;
  text: string;
};

type ChatRequest = {
  message: string;
  history: readonly ChatMessage[];
};

type ChatResponse = {
  text: string;
};

interface ChatService {
  sendMessage(request: ChatRequest): Promise<ChatResponse>;
}
```

## Request contract

- `message` is the trimmed, non-empty current user question.
- `history` contains only completed prior messages, in display order. It excludes the
  current `message`, pending messages, error text, and any conversation from a previous
  New chat action.
- The service MUST NOT mutate the request or any message supplied in `history`.
- An empty `history` is valid and is required for the first question after opening or
  resetting the chat.

## Response and error contract

- A successful response contains non-empty plain text in `text`.
- A provider failure, network failure, missing configuration, or empty provider response
  rejects the promise. The UI maps the failure to a clear non-technical error message,
  keeps the visible user question, and permits retry.
- Error details and credentials MUST NOT be rendered in the user interface or logged by
  the browser application.

## Production implementation rules

- Use `@langchain/google-genai` and `@langchain/core`.
- Use model `gemini-2.5-flash` and a maximum output of 512 tokens.
- Include the CA system prompt from `src/prompts/ca-system-prompt.ts` before the
  conversation history and current user question.
- Read the browser configuration from `VITE_GOOGLE_API_KEY`; never hard-code a key.
- Map the LangChain response to the plain `ChatResponse.text` value.

## Fake implementation rules

- The fake MUST implement the same `sendMessage` signature.
- Unit tests MUST be able to provide deterministic responses, record requests, and
  simulate failures without network access.
- Recorded requests MUST make it possible to assert that a follow-up includes prior
  completed messages and that New chat sends an empty history.

## End-to-end interception

Playwright tests intercept the provider request before it reaches Google and fulfill it
with a fixture response. The route handler MUST be able to inspect the request payload
for context assertions and return success and failure fixtures. E2E tests therefore
validate the real UI-to-ChatService-to-provider request shape without requiring a live
API key or consuming provider quota.
