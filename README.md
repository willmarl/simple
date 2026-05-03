# RESULT.md

## Autonomous Agent Experiment: Todo List Edition

**Repo Purpose:** Testing whether a local AI agent (Claude via homelab) can autonomously commit meaningful code improvements to a real GitHub repository without human intervention. This is a proof-of-concept to validate autonomous development workflows on constrained hardware before scaling infrastructure.

**Agent Mode:** Ralph mode (fully autonomous; agent makes decisions without human approval between commits)

**Test Subject:** Bare-bones todo list application
- Stack: Vanilla HTML + CSS + JavaScript
- Features: Add todo, delete todo
- Limitations: No data persistence (localStorage not implemented), minimal styling
- Commit History: 1 manual verification commit (`hello.txt`), remainder autonomous

---

## Hardware Setup

**Infrastructure:**
- Hypervisor: Proxmox
- VM #1: Ubuntu + Ollama (model serving)
- VM #2: Ubuntu + Claude Code (agent execution, `--dangerously-skip-permissions` flag)
  - Rationale: Isolated throwaway environment; if agent executes destructive commands, VM is disposable
- Host Hardware: 32 CPU cores, 96GB RAM, Samsung 990 Pro NVMe storage

**Performance Profile:**
- CPU: Stable, low utilization (~5-10%)
- RAM: Stable, ~60-70% utilized
- GPU VRAM: **100% saturation** (bottleneck identified)
- NVMe: Minimal I/O pressure

**Model Runtime:**
- Primary: `gpt-oss:20b` (~14GB VRAM) – mediocre results, sustainable
- Alternative: `glm-4.7-flash` (~24GB VRAM) – significantly better outputs, severe VRAM constraints
  - Context Window: ~30k tokens before thermal throttle
  - Practical Limit: ~2-3 file modifications before VRAM exhaustion
  - Assessment: Model quality is promising, but infrastructure cannot scale it

---

## Experimental Results

### What the Agent Accomplished

**Autonomous Capabilities Confirmed:**
- Successfully staged, committed, and pushed code changes ✓
- Understood repository structure (basic) ✓
- Followed git workflow without intervention ✓
- No catastrophic failures (repo integrity maintained) ✓

**Code Quality Observations:**
- Incremental improvements only (not transformational)
- Mechanically competent but architecturally naive
- No forward-thinking design patterns applied

---

## Findings from Prior Experiments

### Test #1: Article Module Removal (`remove-article` branch)

**Task:** Relocate article-related backend and frontend components to `/legacy` folder.

**Agent Mode:** Ralph mode (fully autonomous)

**Outcome:**
- Agent successfully moved files and updated imports
- Code executed without errors on next morning
- Frontend refactoring: Competent (proper file handling)
- Backend refactoring: Incomplete (only deleted `admin-article.*.ts` files, missed others)
- **Assessment:** Better at mechanical frontend tasks than backend architecture

**Verification:** Full results visible at: https://github.com/willmarl/monno/tree/remove-article

**Key Observation:** Agent performed "happy path" execution rather than thorough refactoring. Did not autonomously validate completeness.

### Test #2: Notification System Implementation (Abandoned)

**Task:** Implement notification feature across database schema and service layer.

**Agent Mode:** Ralph mode (fully autonomous)

**What Happened:**
- Schema Changes: Hardcoded `notification` fields into User, Post, Comment, Article models
  - Should have: Added relationships to existing Notification model
  - Did: Duplicated notification logic across multiple entities
- Service Layer: Massive code duplication across `*.service.ts` files
  - No DRY extraction or OOP patterns
  - Entry-level implementation approach
- Pattern Violation: Ignored established architectural conventions in the codebase

**Decision to Abort:** Agent was not being resourceful. It took the most literal interpretation of requirements (A→B automation) without considering maintainability, future extensibility, or repo patterns. This is exactly the kind of autonomous decision that creates technical debt.

