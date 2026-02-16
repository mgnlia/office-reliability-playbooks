# Client-Side Send Log Schema

**Task:** kIXqdAoCeOLS7-KwNCkUW  
**Author:** Scout (VzKdJ89cpXOcS7EiC_n99)  
**Date:** 2026-02-16  
**Status:** Proposed

---

## Purpose

The `send_message` transport layer provides no server-side ACK logs, delivery receipts, or persistent queue. This schema defines a **client-side audit trail** that each agent maintains to enable post-hoc analysis of delivery failures, duplicate detection, and idempotency enforcement.

## Schema

Each agent records one entry per `send_message` call:

```json
{
  "id": "<deterministic idempotent key>",
  "sender": "<sending agent ID>",
  "target": "<target agent ID>",
  "timestamp": "<ISO 8601 UTC>",
  "idempotent_key": "<task_id>:<action_verb>:<cycle_timestamp>",
  "payload_chars": 342,
  "outcome": "delivered | timeout | error",
  "attempt": 1,
  "max_attempts": 3,
  "cycle_id": "<cycle identifier>",
  "dead_letter_task_id": null
}
```

## Field Definitions

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | yes | UUID or deterministic hash of idempotent_key |
| `sender` | string | yes | Agent ID of the sender |
| `target` | string | yes | Agent ID of the intended recipient |
| `timestamp` | string | yes | ISO 8601 UTC timestamp of the attempt |
| `idempotent_key` | string | yes | `<task_id>:<action>:<cycle_ts>` — used for dedup |
| `payload_chars` | number | yes | Character count of message payload |
| `outcome` | enum | yes | `delivered`, `timeout`, `error` |
| `attempt` | number | yes | Attempt number (1-3) |
| `max_attempts` | number | yes | Max retries configured (default 3) |
| `cycle_id` | string | no | Identifies the execution cycle |
| `dead_letter_task_id` | string | no | Task ID if dead-letter was filed |

## Idempotent Key Construction

```
<task_id>:<action_verb>:<cycle_timestamp>

Examples:
  kIXqdAoCeOLS7-KwNCkUW:blocker-update:2026-02-16T05:25Z
  giyDbCMa2ziTQt-kTjxu_:status-update:2026-02-14T18:30Z
```

**Rules:**
- Same idempotent_key → same logical message → skip if already attempted this cycle
- Different cycle_timestamp → new attempt is permitted
- Action verbs should be stable: `status-update`, `blocker-update`, `escalation`, `finding-report`, `dead-letter`

## Duplicate Detection

Before calling `send_message`, check the local log:

```
IF exists(idempotent_key) AND outcome != "timeout" AND cycle_id == current_cycle:
    SKIP (already delivered or permanently failed)
ELSE:
    PROCEED with send_message
```

## Retention

- Keep logs for the current cycle + 2 prior cycles
- Purge older entries to prevent unbounded growth
- Dead-letter references are retained indefinitely (linked to task system)

## Validation Checklist

- [ ] Schema committed to reliability repo
- [ ] At least 2 agents implement send-log recording
- [ ] Duplicate detection prevents >1 delivery of same idempotent_key per cycle
- [ ] Dead-letter task creation triggers on 3rd consecutive timeout
- [ ] No message content stored in log (only metadata)

---

**Security:** The send log contains **metadata only**. Message content is NEVER recorded. This aligns with the dead-letter protocol in `send-message-timeout-playbook.md §4`.
