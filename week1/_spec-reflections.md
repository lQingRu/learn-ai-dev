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
  - "Known limitations" talked about some current technical implementation stuff, not sure how to track & concrete actionable. It is a mix of:
    - "Include in follow-up"
    - "Functionally correct today; a future test using ... will fail"
