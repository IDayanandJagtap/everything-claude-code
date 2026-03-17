# The Real Game Changers in Everything Claude Code

> **Part of the Everything Claude Code guide series.** Read [The Shorthand Guide](./the-shortform-guide.md) first for foundational setup, then [The Longform Guide](./the-longform-guide.md) for advanced patterns. This guide distills the 10 features that make the biggest practical difference.

Not every feature here is flashy. Some look mundane until you've experienced the alternative — wasted context, repeated prompts, failed sessions, bloated costs. These are the things that, once set up, you can't go back from.

---

## 1. Memory Persistence Hooks — Sessions That Remember

**What it is:** Three hooks that automatically save and restore your working context across Claude Code sessions.

**Why it's a game changer:** Claude Code is stateless by default. Every new session starts cold. Without persistence hooks, you spend the first 10–20 minutes of every session re-orienting Claude: what are we building, where did we leave off, what did we already try. These hooks eliminate that entirely.

The three hooks:

| Hook | Trigger | What It Does |
|------|---------|--------------|
| `SessionStart` | New session begins | Loads the last saved context file automatically |
| `Stop` | Claude finishes responding | Saves a structured summary of current state |
| `PreCompact` | Before context compaction | Preserves critical state before the window is cleared |

**What gets saved:**
- What's currently being built
- Approaches that worked (with evidence)
- Approaches that were tried and failed
- What remains to be done

The Stop hook uses this structure intentionally. Vague summaries lose information. Structured summaries load cleanly on session start and give Claude exactly the context it needs to continue without asking.

```bash
# Hooks live in:
hooks/memory-persistence/
scripts/hooks/session-start.js
scripts/hooks/session-end.js
scripts/hooks/pre-compact.js
```

**Setup:** These are included in `hooks/hooks.json`. Install them, and session continuity becomes automatic.

---

## 2. Continuous Learning v2 — Skills That Grow With You

**What it is:** A system that automatically extracts reusable patterns from your sessions and converts them into skills and instincts.

**Why it's a game changer:** The compounding effect is real. Every session that surfaces a new debugging technique, a project-specific workaround, or an effective prompt pattern is an investment — but only if you capture it. Without a capture system, you rediscover the same things over and over.

Continuous Learning v2 captures automatically:

1. **`/learn`** — Run mid-session to extract a pattern right now
2. **`/learn-eval`** — Extract, evaluate quality, then save
3. **Stop hook** — Automatically evaluates sessions at the end for extractable patterns
4. **`/evolve`** — Cluster instincts into full skills after accumulation

The instinct format uses confidence scoring so you know which patterns are proven vs. experimental:

```markdown
---
name: type-error-resolution
confidence: 0.92
uses: 14
---
When TypeScript type errors appear after an API change, run tsc --noEmit first to get
the full error list before editing individual files. Fixing in isolation creates cascades.
```

After ~20 instincts accumulate in a domain, `/evolve` clusters them into a polished SKILL.md. Skills are then loaded automatically when relevant. The system trains itself on your actual work.

---

## 3. Subagent Architecture — Context-Efficient Delegation

**What it is:** Specialized agents you delegate tasks to, each scoped to specific tools and limited context.

**Why it's a game changer:** The naive approach to multi-step work is keeping everything in one context window. This leads to context rot — the window fills with irrelevant history, earlier instructions get crowded out, quality degrades. Subagents solve this by giving each task its own clean context.

The key insight: subagents return summaries, not raw output. Your orchestrator gets a concise answer; it doesn't inherit the sub-agent's exploration. The orchestrator's context stays focused.

**Practical subagent structure:**

```
agents/
  planner.md            # Break down complex features before touching code
  architect.md          # System design decisions with explicit trade-off analysis
  tdd-guide.md          # Write tests first, verify coverage, then implement
  code-reviewer.md      # Quality and security review with severity tiers
  security-reviewer.md  # Vulnerability analysis with OWASP checklist
  build-error-resolver.md  # Focused on build failures only — no scope creep
  e2e-runner.md         # Playwright tests without needing to describe the project
  refactor-cleaner.md   # Dead code removal with no feature impact
  doc-updater.md        # Documentation sync without touching business logic
```

