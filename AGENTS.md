# AGENTS.md

Repository-wide instructions for agents working in this project.

## Scope
- These instructions apply to the entire repository rooted at this directory.

## Delivery Style
- Be concise, direct, and code-first.
- Do not write long explanations about Java, Spring Boot, or Gradle basics.
- End each substantive response with the next exact step for the developer.

## Technical Constraints
- Language: Java 21.
- Framework: Spring Boot 3.x.
- Build tool: Gradle.
- Tests: JUnit 5 and AssertJ.
- Prefer native Java 21 features such as `record`, pattern matching, and `var` where appropriate.
- Do not use Lombok.

## Development Workflow

Follow the EXACT workflow strictly: Example-guided, AI-Collaborative, Test-driven.
**The loop is: EXAMPLE → RED → GREEN → REFACTOR → (next EXAMPLE)**
**Each phase requires explicit human confirmation before advancing. Never skip a phase. Never combine phases.**

---

### Phase 0 — EXAMPLE (Clarify)

- Do not invent requirements.
- The example must originate from the developer, not from the agent.
- If the request is vague or high-level, ask the developer to provide a concrete example: a specific input and the exact expected output.
- If the developer cannot or does not provide one, the agent may **propose** a candidate example — but must clearly label it as a proposal and require the developer to actively verify every field and value before confirming.
- If the example is ambiguous or incomplete, ask one short clarifying question at a time.
- Do not proceed until the expected behavior is expressed as a single, concrete input/output pair that the developer has explicitly authored or verified.

**Gate:** End with:
> `-> Does this example capture the intended behavior? Reply YES to proceed to RED.`

Do not write any code until the developer replies YES.

---

### Phase 1 — RED (Write a Failing Test)

**Entry condition:** Developer confirmed the example.

- Write only the minimal test that encodes the confirmed example.
- Do not write or touch production code in this phase.
- Do not add helper methods, fixtures, or abstractions beyond what the test requires.
- The test must compile and fail for the right reason (assertion failure, not a compile error).
  - If compilation fails due to missing types or methods, add the bare minimum stub (empty class, method returning `null` or `0`) to make it compile — nothing more.

**Gate:** After writing the test, output:
> `-> I will run: ./gradlew test`
> `-> The test is RED for the expected reason. Reply RED to proceed to GREEN.`

The agent should run the tests directly when possible and summarize the failure briefly.
Do not write any production logic until the developer confirms RED.

---

### Phase 2 — GREEN (Make it Pass)

**Entry condition:** Developer confirmed the test is failing with the expected error.

- Write the smallest possible production change to make the failing test pass.
- Do not refactor, rename, or clean up anything in this phase.
- Do not add behavior not required by the currently failing test.
- It is acceptable to hardcode values if that is the smallest passing change.

**Gate:** After writing the production code, output:
> `-> I will run: ./gradlew test`
> `-> All tests are GREEN. Reply GREEN to proceed to REFACTOR.`

The agent should run the tests directly when possible and summarize the result briefly.
Do not proceed to refactoring until the developer confirms GREEN.

---

### Phase 3 — REFACTOR (Clean Up)

**Entry condition:** Developer confirmed all tests are GREEN.

- Identify one specific, targeted cleanup (rename, extract, simplify).
- Propose it with a one-line rationale. Do not apply it yet.
- Do not change behavior. Tests must still pass after refactoring.
- Do not introduce new abstractions unless they directly reduce duplication already present.

**Gate:** After proposing a refactor, output:
> `-> Reply APPLY to make this change, SKIP to leave it as-is, or DONE to start the next example.`

Do not apply the refactor until the developer replies APPLY.
After applying, output:
> `-> Run: ./gradlew test`
> `-> Confirm GREEN before we continue.`

The agent should run the tests directly when possible and summarize the result briefly.

---

## Hard Rules (Never Violate)

- **Never write production code before a confirmed RED test.**
- **Never refactor before confirmed GREEN.**
- **Never combine phases in a single response** (e.g., test + implementation together).
- **Never proceed past a gate without an explicit human reply.**
- **Never treat an agent-proposed example as confirmed until the developer has explicitly verified every field and value.** A YES reply to a proposal is only valid after the developer has had the opportunity to correct it.
- If unsure which phase you are in, ask: `-> Which phase are we in? RED, GREEN, or REFACTOR?`

## Confirmation Keywords
- `YES` = example confirmed
- `RED` = failing test confirmed
- `GREEN` = passing tests confirmed
- `APPLY` = apply proposed refactor
- `SKIP` = leave refactor as-is
- `DONE` = end the current example without further refactoring

## Commit Policy
- Do not commit automatically.
- Only create a commit when the developer explicitly asks after `DONE`.
- Use the developer-provided commit message, or agree one explicitly before committing.

---

## Testing Guidance
- Prefer simple in-memory stubs or fakes over third-party mocking frameworks when possible.
- Keep tests readable and aligned with the current example.
- One test per example. Do not add multiple assertions for unconfirmed behavior.

## Change Discipline
- Make minimal, focused changes.
- Do not fix unrelated issues.
- Keep code transparent and presentation-friendly for live demos.
- Update documentation only when it is directly relevant to the requested change.

## Response Pattern
- State the change or proposed file edits briefly.
- Keep explanations punchy.
- Always end with the exact gate prompt for the current phase.
