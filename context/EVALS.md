# EVALS.md

The verification table from HW3, grown up. Five sections, in this order.
The first two are written and committed BEFORE any tool sees the spec.

## 1. RAT statement

The riskiest assumption in delegating Feature F-03 (Real-Time Candidate Search & Status Filter) is that bolt.new will preserve deterministic filtering against our existing in-memory candidate array fetched from the Cloudflare Worker without inventing parallel client storage or breaking existing DOM structure.

## 2. Prediction Stake (Before build, 2026-10-01 16:45 EDT)

- **Tight:** At least 3 of 4 EARS rows for Feature F-03 will pass on the tool's first output.
  - Resolved 2026-10-01: PASSED (4 of 4 EARS rows satisfied on initial generation).
- **Loose:** bolt.new will attempt to introduce its own mock data or local storage wrapper rather than filtering directly against the active candidate array loaded from our Cloudflare Worker D1 endpoint.
  - Resolved 2026-10-01: PASSED (bolt.new defined an isolated in-memory `candidates = []` array and local push handler instead of consuming the Cloudflare Worker API; resolved during integration by bridging `getFilteredCandidates()` with `loadCandidates()`).
- **Open:** The tool will introduce non-standard CSS utility classes or modern frontend framework dependencies that violate our vanilla tokens and standards in `context/STYLE.md`. Resolves when I read package.json and styles.css.
  - Resolved 2026-10-01: PARTIALLY PASSED (zero runtime framework dependencies introduced, but bolt injected third-party metadata `<meta>` tags and `.hidden { display: none !important; }` utility styling; normalized during integration).

## 3. Success criteria

| EARS row (feature) | Checked by | Where |
| :--- | :--- | :--- |
| WHEN a recruiter types into the search box, THE SYSTEM SHALL filter candidates by name | test | evals/worker.test.js |
| WHILE status filter is evaluated, THE SYSTEM SHALL persist and return the applicant eligibility status | test | evals/worker.test.js |
| THE SYSTEM SHALL display filter controls above evaluated list | human | README, See It Work |
| IF no candidates match, THEN THE SYSTEM SHALL show empty state message | human | README, See It Work |

## 4. Error-analysis log

| Failure (a few words) | Count | Source | Category |
| :--- | :--- | :--- | :--- |
| Disconnected D1 API fetch / used local array | 1 | bolt | ARCHITECTURE |
| Injected third-party metadata / utility CSS | 2 | bolt | STYLE |
| Test runner module resolution mismatch | 1 | developer | TOOLS |

## 5. Evals
- **Code:** `npm test` with `API=https://mgt3745-hw4.kdotll.workers.dev`; 4 tests, 4 passing. Screenshot in README.
- **Judgment:** docs/JUDGMENT.md, 3 questions, two graders, agreement 100%.

## Verification table (carried from HW4)

| Statement | HW3 verdict | HW4 verdict | Reason |
| :--- | :--- | :--- | :--- |
| Return entries in order | PASS | PASS | `GET /entries` queries `ORDER BY id ASC`, rendering records chronologically. |
| Store valid entry | PASS | PASS | `POST /entries` writes parameters via `.bind()` to D1 and returns 201. |
| Reject missing fields (HTTP 400) | PASS | PASS | Sending empty field returns 400 with "Missing required evaluation fields". |
| Survive cleared cache | CANNOT TEST YET | PASS | Evaluated candidate entries reload from D1 in a new Incognito browser tab with empty cache. |
| Server unreachable / Network drop | CANNOT TEST YET | PASS | Tested with offline network throttling in DevTools; page displays `Network error: Unable to connect to backend server` with 0 console errors. |
| Server returns 500 | CANNOT TEST YET | CANNOT TEST YET | D1 schema constraints handle current loads cleanly; simulating unhandled cloud runtime crashes requires fault-injection tools not yet in stack. |
| Second client writes to same table | DEFERRED | DEFERRED | Multiple clients write concurrently to D1, but real-time push synchronization across active tabs is deferred to WebSockets in a future milestone per ADR-002. |