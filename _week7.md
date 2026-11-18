# Good resources

- Quick summary of the best practices for coding agents in prod workflows: https://github.com/orgs/community/discussions/182197
-

# Scope

1. Evaluate how generated specs can be reviewed and interpreted by someone else
2. Think through and try out the workflow if there's change request that requires spec updates/when the spec becomes outdated and needs to be brought back in sync with the code
3. Explore how collaborative spec editing would work. If multiple people are updating specs, how to handle merges/conflicts/overlapping sections
4. Think about how we can guide the agent to follow our own coding principles and conventions

- create own skills?

# What I've Done

- `/setup-matt-pocock-skills` -> `/grill-with-docs` -> `/to-specs` -> (`/to-tickets`) -> `/to-implement`

## Changes / Updating requirements

- After tickets created, there are some ambiguity in the ticket which i want to make changes to:
  - The flow then became: add comment to the ticket on github then update the spec to include that change

## Onboarding for another developer

- Included sequencing of events for docs in README.md (as per proposed by agent)

## `/to-implement`

- After first-cut, the agent itself spun off another few agents for:

```
I’m launching the two required review agents now: one for repo/standards smell review, one for issue-spec coverage. Since this branch has no baseline commit, I’m giving them the concrete scaffold files and the GitHub issue text rather than a normal main...HEAD diff.
```

## Coding principals

### 1. Test

#### Scope

- Build passed, but test somehow doesn't seem to be checking the right things to test

#### Configuration

- e.g.: To run with `TESTCONTAINERS_RYUK_DISABLED=true MAVEN_USER_HOME=/private/tmp/data-aggregator-m2 ./mvnw -B verify`
- Common ones?
  - Docker enabled, for testcontainers
  - Have to ensure testcontainers created via Docker are teared down successfully when running tests

### How-To

- Tried to use `/ask-matt` on how to proceed
  - It actually proposed to write in repo first until stablise after 1-2 tasks then can create custom skill using `/skill-creator

```
Transferable
Yes, this is a good candidate for a custom skill, but I’d make it broader than this repo:
Maybe:
`spring-boot-testcontainers-runtime-scaffold`
or
`backend-runtime-scaffold`
Its job would be a checklist for future projects:
- choose runtime and test runner
- add Docker Compose local services
- add Testcontainers integration tests
- separate unit vs integration test commands
- verify Failsafe/Surefire naming
- handle listener/container shutdown
- document local Docker Desktop caveats
- keep environmental workarounds out of app code
- run clean-output verification, not only green-output verification
My recommendation: first write the repo doc. Once it feels stable after 1-2 more tickets, promote the reusable parts into a custom skill using /skill-creator. That avoids prematurely freezing one project’s quirks into a personal rulebook.
```

### 2. (Generic) Progressive Disclosure

- Do not give your agent all the docs at once, but organize them into categories.
  - So your CLAUDE.md or AGENTS.md file becomes just an index, telling the agent which file to load for a given task:

#### AGENTS.md

https://learn.chatgpt.com/docs/customization/overview

- AGENTS.md gives Codex durable project guidance that travels with your repository and applies before the agent starts work. Keep it small.
  - Use it for the rules you want Codex to follow every time in a repo, such as:
    - Build and test commands
    - Review expectations
    - repo-specific conventions
    - Directory-specific instructions
- A custom skill will not magically run after every task unless the agent chooses/invokes it. For “every task should consider docs,” the better layer is an always-visible closeout rule in AGENTS.md
- When to update AGENTS.md
  - Repeated mistakes: If the agent makes the same mistake repeatedly, add a rule.
  - Too much reading: If it finds the right files but reads too many documents, add routing guidance (which directories/files to prioritize).
  - Recurring PR feedback: If you leave the same feedback more than once, codify it.
  - In GitHub: In a pull request comment, tag @codex with a request (for example, @codex add this to AGENTS.md) to delegate the update to a cloud chat.
  - Automate drift checks: Use scheduled tasks to run recurring checks (for example, daily) that look for guidance gaps and suggest what to add to AGENTS.md.
  - Pair AGENTS.md with infrastructure that enforces those rules: pre-commit hooks, linters, and type checkers catch issues before you see them, so the system gets smarter about preventing recurring mistakes.
- Codex can load guidance from multiple locations: a global file in your Codex home directory (for you as a developer) and repo-specific files that teams can check in. Files closer to the working directory take precedence. Use the global file to shape how Codex communicates with you (for example, review style, verbosity, and defaults), and keep repo files focused on team and codebase rules.

## Making AI-Assisted Work Resumable

- The problem we hit was not really a coding problem. It was a coordination problem: when asking an agent to “do the next task,” the agent did not reliably know what “next” meant. The repo had a phase spec, GitHub issues, issue comments, and some README status text, but the implementation sequence and current progress were spread across places. That made it hard for both humans and agents to navigate.
- We considered a few approaches:
  - Putting live status in README.md, but rejected that because README should be stable onboarding, not a tracker.
  - Adding a separate progress doc, but that risks becoming another stale source of truth.
  - Maintaining a manual Current Next field, but that creates extra bookkeeping.
  - Using GitHub Issues as the source of truth, with one tracker issue containing only the intended sequence, while issue state represents actual status.
- The better model is:
  - GitHub Issues are the live source of truth.
  - A tracker issue records sequencing only, not duplicated status.
  - Individual issues carry task detail, comments, assignment, blockers, and completion state.
  - README and specs only point to the tracker; they do not track live progress.
  - NOTE: But ideally, think it should be part of the process where human trigger the next task explicitly like `/implement Github issue #3` instead of leaving unnecessary ambiguity
