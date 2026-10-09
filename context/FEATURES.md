# Features

## Context
Campus recruiters face high-volume application postings that receive 500 to 800 applications within days. Corporate ATS platforms force recruiters to choose between slow, multi-click manual reviews and blunt keyword algorithms that favor keyword-stuffed resumes over authentic, hands-on shop and design competence. Recruiters want to quickly isolate project execution evidence to build a defensible, diverse candidate shortlist for plant hiring managers without being bottlenecked by untrustworthy match scores. Simultaneously, proactive student applicants need visibility into review timelines so they know their materials are being actively evaluated instead of waiting aimlessly in indefinite post-application silence.

## Users
**Primary User Segment:** High-Volume Diversity-Conscious Campus Recruiter (Profile 1 in `USERS.md`). Evaluates 500+ technical applications per open role, distrusts automated keyword rankings, and needs rapid manual scanning to surface candidates with tangible experience
**Secondary User Segment:** Proactive Business Applicant (Profile 2 in `USERS.md`). Crafts tailored project bullets, experiences friction from repetitive account setups across ATS portals, and seeks clear evaluation timeline estimates to avoid waiting in radio silence.

## Scope
**Included Behavior:**
* Receiving large numbers of applications and extracting plain text of candidate PDF resumes
* Deterministic screening of four operational knockout parameters: graduation date range, degree level, work authorization, and relocation readiness.
* Full-screen, keyboard-driven viewer that displays applicant resumes with sub-second navigation hotkeys.
* Contextual snippet formatting that highlights technical verbs linked directly to technical tools within project bullets while suppressing isolated skills banks.
* Candidate-facing read-only status page that displays active review windows and interview milestone dates for specific requisition IDs.
* CSV export of shortlisted candidate records containing extracted project execution evidence.

**Explicit Non-Goals:**
* **No Algorithmic Stack-Ranking:** The system will not generate composite match percentages or automatically rank candidates based on keyword density.
* **No Automated Disqualification of Eligible Candidates:** The system will not send auto-rejections to non-knockout applicants. All rejection and hold decisions remain strictly human-driven.
* **No Generative Text Rewriting:** The system will not generate, rephrase, or ghostwrite bullet points for students.
* **No Direct ATS API Integration:** The system will not write data back into enterprise ATS platforms like Workday, it will function as an external searching tool utilizing CSV/PDF exports.

### Kano hypotheses
Provide at least six features. For each, name the user segment, date, category, and evidence-based reasoning. These are tentative hypotheses, not validated survey findings.

| Feature ID | Feature | Kano hypothesis | Segment / date | Evidence and reasoning |
|---|---|---|---|---|
| F-01 | Instant Batch PDF Viewer with Hotkey Navigation | Must-be | Recruiter / 09-10-2026 | Recruiter identified multi-click navigation in Workday as the critical operational bottleneck when manually triaging 500+ applicants at 15–30 seconds per resume. |
| F-02 | Deterministic Operational Knockout Gate | Must-be |  Recruiter / 09-10-2026 | Legal and company policy mandates that candidates failing graduation date, visa, or relocation constraints cannot be advanced. These must be screened before human review. |
| F-03 | Contextual Execution Snippet Highlighter | Performance | Recruiter / 09-10-2026 | Recruiter actively searches for physical manufacturing verbs and component execution in project bullets; automated highlighting directly scales scan speed and consistency. |
| F-04 | Open role Review Timeline & Stage Transparency Badge | Attractive | Business Student / 09-10-2026 | Student expressed that the most unsatisfying part of applying is radio silence; providing clear milestone dates addresses this primary anxiety without adding recruiter overhead. |
| F-05 | Universal Multi-Platform Profile Sync | Indifferent | Business Student / 09-10-2026 | While students dislike repetitive account creation, corporate recruiters cannot accept off-platform profile data due to institutional compliance constraints. |
| F-06 | Automated Algorithmic Candidate Match Scoring | Reverse | Campus Recruiter / 09-10-2026 | Recruiter explicitly distrusts automated match scores because they reward artificial AI keyword-stuffing and penalize authentic candidates from non-traditional backgrounds. |

## Behavior
1. **Requisition Setup & Batch Upload:** The recruiter enters a job ID, defines the four knockout parameters (graduation date range, degree, work authorization, relocation consent), sets public review milestone dates, and uploads a `.zip` archive containing applicant PDF resumes.
2. **Knockout Screening:** The system extracts text from each PDF. Candidates failing one or more knockout parameters are tagged as "Ineligible" and placed in an administrative queue. Eligible candidates are queued for further review.
3. **Execution Snippet Tagging:** For eligible candidates, the ATS reader reads project and experience sections, identifying sentences where a technical tool is paired with an action verb and physical component. These sentences are indexed as "Contextual Snippets."
4. **Keyboard-Driven Research:** The recruiter launches Search Mode. The first resume renders instantly. Contextual snippets are highlighted and isolated skills sections at the bottom of the page are dimmed. The recruiter uses single hotkeys like `1` to Shortlist, `2` to Hold, `J`/`K` to Navigate.
5. **Shortlist Export:** When searching concludes, the recruiter exports a CSV manifest containing candidate names, contact details, notes, and the specific extracted project execution snippets to attach to hiring manager review emails.
6. **Student Status Inspection:** A student navigates to the public review status portal, enters their job ID, and views the active hiring stage, review window dates, and projected notification timeline.

