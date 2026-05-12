# Agent Blueprint

> A framework-agnostic reference for building reliable agentic systems.
> Use as a living project checklist — track, address, and document architectural decisions on a per-project basis.

## Origin

Derived from the **Claude Certified Architect – Foundations** exam curriculum — a structured 12-week study plan covering agentic architecture, multi-agent systems, context management, and trust patterns.

The Claude-specific tooling (hooks, CLAUDE.md, MCP) was abstracted away, leaving four universal problems that apply across LangChain, AutoGen, CrewAI, DSPy, OpenAI Assistants, and raw API implementations.

Source: [Claude Certified Architect Study Guide – Free 12-Week Plan](https://claudecertifications.com/claude-certified-architect/study-guide)

---

## The Four Problems

Every agentic system must solve these in some form. When a framework fails you, it's usually failing in one of these four areas.

| Problem | Question |
|---------|----------|
| **Control** | Does the agent do the right thing? |
| **Scale** | Do multiple agents do it reliably? |
| **Trust** | Can you verify they did? |
| **Context** | Are all three constrained by what's in the window? |

---

## 1. CONTROL — Does the agent do the right thing?

Control is the foundation. Before worrying about scale or trust, an agent needs to behave predictably on its own — knowing when to stop, what rules to follow, and what it's allowed to touch.

---

### 1.1 Loop Termination

**Why it matters:** Every agent needs a principled way to know when it's done. Without this, agents either run forever or stop arbitrarily. Arbitrary iteration caps are a crutch — they mask the real problem of not having a well-defined completion signal. Parsing natural language to decide when to stop is worse: it's probabilistic where you need determinism.

**Principle:** Use deterministic signals — a structured stop condition, a tool result that signals completion, a state flag. Design the exit condition before you design the loop.

- [ ] Agent uses a deterministic stop condition (structured signal, state flag, tool result)
- [ ] No arbitrary iteration caps as the primary exit strategy
- [ ] No natural language parsing to decide when to stop

**Notes:**
```
<!-- How is loop termination handled in this project? -->
```

---

### 1.2 Rule Enforcement

**Why it matters:** Prompts are probabilistic. A rule in a system prompt will be followed most of the time — not all of the time. For anything that must always hold (spending limits, permission checks, output format constraints), "most of the time" isn't good enough. Prompt-based rules will fail eventually, and usually at the worst moment.

**Principle:** If a rule must always be followed, enforce it in code — a middleware layer, a validator, a hook. Use prompts for guidance, code for guarantees.

- [ ] Safety-critical rules are enforced in code, not just in prompts
- [ ] Middleware / validation layer exists for non-negotiable constraints (permissions, limits, formats)
- [ ] Prompt-based rules are documented as probabilistic (not guaranteed)

**Notes:**
```
<!-- Which rules are enforced in code vs prompt? Document the boundary. -->
```

---

### 1.3 Tool Scoping

**Why it matters:** Broad tool access increases the blast radius of mistakes. An agent that can read, write, delete, and call external APIs is far more dangerous when it misunderstands a task than one that can only read. Scoping tools to what's needed is the agentic equivalent of least-privilege.

**Principle:** Each agent should only have access to the tools it needs for its task. Review tool access whenever agent responsibilities change.

- [ ] Each agent has access only to the tools required for its task
- [ ] Broad tool access is not used as a shortcut
- [ ] Tool access is reviewed when agent responsibilities change

**Notes:**
```
<!-- List tools per agent and justification for each -->
```

---

## 2. SCALE — Do multiple agents work reliably?

Single-agent control is necessary but not sufficient. When you add more agents, new failure modes emerge: context leaks, coverage gaps, uncoordinated state, and cascading failures. Scale requires deliberate architecture.

---

### 2.1 Orchestrator / Subagent Pattern

**Why it matters:** Without a coordinator, multi-agent systems devolve into chaos — agents stepping on each other, duplicating work, or missing tasks entirely. The hub-and-spoke model (one coordinator, many scoped subagents) is the dominant pattern across LangGraph, AutoGen, CrewAI, and every serious multi-agent framework for a reason: it makes coordination explicit and traceable.

**Principle:** One coordinator holds the plan and delegates. Subagents execute scoped tasks in isolation. Context is passed explicitly, not inherited implicitly.

- [ ] A coordinator agent holds the plan and delegates to subagents
- [ ] Subagents execute scoped tasks in isolation
- [ ] Context is passed explicitly — not inherited implicitly

**Notes:**
```
<!-- Describe the orchestration topology for this project -->
```

---

### 2.2 Context Isolation

**Why it matters:** Leaking coordinator context into subagents creates unpredictable behavior. A subagent that inherits the full history of a coordinator session will be influenced by reasoning that isn't relevant to its task — and may act on it. Subagents should start clean.

**Principle:** Pass only what's needed to each subagent. Treat subagent context the same way you'd treat function arguments — minimal and explicit.

- [ ] Subagents start with clean context
- [ ] Only required context is passed to each subagent
- [ ] No coordinator state leaks into subagent sessions

**Notes:**
```
<!-- Document what context is passed to each subagent and why -->
```

---

### 2.3 Task Decomposition

**Why it matters:** The decomposition strategy is often more important than the framework you choose. Too narrow and you get coverage gaps — tasks that fall between subagents. Too broad and you lose the ability to parallelize. Getting this right is a design problem, not a configuration problem.

**Principle:** Break tasks at natural seams. Map the decomposition before implementing it. Check for gaps.

- [ ] Tasks are broken at natural seams (not too narrow, not too broad)
- [ ] Decomposition strategy is explicit and documented
- [ ] Coverage gaps are checked when tasks are split

**Notes:**
```
<!-- Diagram or describe the task decomposition for this project -->
```

---

### 2.4 Parallel vs Sequential Execution

**Why it matters:** Designing your pipeline around assumed execution order instead of actual dependencies is a common source of subtle bugs. Some subtasks are independent (run in parallel), some depend on prior results (sequential). The dependency graph should drive the design — not the other way around.

**Principle:** Map the dependency graph first. Independent tasks run in parallel. Sequential dependencies are explicit, not assumed.

- [ ] Dependency graph of subtasks is mapped
- [ ] Independent tasks run in parallel where possible
- [ ] Sequential dependencies are explicit, not assumed

**Notes:**
```
<!-- Attach or describe the dependency graph -->
```

---

### 2.5 Session & State Management

**Why it matters:** Long-running agents will eventually crash, timeout, or hit context limits. Without checkpointing, that means starting over. This is distributed systems thinking applied to agents — the same principles that make a database resilient (durable state, recovery, idempotency) apply here.

**Principle:** Long-running agents need checkpointing. You need to be able to resume, fork for branched exploration, and recover from crashes without losing progress.

- [ ] Long-running agents have checkpointing
- [ ] Resume and crash recovery is implemented
- [ ] Branching / forked exploration is handled cleanly

**Notes:**
```
<!-- Document the state management and recovery strategy -->
```

---

## 3. TRUST — Can you verify the output?

An agent that produces output you can't verify is a liability. Trust is the layer most teams skip when moving fast — and the one that causes the most production failures. These mechanisms don't prevent errors; they catch and correct them.

---

### 3.1 Validation + Retry Loops

**Why it matters:** When an agent produces invalid output and you respond with a generic "try again," you're giving it no new information to work with. Specificity of feedback determines quality of correction. The validation loop is only as good as the error message you send back.

**Principle:** Validate output against a schema or rule set. When it fails, feed the specific error back in — not "try again." Set a retry maximum with a graceful fallback.

- [ ] Output is validated against a schema or rule set after generation
- [ ] Failures feed specific error details back into the prompt (not generic "try again")
- [ ] Retry loop has a maximum and graceful fallback

**Notes:**
```
<!-- Document validation rules and what happens on max retries -->
```

---

### 3.2 Separate Generator and Reviewer

**Why it matters:** This is the single biggest trust improvement you can make to an agentic system. An agent reviewing its own output in the same session will rationalize its mistakes — it retains the same reasoning context that produced the error. It's not being dishonest; it genuinely can't see the gap. A separate agent or fresh session doesn't have that context and will catch what the generator missed.

**Principle:** Review is never performed by the same agent in the same session that generated the output. Generator and reviewer roles are architecturally separated.

- [ ] Review is not performed by the same agent in the same session
- [ ] Reviewer uses a fresh session or separate model call
- [ ] Generator and reviewer roles are clearly separated in the architecture

**Notes:**
```
<!-- Document how generator/reviewer separation is implemented -->
```

---

### 3.3 Structured Output Over Free Text

**Why it matters:** Free text output requires interpretation — and interpretation introduces errors. Forcing output into schemas makes structural errors impossible and shifts the problem to semantic errors, which are easier to detect and handle. Parse, don't hope.

**Principle:** Force output into schemas wherever possible — JSON via tool_use, function calling, or output parsers. Validate on receipt. Track semantic errors separately from structural ones.

- [ ] Output is forced into a schema wherever possible
- [ ] Schema is validated on receipt, not assumed correct
- [ ] Semantic errors are tracked separately from structural errors

**Notes:**
```
<!-- Document output schemas in use and validation approach -->
```

---

### 3.4 Provenance Tracking

**Why it matters:** In any agent that synthesizes information from multiple sources, you need to know which claim came from where. Without provenance, you can't audit, debug, or trust outputs at scale. It also becomes impossible to identify when a source is stale, biased, or wrong.

**Principle:** Each output claim is traceable to a source. Temporal data is tagged with retrieval time. An audit trail exists for debugging and verification.

- [ ] Each output claim is traceable to a source
- [ ] Temporal data is tagged with retrieval time
- [ ] Audit trail exists for debugging and verification

**Notes:**
```
<!-- Document how provenance is tracked in this project -->
```

---

### 3.5 Multi-Pass Review

**Why it matters:** A single review pass catches local errors — problems within a component. It misses emergent issues that only appear when you look at the whole: contradictions between components, integration failures, coverage gaps. Complex outputs need two passes: one per component, one across components.

**Principle:** Local analysis pass first (per-component / per-file / per-chunk). Integration pass second (cross-component). Single-pass review should be documented as a known limitation.

- [ ] Local analysis pass runs per component / file / chunk
- [ ] Integration pass runs across components to catch emergent issues
- [ ] Single-pass review is documented as a known limitation if used

**Notes:**
```
<!-- Describe the review pass architecture -->
```

---

## 4. CONTEXT — The constraint everything runs within

Context is not a separate concern — it's the medium everything else operates in. Control, Scale, and Trust all degrade as context quality degrades. Managing it actively is not optional for production systems.

---

### 4.1 Context Degradation Management

**Why it matters:** Long sessions accumulate noise. Models lose track of early context — a well-documented effect called "lost in the middle." The longer a session runs without active context management, the less reliable its outputs become. This isn't a model failure; it's an architectural failure.

**Principle:** Actively manage what stays in context. Summarize where needed, trim verbose tool outputs, move stale state to external storage. Treat context like working memory — curate it intentionally.

- [ ] Long sessions actively manage what stays in context
- [ ] Verbose tool outputs are trimmed before inclusion
- [ ] Stale state is moved to external storage, not left in context

**Notes:**
```
<!-- Document the context management strategy for long-running sessions -->
```

---

### 4.2 Position-Aware Ordering

**Why it matters:** What you put at the start and end of a context window matters more than the middle — models attend more strongly to content at the edges. If critical instructions are buried in the middle of a long context, they will be under-weighted. Structure your context intentionally.

**Principle:** Critical instructions at the edges. Context is ordered deliberately, not appended arbitrarily.

- [ ] Critical instructions are placed at the start and/or end of context
- [ ] Context is structured intentionally, not appended arbitrarily
- [ ] "Lost in the middle" risk is acknowledged for long contexts

**Notes:**
```
<!-- Document the context ordering strategy -->
```

---

### 4.3 Escalation Patterns

**Why it matters:** Without defined escalation criteria, subagents either handle everything (missing cases that need review) or escalate everything (defeating the purpose of delegation). Basing escalation on sentiment or message length is unreliable — a politely worded request can still violate policy; a verbose error message may be fully recoverable locally.

**Principle:** Define escalation criteria based on structured signals — error type, confidence score, policy boundary. Not sentiment, not verbosity.

- [ ] Clear criteria define when a subagent handles locally vs escalates
- [ ] Escalation is triggered by structured signals (error type, confidence, policy boundary)
- [ ] Escalation is not based on sentiment or message length

**Notes:**
```
<!-- Document escalation criteria and thresholds -->
```

---

## Skill Connection Map

```
CONTROL
  └── Loop Termination        — know when to stop
  └── Rule Enforcement        — guarantee critical constraints
  └── Tool Scoping            — limit blast radius
        └── feeds into → SCALE (defines what agents can safely do)

SCALE
  └── Orchestrator / Subagent — coordination structure
  └── Context Isolation       — clean agent boundaries
  └── Task Decomposition      — coverage without gaps
  └── Parallel / Sequential   — execution follows dependencies
  └── Session & State         — survive long runs and crashes
        └── feeds into → TRUST (reliable output to verify)

TRUST
  └── Validation + Retry      — catch and correct errors
  └── Generator / Reviewer    — separate the roles
  └── Structured Output       — remove structural ambiguity
  └── Provenance Tracking     — know where claims came from
  └── Multi-Pass Review       — catch local and emergent issues
        └── depends on → CONTEXT (all trust mechanisms are window-constrained)

CONTEXT
  └── Degradation Management  — curate what stays in
  └── Position-Aware Ordering — put critical things at the edges
  └── Escalation Patterns     — structured signals, not sentiment
        └── underlies → CONTROL + SCALE + TRUST
```

---

## Project Checklist (Copy Per Project)

```markdown
### Project: [Name]
### Date:
### Framework / Stack:

#### CONTROL
- [ ] 1.1 Loop Termination
- [ ] 1.2 Rule Enforcement
- [ ] 1.3 Tool Scoping

#### SCALE
- [ ] 2.1 Orchestrator / Subagent Pattern
- [ ] 2.2 Context Isolation
- [ ] 2.3 Task Decomposition
- [ ] 2.4 Parallel vs Sequential Execution
- [ ] 2.5 Session & State Management

#### TRUST
- [ ] 3.1 Validation + Retry Loops
- [ ] 3.2 Separate Generator and Reviewer
- [ ] 3.3 Structured Output
- [ ] 3.4 Provenance Tracking
- [ ] 3.5 Multi-Pass Review

#### CONTEXT
- [ ] 4.1 Context Degradation Management
- [ ] 4.2 Position-Aware Ordering
- [ ] 4.3 Escalation Patterns

#### Notes
<!-- Key decisions, tradeoffs, and known gaps for this project -->
```

---

*Framework version: 1.0 — May 2026*
*Contributions and project-specific learnings welcome.*
