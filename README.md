# Craftsman

**Plan Mode and "Ralph" implementation loop Workflows for GitHub Copilot**

Craftsman is a collection of 2 specialized AI agent modes that transform
how you plan, implement, and verify complex software changes in VS Code.

Instead of endless manual prompting, Craftsman provides structured workflows
that ensure quality through systematic planning, autonomous implementation,
and continuous validation.

## Features

- Simple 2 files, 2 agent modes: Plan Mode / Ralph Loop.
- **Plan Mode**: An interview-based planning agent that produces reviewable specifications
  and actionable task breakdowns.
- **Ralph Loop Mode**: An orchestration agent that autonomously implements tasks
  with continuous verification
- **Structured Artifacts**: Clear, traceable files for specifications, plans, and tasks
- Integration with external issue trackers via JIRA-ID naming convention
- Optional Human-in-the-loop (HITL) phase review for stakeholder review at critical points during
  implementation.
## Why Craftsman?

Craftsman is an **experiment in structured agent orchestration**, exploring three key questions:

### 1. Can models reliably follow complex workflows?

Using **XML-like tags and structured prompts**, Craftsman tests whether LLMs can execute multi-phase processes (discovery → interview → specification → planning → implementation → verification) without losing track of their role or breaking the workflow.

### 2. Can we integrate external issue trackers seamlessly?

Real teams use JIRA, Linear, or GitHub Issues.
Craftsman uses the **JIRA-ID naming convention** (`.agents/changes/JIRA-123-description/`)
to maintain traceability between planning artifacts and external project management systems.

### 3. Can we implement using a Ralph Wiggum loop directly in GitHub Copilot?

