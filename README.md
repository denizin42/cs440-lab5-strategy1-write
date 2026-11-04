# CS 440 Lab 5 — Repository 1: Strategy 1 (Write) & Model-Free Tabular Q-Learning

**Group 2 Members:** David Jablonski (`@denizin42`), Chris Mills (`@chris-mills03`), James Richardson (`@RichardsonJC`)  
**Course:** CS 440 — Introduction to Artificial Intelligence (Fall 2026)  
**Companion Repository (Strategy 4: Isolate):** [`denizin42/cs440-lab5-strategy4-isolate`](https://github.com/denizin42/cs440-lab5-strategy4-isolate)

---

## 1. Purpose of Repository 1: Context Engineering Strategy 1 (Write)

In the **Agentic AI & Context Engineering** lecture (Slides 21–28), **Strategy 1: Write** solves ephemeral context loss and prompt bloat by persisting knowledge to external files and dynamically loading only what is relevant to the current branch or query.

### How This Repository Contrasts with Repository 2 (`strategy4-isolate`)
| Architectural Layer | **Repo 1: `cs440-lab5-strategy1-write` (This Repo)** | **Repo 2: `cs440-lab5-strategy4-isolate`** |
| :--- | :--- | :--- |
| **Primary Context Strategy** | **Strategy 1: Write (Slides 21–28)** — Externalizing persistent memory, dynamic `@./context/*.md` imports, and procedural skills to disk. | **Strategy 4: Isolate (Slides 33–34)** — Splitting work across an Orchestrator, a Context Filter, and isolated sub-agents/processes to prevent Context Confusion. |
| **Primary `CLAUDE.md` Role** | **3-Tier Persistent Memory** (`./CLAUDE.md`, `examples/user-memory-CLAUDE.md`, and `@./context/*.md` dynamic imports). | **Main Agent Orchestrator** decomposing work via `Task(description, prompt, subagent_type)` through a strict Context Filter. |
| **Primary Hook Focus** | **`UserPromptSubmit` $\rightarrow$ `scripts/context-switcher.sh`**: Dynamically writes `@./context/*.md` imports based on Git branch and query keywords using a `grep -qxF` duplicate guard. | **`PreToolUse` $\rightarrow$ `scripts/prevent-main-commit.sh`**: Enforces an execution boundary blocking direct agent commits to `main`. |
| **Primary Skill & MCP Focus** | **On-Demand Skills (`SKILL.md`)** + **MCP `@mcp.resource` & `@mcp.prompt`** to inject persisted hyperparameters and expert prompt templates. | **Scoped Sub-Agents (`code-reviewer`, `researcher`)** + **Process-Isolated MCP `@mcp.tool`** running 8,000 Q-learning episodes outside the LLM context window. |

*(Note: Both repositories include all 4 Claude Code features—**Hooks, MCP, Skills, and Sub-agents**—and the complete 4x4 Vacuum Q-Learning solution so either repository can be graded standalone.)*

---

## 2. The Target Problem: 4x4 Vacuum Robot with Unknown $T$ and $R$

```text
Row \ Col    0        1        2        3
   0      [start]  [  0  ]  [  0  ]  [  0  ]
   1      [  0  ]  [  X  ]  [  0  ]  [  0  ]
   2      [  0  ]  [  0  ]  [  0  ]  [  0  ]
   3      [  0  ]  [ -1  ]  [  0  ]  [ +10 ]
```

- **Start State `(0, 0)`**: Top-left corner (`0`).
- **Obstacle `X` at `(1, 1)`**: Impassable pillar; bumping into `(1, 1)` or an outer wall leaves the robot in place.
- **Terminal Trap `-1` at `(3, 1)`**: Hazard yielding `-1.0` and ending the episode.
- **Terminal Recharge Goal `+10` at `(3, 3)`**: Recharge station yielding `+10.0` and ending the episode.
- **Why Tabular Q-Learning?** In Lab 3, $T(s, a, s')$ and $R(s, a, s')$ were known offline. Here, $T$ and $R$ are encapsulated inside `VacuumGridEnv`, so `TabularQLearningAgent` learns strictly from sampled transitions $(s, a, r, s', \text{done})$.

---

## 3. Selected Hyperparameters & Justification

| Hyperparameter | Variable | Selected Value | Justification |
| :--- | :--- | :---: | :--- |
| **Living Reward** | `living_reward` | **`-0.04`** | Matches the Lab 3 baseline living cost; penalizes extra steps enough to encourage the shortest safe path without making the robot reckless near the `-1.0` Trap at `(3, 1)`. |
| **Discount Factor** | `discount_gamma` | **`0.90`** | Ensures the `+10.0` terminal reward at `(3, 3)` propagates back across the 6-step path to `(0, 0)` and allows direct comparison against Lab 3 Value Iteration utilities. |
| **Learning Rate** | `learning_rate_alpha` | **`0.10`** | Smoothly averages Temporal-Difference samples over the `80% / 10% / 10%` stochastic slip transitions without oscillating after occasional slips. |
| **Exploration Rate** | `exploration_epsilon` | **`0.20`** | Guarantees 20% uniform random exploration during training so all 13 non-terminal states and 4 actions are thoroughly visited before extracting the greedy policy. |

---

## 4. Repository Structure

```text
cs440-lab5-strategy1-write/
├── README.md                                  # Strategy 1 (Write) documentation, hyperparameters, & CLI guide
├── KANBAN_BOARD.md                            # Combined Group 2 Kanban board
├── TEAM_ONBOARDING.md                         # Teammate clone & multi-agent setup instructions
├── CLAUDE.md                                  # [Strategy 1] Project Memory + @./context/*.md dynamic imports
├── .mcp.json                                  # [Feature 2: MCP] Registers local FastMCP stdio server
├── mcp_server.py                              # [Feature 2: MCP] @mcp.resource, @mcp.prompt, & @mcp.tool server
├── .githooks/
│   └── pre-commit                             # Git pre-commit hook blocking direct commits to main
├── .claude/
│   ├── settings.json                          # [Feature 1: Hooks] UserPromptSubmit (context-switcher) + PreToolUse
│   ├── context/                               # Modular domain context echoed by context-switcher.sh (Slide 27)
│   │   ├── environment.md
│   │   ├── q-learning.md
│   │   └── exploration.md
│   ├── skills/                                # [Feature 3: Skills] On-demand procedural skills (Slide 28)
│   │   ├── github-kanban-sync/
│   │   │   ├── SKILL.md
│   │   │   └── scripts/sync-kanban.sh
│   │   └── q-learning-audit/
│   │       └── SKILL.md
│   └── agents/                                # [Feature 4: Sub-Agents] YAML-configured sub-agents
│       ├── code-reviewer.md
│       └── researcher.md
├── context/                                   # [Strategy 1] Dynamic @./context/*.md files (Slides 25–26)
│   ├── environment.md
│   ├── q-learning.md
│   └── exploration.md
├── examples/
│   └── user-memory-CLAUDE.md                  # [Strategy 1] Reference example for ~/.claude/CLAUDE.md (Slide 23)
├── scripts/
│   ├── context-switcher.sh                    # [Strategy 1] Branch + prompt selector with grep -qxF guard
│   ├── prevent-main-commit.sh                 # PreToolUse guard blocking direct main commits
│   └── setup-teammate.sh                      # Multi-agent onboarding script (Claude, Cursor, Copilot, Gemini)
├── src/
│   ├── __init__.py
│   ├── environment.py                         # Black-box 4x4 VacuumGridEnv simulator (encapsulates T and R)
│   ├── agent.py                               # Hyperparameters & TabularQLearningAgent (TD update + exploration)
│   └── q_learning.py                          # CLI runner printing utilities, Q-table, policy, and rollout logs
└── tests/
    └── test_lab5_progress.py                  # Automated 14-test pytest suite verifying all Lab 5 requirements
```

---

## 5. How to Run the Solution & Tests

```bash
# 1. One-time teammate & pre-commit hook setup:
bash scripts/setup-teammate.sh --agent all

# 2. Run the 14-test verification suite:
pytest -v

# 3. Run the Q-Learning training, policy extraction, and stochastic rollout demo:
python3 -m src.q_learning
```
