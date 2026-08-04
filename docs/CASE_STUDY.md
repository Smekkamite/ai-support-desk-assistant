# Case study: AI Support Desk Assistant

## Project summary

AI Support Desk Assistant is a 12-node n8n workflow for controlled SaaS support triage. It receives a ticket, uses an LLM to classify and summarize it, records the decision in Google Sheets, retrieves an approved Knowledge Base article, and creates a Gmail draft only when explicit workflow checks pass. Tickets that do not meet those conditions return a structured human-review response.

The portfolio scope was completed on 4 August 2026. The result is a stable demonstration rather than a production support platform.

## The problem

Support teams spend time repeatedly classifying requests, evaluating urgency, locating internal guidance, and preparing routine replies. A fully autonomous solution would introduce unnecessary risk because model output can vary, tickets may be ambiguous, and customer-facing communication requires accountability.

The project therefore focused on a narrower question:

> How can AI reduce repetitive support work while keeping routing decisions visible and customer communication under human control?

## The implemented solution

The workflow separates interpretation from control:

1. A webhook receives a structured support ticket.
2. The payload is normalized for downstream nodes.
3. An LLM returns structured triage data.
4. Google Sheets records the triage decision for inspection.
5. n8n applies deterministic status and confidence checks.
6. Google Sheets supplies an active Knowledge Base article when available.
7. A second LLM drafts a response using that article as its source.
8. Gmail saves the response as a draft; it is never sent automatically.
9. Failed confidence or knowledge checks return a human-review result.

## Key engineering decisions

### Keep probabilistic analysis separate from deterministic routing

The LLM interprets the ticket, but n8n decides which branch runs. This makes the control flow visible and prevents natural-language recommendations from silently becoming workflow policy.

### Create drafts instead of sending messages

The Gmail integration deliberately stops at draft creation. A human can inspect the response before it reaches a customer, reducing the impact of incomplete context or incorrect model output.

### Ground responses in an approved Knowledge Base

The response model receives a matching active article rather than being asked to answer from general model knowledge. If the workflow cannot find an article with a usable answer, it chooses human review.

### Use Google Sheets for a demonstrable prototype

Sheets makes audit records and Knowledge Base entries easy to inspect during a demonstration. The trade-off is that it is not a transactional or high-volume datastore.

### Document the real workflow, not an idealized version

The repository distinguishes implemented behavior, current limitations, and future improvements. Authentication, retries, automated tests, and production monitoring are roadmap items rather than claimed capabilities.

## Main challenges

### Constraining model output

Downstream automation needs predictable fields. The triage and response steps therefore use structured JSON output instead of free-form text.

### Designing a safe fallback

A low-confidence result or missing Knowledge Base entry must not continue to Gmail draft creation. Separate branches provide an explicit human-review outcome.

### Preserving traceability

The workflow records triage information before routing. This makes decisions inspectable even when the ticket does not reach the automatic-draft path.

### Publishing a reusable workflow safely

The public export excludes credentials, Google document URLs, pinned execution data, and n8n instance identifiers. Example payloads use synthetic customer data.

## Outcome

The completed portfolio project demonstrates:

- visual workflow orchestration with n8n;
- structured LLM integration;
- deterministic guardrails around probabilistic output;
- retrieval of approved operational knowledge;
- audit logging and explicit fallback behavior;
- human-in-the-loop customer communication;
- honest documentation of trade-offs and limitations.

The workflow logic did not need additional features to meet its portfolio objective. The final phase focused on making the implementation understandable, reproducible, and auditable for another engineer or recruiter.

## What I learned

- AI output is most useful when it is bounded by explicit workflow rules.
- A safe automation can produce drafts and decisions without performing irreversible actions.
- Error behavior and non-goals are part of the design, not documentation afterthoughts.
- A public workflow needs sanitization and examples as much as it needs functional nodes.
- Clear engineering decisions make a small system easier to evaluate than a larger but unexplained demo.

## Future work

The existing roadmap covers the next production-oriented steps: authenticated webhook ingestion, input validation, bounded retries, dead-letter handling, automated routing tests, persistent human-review queues, and operational monitoring.

These improvements are intentionally outside the completed portfolio scope and are tracked as GitHub issues.
