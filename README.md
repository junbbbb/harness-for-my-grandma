# Harness for My Grandma

> **Prompt Engineering → Context Engineering → Harness Engineering**
>
> How we use AI is evolving. We've moved past the era of crafting better prompts, past designing better context, and into **designing the environment in which AI works.** Built on the latest harness engineering research from [Anthropic](https://www.anthropic.com/engineering/harness-design-long-running-apps) and [OpenAI](https://openai.com/index/harness-engineering/) — their design principles (Generator-Evaluator separation, progressive context disclosure, blast-radius classification, mechanical enforcement, garbage collection) are re-engineered here for people who have never written a line of code.

**A tool that lets non-developers tell AI what to build.**

## What Is This?

Think about building a house.

The homeowner doesn't build it themselves. They say "I want a cafe here, about 20 seats, big windows." The architect draws the plans, the builders construct it, the inspector verifies quality. The homeowner sees the finished space and says "move the tables around."

This project creates the same structure for software development.

```
Building a house              This tool
────────────────              ─────────
Owner: "Build me a cafe"      You: "Build me an ordering site"
Architect: draws plans         AI: designs + implements
Inspector: checks quality      AI: automatic quality verification
Owner: "Bigger windows"       You: "Bigger photos"
```

**What you do**: Say what you want. Try the result. Give feedback.
**What AI does**: Everything else.

## Why Do You Need This?

We live in an era where you can ask AI to write code, but just asking without structure causes problems:

- AI asks technical questions you can't answer ("Should I use Redis or in-memory caching?")
- AI explains what it built using code, which you can't understand
- Simple changes go through the same long process every time
- AI builds a hollow shell of important features and says "done"

This tool solves these problems:

- Technical questions get translated into questions you CAN answer ("Is it okay if search history disappears when the server restarts?")
- Results are reported as behavior changes, not code ("Before: goes to homepage. After: goes to dashboard")
- Simple fixes are handled instantly without asking
- Critical changes get verified by a separate QA agent
- The tech stack (programming languages, tools, etc.) is automatically selected by the AI — it picks what it works best with, so you don't have to choose

## A Day in the Life: Building an Online Ordering Site for "Flour Power" Bakery

### 10:00 AM — Getting Started

```
You: /harness-init
AI:  What product do you want to build? Describe it in a sentence or two.
You: Online ordering site for my bakery. Customers pick bread, place orders, and book pickup times
AI:  A few more questions:
     1. How will you handle payments?
        [A] Pay on the website (card/digital wallet)
        [B] Order online, pay in person at pickup
You: B
AI:  Project environment is ready.
     Use /build to tell me what you'd like to create.
```

### 10:30 AM — First Feature

```
You: /build Show bread menu on the main page and let customers order
AI:  Here's what I'll build:
     - Main page displays bread list with photos, names, and prices
     - Customers can select items, set quantities, and place orders
     - Orders include a pickup date and time selection
     Shall I proceed? Let me know if you'd like to add or remove anything.
You: Looks good but make pickup times 30-minute slots only
```

*... AI builds for 3-5 minutes ...*

```
AI:  Done.
     Before: Empty page
     After: Bread menu is displayed. Ordering with pickup reservation is available.
            Pickup times are selectable in 30-minute slots between 10:00 AM and 7:00 PM.
     Try it out and let me know if anything needs changing.
```

### 11:00 AM — Feedback (Just Speak Naturally)

```
You: The bread photos are too small. Make them bigger
AI:  Done. Bread photos are now displayed 2x larger.
```

Simple changes are handled immediately — no questions asked, one-line response.

### 2:00 PM — When a Product Decision Is Needed

```
You: /build Create an admin page
AI:  While building the admin page, I need to check something.
     The admin page shouldn't be accessible to everyone.
     How should access work?

     [A] Single password — simple, anyone with the password can manage
     [B] Username/password login — separate accounts per staff member

     Recommended: A (since you're running the shop solo, simple is better)
You: A
```

Instead of asking "Should I use Redis?", the AI asks "One password or login accounts?" — a question you can actually answer.

### Next Day — What Changed?

```
You: /changes
AI:  Yesterday's changes:
     New features
       - Bread menu and ordering system added
       - Admin page for menu management and order tracking
     Fixes
       - Bread photos enlarged
```

### One Week Later — Status Check

```
You: /status
AI:  "Flour Power" Status
     Working features
       - Bread menu display: OK
       - Online ordering: OK
       - Admin menu management: OK
     Overall completion: ~70%
     Recommended next steps:
       1. /build Notify me when a new order comes in
       2. /build Show today's sales summary
```

### Periodically — Cleanup

```
You: /cleanup
AI:  Cleanup complete
     Auto-fixed:
       - Removed unused internal code
       - Added safety checks
     Recommendation:
       - /build Block orders for out-of-stock bread
```

