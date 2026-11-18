# Vision

1. Shifting our human developers value-add from syntax to intent

| Human Effort     | Year       | Value-add Shift         | Where we are now |
| ---------------- | ---------- | ----------------------- | ---------------- |
| Effort on Syntax | ~2021      | Autocomplete            |                  |
|                  | ~2022      | Inline Code Suggestions | Around here      |
|                  | ~2023      | Chat-Based Generation   | Maybe here       |
|                  | ~2024-2025 | Coding Agents           |                  |
| Effort on Intent | ~2025-2026 | Autonomous Agents       |                  |

2. Shifting our ability on AI-assisted coding from not only Vibe Coding to also Agentic Engineering

| Dimension              | Vibe Coding                          | Structured AI-Assisted Coding                        | Agentic Engineering                                                                  |
| ---------------------- | ------------------------------------ | ---------------------------------------------------- | ------------------------------------------------------------------------------------ |
| Intent Specification   | Casual natural language prompting    | Detailed prompting with examples and constraints     | Formal specifications, architecture documents and memory files                       |
| Verification           | Vibe Checking                        | Manual testing, spot-checking                        | Automated test suites, CI/CD quality gates, LLM verification                         |
| Codebase Understanding | Minimal                              | Selective review of Critical Paths                   | Comprehensive review of architecture. AI handles implementation details              |
| Error Handling         | Copy-paste error messages back to AI | Developer diagnoses root cause and AI implements fix | Agent self-diagnose within acceptable boundary, developer handle architecture issues |
| Appropriate Scope      | Prototypes, Scripts , Hackathons     | Features within established codebases                | Production systems                                                                   |
| Risk Profile           | High, acceptable to dispose code     | Medium, developer judge at key checkpoints           | Low, systematic verification at every stage                                          |

---

# Phase 1 Goals

- Enable developers to have AI subscription
- Projects being developed on the Internet with AI assistance (everyone to be on Chat-Based Generation working towards Coding Agents )
- First-cut process on AI-assisted SDLC (everyone skilled for structured AI-assisted coding)

### 12-weeks to focus on the following stages

We all agree that having _some_ kind of plan — a spec, a standard, a test — before/around AI-generated code is a good idea. Hence we picked the following 3 stages to focus first as we believe these are the areas where getting it right upstream gives us the most confidence in the codebase AI helps produce.

- Requirements (Improve Specification Quality)
  Proficient in managing Specifications with tools like (Openspec or Spec-Kit), and review specifications

- Design (Able to incorporate Architecture Decisions)
  Able to utilize project setup such as .md files, skills and mcp integrations to implement and guard architecture decisions

- Review (Able to verify / judge what was implemented)
  Able to confidently release code into production

### 5 Key Questions

For each of the focused stage, think about these questions:

1. How much to generate. Is there a "right-sized" version of this artifact — enough to be useful, not so much it's noise or wasted effort?
2. How much to review. Where's the point past which more review effort stops catching anything new?
3. How extensible/reusable it is. Does it hold up on the next feature, or does it need to be rebuilt from scratch each time? Could someone else pick it up and use it without you?
4. How to review it. What's the actual technique that catches problems efficiently — not just "read carefully," but a specific method.
5. How AI can help apply it. Can you feed the artifact back to the AI usefully — "does this code satisfy this spec," "does this PR follow this architecture decision," "what's missing here" — so it becomes a live tool, not a document that goes stale the moment it's written?

### Area 1: Requirements

##### Tools:

