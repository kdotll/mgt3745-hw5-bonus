# ARCHITECTURE.md

Decisions, in order. An ADR is never edited after it is accepted; it is superseded.

## The Gate: HW4 rerun

Where should entries live now that they must survive a cleared cache?

| Criterion | Weight | Build (Worker + D1) | Buy (hosted BaaS) | Delegate (AI builder hosts it) |
|---|---|---|---|---|
| Cost to start | 5 | 5 (25) | 3 (15) | 5 (25) |
| Cost to maintain | 3 | 5 (15) | 2 (6) | 4 (12) |
| Time to working | 5 | 4 (20) | 4 (20) | 3 (15) |
| Inspectability | 4 | 5 (20) | 2 (8) | 2 (8) |
| Switching cost | 2 | 4 (8) | 2 (4) | 2 (4) |
| Fit to spec | 4 | 5 (20) | 3 (12) | 3 (12) |
| **Weighted total** | -- | **108** | **65** | **76** |

*HW3 weights carried forward directly (5, 3, 5, 4, 2, 4) because project constraints and budget remain zero-cost and audit-first. Switching cost scored from Session B experience: migrating candidate records from client localStorage to a Cloudflare Worker + D1 required rewriting only two fetch functions and writing a simple schema, whereas vendor-locked BaaS solutions require opaque SDKs and external auth handshakes.*

## ADR-002: Entries move from localStorage to Cloudflare D1

**Status:** Accepted  
**Supersedes:** ADR-001  

### Context
HW3 proved client-side evaluation speed but failed multi-user and persistence constraints: data could not survive a cleared cache or follow a recruiter across workstations. Evaluated candidate records must now leave the browser.

**Crossing Statement:** Evaluated candidate records (name, degree, graduation year, work authorization, relocation status, and eligibility verdict) cross from the user's browser over HTTPS to Cloudflare Workers and D1 database under Cloudflare's Standard Service Terms (free tier), with Kenneth Riley II accountable for schema integrity, credential protection, and student data privacy.

### Decision
Build and deploy a serverless edge endpoint using Cloudflare Workers and Cloudflare D1 SQLite database to ingest, validate, and store candidate evaluation records. The client application communicates exclusively via `GET /entries` and `POST /entries`.

### Alternatives considered
* **Buy (Hosted BaaS - Airtable/Firebase):** Scored 65. Discarded due to proprietary API lock-in, recurring cost tiers after initial quotas, and lack of granular inspectability into database boundaries.
* **Delegate (AI Builder - Bolt.new/v0):** Scored 76. Discarded due to lack of local inspectability, unvetted server-side code generation, and uncontrolled cloud deployment endpoints.

### Consequences
* **Positive:** Candidate records persist across device reloads, browser cache purges, and different recruiter workstations.
* **Negative (What got harder):** Network latency and connectivity failures must now be handled asynchronously. If Cloudflare or the network drops, the application cannot save offline without explicit caching layers. Server-side validation errors (400/500) require visual user feedback without throwing console errors.

### Revisit trigger
Revisit this decision when a second user role (e.g., student applicant direct portal) requires authenticated session cookies, role-based access control, or per-user data isolation.

---

## ADR-001: Store entries in localStorage

**Status:** Superseded by ADR-002

### Title and date:
ADR-001: Client-Side Vanilla JavaScript and LocalStorage for Candidate Knockout Intake (2026-09-17)

### Status:
Accepted

### Door / concrete acquisition and execution choice:
Build — Hand-built implementation using native HTML5, vanilla JavaScript, and browser `localStorage` running in a local browser environment.

### Context:
`FEATURES.md` defines Feature F-02 (Deterministic Operational Knockout Gate), requiring the system to evaluate four strict candidate constraints (graduation date window, degree program, U.S. work authorization, and relocation readiness) in under 2.0 seconds.

### Decision:
Build the candidate knockout intake form and list view directly using semantic HTML5, vanilla CSS, and client-side JavaScript adhering strictly to the load-save-render cycle with browser `localStorage`, rejecting external UI frameworks and hosted SaaS form builders.

### Consequences and revisit trigger:
* **Positive consequences:** Execution latency is under 50ms, easily beating the 2.0-second EARS acceptance limit. Zero out-of-pocket cost and zero external dependencies or API keys. 100% inspectable and transparent code logic for grading and verification.
* **Negative consequences:** Fails to persist candidate data across different devices or browsers because `localStorage` is isolated to the local client. Does not support concurrent multi-user recruiter workflows.
* **Revisit trigger:** Revisit this decision in Module 4 when a shared backend database and authentication layer are introduced to support multi-user persistence.