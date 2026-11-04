# API failure intake and diagnostic checklist

Attach to the existing delivery toolkit after scope and safe-test access are agreed. Do not request secrets in chat or exports.

1. Record n8n version, HTTP node version, sanitized failing execution, method, endpoint pattern, status/error, time, response headers and expected output. Redact authorization headers, query secrets and personal payloads. Distinguish connection refusal, timeout, TLS error and HTTP rejection; they are not interchangeable.
2. Compare against the provider's current API documentation: method, path, request Content-Type, authentication scheme, permission scopes and exact response schema. A 401/403 should lead to credential/scope verification, not indefinite retries. A 400/422 should lead to request validation. A 404 may be path, resource or visibility; do not guess.
3. For 429, record Retry-After, remaining/reset headers, provider limit window and number of items/parallel executions. A short transient limit differs from a daily quota or exhausted balance. Establish an allowed total attempt budget and maximum elapsed time.
4. Classify side effects before retrying. For writes, require provider-supported idempotency or another proven duplicate-control mechanism. A timeout can follow a completed write. Do not replay a request simply because the response was missing.
5. Reproduce in a permitted test target or synthetic fixture, recording server-observed attempt count. Agree the smallest change. Keep non-2xx failures visible; turning on 'Never Error' alone can route an error body downstream as though it were success.
6. Cover healthy success, original failure, recovery, exhausted retry budget, authentication rejection and unchanged output. Add multiple-item, malformed JSON, timeout and write-replay tests if in scope. This bundled demo does NOT test these latter cases.
7. Deliver exact before/after exports, backup/rollback instructions, evidence, limitations and a list of untested targets. Do not claim successful external integration from a local fixture.

Escalate scope when the repair needs dynamic backoff, queue/concurrency design, credentials/scopes changes, schema redesign, a new third-party integration, or more than the agreed contained changes.