Inspired by the **["Ralph Wiggum" pattern](https://www.humanlayer.dev/blog/brief-history-of-ralph)**,
a simple loop that repeatedly delegates to subagents until all tasks are complete.

Craftsman adapts this approach for **VS Code GitHub Copilot**.

Instead of:

- Writing new prompts for each implementation step
- Manually tracking which tasks are done
- Hoping the agent remembers earlier context

**Ralph Loop** does:

- Read the progress file
- Delegate next task to a fresh Coder subagent
- Verify the result with Inspector subagent
- Update progress
- Repeat until complete

This is **linear, stateful, and autonomous** — you start the loop and step away.

## Core Agent Modes

Craftsman currently provides two complementary agent modes:

### � Plan Mode

**A research and planning agent that produces reviewable specifications and actionable task breakdowns.**

Plan Mode systematically explores your change request through:

1. **Deep context discovery** — scans your project structure, documentation, and existing patterns
2. **Structured interviews** — asks 10-15 clarifying questions, then 5-10 technical follow-ups
3. **Specification generation** — produces a reviewable spec with requirements, constraints, and success criteria
4. **Implementation planning** — creates detailed architectural plan with dependencies
5. **Task breakdown** — generates independent, actionable task files for implementation

**Output artifacts** (in `.agents/changes/<JIRA>-<description>/`):

```text
.agents/changes/JIRA-123-feature-name/
├── 00.request.md              # Initial change request
├── 01-specification.md        # Reviewable design decisions and requirements
├── 02-plan.md                 # Technical architecture and dependencies
├── 03-tasks-00-READBEFORE.md  # Critical context for all tasks
├── 03-tasks-01-models.md      # Phase 1, Task 1: Data models
├── 03-tasks-02-api.md         # Phase 1, Task 2: API endpoints
├── 03-tasks-03-tests.md       # Phase 2, Task 3: Unit tests
└── 03-tasks-04-docs.md        # Phase 2, Task 4: Documentation
```

**Key principle**: Plan Mode **never writes implementation code**. It focuses exclusively on thorough planning so implementation agents have clear, complete instructions.

**Files Description**:

- `00.request.md`: The initial human request, often a poorly written JIRA ticket.
- `01-specification.md`: The main output of Plan Mode, containing reviewable design and architectural choices without technical details or code.
- `01-specification.jira.txt`: A JIRA-friendly version of the specification
  for easy putting issues in review in JIRA.
- `02-plan.md`: A highly technical architecture plan that includes task dependencies
  and low-level details. This file is never used after task breakdown is finished.
- `03-tasks-00-READBEFORE.md`: critical context and instructions for all tasks,
  including applicable coding standards, testing requirements, and implementation guidelines.
  These guidelines may be loaded using progressive disclosure by implementation agents
  to ensure consistent adherence to standards.
- `03-tasks-XX-*.md`: individual task files, each containing a single independent task
  with just enough context for a fresh agent to implement it.
  Tasks are grouped into phases, but each task file is self-contained
  to reduce cognitive overload and token waste.

### � Ralph Loop Mode

**An orchestration agent that autonomously implements tasks with continuous verification.**

Ralph Loop manages the complete implementation lifecycle:

1. **Reads planning artifacts** — loads spec, plan, and task files from Plan Mode
2. **Delegates to Coder subagent** — selects next task, triggers implementation subagent
3. **Runs Task Inspector** — verifies each completed task meets acceptance criteria
4. **Manages phase transitions** — validates phase completion before proceeding
5. **Human-in-the-Loop (HITL)** — optional pause points for stakeholder review
6. **Progress tracking** — maintains PROGRESS.md with task status and validation notes

**Two operational modes**:

- **Auto Mode** (default) — continuous implementation until all tasks complete
- **HITL Mode** (Human-in-the-loop) — pauses at phase boundaries for human review. This allows
  to "steer"

**Verification system**:

- **Task Inspector** — validates individual task completion after each Coder run
- **Phase Inspector** — generates comprehensive phase review reports
- **Retry mechanism** — marks incomplete tasks for high priority rework by a new coder subagent

At the end of each coding task AND after each review, a commit message is generated.
This creates a clear, traceable commit history that links implementation decisions
back to the original plan and specification.

It is higly recommended to squash all these commits into a single,
self-contained commit before merging to the main branch,
to avoid polluting the commit history with intermediate implementation steps.

## Personas

- **Orchestrator**: manages the overall workflow, delegates to subagents, and tracks progress
  It nevers codes or even verify the actual job, it just tracks the progress and
  delegates to the right subagent.
- **Coder**: implements individual tasks based on task files
- **Task Inspector**: verifies task completion against acceptance criteria
- **Phase Inspector**: validates phase completion and generates review reports

## Installation

**Current method** (manual):

1. **Add Craftsman to your workspace**:

   ```bash
   # Clone or download Craftsman
   git clone https://github.com/your-org/Craftsman.git ~/Projects/Craftsman
   ```

   Add the Craftsman folder to your VS Code workspace to make agent definitions available.

2. **Copy agent modes to your project**:

   ```bash
   cd /path/to/your-project
   mkdir -p .github/agents
   cp ~/Projects/Craftsman/.github/agents/*.agent.md .github/agents/
   ```

## Quick Start

### Planning a Change

1. Start Copilot Chat and select the Agent **@Craftsman: Plan Mode**

   ![Plan Mode selection screenshot](images/agent-plan-mode.png)

2. Provide your change request (or paste JIRA ticket)
3. Answer the clarifying questions
4. Review the generated specification in `.agents/changes/<JIRA-123>-<short-description>/01-specification.md`
5. Approve plan and task breakdown

### Implementing with Ralph Loop

1. Start Copilot Chat and select **@Craftsman: Ralph Loop**
   ![Ralph Loop selection screenshot](images/agent-ralp-mode.png)
2. By default, the implementation is autonomous without human intervention.
   You can choose to enable HITL (Human-in-the-Loop) mode to pause at phase boundaries for review.
3. Provide path to planning artifacts (e.g., `.agents/changes/IDEA-01-refresh-command/`)

   ![Ralph Loop Trigger](images/agent-ralph-trigger.png)

   ![Ralph Loop Trigger](images/agent-ralph-trigger-hitl.png)

4. Ralph Loop will:
   - Read spec, plan, and tasks
   - Delegate implementation to Coder subagents
   - Verify each task with Task Inspector
   - Track progress in PROGRESS.md
   - Continue until all tasks complete

**Pausing Ralph Loop**: Create `PAUSE.md` in the planning folder to safely pause the loop for manual task edits.

## Concrete advices

- Start with a small request in markdown (`.agents/changes/JIRA-123-description/00.request.md`)
- Use a mid-size model like Claude Sonnet 4.5 for the Plan Mode.
- Reserve Opus only when tasks require complex reasoning or multi-phase implementation (20+ tasks)
- Use Claude Haiku for implementation. But for the moment, you can use Sonnet for the Ralph loop,
  it will not consume 1 premium token per request (to my experience, it consumes 1 premium request
  after ~15 taks + reviewers)

## Typical End-to-End Workflow

```mermaid
graph TD
    A[Change Request] --> B[Plan Mode: Discovery]
    B --> C[Plan Mode: Questions]
    C --> D[Plan Mode: Specification]
    D --> E{Review Spec}
    E -->|Approved| F[Plan Mode: Tasks]
    E -->|Revise| C
    F --> G[Ralph Loop: Read Artifacts]
    G --> H[Ralph Loop: Coder Subagent]
    H --> I[Ralph Loop: Task Inspector]
    I -->|✅ Complete| J{More Tasks?}
    I -->|� Incomplete| H
    J -->|Yes| H
    J -->|No| K{Phase Complete?}
    K -->|Yes, HITL| L[Phase Inspector + Human Review]
    K -->|Yes, Auto| M[Phase Inspector]
    K -->|No| H
    L --> N{All Phases Done?}
    M --> N
    N -->|Yes| O[Implementation Complete]
    N -->|No| H
```

---

## Honest Feedback: Current Limitations

Craftsman is a **production-level proof of concept**.
It works, i use it daily in my workflow BUT it's not perfect.

In this section, I humbly provides the real limitations and known issues
of the current implementation.

I would be so grateful if you could try it out and share your feedback,
especially if you have suggestions for improvement.

### Known Issues

1. **Task selection autonomy**: The orchestrator sometimes chooses tasks and
   sends task numbers to the Coder subagent, despite instructions stating
   "let the subagent choose". This creates unnecessary coupling.

2. **Rate limit recovery failures**: When hitting GitHub Copilot daily/weekly rate limits, retry behavior degrades:
   - Orchestrator "forgets" to trigger subagents
   - Implementation happens in orchestrator instead of Coder subagent
   - **Workaround**: Start a fresh chat session

3. **Feature completeness vs. accessibility gap**: The most significant limitation — at completion:
   - ✅ All features are typically implemented
   - ✅ Complete preflight checks pass
   - ✅ Unit tests pass
   - ✅ Code quality is high
   - ❌ **But features may not be user-accessible** (especially with UI)

   Despite intensive planning with Claude Opus and no visible gaps in specifications,
   implemented features sometimes exist in code but lack integration points, UI bindings,
   or entry points for users to actually use them.

   I would say a human would have caught this gap during implementation,
   but the agent still misses it.

## Acknowledgments

Inspired by the "Ralph Wiggum" loop concept and refined through experimentation with GitHub Copilot's Agent Mode system.

**Read more**:

- [Original Reddit post](https://www.reddit.com/r/GithubCopilot/comments/1qapkdg/ralph_wiggum_technic_in_vs_code_copilot_with/)
- [X/Twitter thread](https://x.com/stibbons31/status/2020456046259589229)