## Constraints
* **Performance:** The resume viewer must render the next document within 200 milliseconds of a navigation keystroke to support quick 15-second evaluations.
* **Determinism:** Given identical PDF text and requisition parameters, the snippet extraction logic must yield the identical highlights across repeated executions.
* **Data Privacy:** Candidate resume data must remain local or within the designated tenant to ensure no application data may be used to train public commercial AI models.
* **File Constraints:** The ingestion engine accepts standard text-based PDF files up to a certain size and flags scanned image files that lack an embedded text layer.

## Acceptance

### Feature F-03: Real-Time Candidate Search & Status Filter
* **Ubiquitous:** The system shall display filter controls (Search by Candidate Name, Filter by Status: All / Eligible / Ineligible) above the evaluated candidate list.
* **Event-driven:** When a recruiter types into the search box, the system shall immediately update the rendered candidate list to show only candidates whose names match the search query (case-insensitive).
* **State-driven:** While the status filter dropdown is set to "Eligible" or "Ineligible", the system shall only display candidate records matching that selected status.
* **Unwanted behavior:** If no candidate records match the active filter criteria, then the system shall display "No candidates match the selected filters." in the candidate list area without throwing console errors.

* **Ubiquitous:** The system shall execute candidate review and navigation without calculating or displaying an automated percentage match score.
* **Event-driven:** When the recruiter presses navigation hotkeys (`J` or `K`), the system shall render the corresponding candidate resume within 200 milliseconds.
* **Event-driven:** When an uploaded resume fails any of the four configured knockout parameters, the system shall mark the record as Ineligible within a few seconds.
* **Unwanted:** If an uploaded PDF contains no embedded text layer, then the system shall flag the file as Unreadable and prompt for manual review.
* **Unwanted:** If a candidate enters a job ID that does not exist on the status portal, then the system shall display an Invalid job message and link to the main career page.
* **State-driven:** While in Search Mode, the system shall highlight sentences containing verified action verbs paired with technical tools and visually dim isolated skills banks.
* **Optional:** Where an applicant includes a direct hyperlink to an engineering portfolio or CAD repository, the system shall render an external link button in the viewer header.

## Handoff reflection
A peer career advisor reviewed this specification to verify whether a developer could implement the research interface without extra context. The reviewer pointed out ambiguity regarding how the text reader separates an "isolated skills bank" from technical skills mentioned within a project bullet, as well as what specific syntax library defines an "action verb." I revised the Scope and Behavior sections to clarify that text inside sections labeled "Projects" or "Work Experience" is evaluated for tool-verb pairings, while text under standalone "Skills" headings is ignored. A remaining limit is that the document does not define a comprehensive glossary of non-standard engineering tool acronyms like mapping "CREO" and "Pro/ENGINEER" to CAD, which a developer will need to establish before building the snippet highlighter.

## Verification

Walk every statement against the deployed page. Mark each PASS, FAIL, CANNOT TEST YET, or DEFERRED with a reason.

| Statement | HW3 verdict | HW4 verdict | Reason |
|---|---|---|---|
| Return entries in order | PASS | PASS | `GET /entries` queries `ORDER BY id ASC`, rendering records chronologically. |
| Store valid entry | PASS | PASS | `POST /entries` writes parameters via `.bind()` to D1 and returns 201. |
| Reject missing fields (HTTP 400) | PASS | PASS | Sending empty field returns 400 with `Missing required evaluation fields`. |
| Survive cleared cache | CANNOT TEST YET | PASS | Evaluated candidate entries reload from D1 in a new Incognito browser tab with empty cache. |
| Server unreachable / Network drop | CANNOT TEST YET | PASS | Tested with offline network throttling in DevTools; page displays `Network error: Unable to connect to backend server` with 0 console errors. |
| Server returns 500 | CANNOT TEST YET | CANNOT TEST YET | D1 schema constraints handle current loads cleanly; simulating unhandled cloud runtime crashes requires fault-injection tools not yet in stack. |
| Second client writes to same table | DEFERRED | DEFERRED | Multiple clients write concurrently to D1, but real-time push synchronization across active tabs is deferred to WebSockets in a future milestone per ADR-002. |