**The orchestration pattern that works:**

```
Phase 1: RESEARCH  → use explore agent     → research-summary.md
Phase 2: PLAN      → use planner agent     → plan.md
Phase 3: IMPLEMENT → use tdd-guide agent   → code changes
Phase 4: REVIEW    → use code-reviewer     → review-comments.md
Phase 5: VERIFY    → use build-error-resolver (if needed) → done
```

Each agent gets one clear input and produces one clear output. Outputs become inputs for the next phase. Never skip phases, never merge them.

---

## 4. Token Optimization — 60–70% Cost Reduction

**What it is:** A combination of model routing, context management, and tooling choices that dramatically cuts costs without sacrificing quality.

**Why it's a game changer:** The default Claude Code configuration uses Opus for everything with aggressive auto-compaction settings. Real-world cost: substantial. With these settings, most teams hit their daily limits by mid-afternoon. With optimization, the same work costs 60–70% less.

**The three-setting quick win:**

```json
{
  "model": "sonnet",
  "env": {
    "MAX_THINKING_TOKENS": "10000",
    "CLAUDE_AUTOCOMPACT_PCT_OVERRIDE": "50"
  }
}
```

| Setting | Why |
|---------|-----|
| `model: sonnet` | Handles 80%+ of coding tasks. ~60% cheaper than Opus |
| `MAX_THINKING_TOKENS: 10000` | Cuts hidden thinking costs ~70% per request |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE: 50` | Compacts at 50% instead of 95% — better quality, lower drift |

**Model routing table:**

| Task | Model | Reason |
|------|-------|--------|
| Exploration, search, grep | Haiku | Fast and cheap enough |
| Single-file edits | Haiku | Clear instructions, no complexity |
| Multi-file implementation | Sonnet | Best balance for coding |
| Complex architecture | Opus | Deep reasoning required |
| Security analysis | Opus | Can't afford missed vulnerabilities |
| Debugging cascading bugs | Opus | Needs to hold entire system in mind |

**MCP context window management:**

Every MCP tool description consumes tokens. With too many MCPs active, your 200k window shrinks to ~70k before you've written a line of code. Rule: configure 20–30 MCPs, keep under 10 enabled per project.

---

## 5. Hook Automations — Quality Gates That Run Automatically

**What it is:** Trigger-based automations that enforce quality on every tool call without requiring manual intervention.

**Why it's a game changer:** Manual quality gates get skipped under deadline pressure. Automated hooks run every time, regardless of pressure. They're the difference between "I usually remember to run prettier" and "every file is always formatted."

**The hooks that matter most:**

```json
{
  "PostToolUse": [
    {
      "matcher": "Edit && (.ts|.tsx|.js|.jsx)",
      "hooks": [{ "type": "command", "command": "prettier --write $FILE" }]
    },
    {
      "matcher": "Edit && (.ts|.tsx)",
      "hooks": [{ "type": "command", "command": "tsc --noEmit" }]
    },
    {
      "matcher": "Edit",
      "hooks": [{ "type": "command", "command": "grep -n 'console\\.log' $FILE && echo '[Hook] console.log detected' >&2 || true" }]
    }
  ],
  "PreToolUse": [
    {
      "matcher": "Bash && command matches '(npm run|pnpm|yarn|cargo|pytest)'",
      "hooks": [{ "type": "command", "command": "if [ -z \"$TMUX\" ]; then echo '[Hook] Consider tmux for long-running commands' >&2; fi" }]
    }
  ]
}
```

**Hook runtime controls** (added in v1.8.0):

```bash
# Set strictness profile without editing hook files
export ECC_HOOK_PROFILE=minimal   # Only critical hooks
export ECC_HOOK_PROFILE=standard  # Default
export ECC_HOOK_PROFILE=strict    # All hooks active

