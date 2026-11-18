<details>
<summary> My raw notes
</summary>

# What I did

1. Used 2 of matt pocock's skills (`/grill-with-docs` and `/to-spec`)

# Speckit (Continuation)

- Uses hooks to perform scripts like `/speckit-plan`
- Did `/speckit-plan`, subsequently it asked to do `/speckit-tasks`
- Done with:

```
The best starting set is:

spec.md — the actual requirements (included Q&A summarised) and acceptance criteria
plan.md — the implementation approach and target architecture
tasks.md — the concrete work breakdown
research.md — the decisions made for unclear parts
Then use these as supporting references:

data-model.md — the types/entities
semantic-search-api.md — the API contract
quickstart.md — how to validate it
```

## Thoughts

- The specs all went very deep into the implementation (even down to API level, implementation, quickstart) which not sure actually how reliable it will be
  - Will be good for those who know the codebase inside out to review the implementation-related Specs
  - This is similar to Superpowers brainstorming but a lot more verbose
- The Spec itself though detailed, but useful to some extent:
  - User story (with priority reason & how to test) + Acceptance Scenarios
  - Edge cases
  - Broken down to functional requirements: Function (e.g.: Search Tab Selection) then different modular tasks
    - But hard to link user story/acceptance scenarios to these functional rqeuirements
  - Assumptions
    - But does this mean need to correct assumptions then change the above again?
    - These were not clarified before
- The constitution quite good for coding standards/principles upfront
  - But not sure the "status" (pass/watch) for what when haven't even implemented
- The plan.md
  - Hard to draw from spec -> plan: like why those plans are there
- The tasks.md
  - Very verbose, needs a lot of linking back to user story and some thoughts on implementation to verify (e.g.: "run tasks in parallel because different domain")
  - Includes "Parallel Execution Plan" which requires a lot of links back to tasks (means need to have this side by side) and also more coupling (like if US wrong / tasks wrong / implementation wrong then needs to change downstream)
- The research.md
  - Useful if want to think about different alternatives/considerations for the bigger concepts that need to be implemented as part of the functionality
- The data-model.md
  - Again, hard to link back on why we do this and very coupled with implementation. May not need to be down to this level??
- The quickstart.md
  - Wondering how these verification steps complement/gel with the "Performance verification" / "Tests" in tasks.md

# Matt Pocock

## `/grill-with-docs`

- Asked a series of questions (~3 sets of 5 questions & 1 set of 3 questions)
- Then ended off asking if I want to implement already, which i later prompted to write specs first
  - This triggered `/feature-spec` skill that didn't come up a very good spec (included snippet of implementation which has no relation to current code base; may cause pollution after)
- Then i specifically put `/to-spec`

## `/to-spec`

- It went to read back our convo during `/grill-with-docs` then gave:
  - Problem Statement
  - Solution
  - User Stories
  - Implementation Decisions
  - Testing Decisions (this even included "What makes a Good Test", "Prior Art / Existing Test Patterns", "Testing Layers")
  - Out of Scope
  - Further Notes (e.g.: Data storage implications)
    - Most of these somehow seems hallucinated / assumptions (e.g.: Performance Profile, Chunk Size Recommendations, Further Extensions)

## Thoughts

- Personally i feel that the specs generated seems more "workable" to review (i.e. to follow) because it focuses more on really what needs to be done without too technical deep dive into solutioning
  - This allows validating whatever the AI understands/generates are aligned to what you want it to do
  - It is also comprehensive in what needs to be laid out front (e.g.: user stories, testing, out of scope) which can easily be refined afterwards because it's modular
- I like that there's "Rationale" and "Implications" for the implementation decisions
  - Implementaiton decisions is more of like "principle/approach" which i thought is a good first step to look at before showing the actual technical implementation

# Comparisons

## Superpowers

- Like that there's some form of visuals in markdown which allows verification on AI's understanding
  - Not sure if this also helps the AI with development and testing later?
- Tasks broken done into modular size with step-by-step on what will be done, including tests
  - This would be useful if have some idea on what should be done / that the steps are straightforward with enough context for relatively newer developer to the codebase
- Will need to verify if there are extra tokens consumed, but there are the plan.md created and the `.superpowers/sdd/task-1-brief.md` that is a similar content for the task 1 in plan.md
- The report contains content like "Concerns / Deviations" section which then highlighted what deviated from the brief.md (e.g.: after running tests) and hence was implemented in certain way
  - Would that mean reviewing could be like:
    - Specs
    - Brief
    - Report
    - Code (?? or assume the report says done means same as actual code produced)
    - Tests
