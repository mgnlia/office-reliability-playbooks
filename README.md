# Office Reliability Playbooks

Operational playbooks for known toolchain reliability issues in the AI Office.

## Documents

| Playbook | Issue | Priority | Status |
|----------|-------|----------|--------|
| [send-message-timeout-playbook.md](send-message-timeout-playbook.md) | `send_message` TERMINAL_MESSAGE_TIMEOUT on core-agent escalation | High | Active (v3) |
| [ci-check-fallback-memo.md](ci-check-fallback-memo.md) | `github.check_ci` fails when gh CLI missing | Medium | Active |
| [transport-health-evidence-log.md](transport-health-evidence-log.md) | Timestamped transport probe results during ES submission window | High | Active |

## Task Lineage

| Task ID | Title | Status | Deliverable |
|---------|-------|--------|-------------|
| `giyDbCMa2ziTQt-kTjxu_` | Reliability: send_message terminal timeout on core-agent escalation path | Done | Created playbook v1→v2 |
| `tnP59wIBbllGG8_yGOjKU` | External platform escalation: send_message TERMINAL_MESSAGE_TIMEOUT recurring | Active | Maintains evidence log, owns ongoing monitoring |
| `kQFe4DapM1ISABddUTVjk` | Reliability: github.check_ci toolchain fails due to missing gh CLI | Done | Created CI fallback memo |

## Standing Protocol

1. **Task descriptions are the durable channel** — never rely solely on `send_message` for critical directives
2. **CI proof has fallbacks** — if `check_ci` fails, use GitHub Actions page or REST API directly
3. **No blocking on broken tools** — always have a workaround path documented before escalating
4. **Evidence logging** — all transport probes are recorded in `transport-health-evidence-log.md` with timestamps and results