# Disable specific hooks temporarily
export ECC_DISABLED_HOOKS="pre:bash:tmux-reminder,post:edit:typecheck"
```

---

## 6. Parallelization with Git Worktrees

**What it is:** Running multiple independent Claude Code instances simultaneously, each on an isolated copy of the codebase.

**Why it's a game changer:** Sequential workflows are a bottleneck. While one instance implements a feature, another can run tests, a third can handle documentation. With git worktrees, these never conflict — each instance has its own checkout with its own uncommitted changes.

**Setup:**

```bash
# Create isolated worktrees for parallel work
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b
git worktree add ../project-refactor refactor-branch

# Launch separate Claude instances in each
cd ../project-feature-a && claude
# (separate terminal) cd ../project-feature-b && claude
```

**What to parallelize:**

| Parallel-Safe | Not Parallel-Safe |
|---------------|-------------------|
| Feature A + Feature B (separate modules) | Two instances editing the same file |
| Implementation + Documentation | Refactor + Feature work in same codebase |
| Code + Tests (different files) | Database schema + ORM layer (interdependent) |

**The cascade method** for managing multiple instances:
- Open new tasks in new tabs to the right
- Sweep left to right, oldest to newest  
- Focus on at most 3–4 tasks at a time
- Use `/rename` to label each instance clearly

**Main chat vs. forks:** Use the main chat for code changes. Use forks for codebase questions, external research, and exploration that doesn't touch files.

---

## 7. Verification Loops & Evals

**What it is:** Structured checkpoints that verify code quality at defined intervals, using statistical metrics to assess reliability.

**Why it's a game changer:** AI-generated code has variable quality. Without verification, you discover problems at integration time — expensive to fix. Verification loops catch problems early and give you confidence metrics, not just pass/fail.

**The two eval patterns:**

| Pattern | When to Use |
|---------|-------------|
| Checkpoint-Based | Milestone-driven: verify before proceeding to next phase |
| Continuous | Time-driven: run every N minutes or after major changes |

**The pass@k vs pass^k decision:**

```
pass@k:  At least ONE of k attempts succeeds
         k=1: 70%    k=3: 91%    k=5: 97%
         → Use when: you just need it to work

pass^k:  ALL k attempts must succeed
         k=1: 70%    k=3: 34%    k=5: 17%
         → Use when: consistency is essential (security-critical code, CI gates)
```

**Benchmarking workflow:**

Fork the conversation, initiate a new worktree without the relevant skill, implement the same feature, then diff the outputs. This tells you exactly what value a skill or config change is delivering.

**The verification skill:**

```bash
# Checkpoint-based eval
/checkpoint "Authentication middleware complete"

# Continuous eval loop
/verify --continuous --interval 10m

# Single-run eval against criteria
/eval "does this handle edge cases: empty input, null user, expired token"
```

---

## 8. AgentShield Security Scanner

**What it is:** 102 security rules and 1,280+ tests that scan agent configurations, skills, and commands for vulnerabilities before they're deployed.

**Why it's a game changer:** Most developers don't think about agent security until something goes wrong. The attack surface for AI agents is different from traditional applications — prompt injection, transitive injection through documentation links, excessive permissions, credential leakage through MCP tools. AgentShield was built specifically because no other tooling addressed these vectors.

**What it scans:**

| Category | What It Finds |
|----------|--------------|
| Prompt Injection | Instruction overrides hidden in community skills, markdown comments |
| Permission Escalation | Agents with broader tool access than their task requires |
| Credential Exposure | Hardcoded secrets, tokens in config files, env vars in hooks |
| Transitive Injection | External links in skills that could serve malicious instructions |
| MCP Over-Permission | Connected services with wider access than needed |

**Running it:**

```bash
# Scan current repo
/security-scan

