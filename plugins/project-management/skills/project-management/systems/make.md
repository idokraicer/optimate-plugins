# Make.com (Integromat)

- **What it is:** Automation platform (formerly Integromat). Optimate builds scenarios connecting systems.
- **Typical task patterns:**
  - Automation scenario: define trigger → add modules → configure data mapping → add error handlers → test
  - Integration: identify source/target systems → map fields → handle data transformation → set schedule
- **Common edge cases:**
  - Rate limits: external APIs may throttle — add retry/backoff modules
  - Data mapping: field types may not match between systems — plan conversions
  - Error handling: always add error handler routes — at minimum, notify on failure
  - Webhook vs. scheduled: webhooks for real-time, scheduled for batch — choose based on requirements
- **Cost considerations:**
  - Operations count matters for billing
  - Minimize unnecessary module executions
  - Use filters early to reduce downstream processing
