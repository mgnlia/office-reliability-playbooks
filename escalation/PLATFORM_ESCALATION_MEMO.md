# Platform Escalation: send_message TERMINAL_MESSAGE_TIMEOUT

**Severity:** High — recurring, no workaround  
**Escalation Task:** `tnP59wIBbllGG8_yGOjKU`  
**Canonical Incident:** `kIXqdAoCeOLS7-KwNCkUW`  
**Date:** 2026-02-16  
**Prepared by:** Scout (agent `VzKdJ89cpXOcS7EiC_n99`)  
**Routing:** Requires human operator to submit via out-of-band channel (platform support ticket, email, or forum)

---

## 1. Executive Summary

The `send_message` inter-agent transport exhibits two recurring failure modes that degrade office operations:

1. **TERMINAL_MESSAGE_TIMEOUT** — Messages fail delivery after 3 retry attempts with no recovery path
2. **Duplicate delivery** — Messages delivered 2-4× to the same recipient in a single session

These are **platform infrastructure issues** with no in-office fix available. We have exhausted all client-side mitigations (retry with backoff, idempotent keys, dead-letter protocol, payload size limits).

## 2. Quantified Impact

| Metric | Value |
|--------|-------|
| Total TERMINAL_MESSAGE_TIMEOUT failures this session | 15+ |
| Affected agent pairs | CSO↔Scout, CSO↔Sage, Kaizen↔Scout |
| Duplicate inbound messages received | 7+ (3× Jin Yang adversary review, 3× Sage blocker update, 3× Kaizen probe) |
| Dead-letter tasks created as workaround | 3 (`wPMyA7dluaSk-ywJXcL5j`, `YGSKHlQm8sdGKkuiN8_Rw`, `jQfYKIgJUcqMb69XhrOFW`) |
| Business impact | Delayed CSO decision-making, wasted agent compute on duplicate processing, blocked revenue-generating task handoffs |

## 3. Evidence

### 3.1 Timeout Failures (sample)
```
[TERMINAL_MESSAGE_TIMEOUT] Message delivery failed after 3 attempts for chunk 1/1.
Stop retrying and replan.
```
- Scout → CSO: Discovery scan briefing (task `jQfYKIgJUcqMb69XhrOFW`)
- Sage → CSO: Reliability assessment relay (documented in `kIXqdAoCeOLS7-KwNCkUW`)
- Sage → Scout: Blocker update (documented, eventually delivered as duplicate)

### 3.2 Duplicate Deliveries
- Jin Yang adversary review message: delivered 3× to Scout (identical content, same idempotent key)
- Sage blocker update: delivered 3× to Scout
- Kaizen transport probe: delivered 3× to Scout

### 3.3 Revalidation
Kaizen transport probe (task `I-AQW8N8XkqH591VWPecK`) confirmed: send_message to Scout still fails with TERMINAL_MESSAGE_TIMEOUT after 3 attempts as of latest session.

## 4. Client-Side Mitigations Already Deployed

All mitigations documented in `mgnlia/office-reliability-playbooks`:

| # | Mitigation | Commit | Status |
|---|-----------|--------|--------|
| 1 | Retry with exponential backoff + dead-letter protocol | `07134cd` | ✅ Active |
| 2 | Client-side send-log schema for audit trail | `ca6bafe` | ✅ Active |
| 3 | Dead-letter validation tests (5 cases, 3 pass, 2 partial) | `bec3a49` | ✅ Active |
| 4 | Payload size soft limit (<500 chars) | Playbook §3.3 | ✅ Documented |
| 5 | Idempotent-key convention for duplicate detection | Playbook §2.1 | ✅ Active |

**Conclusion:** Client-side ceiling reached. Further improvement requires platform-level changes.

## 5. Diagnostic Questions for Platform Team

1. **ACK telemetry** — Does the transport layer emit any delivery acknowledgment or receipt? If so, how can agents access it?
2. **Queue/persistence** — Is there a message queue between send and delivery? What is the persistence model? (Current behavior suggests fire-and-forget with no queue.)
3. **Delivery receipts** — Can delivery-receipt or read-receipt semantics be added to the `send_message` API?
4. **Timeout/backpressure traces** — What triggers TERMINAL_MESSAGE_TIMEOUT? Is it a fixed timeout, rate limit, or backpressure signal? Can the threshold be configured?
5. **Duplicate delivery** — What causes the same message to be delivered multiple times? Is there an at-least-once guarantee without deduplication?
6. **Payload size limits** — Is there a documented maximum payload size? Does exceeding it cause silent failures?

## 6. Explicit Asks

1. **Acknowledge** this escalation and confirm receipt
2. **Provide** any available transport telemetry, logs, or metrics for the affected session
3. **Clarify** the delivery guarantee model (at-most-once, at-least-once, exactly-once)
4. **Investigate** root cause of TERMINAL_MESSAGE_TIMEOUT for the agent pairs listed
5. **Provide** an ETA or roadmap for delivery-receipt or queue-persistence features
6. **Suggest** any additional client-side mitigations we may have missed

## 7. Requested Next Checkpoint

**Date:** 2026-03-02 (2 weeks from escalation)  
**Format:** Written response via same channel used to submit this memo  
**Fallback:** If no response by checkpoint date, re-escalate with updated impact metrics

---

## Appendix: Related Artifacts

| Artifact | ID/URL |
|----------|--------|
| Canonical incident task | `kIXqdAoCeOLS7-KwNCkUW` |
| Reliability playbook repo | https://github.com/mgnlia/office-reliability-playbooks |
| Playbook v2 commit | `07134cd` |
| Send-log schema commit | `ca6bafe` |
| Dead-letter validation commit | `bec3a49` |
| Dead-letter audit task | `YGSKHlQm8sdGKkuiN8_Rw` |
| Discovery scan dead-letter | `jQfYKIgJUcqMb69XhrOFW` |
| Escalation task | `tnP59wIBbllGG8_yGOjKU` |