- The spec doesn't seem to have the same format throughout like "Objective -> Functionality (What, Trigger, Expected Behaviour)". Now: "Overview, Hierarchy, <diff parts of the design>, Open questions, known limitations, out of scope"
  - May be hard to review / loose details
  - "Known limitations" talked about some current technical implementation stuff, not sure how to track & concrete actionable. It is a mix of: - "Include in follow-up" - "Functionally correct today; a future test using ... will fail"
  </details>

# Notes

I tried three approaches to generating specs before implementation — Speckit, Matt Pocock's `/grill-with-docs` + `/to-spec`, and Superpowers. Writing this up because a pattern kept showing up across all three: **the tools are good at producing detail, but bad at showing why the detail is there.** That's the thing worth discussing as a team, more than which tool "wins."

## The core insight

Every tool eventually asks the same question, just at different altitudes: _how much should the spec decide, vs. leave for implementation?_

- Speckit answers "a lot" — it goes all the way to API contracts and data models before a line of code is written.
- Matt Pocock's skill answers "not much" — it stays at problem/solution/decision-rationale level and stops short of solutioning.
- Superpowers sits in between, but inconsistently — the spec format changes doc to doc, so how much gets decided upfront isn't predictable.

None of these is objectively right. But it means **"write a spec" isn't one task** — it's a choice about altitude, and right now that choice is being made implicitly by whichever tool we happen to run, not deliberately by us. That's probably the first thing worth agreeing on as a team.

## Comparison

|                              | Speckit                                                    | Matt Pocock skills                                                     | Superpowers                                                                                                                   |
| ---------------------------- | ---------------------------------------------------------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Altitude                     | Implementation-level (API, data model)                     | Problem/decision-level, deliberately avoids solutioning                | Mixed, varies by section ("Overview, Hierarchy, <diff parts of the design>, Open questions, known limitations, out of scope") |
| Easiest to review            | If you know the codebase deeply                            | If you want to sanity-check shared understanding first                 | If you want a visual gut-check                                                                                                |
| Traceability spec→plan→tasks | Weak — hard to see why a plan follows from a spec          | N/A (doesn't go this deep)                                             | Mixed — spec structure isn't consistent enough to trace against                                                               |
| Where it hallucinates        | Assumptions section, listed after the fact                 | "Further Notes" (performance profile, chunk sizing, future extensions) | "Known limitations" mixes real gaps with vague technical asides                                                               |
| Useful artifact              | `research.md` — genuinely useful for weighing alternatives | Implementation Decisions w/ Rationale + Implications                   | Post-implementation "Concerns/Deviations" report                                                                              |

## What each one is actually good for

**Speckit** front-loads so much that it starts to resemble a design doc more than a spec — which is _useful_ if your team already trusts the direction and just needs the detail nailed down. But this is also why it feels hard to review: you're being asked to approve implementation choices before you've agreed the direction is right. The constitution doc (coding standards upfront) is a genuinely reusable idea independent of the rest — worth stealing even if we don't adopt Speckit wholesale.

**Matt Pocock's flow** is the most "reviewable" precisely because it refuses to solutionize. The Rationale + Implications pairing on each decision is doing real work — it's forcing the AI to justify _why_, which is exactly what's missing from Speckit's plan.md. The failure mode (hallucinated "Further Notes") only shows up when you skip straight to `/feature-spec` — which suggests the two-step process (`grill-with-docs` → `to-spec`) matters more than either step alone. Worth testing: does skipping the grilling step and going straight to `/to-spec`-style output on a vaguer prompt produce the same hallucinations?

**Superpowers** is the only one that tries to close the loop back to reality — the "Concerns/Deviations" report after implementation is a good idea nobody else has. But it's undermined by the spec itself not having a fixed shape, so you can't reliably check "did the deviations report match what the spec actually promised." Fixing the format would probably make this the strongest end-to-end tool of the three.

## Questions worth raising with the team

1. **Altitude first, tool second.** Before picking a tool, should we agree on how deep a spec should go — problem/decision level (Pocock-style) vs. implementation-ready (Speckit-style) — and let that decision drive which tool/template we use per project type?
2. **Where does assumption-clarification belong?** All three tools surface assumptions _after_ generating the spec rather than _during_ clarification. Should "assumptions" be a live checklist reviewed with a human before the spec is considered final, rather than an appendix?
3. **Is traceability worth the coupling cost?** Speckit's tight linking (spec→plan→tasks→parallel execution) gives traceability but means one wrong upstream decision cascades. Is that a fair trade, or should we deliberately keep looser coupling and accept some drift?
4. **Should we standardize our own spec template** rather than rely on each tool's default structure? A shared shape (e.g., always: Problem → User Stories → Decisions+Rationale → Testing → Out of Scope → Open Questions) would make specs from different tools/people comparable, instead of each tool teaching us its own format.
