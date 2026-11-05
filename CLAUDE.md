# Project Context: CS 440 Lab 5 — Strategy 1: Write (Persistent Memory & Dynamic Context Injection)

## 1. Strategy 1 (Write) Architecture Overview (Lecture Slides 21–28)
This repository (`cs440-lab5-strategy1-write`) focuses on **Context Engineering Strategy 1: Write**—persisting domain knowledge, coding standards, and procedural skills **outside** the ephemeral conversation history and loading them dynamically on demand so the AI agent can generate the 4x4 Vacuum Q-Learning solution in `src/` from scratch.

### How Strategy 1 (Write) Is Implemented Here
1. **Three-Tier Persistent Memory Hierarchy (Slides 21–23)**:
   - **Tier 1 — Project Memory (`./CLAUDE.md`)**: Shared via Git; stores the 4x4 Vacuum Grid coordinates, selected hyperparameters, unknown $T$/$R$ mandate, and PEP 484 coding standards.
   - **Tier 2 — User Memory (`examples/user-memory-CLAUDE.md` $\rightarrow$ `~/.claude/CLAUDE.md`)**: Stores personal developer workflow preferences across sessions.
   - **Tier 3 — Dynamic Context Imports (`@./context/*.md`)**: Modular markdown files (`environment.md`, `q-learning.md`, `exploration.md`) imported at the bottom of `CLAUDE.md`.
2. **Hook-Driven Context Writing (`.claude/settings.json` + `scripts/context-switcher.sh` — Slides 24–27)**:
   - Registers a `UserPromptSubmit` hook that checks the active Git branch (`feature/environment*`, `feature/q-learning*`, `feature/exploration*`) and prompt keywords.
   - Uses an idempotent `grep -qxF` check before appending `@./context/*.md` references so repeated prompts never bloat `CLAUDE.md`.
3. **On-Demand Procedural Skills (`.claude/skills/` — Slide 28)**:
   - Stores specialized verification procedures (`q-learning-audit` and `github-kanban-sync`) on disk that the agent loads only when needed.
4. **MCP Resources & Reusable Prompts (`mcp_server.py` — MCP Lecture Slides 20–28)**:
   - Uses `@mcp.resource` (`vacuum://hyperparameters`, `vacuum://context`) to expose persisted config/docs like GET handlers and `@mcp.prompt` (`/audit_hyperparameters`) to ship tested prompt templates.

---

## 2. 4x4 Vacuum Grid World & Selected Hyperparameters (Project Memory — Slide 22)
```text
Row \ Col    0        1        2        3
   0      [start]  [  0  ]  [  0  ]  [  0  ]
   1      [  0  ]  [  X  ]  [  0  ]  [  0  ]
   2      [  0  ]  [  0  ]  [  0  ]  [  0  ]
   3      [  0  ]  [ -1  ]  [  0  ]  [ +10 ]
```
- **Start State `(0, 0)`**, **Obstacle `X` at `(1, 1)`**, **Terminal Trap `-1` at `(3, 1)`**, **Terminal Recharge Goal `+10` at `(3, 3)`**.
- **Unknown $T$ and $R$**: When the agent generates the Python solution in `src/`, the learning agent MUST NOT read private environment transition weights (`0.8` intended, `0.1` slip left, `0.1` slip right) or reward tables; it must learn strictly from sampled tuples $(s, a, r, s', \text{done})$.

### Selected Q-Learning Hyperparameters (Agent Must Use These Values When Generating `src/`)
<!--
TODO (Part 1 - Select Hyperparameters in CLAUDE.md):
Replace `TODO_SET_VALUE` in the table below with your chosen numbers:
  - living_reward       : -0.04
  - discount_gamma      : 0.90
  - learning_rate_alpha : 0.10
  - exploration_epsilon : 0.20
-->
| Hyperparameter | Variable Name | Selected Value |
| :--- | :--- | :--- |
| Living Reward | `living_reward` | `-0.04` |
| Discount Factor ($\gamma$) | `discount_gamma` | `0.90` |
| Learning Rate ($\alpha$) | `learning_rate_alpha` | `0.10` |
| Exploration Rate ($\epsilon$) | `exploration_epsilon` | `0.20` |

### Agent Implementation Instructions for `src/`
When prompted to solve Lab 5, the agent must create `src/environment.py`, `src/agent.py`, and `src/q_learning.py` (runnable via `python3 -m src.q_learning`) using the hyperparameters above, explicit PEP 484 type annotations, and strict black-box encapsulation of $T$ and $R$.

---

## 3. Cross-Reference: Sub-Agent Delegation (Supported Alongside Strategy 1)
To ensure all 4 Claude Code features work seamlessly in this repository, isolated sub-agents are also registered in `.claude/agents/`:

```javascript
// ============================================================================
// TODO (Part 4B - Orchestrator Task Delegation, Slide 34):
// Replace "TODO_SET_REVIEWER_SUBAGENT" with "code-reviewer" and
// replace "TODO_SET_RESEARCHER_SUBAGENT" with "researcher".
// ============================================================================

// Code review agent - focused context (Slide 34)
Task(
  description: "Code review",
  prompt: "Review this PR focusing only on security, performance, and model-free T/R encapsulation",
  subagent_type: "TODO_SET_REVIEWER_SUBAGENT"
)

// Research agent - broad architectural query (Slide 34)
Task(
  description: "Research implementation",
  prompt: "Compare epsilon-greedy vs optimistic exploration f(u,n)=u+k/n for the 4x4 vacuum grid",
  subagent_type: "TODO_SET_RESEARCHER_SUBAGENT"
)
```

---

## 4. Dynamic Context Imports (Strategy 1: Write — Slides 21, 23, 25)
<!--
TODO (Part 2C - Dynamic Context Imports, Slide 25):
Uncomment or add the three `@./context/*.md` lines below so Claude Code dynamically
loads the modular environment, Q-learning, and exploration context files:
  @./context/environment.md
  @./context/q-learning.md
  @./context/exploration.md
-->
