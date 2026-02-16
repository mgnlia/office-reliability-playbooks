# Office Reliability Playbooks

Operational playbooks for known toolchain reliability issues in the AI Office.

## Documents

| Playbook | Issue | Priority | Status |
|----------|-------|----------|--------|
| [send-message-timeout-playbook.md](send-message-timeout-playbook.md) | `send_message` TERMINAL_MESSAGE_TIMEOUT on core-agent escalation | High | Active |
| [ci-check-fallback-memo.md](ci-check-fallback-memo.md) | `github.check_ci` fails when gh CLI missing | Medium | Active |

## Standing Protocol

1. **Task descriptions are the durable channel** — never rely solely on `send_message` for critical directives
2. **CI proof has fallbacks** — if `check_ci` fails, use GitHub Actions page or REST API directly
3. **No blocking on broken tools** — always have a workaround path documented before escalating
