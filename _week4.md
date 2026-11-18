# What I did

1. /setup-mattpocock-skills

- custom tracker (jira format/development story)

2. /grill-me-with-docs

- search
- llm

3. /codebase-design

4. /code-review

- /code-review is a diff review skill. It looks at the changes since a point you choose and reviews them on two axes:
  - Standards
    - does the code follow this repo’s conventions and quality bar?
  - Spec
    - does the code match what this slice was supposed to achieve?
- Ran `/ask-matt` on how to best use `/code-review`
  - it came up with a prompt:
  ```
  /code-review since HEAD for the reference end-to-end slice implementing search candidate assembly, raw hydration, preview drill-down, and the SearchReference UI. Prioritize seam quality, spec alignment, misleading coupling, and whether
  the tests validate the right behavior.
  ```

## Coding principles

1. Used `/ask-matt` on what is the correct way to specify coding principles etc.

```
Tier 1 — CLAUDE.md (exists, mechanical/stack-level)
Tech stack, file structure conventions, the API-layer pattern, component conventions. This is "how we write code here" — general, slow-changing, applies regardless of what feature you're building.

Tier 2 — CONTEXT.md + docs/adr/ (documented, but doesn't exist yet)
docs/agents/domain.md (line 15) declares the intended layout — a root CONTEXT-MAP.md, per-package CONTEXT.md (glossary of domain terms) and docs/adr/ (decisions, including functionality-specific "why did we build X this way, and why not Y"). I checked: none of these files exist anywhere in the repo yet. That's by design, not a gap — line 11 of that doc says they're created lazily by /domain-modeling (invoked from /grill-with-docs) the first time a term or decision actually needs resolving, not scaffolded upfront.

What this means for you concretely:
- Coding principles (formatting, patterns, structure) → CLAUDE.md. Keep it there.
- Functionality-specific stuff (domain vocabulary, "we chose approach A over B because...", package-scoped tradeoffs) → doesn't live in CLAUDE.md at all. It accumulates in apps/gv/CONTEXT.md / apps/gv/docs/adr/ etc., but only once you actually run /grill-with-docs on a real feature and a decision crystallizes. Right now that layer is empty, so any functionality-specific reasoning you've done so far is only in your head or in past conversations — nowhere durable.

On "is CLAUDE.md comprehensive and up to date": I can't answer that from the doc itself — it can only be checked against actual code. Want me to spot-check it (e.g. pull 2-3 real api/* modules and component folders and diff their actual shape against what CLAUDE.md claims), so you know whether it's drifted before you start building on top of it?
```

- By right CLAUDE.md will be applied at project-level and looked at by agent, and during the `/grill-with-docs`, will also document the principles to follow-through during implementation

```
- CLAUDE.md says "conventional commits are enforced via commitlint" — but there's no commitlint config, no Husky, no lint-staged anywhere in the repo.
- No CI at all (.github/workflows doesn't exist).
- No MCP servers configured (.mcp.json absent) — despite Jira (project CP) being your documented issue tracker.
- No project-level subagents (.claude/agents/*.md) — only the global skill plugin cache.

So right now, the only thing standing between "agent writes code" and "code merges" is the LLM judgment in /code-review's Standards axis, plus whether anyone remembers to run pnpm check by hand. That's a soft gate, not a hard one — an agent (or a rushed human) can just skip it.

Ranked by leverage, here's what closes that gap:

1. Pre-commit hooks — highest leverage, cheapest to add
Husky + lint-staged running Biome format/lint + typecheck (+ maybe affected tests via Nx) on every commit. This is a hard local gate — no reliance on anyone remembering. There's a dedicated agent for exactly this: setup-pre-commit. Since CLAUDE.md already claims commitlint is active, this would also true-up the doc. I'd want to include commitlint here too, to actually deliver on what's documented.

2. CI workflow — the backstop hooks can't provide
Hooks can be bypassed (--no-verify) or simply never installed after a fresh clone. A GitHub Actions workflow running lint / typecheck / test on every PR is what actually guarantees nothing non-conforming merges, regardless of local setup. Not part of the Matt skillset — happy to set it up separately if you want it.

3. Project-level subagent — for conformance checks outside the skill flow
A .claude/agents/ entry (e.g. a repo-tuned code-reviewer) whose system prompt bakes in your exact CLAUDE.md rules is useful if you want the same standards check invokable ad hoc ("review this file") rather than only inside /code-review. Once CONTEXT.md/ADRs start existing, this subagent is also where you'd point it at domain-vocabulary drift. Lower priority than 1–2 since /code-review's Standards axis already covers this inside the main flow.

4. MCP — capability, not conformance
A Jira MCP server would let /triage, /to-issues, /to-prd read/write real tickets in project CP directly instead of you copy-pasting. Doesn't enforce coding principles — it's about closing the loop on the issue-tracker workflow you've already documented. Worth doing, but it's a different problem than "does the code conform."
```

