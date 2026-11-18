# Design & Implementation

## What I explored

I focused on the workflow from **design/specification to implementation**, especially around:

- whether generated specs are actually usable by another developer;
- how requirements and specs should evolve once implementation starts;
- how agents should understand task sequencing and current progress;
- how repository conventions should be communicated to agents;
- how unfinished AI-assisted work can be resumed or handed over safely.

I used Matt Pocock's skills as a starting point, with a flow roughly like:

`/setup-matt-pocock-skills` → `/grill-with-docs` → `/to-specs` → `/to-tickets` (if not already created) → `/to-implement` (this itself triggered it's own subagents and skills like `/code-review`)

The main learning was that the coding itself was rarely the hardest part. Most friction came from **coordination, state management and making the surrounding workflow explicit**.

---

## 1. Generated specs need to be usable as handover artifacts

The generated Phase 1 spec contained useful domain and implementation details, but another developer or agent could still struggle to answer:

- What has already been done?
- What should I work on next?
- Where is current progress tracked?
- Has anything changed since the spec was written?

This led me to review generated specs not just for completeness, but for **navigability**.

A useful check is whether someone without access to the original AI conversation can understand the goal, locate the tracker, distinguish stable scope from live status, and continue the work.

**Learning:** A detailed generated spec is not automatically a good handover artifact. It needs to be understandable and actionable independently of the original conversation.

---

## 2. Give each artifact a clear responsibility

When requirements changed after tickets were created, I prompted the updated of both the GitHub issue and the spec. This raised the question of whether every change should be replicated across multiple documents.

The model I found clearer was:

| Artifact      | Responsibility                                                  |
| ------------- | --------------------------------------------------------------- |
| Spec          | Stable scope, architecture intent and implementation boundaries |
| GitHub Issues | Task details, sequencing, progress, blockers and discussion     |
| ADR           | Durable technical decisions                                     |
| `CONTEXT.md`  | Domain terminology                                              |
| Code/tests    | Implemented behaviour                                           |
| README        | Stable onboarding and pointers                                  |

I also added a part in AGENTS.md (keeping it concise since it's a context all agents will read) to ensure this:

```
### Documentation closeout

At task closeout, update docs only when their source of truth changed: README for stable onboarding, `docs/specs` for phase scope or implementation order, GitHub Issues for live status, `docs/adr` for durable decisions, `CONTEXT.md` for domain language, and `docs/agents` for agent operating rules. Do not use README as a live tracker.
```

This makes change handling more explicit:

- scope change → spec;
- task clarification/progress → issue;
- architecture decision → ADR;
- terminology change → `CONTEXT.md`.

**Learning:** Documentation sync does not mean updating everything. It means updating the correct source of truth.

---

## 3. Make implementation state and sequencing explicit

A problem appeared when asking the agent to "do the next task". The meaning of "next" was spread across specs, issues, comments and README status.

I considered maintaining progress in README or adding another status document, but both introduce another source that can become stale.

The approach I preferred was:

- GitHub Issues are the live source of truth;
- a tracker issue records intended sequencing;
- individual issues contain task state and discussion;
- README/specs only point to the tracker.

But a better way would be to explicitly trigger work, for example:

`/implement GitHub issue #3`

rather than expecting the agent to continuously infer what should happen next.

**Learning:** AI-assisted workflows become more reliable when task state, sequencing and ownership are explicit rather than inferred.

---

## 4. Make unfinished work resumable

Another gap appeared when implementation stopped halfway.

A general handover document transfers broad context, but unfinished implementation needs something more precise: **where exactly did we stop, and what happens next?**

I created a local `/checkpoint-work` skill with `/skill-creator` to capture:

- active task and branch;
- current code state;
- test/verification state;
- decisions made;
- unresolved questions;
- exact next step;
- ownership state.

I used three states:

- **paused** — current worker intends to resume;
- **handoff-ready** — another developer/session can continue;
- **blocked** — progress depends on something external.

The checkpoint is stored with the task, currently as a GitHub issue comment.

This differs from matt pocock's `/handoff` skill:

- `/handoff` transfers broader context;
- `/checkpoint-work` preserves the exact state of incomplete implementation.

**Learning:** Specs capture intent, issues capture work, branches capture code state, and checkpoints preserve interrupted context. Each artifact works better when it has one clear role.

---

## 5. Use progressive disclosure for agent guidance

Another question was how to make agents consistently follow our engineering conventions.

Putting everything into `AGENTS.md` would make it large and expensive to process for every task. Instead, I explored **progressive disclosure**:

- `AGENTS.md` contains always-relevant rules and routing;
- specialised guidance lives in smaller documents;
- reusable workflows become skills only when they prove useful repeatedly.

For example:

- `AGENTS.md` → build/test commands, review expectations, documentation closeout, pointers;
- `docs/agents/testing-and-runtime.md` → integration testing, Testcontainers, runtime setup;
- `docs/agents/issue-tracker.md` → issue sequencing and tracker conventions;
- `CONTEXT.md` → domain terminology;
- ADRs → architecture decisions;
- skills → repeatable workflows.

A useful decision rule is:

> Is this a stable convention, a task-specific instruction, or a repeated workflow?

Repeated mistakes, recurring review comments or agents repeatedly reading the wrong files are good signals that repository guidance should be improved.

**Learning:** Agent guidance should be treated like project infrastructure: discoverable, scoped and maintained over time.

---

## 6. Use implementation gaps to refine coding principles

During implementation, I encountered a case where the build passed, but the tests were not necessarily exercising what I expected and there were still runtime/integration issues.

For the Spring Boot/Testcontainers setup, this meant looking more closely at things such as:

- whether integration tests were actually discovered and run;
- Testcontainers/Docker configuration;
- Surefire/Failsafe naming;
- clean container/listener shutdown;
- keeping environment-specific workarounds outside application code.

This exposed a broader question: **how should recurring engineering expectations like these be encoded for coding agents?**

I initially considered creating a reusable skill. After discussing it through `/ask-matt`, the recommendation was to first trial and refine the principles at repository level rather than immediately generalising them.

I therefore added a lightweight pointer in `AGENTS.md`:

> **Runtime and integration testing**
> For runtime scaffolds, container-backed integration tests, message consumers, local service orchestration, or CI verification work, see `docs/agents/testing-and-runtime.md`.

The detailed guidance lives in the linked document and is loaded only when relevant. If the same principles prove stable across more tasks or repositories, the reusable parts can then be promoted into a custom skill.

**Learning:** Implementation gaps are useful signals for identifying missing coding principles. Encode and refine those principles close to the repository first, then promote them into reusable skills only when they have proven stable and generalisable.

---

## 7. Collaborative spec editing needs ownership boundaries

Multiple developers or agents editing a spec introduces more than Git merge conflicts. Two changes may be modifying the same underlying decision.

One way to reduce ambiguity is to give spec sections clear responsibilities, for example:

- Goals → what we are trying to achieve;
- Out of Scope → deliberate exclusions;
- API Surface → contracts;
- Data Model → persistence shape;
- Acceptance Tests → observable behaviour;
- Tracking → pointer to the live tracker.

When edits overlap, the useful question is:

> Are these changes modifying the same decision, or adding separate detail?

If they change the same decision, there should ultimately be one authoritative wording.

**Learning:** Spec sections can act as ownership boundaries, not just headings.

---

## 8. Human control is still useful at workflow boundaries

Early on, I still had to prompt the agent about steps such as:

- updating the GitHub issue;
- reviewing the implementation;
- checking whether documentation had become stale;
- deciding whether to continue with another task.

Some of these can be encoded into a standard closeout workflow:

`implement → verify → review → update task state → update relevant docs → checkpoint/stop`

However, I would still keep important transitions explicit — particularly deciding what task the agent should work on next.

Long-running sessions also raise a related question around context usage: when should we checkpoint, compact, or move to a fresh session? I have not settled on the exact policy yet, but durable checkpoints reduce the risk of losing implementation state.

- Matt Pocock's research shown ~100k is the smart zone and beyond that is "dumb" — for now we can treat it as part of the process where we do `/status` (in codex) to check this, otherwise may need to update the `/compact` skill to compact at `X` amount of tokens (so this can be triggered automatically by the agent)

---

## Current Working Model

The workflow I am currently leaning towards is:

### Design

Requirements -> generate spec -> review the spec as a standalone handover artifact.

A generated spec is not considered usable just because it is detailed. It should be reviewable by someone who was not in the original conversation and should clearly explain scope, non-scope, key decisions, acceptance expectations, and where live tracking lives.

### Planning

Spec -> GitHub implementation issues -> tracker issue with explicit sequencing.

The spec owns stable scope and intent. GitHub Issues own live work. For multi-ticket specs, one tracker issue records the intended implementation sequence, while individual issue state, assignment, labels, dependencies, comments, and PRs represent live status.

### Implementation

Resolve exact issue -> implement -> test/verify -> review.

Agents should not infer “next task” from the broad spec or issue number order. They should use the tracker issue sequence and select the first open, unassigned, unblocked issue. If tracker state is unavailable or ambiguous, stop and ask.

### Closeout

Update issue state -> update only affected authoritative docs.

Do not update every doc by default. Route updates by source of truth:

- live task status -> GitHub issue / PR
- stable scope change -> spec
- durable technical decision -> ADR
- domain language -> `CONTEXT.md`
- agent workflow/convention -> `docs/agents/*` or skill

### Pause / Handover

Use `/checkpoint-work` for unfinished implementation.

A checkpoint should live with the task and capture task, branch, code state, verification state, exact next step, decisions, open questions, and ownership state.

Use `/handoff` for broader context transfer.

A handoff is for moving context to another thread, person, directory, or phase. It is not the default way to pause mid-task.

### Continuous Improvement

Repeated friction -> repository guidance -> reusable skill when the workflow proves general enough.

## Start with lightweight repo guidance when the convention is project-specific. Promote to a reusable skill when the same workflow appears across tasks or projects and needs a repeatable protocol.

## Main takeaways

The biggest shift in my thinking is that effective AI-assisted development is less about finding a better coding prompt and more about **designing the workflow around the coding agent**.

The principles I would carry forward are:

1. Generated specs should be reviewed for handover quality, not just completeness.
2. Each artifact should have a clear source-of-truth responsibility.
3. Task sequencing and ownership should be explicit.
4. Interrupted work needs durable checkpoints.
5. Agent guidance should use progressive disclosure.
6. Repeated workflows can become skills, but only after they stabilise.
7. Passing tests are not enough — verify what they actually cover.
8. Keep humans explicit at important workflow boundaries rather than allowing agents to infer indefinitely.
