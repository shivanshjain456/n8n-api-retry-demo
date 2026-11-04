# Synthetic API rate-limit repair diagnostic

A runnable, synthetic before-and-after demonstration of a bounded HTTP retry change. This is not client work, revenue evidence, a production integration, or an n8n editor walkthrough.

## Failure and bounded change
A GET lookup encounters one HTTP 429 response. Without retry settings the workflow fails. The repaired HTTP Request node enables Retry On Fail, Max Tries 3 and Wait Between Tries 1100 ms. The fixture advertises a 1-second Retry-After value. The same timeout of 3000 ms remains in both workflows. No credentials, payload mapping, external endpoint or downstream side effect changes.

This fixed delay is appropriate only for the agreed synthetic fixture. It does NOT dynamically read Retry-After, implement exponential backoff/jitter, or fix an exhausted daily quota. For a real provider, confirm its rate-limit contract and allowed retry timing first.

## Run
Requires an existing n8n 1.121.3 installation and a disposable test profile. The harness uses Node built-ins only. Set N8N_BIN to the n8n executable (default: n8n on PATH) and N8N_TEST_USER_FOLDER to a disposable writable test user folder (default: .test-profile beside the script), then run `node run.cjs`.

The harness binds an ephemeral port on 127.0.0.1, generates inactive workflow copies, imports them and executes each through n8n CLI. Generated exports reference that particular loopback port; regenerate with the harness before reusing. Import never activates a trigger. Test IDs use the synthetic-api- prefix and may overwrite these same synthetic workflows in the chosen profile. Never point it at a client profile. It closes the local listener afterward. No n8n server or publicly reachable webhook is started.

## Acceptance and evidence
- Baseline transient 429: failure after one request.
- Repaired transient 429: success after two requests, one output record.
- Healthy endpoint: success after one request, unchanged output schema.
- Persistent 429: bounded failure after three requests; no invented successful result.
- 401 unauthorized: failure after three requests, exposing the broad retry behavior. This is a diagnostic limitation, not an authentication fix.
- Every repeated attempt must be separated by at least one second, checked against the HTTP server's observed timestamps.

`evidence/summary.json` contains asserted outcomes; per-case execution and import logs preserve n8n output, with installation and disposable profile paths replaced by [N8N_INSTALL] and [TEST_PROFILE]; `evidence/requests.json` preserves server-observed method, path, timestamps, count and statuses. Read all three together. Some n8n CLI aggregate status values can remain 'running'; successful assertions use finished=true, HTTP node success and exact output, plus process exit code.

## Operational safeguards
Use automatic retries only after identifying duplicate-side-effect risk. This demo uses a synthetic GET with no mutation. Do not copy these settings onto POST/payment/email/create-record actions without the provider's idempotency contract, authorization and a safe test. The node retries all failures, including this fixture's 401. For selective retries, variable Retry-After, long quota windows or multiple items, agree a separate explicit branch/Wait design and tests rather than promising this one-node change solves them.

## Rollback
Restore the original HTTP node settings from the baseline export: remove retryOnFail, maxTries and waitBetweenTries, preserving the authorized endpoint and all other fields. Confirm workflow identity and backup before client-side changes. The local fixture exports are not client deployment artifacts.

## Source references checked 2026-11-04
- https://docs.n8n.io/integrations/builtin/handle-rate-limits.md
- https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/common-issues/
The official documentation describes node retries and fixed waits. Test evidence, rather than current documentation alone, establishes behavior on the recorded installed version. No latest-version compatibility claim.