**Root Cause Analysis:** With `gpt-oss:20b`, the agent lacks sufficient model capacity to:
1. Perform deep context analysis of existing patterns
2. Reason about architectural implications
3. Distinguish between "works" and "works well"

### Test #3: Frontend Modal Implementation (Mid-2025, Pre-Claude Era)

**Context:** Running Copilot on auto-accept while unattended; using GPT model instead of Claude

**Agent Mode:** Autonomous (equivalent to ralph mode)

**What Happened:**
- Existing Infrastructure: Zustand-based global modal provider
- Agent Decision: Created fresh modal instance for each new modal component
- Result: Redundant state management, ignored architectural standard

**Implication:** This pattern repeats—agents optimize for "task completion" not "systemic design."

---

## Analysis & Hypothesis

### Why the Results Were Mixed

**Confirmed Limitations:**
- Model capacity (VRAM/parameters) directly correlates with code quality
- `gpt-oss:20b` is functionally capable but strategically limited
- `glm-4.7-flash` is substantially better but infrastructure-constrained

**Unconfirmed Variables (Low Sample Size):**
- **Prompt Engineering:** Single trial per task. Iterative refinement of system prompts might yield dramatically different results. Investment in "repo constitution" documentation (patterns, anti-patterns, architectural rules) was not attempted.
- **Experiment Design:** No controlled A/B testing across models or prompt variations
- **Statistical Significance:** N=1 (or N=3 for broader tests) is anecdotal, not scientific

**Honest Assessment:** I may have caught the agent at its worst. Without more rigorous trial-and-error on prompting and test replication, it's impossible to claim with confidence whether poor results stem from:
- Fundamental model limitations
- Infrastructure constraints (VRAM = context depth, reasoning time)
- Suboptimal prompt strategy
- Natural variation in autonomous execution

The honest answer: **Unknown without more data.**

---

## What Would Be Required to Scale

### Infrastructure Bottleneck (Primary)
Current GPU VRAM is the hard constraint. Solutions:

**Option A: Upgrade GPU**
- Target: A100 (80GB) or RTX 6000 Pro (48GB)
- Benefit: Run `glm-4.7-flash` at full capacity, longer context windows, multiple concurrent agents
- Limitation: Expensive, single point of failure

**Option B: Multi-GPU Setup**
- Original Plan: Spawn VM #3, VM #4, etc. as parallel agents with orchestration UI
- Issue: Underestimated VRAM requirements; model serving scales poorly without proper infrastructure
- Feasibility: Requires investment in clustering/distributed model serving (Vllm, Ray, etc.)

### Prompt Engineering (Secondary)

**Not Yet Attempted:**
- Repo-specific "constitution" (architectural rules, patterns, anti-patterns)
- Few-shot examples of desired code quality for this specific codebase
- System prompt iterations targeting architectural reasoning vs. mechanical execution
- Feedback loops (agent generates code → validation → prompt refinement)

**Hypothesis:** With 5-10 iterations of prompt tuning, agent output quality could improve 20-40% without hardware changes. This was not tested due to time constraints and low expectations from initial results.

### Token Budget (If API Route)

Running Opus autonomously would be ideal (better reasoning) but prohibitively expensive:
- Single notification feature attempt = ~$2-5 in API costs
- Scale to 10-15 agents running nightly = $20-75/night
- Local GPU = electricity cost only (~$0.50/night for a 4090 at $0.15/kWh)

Trade-off: Local = free but limited; API = expensive but capable.

---

## Current State of Infrastructure

**What Works:**
- Proxmox hypervisor is stable and over-provisioned (CPU/RAM not bottlenecked)
- Ollama + Claude Code integration functions reliably
- Throwaway VM approach prevents catastrophic failures
- Git workflow automation is robust
- Ralph mode executes reliably without intervention

**What Doesn't Work Yet:**
- VRAM saturation prevents concurrent agents (original vision = abandoned)
- Model quality insufficient for architectural decisions (but may improve with prompt tuning)
- No feedback mechanisms (no validation → no learning)

---

## Realistic Assessment

### What This Proves

