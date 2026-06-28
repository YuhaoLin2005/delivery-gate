# I Run DeepSeek on Claude Code — Here's How I Swap Models by Changing Only One File

> Most CLAUDE.md files are 500-line monoliths. When you switch LLMs, you rewrite everything. I built a three-layer architecture that makes model swaps trivial. Here's how.

---

## The Problem

I use DeepSeek V4 Pro as my daily driver for Claude Code. But sometimes I need Claude Opus for complex reasoning, or Sonnet for fast iterations.

Every time I swapped, I had to rewrite my CLAUDE.md because DeepSeek needs different behavior tuning than native Claude. DeepSeek is more creative but less consistent with tool calls. Claude Opus is precise but verbose. The same rule file can't serve both.

After the third rewrite, I realized: **the problem isn't the models. It's that I'm mixing identity, behavior, and process rules in one file.**

## The Architecture

Three files. One changes. That's it.

```
SOUL.md        ← Who I am (identity, goals, growth). NEVER changes
INTERFACE.md   ← How this brain works (model-specific tuning). ONLY file that changes
BODY.md        ← What I do (process, reviews, delivery gates). Model-agnostic
```

### SOUL.md — The Identity Layer

This is who the AI assistant IS. What's your role? What are you building? What are your long-term goals?

```markdown
# SOUL
## Identity
- Python backend developer, functional style preferred
## Goals  
- Learning Rust and system design
- Shipping a production SaaS by Q3
```

This file stays identical regardless of whether you're running Claude, DeepSeek, Gemini, or Qwen. It's your digital twin's soul — model-agnostic identity.

### INTERFACE.md — The Brain Layer

This is the ONLY file that changes when you swap models. Each model has different quirks that need compensating:

```markdown
# INTERFACE — DeepSeek V4 Pro
## Behavior Calibration
- TOOL: Match param names exactly; after 2 failures, change strategy
- OUTPUT: Split >500 words into sections
- VERIFY: Post-edit verification is mandatory
- CONTEXT: Reconfirm info >5 turns old
```

When I switch to Claude Opus, I swap in:

```markdown
# INTERFACE — Claude Opus
## Behavior Calibration
- TOOL: Standard Anthropic tooling — no compensation needed
- OUTPUT: Long-form analysis OK, no hard split needed
- VERIFY: Standard verification, less aggressive
- CONTEXT: Full context window available
```

Same structure, different calibration values. Each model gets exactly what it needs, no more, no less.

### BODY.md — The Process Layer

Model-agnostic rules that govern HOW you work regardless of which LLM is running:

```markdown
# BODY
## Session Startup
- Health check: disk space, config files, growth log freshness
- Classify session: simple / complex / strategic

## Shutdown Sequence
1. Self-audit → 2. Teaching output → 3. Delivery gate → 4. Learning capture

## System Health
- Disk: warn at 30GB, block at 15GB
- Context: compact at 70% token usage
```

These rules work identically whether Claude Opus or DeepSeek V4 is the engine. They're process, not personality.

## The Self-Model Loop

There's one more piece. SOUL.md isn't purely static — it evolves through a feedback loop:

```
Each session starts → reads current self-model (how I see myself)
    ↓
Session produces new experiences (failures, methodologies, capability changes)
    ↓
Session end → growth data updates the self-model
    ↓
Next session → reads an evolved self-model (a different "me")
```

This is what I call the **strange loop** — reading yourself, being influenced by what you read, producing new data, rewriting yourself. It's stored in a separate `memory/self-model.md` that evolves session over session.

## Why This Matters Beyond Multi-Model

Even if you only use one model, this separation prevents config drift:

- **Identity doesn't get buried** in 500 lines of behavior rules
- **Process rules don't accidentally get model-specific** (e.g., "split at 500 words" doesn't belong in BODY.md if only DeepSeek needs it)
- **New team members** can adopt your BODY.md without changing their INTERFACE.md

## The Migration

From monolithic CLAUDE.md to three files, it takes about 15 minutes:

1. **Audit**: Classify every line of your current CLAUDE.md as SOUL, INTERFACE, or BODY
2. **Extract**: Move each line to the right file
3. **Test**: Swap the model in INTERFACE.md, verify nothing breaks
4. **Replace**: Your CLAUDE.md becomes just an architecture diagram + `@SOUL.md @INTERFACE.md @BODY.md`

## Real Numbers

This architecture has survived:
- 4 LLM reconfigurations without identity or process drift
- 200+ sessions across 5+ projects
- 6 open-source PRs to 4 different communities (including ECC and anthropics/skills)
- All while maintaining a 5-library learning capture system and 7 automated hooks

It runs on a Dell G15 with a DeepSeek V4 Pro backend. One line changes when I swap models. Everything else stays.

---

*The three-layer architecture is available as a Claude Code skill: [claude-md-architecture](https://github.com/anthropics/skills/pull/1365). The full configuration system is open-source.*
