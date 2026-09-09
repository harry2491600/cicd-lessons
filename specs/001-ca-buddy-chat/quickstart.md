# Quickstart: CA Buddy tax guidance chat

This guide validates the feature after implementation. It does not replace the task
breakdown or contain application implementation code.

## Prerequisites

- Node.js 22.22.2 and npm 10.9.7, or a compatible Node.js 22.12+ environment.
- A browser for local review.
- A Gemini API key restricted to the Gemini API and the deployed site origin for live
  local use. The direct-browser architecture makes this key visible to the browser;
  never reuse an unrestricted key.

## Install and configure

```bash
npm ci
```

For live local chat, create an untracked `.env.local` file containing:

```text
VITE_GOOGLE_API_KEY=replace-with-a-restricted-key
```

Do not commit `.env.local` or print the value in a terminal or CI log. Unit and
end-to-end tests use a fake service or intercepted provider responses and must not
depend on a live Gemini response.

## Local commands

```bash
npm run dev
npm run test:unit
npx playwright install chromium
npm run test:e2e
npm run build
npm run preview
```

`npm run test:e2e` starts the local app through the Playwright configuration and runs
the browser acceptance suite. `npm run preview` serves the built `dist/` output for a
manual production-style check.

## Validation scenarios

1. Open the app and confirm the CA Buddy header, one chat panel, message input, New chat
   action, and one-line disclaimer are visible in the first viewport.
2. Submit `What is TDS and when does it usually apply?`; confirm the user message and
   a concise assistant response appear.
3. Submit a follow-up that refers to TDS; inspect the fake-service or intercepted
   request and confirm the prior completed exchange is included in order.
4. Submit an out-of-scope question and a business-specific filing decision; confirm
   each response explains the limit and recommends consulting a Chartered Accountant.
5. Select New chat; confirm all prior messages disappear, a new request has empty
   history, and no earlier context influences the new exchange.
6. Submit empty and whitespace-only input; confirm no message or service request is
   created.
7. Simulate a provider failure; confirm a non-technical retryable error appears while
   the visible user question remains.
8. Reload or reopen the app; confirm no prior conversation is restored.

## CI validation

- Pushes and pull requests run `npm ci`, unit tests, and Playwright tests with Gemini
  requests intercepted.
- CI provides `VITE_GOOGLE_API_KEY` through the repository secret for any build step
  that requires the configured production value. No workflow step logs environment
  values.
- The deployment job runs only on a successful push to the default branch after both
  test jobs complete successfully.
- GitHub Pages must be configured to use GitHub Actions as the publishing source. The
  deployment uploads the Vite `dist/` directory as a Pages artifact and uses the
  `github-pages` environment.

## Expected result

The unit suite proves UI, fake-service, scope, context, reset, empty-input, and error
behavior. The Playwright suite proves the visible happy path and intercepts every
provider call. A successful production build can be previewed locally and is eligible
for GitHub Pages deployment only after the required checks pass.
