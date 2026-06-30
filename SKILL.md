---
name: delivery-gate
description: Dual-layer mechanical Stop hook — config-health (process monitor, soft) + quality-gate (output enforcer, hard). Deterministic checks only: regex markers, file timestamps, disk space. Zero AI inference.
---

# Delivery Gate

A **dual-layer mechanical gate** for Claude Code Stop hooks. Two Python scripts, zero dependencies, deterministic checks only.

```
config-health（Process Monitor）→ tracks rule execution → soft feedback (never blocks)
         ↓
quality-gate（Output Enforcer）→ checks output completeness → hard block (≥3 stale → exit 2)
         ↓
       Delivery
```

## When to Activate

- Any project where you want Claude to consistently capture learning across sessions
- Teams that want mechanical enforcement of output quality (not just trust the AI's word)
- Solo developers who want the system to remind them, not nag them

## Installation

```bash
# Copy both scripts into your Claude Code scripts directory
cp config-health.py ~/.claude/scripts/
cp quality-gate.py ~/.claude/scripts/
```

Add to `~/.claude/settings.json` (order matters — config-health first):

```json
{
  "hooks": {
    "Stop": [
      {
        "type": "command",
        "command": "python3 ~/.claude/scripts/config-health.py --hook",
        "timeout": 5000
      },
      {
        "type": "command",
        "command": "python3 ~/.claude/scripts/quality-gate.py",
        "timeout": 5000
      }
    ]
  }
}
```

Manual health dashboard:
```bash
python3 ~/.claude/scripts/config-health.py --check
```

## How It Works

### config-health.py — Process Monitor (soft, never blocks)

| Check | Mechanism | On Hit |
|-------|-----------|--------|
| Rule marker counting | Regex `[✓THINK]` `[✓CONTEXT]` `[✓DELIVERY]` in transcript | Log to JSONL, update pending-verifications.md |
| Verification tracking | 3-session window per rule | Verified → auto-remove from pending list |
| Config integrity | Core files exist + non-empty | Dashboard alert |
| Session cost tier | Cumulative sessions count | Dashboard tier (L0-L3) |

### quality-gate.py — Output Enforcer (hard, blocks when stale)

| Check | Mechanism | On Hit |
|-------|-----------|--------|
| Stale learning libraries | File modification timestamps | Block if ≥3 stale OR growth-log stale after complex task |
| Disk space < 15GB | `shutil.disk_usage` | Block (exit 2) |
| Rationalization patterns | Regex on transcript tail | Warning only (never blocks alone) |

### The Dual-Layer Boundary

The boundary between soft and hard is not "importance" — it's **"can this be fixed later?"**

- Missed rule markers → can retroactively mark and count → **soft**
- Missed output records → lost forever → **hard**

## Customization

### config-health.py
- `RULES` — rules to track with regex markers
- `VERIFICATION_WINDOW` — sessions needed for a rule to "pass"
- `MIN_TOOL_CALLS_FOR_CHECK` — minimum tools to consider session complex
- `MIN_EDITS_FOR_DELIVERY` — minimum edits to expect DELIVERY marker

### quality-gate.py
- `RATIONALIZE` — regex patterns for rationalization detection
- `LIBS` — files/dirs to check for today's updates
- `COMPLEX_THRESHOLD` — Edit/Write calls to classify as complex
- `DISK_CRIT_GB` — block below this

## Examples

### Normal path — silence

```
Session: 2 edits, growth-log updated, rules followed
→ config-health: 85% marker rate, wrote to JSONL (exit 0, no stdout)
→ quality-gate: 1/5 stale (exit 0, no stdout)
→ Delivery. Zero tokens consumed.
```

### Rule drift — surfaced, not blocked

```
Session: rules followed but THINK marker rate dropped to 40%
→ config-health: wrote "THINK: 40%" to pending-verifications.md (exit 0)
→ quality-gate: all libs fresh (exit 0)
→ Delivery. Next session: BODY.md surfaces pending item → AI pays attention.
```

### Output incomplete — BLOCKED

```
Session: 5 edits, nothing written to memory
→ config-health: normal (exit 0)
→ quality-gate: 5/5 libs stale → exit 2
  stderr: "BLOCK: 5 libraries stale (threshold: ≥3)"
→ Claude CANNOT finish. Must update libraries first.
```

## Limitations

The mechanical gate enforces **habits** and **completeness**, not content quality. It checks that you captured learning, not whether the learning is correct. For reasoning quality, pair with [self-audit](https://github.com/gategrow/self-audit).

## Compatibility

- Python 3.8+
- Windows, macOS, Linux
- Zero dependencies beyond stdlib

## Related

- [checkgrow](https://github.com/gategrow/checkgrow) — The methodology behind the mechanical gate: failure patterns, hybrid architecture, T-CBB convergence
- [self-audit](https://github.com/gategrow/self-audit) — Four-dimension reasoning quality audit (C/C/G/H)
- [dual-pool-review](https://github.com/gategrow/dual-pool-review) — Multi-persona adversarial review methodology

## License

MIT
