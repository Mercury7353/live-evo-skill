# Live-Evo: Self-Evolving Memory for Claude Code

A Claude Code skill that gives your AI agent **persistent, self-improving memory**. Live-Evo learns from past mistakes through experience accumulation and adaptive evaluation — so the same mistake is never repeated twice.

Inspired by the research on [online self-evolving memory for LLM-based agents](https://arxiv.org/abs/2402.11890).

## How It Works

```
Task arrives → Retrieve relevant past experiences → Generate guideline
     ↓
 Solve task (with or without contrastive evaluation)
     ↓
 Observe outcome (tests, user feedback, errors)
     ↓
 Store lesson if failure detected → Update experience weights
```

### Key Mechanisms

- **Experience Store**: Lessons learned from past failures, stored as structured JSONL
- **Weighted Retrieval**: Relevant experiences are found via keyword similarity, ranked by quality weight (0.1–2.0)
- **Adaptive Evaluation**: The model decides whether to do full contrastive testing (two attempts) or direct apply based on cost/feasibility
- **Feedback-Based Learning**: When verification isn't feasible, learns directly from user corrections, test results, and error messages
- **Self-Correcting Weights**: Useful lessons get promoted (+0.3), unhelpful ones decay (-0.2) — bad advice naturally fades

## Install

### As a Plugin (via Marketplace)

```bash
# Add the marketplace
/plugin marketplace add <your-github-username>/live-evo-skill

# Install the plugin
/plugin install live-evo@live-evo-marketplace
```

### Manual Install

Copy the skill directory to your Claude Code skills folder:

```bash
cp -r plugins/live-evo/skills/live-evo ~/.claude/skills/live-evo
```

## Usage

### Automatic

Live-Evo activates automatically when Claude encounters verifiable tasks. It retrieves relevant past experiences and applies learned lessons.

### Manual

Invoke directly with `/live-evo` to trigger the full workflow on the current task.

### CLI Commands

```bash
# View all stored experiences
python ~/.claude/skills/live-evo/scripts/list_experiences.py

# Search for relevant experiences
python ~/.claude/skills/live-evo/scripts/retrieve.py --query "your task description"

# Add a new experience manually
python ~/.claude/skills/live-evo/scripts/add_experience.py \
  --question "What was the task" \
  --failure-reason "What went wrong" \
  --improvement "What to do differently" \
  --category coding

# View statistics
python ~/.claude/skills/live-evo/scripts/stats.py
```

## Workflow Details

### Step 1: Retrieve & Compile

Before tackling a task, Live-Evo searches past experiences and synthesizes a task-specific guideline.

### Step 2: Smart Verification Decision

The model evaluates whether contrastive testing (two independent attempts) is worthwhile:

| Factor | Do Contrastive Eval | Direct Apply |
|---|---|---|
| Cost of re-running | Low (run a test) | High (long build, API costs) |
| Verifiability | Clear ground truth | No easy verification |
| Task complexity | Simple enough for two attempts | Too complex to duplicate |

### Step 3: Learn from Outcome

- **With contrastive eval**: Compare both attempts, update weights based on which was better
- **Without contrastive eval**: Learn from user feedback, test results, or error messages

## Seed Experiences

The skill comes with 2 seed experiences to demonstrate the format:

1. **React useEffect infinite loop** — Memoize objects/arrays in dependency arrays
2. **Python memory leak** — Use context managers for resources; use `tracemalloc` for diagnosis

These will naturally evolve as you use the system. Your own experiences will accumulate and improve over time.

## Architecture

```
live-evo/
├── SKILL.md                    # Main instructions for Claude
├── experiences/
│   └── experience_db.jsonl     # Experience database (JSONL)
└── scripts/
    ├── experience_manager.py   # Core logic (retrieval, storage, weights)
    ├── retrieve.py             # Search past experiences
    ├── update.py               # Update weights after verification
    ├── add_experience.py       # Store new experiences
    ├── list_experiences.py     # List all experiences
    └── stats.py                # Database statistics
```

- **Pure Python** — no external dependencies, only stdlib
- **JSONL storage** — simple, human-readable, git-friendly
- **Keyword-based retrieval** — Jaccard similarity with phrase boosting (no embeddings needed)

## Cross-Platform

Live-Evo follows the [Agent Skills](https://agentskills.io) open standard. The SKILL.md format is compatible with:

- Claude Code
- OpenAI Codex CLI
- Gemini CLI
- Cursor
- Other Agent Skills-compatible tools

## License

MIT