## Installation

Requires [Claude Code](https://claude.com/claude-code).

### Option 1: Ask Claude Code to do it (easiest)

Open Claude Code and paste this one line:

```
Take the files in the commands/ folder from this repo and set them up as skills in ~/.claude/commands/:
https://github.com/junbbbb/harness-for-my-grandma
```

Claude Code will download and install them for you.

### Option 2: Manual install

```bash
# 1. Download this project
git clone https://github.com/junbbbb/harness-for-my-grandma.git

# 2. Copy the command files
cp -r harness-for-my-grandma/commands/ ~/.claude/commands/

# 3. Done! Run /harness-init in any folder to get started
```

## Command Summary

| Command | What It Does | When to Use |
|---------|-------------|-------------|
| `/harness-init` | Set up project environment | Once per project |
| `/build [what you want]` | Build or modify features | Daily |
| `/changes` | See what changed recently | Anytime |
| `/status` | View overall project health | Anytime |
| `/cleanup` | Internal code cleanup | Weekly recommended |

---

# Technical Reference (For Developers)

Design rationale and technical details below.

Built on Anthropic's [Harness Design for Long-Running Apps](https://www.anthropic.com/engineering/harness-design-long-running-apps) and OpenAI's [Harness Engineering](https://openai.com/index/harness-engineering/), re-engineered for non-developer solo/small teams.

## Architecture

### `/build` 5-Phase Pipeline

```
User: "Build an ordering feature"
         |
         v
  Phase 1: Understand (Planner) ← Anthropic
         |  Expand request into product spec
         |  Assess scope/risk via blast-radius
         |  Auto-split large requests into stages
         v
  Phase 2: Implement (Generator) ← Anthropic
         |  Write code, tests, linter
         |  Core changes → translate to product question, ask user
         v
  Phase 3: Verify (Evaluator) ← Anthropic + OpenAI
         |  Shell: lint + test + build (mechanical checks only)
         |  Mid: + boot app, verify key routes
         |  Core: separate sub-agent verification (prevents self-evaluation bias)
         v
  Phase 4: Report (Translation Layer)
         |  Shell: "Done. [one line]"
         |  Mid/Core: before/after detailed report
         v
  Phase 5: Feedback
         User's natural language feedback → repeat Phase 2-4
```

### Blast-Radius Based Behavior

All skills automatically adjust behavior based on change risk level:

```
              Shell             Mid               Core
Confirm       Skip              Show spec          Show spec + Core question
Verify        lint+test+build   + boot app check   + separate sub-agent
Report        One line          before/after        before/after + risk level
Auto-cleanup  Auto-fix          Report only         Report only
```

Analogy: Changing wallpaper color (Shell) and moving a load-bearing column (Core) don't need the same level of review.

### Generator-Evaluator Separation (Core Changes)

Anthropic's key finding: "When an agent evaluates its own code, it identifies legitimate issues then rationalizes why they don't matter."

Analogy: A chef judging their own dish gives generous scores. You need a separate judge for honest evaluation.

```
Generator (Chef)                  Evaluator (Judge)
     |                              |
     |  Code complete                |
     |  --------------------------> |
     |  Spec + file list             |  "I'm seeing this code for the first time"
     |                              |  PASS/FAIL against spec
     |                              |  Anti-pattern checks:
     |                              |   - Hollow implementation
     |                              |   - Hardcoded values
     |                              |   - Missing routes
     |                              |   - Silently swallowed errors
     |                              |   - Security holes
     |  <-------------------------- |
     |  Fix FAIL items               |
     |  --------------------------> |  Re-verify
     |  <-------------------------- |
     |  All PASS → report to user    |
```

Shell/Mid changes are NOT separated. Lint, test, and build are mechanical tools — they're not LLMs, so self-evaluation bias doesn't apply.

## Design Principles

### Principle 1: Non-developers do exactly two things

```
1. Express intent in natural language    ← OpenAI: "humans steer, agents execute"
2. Try the result and react             ← Anthropic: product-level Evaluator judgment
```

Tech decisions, code review, merging, doc management, cleanup — all handled autonomously by the agent.

### Principle 2: Translate tech questions → product questions

Every technical decision originates from a product requirement. Reverse the question and non-developers can give accurate answers.

### Principle 3: Repository = source of truth

Knowledge in Slack, notes, or someone's head is invisible to the agent. Everything is version-controlled in the repo (OpenAI principle). Start from AGENTS.md (100 lines, table of contents) and progressively disclose via docs/ (OpenAI's progressive disclosure). Load only relevant docs per task type (Anthropic's context injection).

### Principle 4: Boring Technology

The tech agents handle best = stable, well-represented in training data, APIs that don't change often. Cutting-edge tech increases agent mistakes, and non-developers can't catch those mistakes.

## Differences from Anthropic/OpenAI Source Articles

Parts where we intentionally diverged from the original designs, and why.

### 1. Evaluator Separation: All changes → Core only

| | Anthropic Original | This Project |
|---|---|---|
| **Approach** | Generator-Evaluator separation for all changes | Sub-agent separation for Core changes only |

**Why**: Running a sub-agent for Shell changes ("change button color") adds time and cost for no benefit. Shell/Mid verification through lint, test, build is sufficient — these tools aren't LLMs, so self-evaluation bias doesn't apply. Core-only separation blocks the "rationalization" problem Anthropic warned about while maintaining Shell/Mid speed (OpenAI: "corrections are cheap, waiting is expensive").

### 2. Merge Philosophy: Minimal gates → blast-radius gates

| | OpenAI Original | Anthropic Original | This Project |
|---|---|---|---|
| **Approach** | Minimal blocking, fast merge | Quality gates required | Branch by blast-radius |

**Why**: Applying both philosophies simultaneously creates a contradiction: "enforce gates while minimizing gates." Blast-radius classification resolves it — Shell (~80%) follows OpenAI's fast path, Core (~20%) follows Anthropic's strict verification.

### 3. Human Role: Code review → Product review

| | OpenAI Original | Anthropic Original | This Project |
|---|---|---|---|
| **Human's role** | Write prompts, translate feedback | Tune Evaluator, analyze logs | Express intent, confirm results only |

**Why**: Both articles assume "humans" are engineers. Non-developers can't read Evaluator logs or write technical acceptance criteria. We focus on the non-developer's irreplaceable ability: **"Is this what I wanted?"**

### 4. Context Management: Dual system → Single store + injection rules

| | OpenAI Original | Anthropic Original | This Project |
|---|---|---|---|
| **Approach** | Structured docs/, progressive disclosure | Per-task precision context injection | Single docs/ + injection rules in AGENTS.md |

**Why**: Maintaining both systems creates dual overhead that non-developers can't sustain. We unify storage OpenAI-style (single docs/) and specify access rules Anthropic-style (in AGENTS.md).

### 5. Scaffolding: Full infrastructure → Minimum viable set

| | OpenAI Original | This Project |
|---|---|---|
| **Infra** | Per-worktree app boot, Chrome DevTools, observability stack, agent review loops, doc gardener | Standard linter + curl-based app verification |

**Why**: OpenAI's infrastructure was built by 3-7 engineers over 5 months. Impossible for a solo non-developer. We include only the minimum set that delivers 80% of value. OpenAI acknowledges: "This behavior should not be assumed to generalize without similar investment."

### 6. Architecture Enforcement: Layered architecture → Omitted

**Why**: Layered domain architecture prevents structural decay in million-line codebases. Non-developer solo projects (thousands of lines) never reach that scale. Blast-radius classification (Core/Mid/Shell) serves as a lightweight architectural boundary.

### 7. Sprint Contracts, Evaluator Tuning: Omitted

**Why**: Sprint contracts are internal agent-to-agent agreements. Evaluator tuning requires reading logs and adjusting prompts. Both are impossible or meaningless for non-developers.

### 8. Execution Plans: Separate management → Auto-splitting

**Why**: OpenAI has engineers judge task scope and write plans. Non-developers don't know whether "build an admin dashboard" is a big task. The agent auto-detects scope and splits into stages, showing the user: "This is a large task. I'll do it in 3 stages. Start with stage 1?"

## Intentional Omissions

| Item | Source | Why Omitted |
|------|--------|-------------|
| Custom linters + structural tests | OpenAI | Requires dev expertise. Standard linters cover 80% |
| Chrome DevTools / DOM snapshots | OpenAI | Infrastructure setup cost too high. curl verification instead |
| Observability stack (LogQL, PromQL) | OpenAI | Overkill for solo scale. docs/quality.md instead |
| Agent-to-agent review | OpenAI | Meaningful at team scale. Reduced to Core sub-agent |
| Doc gardener agent | OpenAI | Unnecessary at 5-6 docs scale. /cleanup instead |
| Reference docs (references/) | OpenAI | Boring tech is already well-covered in training data |
| Layered domain architecture | OpenAI | Overkill for project scale |
| Sprint Contract pre-validation | Anthropic | Meaningless internal process for non-developers |
| Evaluator tuning loop | Anthropic | Non-developers can't read logs or adjust prompts |
| Explicit context reset management | Anthropic | Auto-splitting large requests provides natural resets |

## References

- [Anthropic - Harness Design for Long-Running Application Development](https://www.anthropic.com/engineering/harness-design-long-running-apps)
- [OpenAI - Harness Engineering: Leveraging Codex in an Agent-First World](https://openai.com/index/harness-engineering/)

## License

MIT