**Autonomous agents CAN:**
- Execute mechanical tasks (file moves, boilerplate generation, basic CRUD)
- Understand git workflows
- Operate unsupervised without destroying the repo
- Run in ralph mode for extended periods without human intervention

**Autonomous agents CANNOT (yet):**
- Reason about architectural trade-offs
- Apply domain-specific patterns consistently
- Validate their own work against quality standards
- Prioritize maintainability over "works now"

### The Gap Between "Works" and "Production-Ready"

The notification system test was illuminating. The code *functioned*. It would pass tests. But it violated every principle I care about (DRY, modularity, future-proofing). This gap—between "technically correct" and "architecturally sound"—is where autonomous agents currently fail.

Closing that gap requires either:
1. **Model Capacity:** Larger models with better reasoning (need better GPU)
2. **Prompt Sophistication:** Highly tuned instructions + repo-specific patterns (not yet attempted)
3. **Human Feedback Loop:** Agent → generate → validate → refine (time-intensive, not autonomous)

---

## Lessons Learned

### Infrastructure Is Real

The original vision—"spawn VMs 3, 4, 5 with a web orchestrator"—was naive. Model serving is resource-hungry. A single 4090 running a decent model at full capacity leaves no room for parallelization. I underestimated VRAM requirements by roughly 2-3x.

### Model Quality Matters More Than I Expected

The jump from `gpt-oss:20b` to `glm-4.7-flash` is noticeable. GLM is worth the VRAM cost, but the cost itself is prohibitive on current hardware. This is a strong signal that better infrastructure would meaningfully improve results.

### Sample Size of 1 Is Useless

I stopped after the notification system failed, but I didn't rigorously test whether the failure was due to:
- Model limitations
- Hardware constraints
- Bad prompts
- Natural variance

A proper experiment would include 10+ iterations with controlled variables. I took a shortcut because I was skeptical and hit friction early.

### Ralph Mode Works, But Needs Guardrails

Ralph mode successfully operated autonomously for extended periods. However, without validation mechanisms or architectural guardrails baked into prompts, the agent made decisions that degraded code quality over time. Future iterations need feedback loops or constraint-based prompting.

### Local GPU Economics Win Long-Term

Running this on Claude Opus API would cost $5-10/night. The 4090 costs ~$0.50/night in electricity. Even at current mediocre results, the unit economics favor local hardware for experimentation. Once I upgrade GPU and refine prompts, that advantage widens dramatically.

---

## What's Next (If I Had More Time)

1. **Prompt Engineering Sprint:** Invest 2-3 hours building a detailed system prompt with architectural rules and examples. Rerun notification test in ralph mode.
2. **Replicate Tests:** Run article removal and notification tests 5+ times each, compare outputs.
3. **Infrastructure Planning:** Research A100 pricing, cooling requirements, model serving orchestration.
4. **Structured Validation:** Build automated code quality checks (linting, architecture rule enforcement) so agent can self-validate during ralph mode.
5. **Ralph Mode Refinement:** Implement checkpoints where agent pauses for human review before committing controversial changes.

---

## Conclusion

**Can autonomous agents replace developers on a 4090?**

Not yet. But the foundation is solid. The repo is safe, the workflow works, and the bottleneck is identified: VRAM and prompt engineering, not architectural flaws. Ralph mode proves the agent can operate unsupervised; it just needs better reasoning capability and guardrails.

**Would better infrastructure help?**

Absolutely. GLM 4.7 Flash quality at 30k token windows is promising. With an A100 or equivalent, I'd expect 50-70% improvement in code quality without changing a single prompt.

**Is it worth doing at all on current hardware?**

Yes. Local experimentation is free. I've learned what works (mechanics), what fails (architecture), and where to invest next. For a hobbyist homelab, that's valuable.

**Final Take:** This is a technology-and-infrastructure problem, not a fundamental AI limitation. With capital (better GPU) and time (prompt engineering), I'm confident autonomous agents can handle non-trivial features reliably in ralph mode. I just haven't paid the upfront investment yet.

---
