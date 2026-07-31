# Validation guide

This checklist validates the behavior that the public workflow currently implements. It does not claim production reliability or complete test automation.

## Before testing

- Import `workflow/AI Support Desk Assistant.json`.
- Configure both AI message-model nodes.
- Configure the Audit Log and Knowledge Base Google Sheets nodes.
- Configure the Gmail draft node.
- Create the required sheet columns documented in the README.
- Use synthetic customer data only.
- Keep the workflow inactive until test execution is complete.

Record the n8n version, model provider, model ID, test date, and execution ID with each result. LLM output may vary between runs.

## Case 1 — Knowledge-grounded Gmail draft

### Setup

1. Add an active `PASSWORD_RESET` row to the Knowledge Base.
2. Send `examples/sample-ticket.json` to the n8n test webhook.

### Verify

- `Normalize Ticket` contains all seven expected fields.
- `AI Ticket Triage` returns JSON.
- `Log Triage Decision` appends one audit row.
- `Safe for Auto Response?` takes the true route only when status is `OK` and confidence is at least `80`.
- `Knowledge Base Lookup` returns the active row selected by the triage knowledge key.
- `Knowledge Found?` takes the true route when `Answer` is present.
- Gmail contains a draft addressed to `customer@example.com`.
- No email is sent.
- The webhook response follows `examples/sample-auto-draft-response.json`.

## Case 2 — No Knowledge Base match

### Setup

1. Use a ticket whose selected knowledge key has no active matching row, or temporarily deactivate the matching synthetic row.
2. Send the ticket to the n8n test webhook.

### Verify

- The triage decision is still logged.
- `Knowledge Base Lookup` emits an empty item.
- `Knowledge Found?` takes the false route.
- No Gmail draft is created.
- The webhook response follows `examples/sample-human-review-response.json`.

## Case 3 — Confidence gate

The triage model may not naturally produce a predictable confidence score. Test this gate by pinning synthetic triage output or executing the downstream branch with controlled test data.

### Verify

- Confidence `79` takes the false route.
- Confidence `80` takes the true route when status is `OK`.
- Any status other than `OK` takes the false route.
- The recommendation fields do not currently affect this gate.

## Case 4 — Missing input

Send a synthetic request with one expected field omitted.

### Verify

- The workflow has no explicit input-validation response.
- The missing value is visible during normalization or causes a downstream failure.
- The observed behavior is recorded as a current limitation, not described as handled.

## Case 5 — Provider or OAuth failure

Use an n8n test environment and a deliberately invalid test credential only if it is safe to do so.

### Verify

- The failing node stops the execution.
- No automatic retry occurs.
- No success webhook response is returned.
- No claim of automatic recovery is added to the documentation.

## Evidence checklist

For a portfolio validation run, retain sanitized evidence of:

- the complete workflow execution;
- the route taken by each IF node;
- the appended audit row;
- the Gmail draft;
- the final webhook response;
- the n8n version and test date.

Never publish real customer data, credential names, access tokens, Google document identifiers, or production webhook URLs.