- [OpenSpec](https://github.com/Fission-AI/openspec)
- [Spec Kit](https://github.com/github/spec-kit)

Task:

1. Pick a tool (can start with OpenSpec/Spec Kit/Matt Pocock's Grill-me)
2. Run its standard flow on a real feature you'd otherwise hand-write a spec for
3. Do this for at least 2 real changes: one regular feature, one small/urgent one (hotfix-sized), refactor.
   See how the tool's process scales down — does it still help, or does it feel like overhead for something small?

Consider these questions during exploration:

- What's explicitly in scope, and what's explicitly **out of scope**?
- What assumptions did it make, and were they **flagged** as assumptions or stated as fact?
- What's left as an **open question** for you to resolve?
- What's the acceptance criteria / how would you know this is done?

What to look out for:

1. Did you have to fight the tool's structure for a small change, or did it flex down reasonably?
2. Could a teammate pick up the spec/proposal folder and understand the change without talking to you?

### Area 2: Design

##### Tools:

- [Superpowers](https://github.com/obra/superpowers)
- [Agent-skills](https://github.com/addyosmani/agent-skills)
- [mattpocock skills](https://github.com/mattpocock/skills)
- [agents.md](https://github.com/agentsmd/agents.md)

##### Tools(Domain Specific)

- [agentskills.io](https://agentskills.io/home)
- [www.skills.sh](https://www.skills.sh)
- [anthropics/skills](https://github.com/anthropics/skills)

Task:

1. Pick one real architecture decision / coding standard on your project.

- Examples: "all external calls go through the retry wrapper", "no direct DB access outside the repository layer," "this service must stay stateless."

2. Write it once as a plain markdown note (e.g. a project .md file / requirement doc from (1)) and have the AI generate code that should comply.

- Check whether it actually does.

3. Then try to make the decision harder to violate, not just documented

- Examples: a skill that checks new code against the decision, or an MCP integration/tool that surfaces a violation automatically (e.g. a lint rule, a fixture, a custom check the AI is told to run).
- Compare: did the stronger version actually catch violations the plain markdown note didn't?

4. Explore processes / skills that can help move from intent --> explicit behavior and design artifacts --> implementation

What to look out for:

1. Decisions that get followed when stated explicitly in-prompt but quietly drift as the conversation continues — this is the main risk this capability is meant to catch.
2. Whether "encoding" the decision (skill/MCP/tool) is something a teammate could reuse, or whether it's too bespoke to your project to transfer.

### Area 3: Review

##### [Traditional Review]

Focus: Syntax, Typos, Local Logic -> Low Volume

##### [AI-Era Review]

Focus: Architecture, Edge Cases, Intent -> High Volume

##### Tools:

- [Understand-Anything](https://github.com/Egonex-AI/Understand-Anything)
- [graphify](https://github.com/safishamsi/graphify)
- [open-code-review](https://github.com/alibaba/open-code-review)
- [pr-agent](https://github.com/the-pr-agent/pr-agent)
- [Mira](https://github.com/miracodeai/mira)

Task:

1. Generic review template that our team can align with (example below)
2. Achieve a review process consist of

- Automated Quality Checks - Linting, formatting, type-checking,security scans
- AI LLM Review - Prompts checking against a predefined set of guidelines
- Human Review - Architecture, Business Logic

3. Propose best practices for AI-assisted code reviews

- Proof - able to explain how code works, edge cases, design decisions / approaches
- Size Limit - max. number of code lines and require breaking down to reviewable chunks
- Automated Quality Checks before Human Review
- Review Checklist
- Judge individual modules for their structural sense - AI generated flaws have syntactically perfect layout which makes structural/logical gaps "invisible". Human reviewers should evaluate code as raw, isolated logic, they are not there to catch messy formatting or typos, inconsistent naming or syntax mistakes.
- Elevate Test Coverage expectations

What to look out for:

1. Consider the difference between reviewing what the AI has implemented during your development session vs peer reviewing another human developer's AI-assisted changes.
2. Did you have to read the code to understand the behaviors and design decisions? If so, when will you need to read the code?

### Example of Review Template

#### 🎯 Core Intent & Context

<!-- Provide a clear, human-written summary of what this PR accomplishes and why. -->

- **Business Objective:**
- **AI Tool(s) Used:** (e.g., GitHub Copilot, Gemini, Claude, Cursor)

---

#### 🛠️ Validation Checklist

###### 1. Functional Verification

- [ ] **Local Execution:** I have successfully pulled and executed this code locally. (Do not merge unrun code).
- [ ] **Automated Test Results:** All existing tests pass, and new test cases have been added for the generated logic.
- [ ] **Edge Case Audit:** I have manually verified or added tests for:
  - [ ] Null, empty, or unexpected input types
  - [ ] Boundary limits (min/max values)
  - [ ] Unhappy paths (network drops, timeout handling, database down)

###### 2. Architectural Alignment

- [ ] **Zero Library Duplication:** I have checked that the AI did not rewrite internal helper functions, utilities, or components that already exist in our codebase.
- [ ] **Pattern Match:** The design patterns used here match our existing architectural standards (e.g., folder structure, state management, naming conventions).
- [ ] **Simplicity Check:** The code is as concise as possible. It is free of AI-generated "scope creep" or overly verbose, rigid structures.

###### 3. Trust & Safety Auditing

- [ ] **Security Scan:** No hardcoded secrets, injection risks, or insecure fallback parameters were introduced.
- [ ] **Hallucination Verification:** Every external API method call, third-party library parameter, and configuration setting used by the AI has been cross-referenced with official documentation to ensure it actually exists and is not deprecated.
- [ ] **Licensing:** Any net-new external dependencies introduced by the AI comply with our company's open-source licensing policy.

---

###### 👤 Human Ownership Sign-off

> ⚠️ **Reminder:** By checking the box below, you take 100% personal accountability for this code. The AI is a tool; you are the author.

- [ ] **I have thoroughly read, understood, and tested every line of code in this PR. I am prepared to debug, maintain, and support it in production.**

**Author Signature:** @username

<details>

<summary>
Phase 3 — Additional Details
</summary>

# Area 3: Review — Verifying AI-Assisted Changes

## Goal

Explore how code review should evolve when AI is involved in implementation.

The aim is not simply to add an AI reviewer or require humans to inspect every generated line. We want to understand:

> **What needs to be verified, who/what should verify it, and what evidence is sufficient for us to confidently own and release an AI-assisted change?**

By the end of this phase, we should be able to converge towards a lightweight and reusable review process for the rest of the team.

---

## Why Review Changes in the AI-Assisted Development Process?

AI can produce code quickly and often make it look structurally clean and plausible. This shifts the review problem.

Traditional review often spends significant effort on:

- syntax;
- formatting;
- naming;
- obvious implementation mistakes.

Many of these can increasingly be handled by tooling.

Greater attention should instead go towards:

- whether we built the **right behaviour**;
- assumptions made during implementation;
- architecture and codebase fit;
- failure and edge cases;
- whether tests actually prove what we think they prove;
- security and operational risks;
- unnecessary complexity or scope introduced by AI.

The three phases therefore connect:

```text
Phase 1 — Requirements
What should we build?
        ↓
Phase 2 — Design & Implementation
How should it be built?
        ↓
Phase 3 — Review
Did we build the right thing correctly,
and what evidence gives us confidence?
```

---

# The Change as the Unit of Review

Where possible, review the **change and its intent**, rather than asking AI to generally review an entire codebase.

A change could be:

- a PR;
- a feature branch;
- commits since `main`;
- staged/WIP changes;
- a bug fix;
- a refactor.

The reviewer ideally has access to:

```text
Requirement / Spec
        +
Architecture / Project Rules
        +
      Code Diff
        +
       Tests
        +
Runtime / Verification Evidence
        ↓
    Review the Change
```

This gives the reviewer enough context to answer more useful questions than simply:

> "Does this code look okay?"

---

# Suggested Review Process

The exact process is something we want to refine through this phase, but a useful starting point is:

```text
            AI-Assisted Implementation
                       │
                       ▼
               Author Verification
                       │
                       ▼
             Automated Verification
          lint / build / types / tests
          security / static analysis
                       │
                       ▼
              AI-Assisted Review
             where it adds value
                       │
                       ▼
                  Peer Review
                       │
                       ▼
                     Merge
```

Different checks belong at different layers.

The goal is not to make every layer perform the same review.

---

# 1. Author Verification — Before Requesting Review

AI assistance does **not** move responsibility for correctness from the author to the reviewer.

Before requesting review, the author should establish that they understand and can defend the change.

## The author should verify:

### Intent

- [ ] I understand what behaviour this change is intended to introduce or modify.
- [ ] I have checked the implementation against the requirement / acceptance criteria.
- [ ] I have checked that AI has not silently expanded the scope or added behaviour that was not requested.
- [ ] Any assumptions made during implementation are understood and acceptable.

### Implementation

- [ ] I understand the major implementation and design decisions.
- [ ] I understand which parts of the system are affected by the change.
- [ ] I have checked that existing project abstractions, utilities and patterns are reused where appropriate.
- [ ] I can explain any significant new abstraction, dependency or complexity introduced.

### Verification

- [ ] The change has actually been executed / exercised where applicable.
- [ ] Relevant automated tests pass.
- [ ] New behaviour has appropriate tests.
- [ ] I have considered relevant edge cases and failure paths.
- [ ] I understand what the existing verification **does and does not prove**.

### Ownership

The expected standard should not necessarily be:

> "I manually inspected every AI-generated line."

Instead:

> **I understand the behaviour introduced by this change, the significant implementation decisions, how it has been verified, and I am prepared to own, debug and maintain it.**

Where the implementation is complex, novel, security-sensitive or otherwise high-risk, deeper inspection should naturally be expected.

---

# 2. What Must Be Verified?

Regardless of whether verification is performed by a human, AI or deterministic tooling, these are the main dimensions we should establish confidence in.

## A. Intent & Functional Correctness

> **Did we build the right thing?**

Verify:

- [ ] Requirement / acceptance criteria are satisfied.
- [ ] No required behaviour is missing or partially implemented.
- [ ] No unintended behaviour or unnecessary scope has been introduced.
- [ ] Assumptions made by AI are valid.
- [ ] Important boundary conditions are handled.
- [ ] Relevant unhappy paths and failure scenarios are handled.

Useful review question:

```text
Given the requirement and this change, identify:
- missing requirements;
- partially implemented requirements;
- assumptions introduced by the implementation;
- behaviour introduced beyond the requested scope.

For every finding, point to the relevant requirement and code.
```

---

## B. Architecture & Codebase Fit

> **Did we build it in a way that fits the system?**

Verify:

- [ ] Relevant architecture decisions and project rules are followed.
- [ ] Existing abstractions / components are reused where appropriate.
- [ ] Functionality has not been unnecessarily duplicated.
- [ ] The change does not introduce a conflicting architectural pattern.
- [ ] Dependencies and coupling are reasonable.
- [ ] Complexity is proportional to the problem being solved.
- [ ] The change remains understandable and maintainable.

Reuse artifacts established during Phase 2 where possible:

- `AGENTS.md`;
- architecture decisions;
- project instructions;
- coding standards;
- skills;
- repository conventions.

Example:

```text
Review this change against the project's architecture decisions
and repository conventions.

Identify concrete:
- architecture violations;
- duplicated existing functionality;
- inconsistent patterns;
- unnecessary abstractions;
- unexpected coupling.

For each finding, provide evidence from the project rules or existing codebase.
```

---

## C. Tests & Verification Evidence

> **Do our tests actually give us confidence?**

Do not stop at:

> "The tests are green."

Verify:

- [ ] Relevant existing tests still pass.
- [ ] New behaviour is covered.
- [ ] Tests correspond to requirements / expected behaviour.
- [ ] Important boundary and negative cases are covered.
- [ ] Tests are not simply reproducing the implementation logic.
- [ ] Mocks and fixtures do not make tests pass trivially.
- [ ] We understand meaningful behaviour that remains untested.

A particularly useful question is:

> **What bug could exist while all of these tests still remain green?**

Example:

```text
Treat the implementation as potentially incorrect.

Using the requirement as the source of truth, review whether
the tests independently establish the required behaviour.

Identify:
- acceptance criteria without tests;
- missing boundary / failure scenarios;
- tests coupled too closely to implementation;
- bugs that could exist while all current tests remain green.
```

Where appropriate, tools such as **mutation testing** can provide stronger evidence by deliberately introducing implementation faults and checking whether the test suite detects them.

Examples:

- PIT — Java/JVM
- Stryker — JavaScript/TypeScript

---

## D. Failure & Edge-Case Behaviour

> **How can this change fail?**

Rather than only asking an AI reviewer to confirm correctness, it can be useful to deliberately attempt to **disprove correctness**.

This is sometimes called adversarial, doubt-driven or failure-seeking review.

```text
Normal review

"Does this look correct?"
        ↓
Inspect implementation


Failure-seeking review

"Assume there is a bug.
How can this fail?"
        ↓
Construct concrete scenarios
        ↓
Check whether the implementation handles them
```

Consider where relevant:

- null / empty / unexpected inputs;
- min/max boundaries;
- partial failures;
- network failures and timeouts;
- retry behaviour;
- idempotency;
- concurrency and race conditions;
- event ordering;
- state transitions;
- data consistency;
- dependency failures;
- permission / trust boundaries.

Example:

```text
Assume this implementation contains a bug.

Construct realistic inputs, event sequences or failure
conditions that could cause it to violate the requirement.

Prioritise concrete scenarios that can be verified or tested
over generic best-practice observations.
```

AI findings here are **hypotheses**, not proof.

Where possible:

```text
AI identifies scenario
        ↓
Check whether it is possible
        ↓
Reproduce / add test
        ↓
Confirm or reject finding
```

---

## E. Security & Data Handling

Where applicable, verify:

- [ ] Inputs crossing trust boundaries are appropriately validated.
- [ ] Authentication / authorization has not been weakened.
- [ ] Sensitive information or secrets are not introduced or leaked.
- [ ] Database / command / template injection risks are considered.
- [ ] New external APIs and dependencies are legitimate and appropriate.
- [ ] Configuration parameters actually exist and are correct for the version being used.
- [ ] Error handling does not expose sensitive information.
- [ ] Data access and permissions match expected behaviour.

AI may confidently generate APIs, parameters or configuration that look valid but do not exist or are outdated.

Novel external APIs / libraries / configurations should therefore be verified against authoritative documentation rather than trusted solely because the generated code compiles or looks plausible.

---

## F. Operational Behaviour

Where relevant to the change, verify:

- [ ] Timeout behaviour is appropriate.
- [ ] Retry behaviour is safe.
- [ ] Operations that may be repeated are appropriately idempotent.
- [ ] Partial failures leave the system in an acceptable state.
- [ ] Concurrency / parallel execution is safe.
- [ ] Logs / metrics / traces provide sufficient visibility for important failure modes.
- [ ] Resource / performance implications are reasonable.
- [ ] Rollback or recovery behaviour is understood for higher-risk changes.

Not every change needs every check.

**Review depth should be proportional to the risk and behaviour affected by the change.**

---

# 3. Use Automation for Deterministic Checks (lower priority for this initiative)

Do not use human or LLM reasoning for problems that deterministic tooling can detect reliably.

Examples:

```text
Formatting             → Formatter
Code style              → Linter
Compilation             → Compiler
Type correctness        → Type checker
Regression              → Automated tests
Known security patterns → SAST / Semgrep
Dependency risks        → Dependency scanner
Test strength           → Mutation testing where useful
```

Ideally:

```text
                   CODE CHANGE
                        │
                        ▼
              Automated Quality Gates
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Build          Tests        Static /
       Lint           Types        Security
          └─────────────┼─────────────┘
                        ▼
                  Review Evidence
```

A useful principle:

> **Automate checks that are repeatable and deterministic; reserve human and AI reasoning for questions that require context and judgment.**

---

# 4. Where AI-Assisted Review Can Help

AI review is useful where reasoning across multiple pieces of context is required.

Examples:

### Specification review

```text
Requirement ↔ Implementation
```

Does the implementation actually satisfy the intended behaviour?

### Architecture review

```text
Project rules ↔ Change
```

Does the change fit the existing system?

### Failure-seeking review

```text
Requirement + implementation
        ↓
Potential scenarios that break it
```

### Test review

```text
Requirement + tests
        ↓
What behaviour is not actually proven?
```

### Change-impact review

```text
Changed code
     ↓
What other components / behaviours might be affected?
```

AI can therefore act as an additional **reasoning layer**, rather than another linter.

---

# 5. Gotchas When Using AI for Code Review

## Polished Code Can Hide Incorrect Logic

AI-generated code can be syntactically correct, consistently formatted and well structured while still misunderstanding the problem.

Do not confuse:

> **Looks correct**

with:

> **Behaviour has been verified.**

Human attention should increasingly focus on:

```text
Intent
  ↓
Behaviour
  ↓
Assumptions
  ↓
Architecture
  ↓
Failure modes
```

rather than spending most review effort on formatting and syntax.

---

## Implementation and Tests May Share the Same Wrong Assumption

When AI generates both implementation and tests:

```text
        Same interpretation
          /            \
Implementation        Tests
```

both can be internally consistent and still fail the actual requirement.

Where possible, anchor verification back to:

```text
Requirement
     ↓
Expected behaviour
     ↓
Tests / evidence
     ↓
Implementation
```

---

## AI Review Findings Can Also Be Wrong

AI reviewers can:

- hallucinate problems;
- misunderstand project conventions;
- miss repository context;
- incorrectly describe APIs;
- overstate theoretical risks;
- recommend unnecessary abstractions;
- generate large amounts of low-value feedback.

Prefer **evidence-backed findings**:

```text
Finding
  ↓
Where is it?
  ↓
What scenario triggers it?
  ↓
What requirement / rule does it violate?
```

Avoid accepting a review comment simply because the model sounds confident.

---

## The Implementing Agent May Retain Its Own Assumptions

An AI that implemented a solution may reproduce the same reasoning when asked to review itself.

Self-review can still be useful because it has rich context, but a fresh reviewer / sub-agent may provide a more independent perspective for important changes.

Potential pattern:

```text
During implementation
        ↓
Implementation agent self-check

Before completion / merge
        ↓
Fresh-context review
        ↓
Spec / architecture / failure checks
```

Whether this additional review is worthwhile may depend on change risk.

---

## Bigger AI-Generated Changes Can Become Hard to Review

AI can generate large changes much faster than humans can understand them.

Watch for:

- unrelated refactoring;
- unnecessary files;
- excessive abstractions;
- multiple conceptual changes mixed together;
- generated boilerplate that obscures important behaviour.

Instead of prescribing an arbitrary maximum LOC, consider:

> **Is this change conceptually small enough for someone to understand its intent, impact and verification evidence?**

---

# 6. Supporting Tools / Skills

The focus of this phase is **the review process**, not learning a particular tool.

Tools and skills are possible ways of implementing parts of that process.

They will change over time; the underlying verification principles should remain reusable.

## Reusable Review Skills

Examples:

- Matt Pocock `/code-review`
- Superpowers review skills
- Addy Osmani review / doubt-driven skills

Useful patterns from these approaches include:

```text
Git Diff
   │
   ├──► Specification review
   │
   ├──► Standards / architecture review
   │
   ├──► Correctness / failure review
   │
   └──► Test evidence review
```

Rather than adopting a skill wholesale, consider what review behaviours are useful enough to standardise.

For example, we may eventually create our own:

```text
/review-change

1. Read the change.
2. Read its requirement/spec.
3. Read relevant project rules.
4. Check automated verification evidence.
5. Check requirement compliance.
6. Check architecture compliance.
7. Look for realistic failure scenarios.
8. Review test evidence.
9. Return only evidence-backed findings.
```

---

## Dedicated AI Code-Review Tools

Examples:

- OpenCodeReview
- Qodo / PR-Agent
- GitHub Copilot Code Review
- other PR-integrated review agents

These may help automate review as part of the Git/PR workflow.

The important questions are not which tool is "best", but:

- What part of our review process does it automate?
- What context does it have access to?
- Are its findings useful and evidence-backed?
- Does it reduce review effort or simply generate more comments?
- What must a human still verify?

---

## Code / Change Understanding

Examples:

- Understand Anything
- Graphify

These may help answer:

> **What is the impact or blast radius of this change?**

They may be useful for larger or unfamiliar codebases where reviewers need help locating dependencies and affected components.

---

## Runtime Verification

Where useful, agents can also be given tools that let them inspect the actual running application.

Examples:

- browser automation;
- Chrome DevTools MCP;
- API testing tools;
- logs / traces;
- database inspection;
- test runners.

This allows review to move from:

> "The code appears to do X."

towards:

> "We have evidence that the system actually does X."

---

# 7. Areas to Explore During These 4 Weeks

There are **no fixed experiments that everyone must complete**.

Apply these ideas naturally to real development work and explore whichever areas are useful.

Things to pay attention to:

### Author verification

- What should I personally verify before asking someone else to review?
- What evidence made me confident in my own change?
- Did I understand the AI-generated implementation well enough to own it?

### What needs verification

- Which review dimensions repeatedly mattered?
- Which checklist items were irrelevant for certain changes?
- Are there important checks missing from the baseline?

### Automation

- Which checks should happen automatically before review?
- Are humans repeatedly checking something that tooling could enforce?

### AI-assisted review

- Where did AI genuinely help?
- What did it catch that I might otherwise have missed?
- What types of comments were mostly noise?
- What context made its review better?

### Human review

- What still required engineering judgment?
- Which parts of the implementation required deeper reading?
- Did higher-risk changes require a different review depth?

### Verification evidence

- Did tests actually establish expected behaviour?
- Would runtime evidence have helped?
- Could a supposedly "verified" bug still exist?

---

# 8. Individual Journal

### What did you review?

Briefly describe the change and its risk / complexity.

### How did you verify it?

For example:

- manual execution;
- tests;
- static analysis;
- requirement mapping;
- AI-assisted review;
- runtime verification;
- peer review.

### What worked well?

Share concrete examples where possible.

### What did not work?

For example:

- noisy AI findings;
- missing context;
- tests that gave false confidence;
- overly large changes;
- architecture issues not caught automatically.

### What still required human judgment?

### What should become part of our normal process?

Think in terms of:

- author responsibilities;
- automated checks;
- AI-assisted checks;
- peer review;
- deeper review for higher-risk changes.

---

# Intended Outcome

At the end of the four weeks, aim to converge on a lightweight model consisting of:

```text
                    CODE CHANGE
                         │
                         ▼
                Author Verification
                         │
                         ▼
              Verification Evidence
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
     Automated       AI-assisted      Runtime /
       checks          checks          testing
          └──────────────┼──────────────┘
                         ▼
                 Human Judgment
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
       Author ownership       Independent /
                             Peer review where
                           required by risk-based guidance/verification checklist
                         │
                         ▼
                       Merge
```

Potential reusable outputs:

1. **Author verification checklist**
2. **Code-change verification checklist**
3. **Recommended automated quality gates**
4. **Guidance for effective AI-assisted review**
5. **Common AI-review / AI-generated-code gotchas**
6. **Guidance on when deeper human review is needed**
7. **Optional reusable prompts / skills / tooling to support the process**

The intended outcome is not more review ceremony.

> **It is a repeatable way to determine what must be verified, establish appropriate evidence, and allow developers to confidently own and release AI-assisted changes.**

## Resources

- https://shiftasia.com/column/how-to-review-ai-generated-code-the-complete-developers-guide/#toc0
- https://tenki.cloud/blog/reviewing-ai-generated-code-checklist
</details>
