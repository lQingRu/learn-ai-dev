# Design & Implementation (Mid-way)

## What I Explored

I experimented with:

- `/grill-with-docs`
- `/codebase-design`
- `/code-review`
- `/ask-matt`

The main test case was revisiting and implementing parts of our unified search design.

---

# Key Learnings

## 1. The developer still needs to drive the thinking

The largest portion of the exercise was spent discussing requirements, design choices, constraints, interfaces, and possible edge cases rather than writing code.

The tools helped structure the thinking, but they did not replace it. I still needed to proactively consider:

- Alternative implementation approaches
- Unknown or unresolved product behaviour
- Potential architectural coupling
- Edge cases and failure scenarios
- Which parts of the work were sufficiently certain to implement
- Whether the AI’s recommendations were actually appropriate for the codebase

My current impression is that this remains true regardless of the tool used: the developer must continue to supply judgment, context, and challenge.

The value is less about the AI independently producing the right design, and more about using it as a structured discussion partner that helps expose and organise decisions.

---

## 2. `/grill-with-docs` was useful for refining existing specifications

It helped revisit a rushed unified search specification and separate concerns such as:

- Retrieval
- Grouping and aggregation
- Hydration and preview
- LLM relevance scoring
- Synchronous and asynchronous processing

This seems useful for both new and existing features, especially when a specification needs to be patched, split, or updated after assumptions change.

A practical flow is:

1. Review the existing specification with `/grill-with-docs`.
2. Decide whether it needs a small patch, an ADR update, or a broader split.
3. Update the documentation before implementation.

---

## 3. Defining narrow implementation seams helped manage uncertainty

`/codebase-design` helped translate the design into:

- Inputs and outputs
- Invariants
- Interfaces
- Ownership boundaries

This made it easier to start with a small part of the feature without committing to the full design.

However, the slice must still represent meaningful production behaviour. Otherwise, it can become unclear whether the implementation is a prototype, test harness, or intended architecture.

---

## 4. TDD only helps when tests exercise the implementation

The workflow initially created types, fixtures, and contracts before the implementation.

However, some tests only validated the fixtures rather than invoking the actual implementation. They could therefore pass even if the implementation was incorrect.

Contract tests should execute the real public interface and verify behaviour such as determinism, duplicate handling, and output invariants.

---

## 5. AI code review is useful, but still needs human verification

`/code-review` identified possible issues such as:

- Tests that did not validate actual behaviour
- Order-sensitive processing
- Differences between the implementation and specification
- Temporary UI behaviour that could be misleading

However, some findings felt high-level or overstated. I still needed to verify whether:

- The specification actually required the claimed behaviour
- The deviation was intentional
- The reviewer understood the slice’s scope

I currently see AI review as a way to generate review leads, not as an authoritative verdict.

---

# Engineering Standards

## 6. Separate repository-wide rules from feature-specific context

`AGENTS.md` or `CLAUDE.md` should contain stable guidance such as:

- Repository structure
- Coding conventions
- API and component patterns
- Commands for linting, testing, and type-checking

Feature-specific reasoning should live in specifications, ADRs, domain documentation, or focused policy documents.

- Something that `/grill-with-docs` helps with

For example, `AGENTS.md` could briefly state the preferred mocking approach and link to a detailed testing policy.

---

## 7. Standards should be enforced by tooling, not only prompts (KIV)

A repository may document standards without actually enforcing them through:

- Pre-commit hooks
- CI checks
- Linters
- Type-checkers
- Automated tests

A recurring community view is that engineering standards are more effective when embedded into the workflow rather than relying on prompt instructions alone.

- https://www.reddit.com/r/softwarearchitecture/comments/1urpnxr/how_are_you_enforcing_engineering_standards_with/

My takeaway is:

1. Use deterministic local and CI checks as the foundation.
2. Use AI review for higher-level conformance.
3. Use MCP integrations for workflow convenience, not quality enforcement.

---

# Model and Workflow Concerns

## 8. Model choice really matters

The same skill behaved differently across models.

Claude Code generally followed the intended workflow more closely. During one experiment, another model skipped the expected clarification process and directly edited the specification.

A skill should therefore be evaluated together with the underlying model and agent runtime.

---

## 9. Scope expansion remains a risk

The agent sometimes considered fixing nearby technical debt or unrelated errors while implementing the requested change.

This may be useful, but it makes review harder and increases the risk of unintended changes.

Prompts and plans should state clearly:

- What is in scope
- What should remain untouched
- Whether opportunistic cleanup is allowed

---

## 10. Generated documents need lifecycle states

AI-generated specifications and ADRs may be created during planning before the decision is accepted or implemented.

- https://github.com/mattpocock/skills/issues/299, actually state how ADR is created during planning but not yet implemented/decided which results in downstream consuming as "truth"
- If downstream agents treat these as authoritative, proposed decisions may become repository “truth” too early.

Documents should clearly indicate states such as:

- Proposed
- Accepted
- Implemented
- Superseded
- Rejected

Existence in the repository should not automatically imply that a decision is in force.

---

# Overall Takeaway

The workflow was most useful for:

- Structuring ambiguous requirements
- Refining existing specifications
- Making design boundaries explicit
- Breaking large work into smaller slices
- Generating possible review concerns

It was less reliable for:

- Independently identifying all important design considerations
- Preventing scope drift
- Providing review findings that can be accepted without verification

My current view is that AI-assisted development works best within a disciplined engineering workflow supported by clear documentation, executable tests, automated quality gates, controlled scope, and human judgment.

---

# Next Areas to Explore

- Encoding testing and abstraction principles in repository guidance
- Comparing AI review findings with human reviews
- See how to better manage lifecycle states for generated specifications and ADRs
- Some further guardrails for AI??
  - e.g.: Preventing unrelated changes from entering a feature slice
