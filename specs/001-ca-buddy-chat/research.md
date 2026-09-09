# Research: CA Buddy tax guidance chat

**Date**: 2026-09-09

## Decision 1: Runtime and package management

**Decision**: Use Node.js 22.22.2 locally and in GitHub Actions, with npm and a
committed `package-lock.json`. Use TypeScript 5.x, React, and the current Vite release
that is compatible with the selected Node.js version. Exact dependency versions are
resolved during project setup and recorded in the lockfile.

**Rationale**: The local environment already provides Node.js 22.22.2 and npm 10.9.7.
Vite documents support for Node.js 20.19+ and 22.12+, while current Vitest documentation
requires Node.js 22.12+ and Vite 6.4+. npm is already available and is the package
manager used in Vite's official deployment examples, so adding a second package-manager
requirement would not add value.

**Alternatives considered**:

- Node.js 24 LTS: viable, but it differs from the available local runtime and is not
  required by the documented Vite or Vitest minimums.
- pnpm: viable for lockfile reproducibility, but it is not installed locally and is
  unnecessary for this single frontend project.
- Floating dependency ranges without a lockfile: rejected because CI and local tests
  could resolve different tool versions.

**Risk**: The dependency set must be installed as a compatible group during the first
implementation task. The lockfile, Node version file, and CI setup must then be kept in
sync.

## Decision 2: Gemini integration boundary

**Decision**: Implement the production ChatService with
`@langchain/google-genai` and `@langchain/core`, using `ChatGoogleGenerativeAI` with the
constitution-mandated `gemini-2.5-flash` model and a 512-token output cap. The service
receives the browser-exposed `VITE_GOOGLE_API_KEY` configuration and sends requests
directly from the browser as required by the PRD.

**Rationale**: The constitution explicitly fixes the provider, package, model, token
limit, and environment variable. The official LangChain integration documents the
`ChatGoogleGenerativeAI` class, installation packages, model invocation, and generation
configuration. The UI remains independent of that provider through the ChatService
contract.

**Alternatives considered**:

- The newer LangChain `ChatGoogle` integration: not selected because the constitution
  explicitly requires `@langchain/google-genai`.
- Direct Google SDK calls: rejected because they would bypass the mandated LangChain
  boundary.
- A backend proxy: rejected because the PRD and constitution require a frontend-only
  browser call.

**Risk**: Current LangChain documentation marks this integration as deprecated in favor
of a newer package. This plan follows the governing constitution and records the
maintenance risk; changing providers requires a constitution amendment before work
begins. Browser bundling and the explicit API-key option must be verified against the
locked package release during implementation.

**Security implication**: A `VITE_` value is included in a browser build and cannot be
treated as a server secret. The GitHub key must be restricted to the Gemini API and the
deployed site origin, must never be logged, and must not be committed to source control.

## Decision 3: Unit and component testing

**Decision**: Use Vitest with a DOM environment, Testing Library for React and DOM
queries, and user-event interactions. Keep the model out of unit tests by injecting a
fake ChatService. Use explicit behavior assertions instead of snapshots.

**Rationale**: Vitest is Vite-native, and its current documentation requires the chosen
Node/Vite baseline. Testing Library exercises the same roles, labels, and visible
content that the acceptance criteria require. A fake service makes response, context,
reset, error, and empty-input behavior deterministic and fast.

**Alternatives considered**:

- Playwright for every test: rejected because browser tests are slower and less precise
  for isolated component and service-contract behavior.
- Snapshot-heavy testing: rejected because the important contracts are interaction and
  content behavior, not serialized markup.
- A service-worker mock for unit tests: rejected because a direct fake service is the
  smallest boundary for this UI.

## Decision 4: Browser acceptance testing and request interception

**Decision**: Use standalone Playwright Test with a Chromium project covering desktop
and mobile viewport scenarios. Intercept Gemini requests with `page.route()` or a
browser-context route and return fixture responses with `route.fulfill()`. Assert the
request body for follow-up tests so that context propagation is verified rather than
only the rendered answer.