- We also identified a second gap: what happens when work stops halfway. A normal handoff document is useful when moving context to another thread or person, but mid-task implementation needs something more specific: a checkpoint that lives with the task.
  - So we created a new local skill: `/checkpoint-work` (so stored in `.agents/skills/checkpoint-work`) using `/skill-creator` skill
  - It captures a concise resume packet for unfinished work:
    - active task
    - branch
    - code state
    - verification state
    - exact next step
    - decisions made
    - open questions
    - ownership state: paused, handoff-ready, or blocked
  - The key idea is that a checkpoint should live where future workers naturally look: the configured task tracker. In this repo, that means a GitHub issue comment. In another repo, it could be Linear, Jira, or a local markdown issue file.
  - How to use it:
    - Use /checkpoint-work when stopping mid-task and future-you, another agent, or another developer may need to resume.
    - Use /handoff when moving broader context to another thread, directory, or person.
    - When resuming, read the latest Work Checkpoint before implementing.
    - If the checkpoint says paused, do not take over unless explicitly asked.
    - If it says handoff-ready, continue from the branch/diff and next step.
- The broader lesson: AI-assisted development needs explicit workflow surfaces, not just good prompts. Specs describe scope, issues track work, branches hold code state, and checkpoints preserve interrupted context. When each artifact has one job, both humans and agents move faster with less guessing.

# Some thoughts

1. At the start, may need to ask a few times what to do next (e.g.: after `/implement`, the agent created relevant code plus test, then stopped there. Prompted if need to update issue in Github status and if need to review.)
2. Should we create some skill / rules ourselves such that we should /compact or highlight that we have potentially reached the dumb zone (~100k is smart zone)?

- codex: `/status`: after running 1 task already like 200k used up

3. Flow can be:

- ...`/to-tickets`
  - issue-tracker.md
    - explicit on the need for sequencing of issues (for human and agents)
      - currently set to `Github issues`: will need to explicitly use for other platforms (e.g.: jira)
- AGENTS.md
  - Include Documentation closeout to update relevant docs
  - Include some coding principles (those that we may want to trial on the repo first before thinking if we need to switch to something more reusable through skills if we see it needed in repeatable workflows) — e.g.: runtime and integration testing
    - But this part here is to serve as an index to the right doc where needed