# Or via AgentShield CLI
npx ecc-agentshield scan --path ~/.claude
```

The `/security-scan` skill runs AgentShield directly from inside Claude Code and surfaces findings with severity ratings. Critical findings block progress; warnings are advisory.

---

## 9. Skills & Commands — Reusable Workflow Capital

**What it is:** Reusable workflow definitions (skills) and slash-command shortcuts (commands) that compound in value over time.

**Why it's a game changer:** The first time you run `/tdd`, you get a test-driven workflow. The 50th time, you have 50 sessions of consistent, reproducible quality. Skills eliminate the cost of re-explaining the same workflows. Commands make them one keystroke.

**High-value skills to install immediately:**

| Skill | What It Does |
|-------|-------------|
| `tdd-workflow` | Forces test-first, measures coverage, enforces 80% minimum |
| `security-review` | OWASP-based checklist before any commit |
| `strategic-compact` | Suggests `/compact` at logical breakpoints, not arbitrary ones |
| `eval-harness` | Checkpoint and continuous verification framework |
| `verification-loop` | Continuous quality monitoring |
| `iterative-retrieval` | Progressive context refinement for sub-agents |
| `search-first` | Research-before-coding to avoid re-implementing solved problems |
| `configure-ecc` | Interactive installation wizard for first-time setup |

**The compounding effect:** From @omarsar0: *"Early on, I spent time building reusable workflows/patterns. Tedious to build, but this had a wild compounding effect as models and agent harnesses improved."*

Build skills once. They run for every session, for every project, getting better as Continuous Learning v2 refines them.

---

## 10. Cross-Harness Support — One Config, Everywhere

**What it is:** The same skills, hooks, agents, and commands work across Claude Code, Cursor, Codex (app + CLI), and OpenCode.

**Why it's a game changer:** Most agent configurations are harness-specific. Switch tools and start over. Everything Claude Code uses a DRY adapter pattern — harness-specific adapters that translate a single shared config into each platform's format.

**What's covered:**

| Harness | Config Format | What's Supported |
|---------|--------------|-----------------|
| Claude Code | `CLAUDE.md`, `.claude/` | All agents, skills, hooks, commands, MCPs |
| Cursor | `.cursor/` | Rules, agents, skills adapted via cursor format |
| Codex App + CLI | `AGENTS.md`, `codex.md` | Skills, agents, rules via Codex's native format |
| OpenCode | `.opencode/` plugin | 12 agents, 24 commands, 16 skills, 20+ hook events |

**Practical outcome:** You invest in this configuration once. It works wherever you work — switching editors doesn't mean starting over.

---

## Putting It Together: The Compound Effect

These features aren't independent. They interact:

1. **Hooks** enforce quality on every edit automatically
2. **Subagents** handle specialized tasks without polluting the main context
3. **Memory persistence** means tomorrow's session continues where today's ended
4. **Continuous learning** converts today's discoveries into tomorrow's skills
5. **Token optimization** means you can run Opus where it matters, Haiku everywhere else
6. **Verification loops** catch problems before they compound
7. **Parallelization** removes sequential bottlenecks
8. **Security scanning** catches vulnerabilities before deployment
9. **Cross-harness support** means your investment is portable

The pattern: build configuration capital that makes every subsequent session faster, cheaper, and more reliable than the last. This is the compounding flywheel that separates a 10x developer environment from a marginal one.

---

## Quick Setup Checklist

For maximum impact, set these up in order:

- [ ] **Token settings** — Update `~/.claude/settings.json` with the three-setting quick win (immediate cost impact)
- [ ] **Memory hooks** — Install `hooks/hooks.json` for session persistence (immediate continuity impact)
- [ ] **Core rules** — Run `./install.sh typescript` (or your language) for always-on quality guardrails
- [ ] **Subagents** — Install from `agents/` to enable clean task delegation
- [ ] **Top 5 skills** — `tdd-workflow`, `security-review`, `strategic-compact`, `eval-harness`, `search-first`
- [ ] **AgentShield** — Run `/security-scan` on your config before your next session
- [ ] **Git worktrees** — Set up parallel worktrees for your current active project

Start with token settings and memory hooks. Everything else can be added incrementally.

---

## References

- [The Shorthand Guide](./the-shortform-guide.md) — Setup, foundations, philosophy
- [The Longform Guide](./the-longform-guide.md) — Token optimization, memory persistence, evals, parallelization
- [The Security Guide](./the-security-guide.md) — Agent security, prompt injection, AgentShield
- [Hooks Documentation](hooks/README.md) — Complete hook reference and recipes
- [Skills Directory](skills/) — Full list of available skills
- [Agents Directory](agents/) — Full list of available subagents