**Rationale**: Playwright natively observes and mocks fetch/XHR traffic without a live
provider, quota, or API key. Its route API supports exact URL glob matching and response
fulfillment. Chromium desktop and mobile viewport coverage is sufficient for the
single-screen acceptance flow while keeping CI time and browser downloads bounded.

**Alternatives considered**:

- A three-engine Chromium/Firefox/WebKit matrix: deferred because the PRD does not
  require cross-engine certification and the first delivery must remain simple.
- MSW: rejected for E2E because Playwright's native network route is closer to the
  external request boundary and avoids another mocking layer.
- Live Gemini calls: rejected because tests must be deterministic and must not consume
  provider quota.

**Risk**: The mock response shape must match the locked LangChain provider behavior. A
  fixture should be kept beside the E2E tests, and a direct unit test should verify the
  service's response extraction separately.

## Decision 5: Browser target and deployment

**Decision**: Target the modern browsers covered by the current Vite production
Baseline Widely Available target. Do not add legacy-browser polyfills. Build the static
site to `dist/`, use a relative asset base (`./`) because the repository name is not
available in this workspace and the app has no client-side routes, and deploy through
the GitHub Pages Actions artifact workflow.

**Rationale**: Vite documents a modern production browser baseline and recommends
setting an explicit repository base when a project site has a known repository name.
The relative base keeps this route-free single-screen app usable under a project-site
path without inventing a repository name. GitHub's Pages workflow requires Pages and
OIDC permissions, an uploaded Pages artifact, and a deployment job that depends on the
build job.

**Alternatives considered**:

- A hard-coded `/<repository>/` base: rejected because this workspace is not connected
  to a Git repository and the final repository name is unknown.
- Legacy browser support with a compatibility plugin: rejected as out of scope for the
  first single-screen release.
- A `gh-pages` branch or third-party deploy action: rejected because the official Pages
  artifact actions provide an atomic build-to-deploy path with explicit permissions.

**Risk**: The final repository's Pages settings must use GitHub Actions as the source.
If the repository later requires absolute asset URLs, the Vite base can be changed to
the documented `/<repository>/` form in the deployment task.

## Decision 6: CI job graph and secret handling

**Decision**: Run unit and Playwright jobs on every push and pull request. They use
`npm ci` from the committed lockfile. The Playwright job runs against the local app and
intercepts all Gemini requests. A deployment job runs only for a successful push to the
default branch and declares `needs: [unit, e2e]`; it builds `dist/`, uploads the Pages
artifact, and deploys it with the official Pages actions. The production build receives
`VITE_GOOGLE_API_KEY` from the GitHub Actions secret without logging it.

**Rationale**: Parallel test jobs keep feedback fast, while the native `needs` graph
enforces the constitution's no-deploy-before-tests rule. GitHub Pages documentation
requires `pages: write` and `id-token: write` for artifact deployment and recommends a
dedicated `github-pages` environment.

**Alternatives considered**:

- Deploy on pull requests or every branch: rejected because the PRD requires deployment
  only after the required checks and does not request preview environments.
- A live-provider smoke test as a required gate: rejected because it would make the
  required path depend on quota, network availability, and a secret.
- Logging environment configuration for debugging: rejected because it can expose the
  browser API key in workflow logs.

**Risk**: The direct browser architecture means the configured Gemini key is public in
  the deployed bundle. Restricting the key by API and referrer, monitoring quota, and
  rotating it are operational requirements outside the feature's code path.

## Sources

- [Node.js release schedule](https://nodejs.org/en/about/previous-releases)
- [Vite getting started and browser support](https://vite.dev/guide/)
- [Vite static deployment and GitHub Pages](https://vite.dev/guide/static-deploy.html)
- [LangChain ChatGoogleGenerativeAI integration](https://docs.langchain.com/oss/javascript/integrations/chat/google_generative_ai)
- [Vitest getting started](https://vitest.dev/guide/)
- [Playwright network mocking](https://playwright.dev/docs/network)
- [GitHub Pages custom workflows](https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages)