- `/checkpoint-work` custom skill — to enforce as part of a process for people to
  - Different states
    - paused = same worker intends to resume, so others should avoid taking it.
    - handoff-ready = incomplete work is documented and another worker/session may take over.
    - blocked = no one can continue until some external question/state changes.
  - Difference from `/handoff` (matt pocock's )

| Skill              | Main Purpose                                                                                                               | Output                                                   |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| `/handoff`         | Transfer context to a new place/person/thread <br/> “What should this next context know?”                                  | Markdown handoff artifact                                |
| `/checkpoint-work` | Park incomplete work on an active issue <br/> “Exactly where did we stop, and how does someone continue without guessing?” | GitHub issue checkpoint comment plus branch/status notes |

4. `/writing-for-agents` skill from matt pocock was also reached and executed successfully when i wanted to update the custom skill

## Questions

### 1. How should generated specs be reviewed and interpreted by someone else?\*\*

Learning: **A generated spec is not automatically a usable handover artifact.**

AI can produce a detailed spec that looks complete, but a future developer or agent may still struggle to act on it. In `data-aggregator`, the Phase 1 spec had a lot of useful domain and implementation detail, but it did not clearly answer: “Where do I see progress, and what is the next ticket?”

So generated specs need a specific review pass for navigability:

- Can someone understand the goal without the original chat?
- Can they separate stable scope from live status?
- Can they find the canonical tracker?
- Can they tell where sequencing lives?
- Are issue links full URLs in repo docs?
- Is any generated text stale after implementation has started?

Takeaway: **Review generated specs as handover documents, not just as content dumps.**

### 2. What happens when change requests require spec updates, or the spec drifts from code?

Learning: **Not every change belongs in the spec. First decide which artifact is the source of truth.**

We clarified this split:

- Specs: stable scope, architecture intent, implementation boundaries
- GitHub Issues: live task status, sequencing, comments, blockers
- ADRs: durable decisions
- `CONTEXT.md`: domain language
- Code/tests: implemented behavior

So if a change request comes in, the first move should be source-of-truth routing:

- Is this a scope change? Update the spec.
- Is this live progress? Update GitHub Issues.
- Is this a durable technical decision? Update or add an ADR.
- Is this terminology drift? Update `CONTEXT.md`.
- Is the code intentionally different from the spec? Update the spec or create a follow-up issue.

Takeaway: **Spec sync is not “update all docs.” It is choosing the right authoritative artifact.**

### 3. How would collaborative spec editing work?

Learning: **Collaboration needs section ownership and conflict rules, otherwise specs become sediment.**

If multiple people or agents edit the same spec, conflict is not just a Git merge issue. It is often an authority issue: two edits may be trying to change the same decision.

A useful rule is to make each section have one job:

- `Summary` / `Goals`: what we are trying to prove
- `Out of Scope`: what we are deliberately not doing
- `API Surface`: contract
- `Data Model`: persistence shape
- `Acceptance Tests`: expected observable behavior
- `Tracking`: stable link to the tracker, not live status

When resolving overlapping edits, ask:

> Are these edits changing the same decision, or adding separate detail?

If same decision, resolve into one authoritative wording. If separate detail, co-locate under the right existing section instead of scattering similar ideas around.

Takeaway: **Collaborative spec editing works better when sections are treated as ownership boundaries, not just headings.**

### 4. How do we guide agents to follow our coding principles and conventions?

Learning: **Team conventions need to be encoded as discoverable operating guidance, not repeatedly re-explained in prompts.**

Agents are much better at following conventions when those conventions live in the project in a structured way. The goal is not to put everything into one giant instruction file. The goal is to create the right level of guidance at the right level of reuse.

A useful pattern is **progressive disclosure**:

- Put always-relevant pointers in `AGENTS.md`.
- Put detailed rules in smaller docs that agents read only when relevant.
- Turn repeated workflows into skills once they prove useful across tasks.

For example, in `data-aggregator`:

- `AGENTS.md` points agents to tracker, domain, and runtime testing rules.
- `docs/agents/testing-and-runtime.md` is loaded when work touches integration tests, containers, runtime scaffolding, or CI.
- `docs/agents/issue-tracker.md` defines how agents should interpret GitHub Issues and sequencing.
- `/checkpoint-work` emerged because “pause unfinished work safely” was a repeated workflow worth making reusable.

The key design question is:

> Is this a stable convention, a task-specific instruction, or a repeated workflow?

Then choose the artifact accordingly:

- Stable repo convention → `AGENTS.md` pointer or `docs/agents/*.md`
- Domain language → `CONTEXT.md`
- Durable architecture decision → ADR
- Task-specific requirement → GitHub issue
- Repeated agent workflow → skill
- One-time handover → issue comment or handoff doc

This prevents two bad extremes:

- stuffing every rule into the prompt every time
- hiding important conventions in random docs agents may never read

Takeaway: **Agent guidance should be designed like project infrastructure: discoverable, scoped, reusable where valuable, and lightweight where one-off.**