- Once implementation is done, can run `code-review` skill which reviews the diff between HEAD and a fixed point you supply — a commit, branch, tag, or merge-base — along two separate axes: Standards (does the code follow this repo's documented conventions?) and Spec (does it implement what the originating issue or spec asked for?)

2. Quick research on community thoughts

- https://www.reddit.com/r/softwarearchitecture/comments/1urpnxr/how_are_you_enforcing_engineering_standards_with/
  - "Had better results by treating standards as part of the workflow instead of the prompt. architecture rules, review gates, and shared context made a bigger difference than prompt tweaks"
  - "Included linters, style checker, type checker, security checker. So now I only review functional test code and implementation code that an AI code reviewer points out should be reviewed by a human."

3.  Through `/grill-with-docs`, it actually already included the coding principles as part of the specs (like ownership, interface) during discussion

4.  Will try out:

- Include in AGENTS.md for things like mock data UI validation with a few notes (like prefer MSW), then point to `docs/agents/testing-and-mocks.md` for the detailed policy

## Switched to codex

- `graphify install codex`
- Move CLAUDE.md to AGENTS.md
- Fixed issues with codex not running the pnpm using corepack (differing versions as stated in package.json)

## Focus: search

### Continuation

1. Previously I already drafted specs/UNIFIED_SEARCH.md, but then it was created in a rush, so I wanted to relook and revisit to segregate them

- Used `/ask-matt` to see the right approach/skill to do so

```
This is a good fit for /grill-with-docs — it's built for exactly this: taking an existing spec, challenging its assumptions against your domain model, and updating docs inline as decisions get re-made. Since you have a codebase (and this file already lives under docs/specs/), that's the version to use over /grill-me.

Concretely I'd frame the grilling session around the split you already see: your doc mixes two concerns — retrieval (metadata/content/semantic search + merge/dedup, ID1–ID3, ID6–ID11) and LLM relevance scoring (ID4, ID5, ID12 — the async scoring, batching, streaming-scores UX). Semantic search itself (embedding-based retrieval) arguably stays in "search"; it's the scoring/ranking layer that's the LLM concern worth pulling out into its own spec.

Main tradeoff to flag going in: right now sync-vs-async is entangled — parallel retrieval (ID1) is sync-ish/blocking-until-merged, while LLM scoring (ID4) is deliberately decoupled and streamed. If you split the docs, you'll want the search spec to define the "matchedBy + pending llmScore" contract as its public interface, so the LLM spec can evolve independently without the search spec caring how scoring gets fulfilled.

Want me to kick off /grill-with-docs now against this file, with that split as the opening frame?
```

- This allowed me to run through targeted "grilling" again to refine the docs, and eventually split out the docs with better demarcations
  - This also seems to simulate cases where we may have an implemented feature / old docs that require updating
  - It says

  ```
  # Unified Search

  **Status**: Superseded
  **Date**: 2026-07-24
  ```

2. After splitting out the plans into more concrete parts. I used `/codebase-design` which prompts me for some technical designs (such as invariants, inputs, outputs) which will serve as use for contract

- Created a lot of more granular specs with good amount of abstraction

3. Before moving to implementation, it tells me what to focus on first (which i prev. highlighted i want modularised to run through e2e with code review first)

```
Then the next step is implementation.

Start with the smallest pure slice:

- code contract types for CandidateAssembler
- fixture-driven tests
- minimal implementation to satisfy them

That gives you a real seam in code without committing to hydration strategy, preview wiring, or ranking yet.

If you want, I can start creating that first TDD slice now.
```

4. It began to start with it's TDD approach, and let me know the changes and what to do next

```
I codified the seam in code without implementing the grouping engine yet.

Added:

- apps/gv/src/api/incp-search/candidate-assembler.types.ts:1
- apps/gv/src/api/incp-search/candidate-assembler.fixtures.ts:1

These mirror the design note:

- chunk-grain retrieval input
- INCP/Raw-group candidate output
- duplicate chunkId example with merged retrievalTypes
- components["schemas"]["Incp"] carried through on the candidate

I also exported the types from apps/gv/src/api/incp-search/index.ts:1.

The next step is to add contract tests against these fixtures, still without committing to the real assembly implementation.
```

- codebase-design for placing the seams and defining the interfaces
- tdd only for the narrow “codify the contract in code” step, not for implementing the real grouping engine yet

```
No, I do not need to compact first.

This thread is still coherent:

- the search split is done
- the three design seams are written
- the first code-facing contract files are in place

I would compact only if we were about to start a much larger implementation pass across multiple files and the context was getting noisy. Right now, staying in the same thread is better because the design
decisions are still active and specific.
```

### Code Review

- Did `/code-review`

```
## Standards

  1. candidate-assembler.contract.test.ts is testing the fixture document, not the seam implementation, so it can pass while createCandidateAssembler().assemble() diverges from the contract. That is a test-quality problem at the public
     seam, not a real behavior check. See apps/gv/src/api/incp-search/candidate-assembler.contract.test.ts:8. This is close to the TDD anti-pattern of tautological/specimen testing: the expected output is hand-authored and never compared
     against the implementation.

  2. SearchReference is carrying real assembly, hydration, and drill-down adapter creation inside the component via three local factories (apps/gv/src/components/search-reference/search-reference.tsx:22). As a reference slice that is
     acceptable, but as a seam it is a possible “Middle Man / Speculative Generality” mix: the adapters exist, but the caller still knows too much about how many stages it must instantiate and sequence.

  ## Spec

  1. The assembler is order-sensitive in ways the design says it should not be. Duplicate chunk fusion is supposed to preserve origin while remaining deterministic (docs/specs/SEARCH-CANDIDATE-ASSEMBLY-DESIGN.md:111, docs/specs/SEARCH-
     CANDIDATE-ASSEMBLY-DESIGN.md:203). But apps/gv/src/api/incp-search/candidate-assembler.ts:35 keeps the first-seen incp/raw, and apps/gv/src/api/incp-search/candidate-assembler.ts:70 use ??= for excerpt, highlight, score, and
     chunkIndex. If parallel retrieval returns the same chunk in a different order, richer later data is dropped. That violates the stated invariant that the same retrieval input yields the same candidate output and risks losing projected
     parent fields.

  2. The reference UI silently drops every INCP candidate except the first one. apps/gv/src/components/search-reference/search-reference.tsx:35 takes hydratedResult.candidates[0] and the rest of the component only operates on that one
     candidate. The assembly and hydration specs describe a candidate set, not a single candidate (docs/specs/SEARCH-CANDIDATE-ASSEMBLY-DESIGN.md:149, docs/specs/SEARCH-RAW-HYDRATION-DESIGN.md:172). For a review slice this may be
     intentional, but it is still a misleading coupling if readers assume this proves multi-INCP behavior.

  3. The drill-down slice does not actually prove the preview integration the spec asked for. The drill-down spec says the first slice should use the existing preview pattern and show chunk hits in the preview text-extract tab (docs/specs/
     SEARCH-RAW-HYDRATION-DESIGN.md:100, docs/specs/SEARCH-PREVIEW-DRILLDOWN-DESIGN.md:133). But apps/gv/src/components/search-reference/search-reference.tsx:112 renders a bespoke local Tabs panel and only states it is “Similar to
     PreviewViewerWrapper”. It demonstrates the data shape, not the real preview seam.

  Summary: Standards 2 findings, worst is the seam tests validating fixtures rather than behavior. Spec 3 findings, worst is the order-sensitive assembler that can drop richer duplicate-hit data depending on retrieval order.
```

- Asked reasons on why the drifts:

```
A few things.

  1. The reference-slice goal should have been written down more sharply before coding.
     Right now there were two competing goals:

  - prove the seams end-to-end
  - stay close to the real preview path
    Those are not identical. If you had explicitly said “this slice may use a temporary reference UI, but the seam contracts must remain production-valid,” the later review would have had less room to call the temporary UI a spec miss.

  2. The “temporary vs production” boundary should have been encoded in the code structure.
     The SearchReference path should have been obviously marked as a reference harness, not something that looked halfway like the real flow. When temporary review code sits next to real paths without a strong label, reviewers reasonably treat it as intended architecture.

  3. The seam tests should have tested the implementation, not just the fixtures.
     That is probably the biggest preventable issue. If the contract tests had exercised createCandidateAssembler().assemble(...) against the worked examples, the review would have had less ground to say the seam wasn’t really verified.

  4. The design docs should have recorded explicit temporary allowances.
     For example:

  - first slice may handle only one INCP in the review harness
  - first slice may use a reference drill-down component instead of the real preview integration
  - these are evaluation shortcuts, not target behavior
    That would have turned some “spec deviations” into documented non-goals for the slice.

  5. One more design checkpoint was needed before implementation.
     Specifically:

  - “What is the smallest production-valid behavior?”
  - “What is the smallest review-valid shortcut?”
    You built something between those two, which is where ambiguity creeps in.

  So the short answer is:
  better slice definition, clearer temporary-vs-real boundaries, and behavior tests at the seam would have prevented most of the drift.
```

- Not sure how much I can trust this `/code-review` outcomes also. seems a bit fluff / too high-level (so ended up i had to cross check myself such as cases where it says implementation doesn't align to specs)
  - specs itself indeed is what i expect

## Thoughts

1. Spent bulk of the time talking about design / considerations etc

- noticed that u need to be very proactive yourself too to think about possible approaches/gotchas (seems to be the case regardless of which tools, like they don't "think" so much for u)

2. A good practical flow for a case of changes in existing codebase:

- `/grill-with-docs` first
- decide whether the change is:
  - spec patch
  - ADR update
  - spec split

- if the change is still small after grilling, skip `/to-spec` and go straight to `/implement`
- if the change alters behavior or assumptions, start by updating the spec shape through /grill-with-docs before implementation

## Side notes

- The choice of LLM really matters, previously I was using Claude Code then it was able to execute the skill quite well, but when I switched to use CoPilot (free) as an interim which the model was shown to be routed to MAI-Code-1-Flash, it did not even care to run the skill well and even went to autonomously edited my spec file without any clarifications / the expected behaviour of the skill
- Struggled with how to split and focus on the work that is more certain first (since many unknowns like what search behaviour, what approach we want to do grouping/aggregation) which i honestly think the `/grill-with-docs` helped quite a bit to bounce ideas and organise them
- Possibility of out-of-scope changes if didn't prompt properly?
  - e.g.: trying to solve an existing tech debt

```
I’ve confirmed the Search schema mismatch. I’m fixing the fixture to use field/order and pageSize/page, and I’m checking whether the table-raw-view story error is genuinely pre-existing or trivial to clean up while I’m here.

• The story type error is cheap to clean up and already sitting in the codebase. I’m fixing both that and the new fixture schema mismatch, then I’ll rerun tests and direct typecheck.
```

- Strange thing is somehow the specs were created with some line wrap that is inconsistent with actual repository setup

## Concerns

- Important to check when artefacts are created and the status of these artefacts (e.g: ADR)
  - In particular this https://github.com/mattpocock/skills/issues/299, actually state how ADR is created during planning but not yet implemented/decided which results in downstream consuming as "truth"
  - Planned decisions should be clearly separated from in-force ADR context or lifecycle-gated until implementation status justifies promotion

# What's Next

- Will continue to explore incorporating some coding principles for different use cases (such as testing / abstraction layer)
- Will continue to see what can be done to verify design and implementation
