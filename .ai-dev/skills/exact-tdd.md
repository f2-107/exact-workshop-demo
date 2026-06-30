# exact-tdd Guide

## Purpose
Enforce strict EXACT (Example-guided, AI-Collaborative, Test-driven) development workflow. This guide ensures Red-Green-Refactor discipline with human validation at each phase.

Works with: Claude Code, OpenAI Codex, OpenCode/pi.dev, and other AI-assisted development tools.

## When to Use
Apply this workflow when:
- Implementing a new feature or behavior
- Fixing a bug with a concrete example
- Refactoring with test coverage
- Team explicitly requests TDD workflow

## The Four Phases (Must Follow Strictly)

### Phase 1: Example-Guided (GATHERING)
**Goal:** Understand the requirement with concrete input/output examples before any code.

**AI Actions:**
1. Ask developer for a **concrete domain example**: input → expected output
2. Ask about **edge cases** (empty lists, null values, negative numbers, etc.)
3. Clarify **behavior** (What happens if condition X? What should be thrown?)
4. DO NOT write any code yet

**Example Dialog:**
```
Q: "What's a concrete example? Give me: input value(s) → expected output"
Q: "What happens if the list is empty?"
Q: "Should this throw an exception or return null?"
```

**When to Proceed:** Developer has answered all clarifying questions with concrete examples.

---

### Phase 2: Test-Driven (RED)
**Goal:** Write ONE failing test for the current example only. Do not write tests for other examples yet.

**AI Actions:**
1. If the class under test doesn't exist yet, create a **minimal skeleton** — empty class with a method stub that throws to make the code compile (see skeleton patterns below)
2. Create test file (if needed) in `src/test/java/com/example/beerapp/`
3. Write ONLY ONE test covering the current example — not all gathered examples at once
4. Use JUnit 5 annotations: `@Test`, `@SpringBootTest`, `@WebMvcTest`, etc.
5. Use AssertJ assertions: `assertThat(...).isEqualTo(...)`, `.contains()`, `.isEmpty()`, etc.
6. Keep tests focused — one behaviour per test, assert everything relevant to it
7. DO NOT write any real production logic

**Skeleton Patterns (compile-time stubs only):**

Java:
```java
public class BeerValidator {
    public static boolean isValid(String name) {
        throw new UnsupportedOperationException("not implemented");
    }
}
```

Kotlin:
```kotlin
class BeerService {
    fun isValidName(name: String?): Boolean {
        throw NotImplementedError("not implemented")
    }
}
```

TypeScript:
```typescript
export class BeerValidator {
    static isValid(name: string | null): boolean {
        throw new Error('not implemented');
    }
}
```

**Code Patterns:**

See your stack file for language-specific test patterns:
- Java → `exact-stack-java` — JUnit 5 + AssertJ patterns
- Kotlin → `exact-stack-kotlin` — JUnit 5 + AssertJ patterns
- Angular/TypeScript → `exact-stack-angular` — Jest patterns

**Next Step:**
```
→ Run the test command for your stack (see stack file)
Expected: Test fails (RED state) — not a compile error, a test failure
```

**When to Proceed:** Developer confirms test fails and shows RED output.

---

### Phase 3: AI-Collaborative (GREEN)
**Goal:** Write ONLY minimum production code to pass the failing test.

**Rules:**
1. Write ONLY what's needed to make the one failing test pass — no extra features
2. No over-engineering, no "just-in-case" code
3. Handle the current example only; ignore hypotheticals
4. Follow the language-specific patterns from the active stack file (`exact-stack-java`, `exact-stack-kotlin`, or `exact-stack-angular`)

**Code Patterns:**

**Simple Service Method:**
```java
public class BeerService {
    public boolean isValid(String beer) {
        return beer != null && !beer.isEmpty();
    }
}
```

**REST Controller Endpoint:**
```java
@RestController
@RequestMapping("/beers")
public class BeerController {
    private final BeerService service;
    
    public BeerController(BeerService service) {
        this.service = service;
    }
    
    @GetMapping
    public List<BeerDto> getBeers() {
        return service.getAllBeers();
    }
}
```

**Record (Java 21):**
```java
public record BeerDto(String name, double abv, String style) {}
```

**Next Step:**
```
→ Run the test command for your stack (see stack file)
Expected: Tests pass (GREEN state)
```

**When to Proceed:** Developer confirms tests pass and shows GREEN output. Then loop back to Phase 2 for the next example.

---

### Phase 4: Refactor (IMPROVE)
**Goal:** Suggest code cleanups ONLY after developer approval. Then pick the next example and repeat from Phase 2.

**Rules:**
1. DO NOT modify code without explicit developer approval
2. Suggest precise, targeted improvements only
3. Focus on: readability, validation, error handling, naming, duplication
4. Ensure all tests still pass after refactoring
5. Provide exact code changes needed

**Refactor Ideas (suggest, not apply):**
- Extract validation into separate method
- Use meaningful variable names
- Remove dead code
- Simplify conditional logic
- Extract common patterns

**Example Suggestion:**
```
Current code has validation mixed in the controller. Suggest extracting to a validator 
method in the service layer for better separation of concerns. Approval needed?
```

**When to Proceed:** Developer approves refactoring suggestion and you apply changes.

---

## Workflow Rules

### Never Skip Phases
- ❌ DO NOT write production code before tests exist
- ❌ DO NOT write multiple tests before cycling RED→GREEN
- ❌ DO NOT suggest refactoring during GREEN phase
- ❌ DO NOT assume examples; always ask explicitly
- ✅ Always gather examples first, implement one at a time
- ✅ Always write ONE test before any code
- ✅ Always let developer verify RED and GREEN before moving on

### Communication Style (Live Demo Optimized)
- **No prose walls** — Keep explanations short and punchy
- **Code-first** — Show code immediately, not explanations
- **Next step clear** — Always end with explicit next action
- **Ask before acting** — Never refactor without approval

### Stack Reference

For language-specific test patterns, assertion libraries, and CLI commands, see your stack file:

| Stack | File |
|-------|------|
| Java 21 / Spring Boot | `exact-stack-java` |
| Kotlin / Spring Boot | `exact-stack-kotlin` |
| Angular / TypeScript | `exact-stack-angular` |

---

## Checklist: Did You Follow EXACT?

- [ ] **Phase 1 (GATHERING):** Asked for concrete example, clarified edge cases
- [ ] **Phase 2 (RED):** Created skeleton if class was new; wrote ONE test; developer ran test and confirmed failure
- [ ] **Phase 3 (GREEN):** Wrote minimum production code; developer ran test and confirmed pass
- [ ] **Phase 4 (REFACTOR):** Suggested improvements (if any); got approval before applying; looped back for next example
- [ ] **Communication:** Short explanations, code-first, clear next steps

---

## Example Full Cycle

**Developer:** "Add a method that validates beer names—name must not be empty or null."

**Phase 1 (GATHERING):**
```
Q: Give me an example:
   Input: null → Expected: false (or exception?)
   Input: "" → Expected: false?
   Input: "IPA" → Expected: true?

A: null throws IllegalArgumentException, empty string returns false, "IPA" returns true.
```

**Cycle 1 — example: `""` → `false`**

Skeleton (class doesn't exist yet):
```java
public class BeerValidator {
    public static boolean isValidName(String name) {
        throw new UnsupportedOperationException("not implemented");
    }
}
```

Phase 2 (RED) — one test:
```java
@Test
void shouldReturnFalseForEmptyBeerName() {
    var result = BeerValidator.isValidName("");
    assertThat(result).isFalse();
}
```
→ Run `./gradlew test` → RED (UnsupportedOperationException). Developer confirms.

Phase 3 (GREEN) — minimum code to pass this test only:
```java
public static boolean isValidName(String name) {
    return !name.isEmpty();
}
```
→ Run `./gradlew test` → GREEN. Developer confirms.

Phase 4 (REFACTOR) — nothing to clean up yet, move on.

---

**Cycle 2 — example: `null` → throws `IllegalArgumentException`**

Phase 2 (RED) — one new test:
```java
@Test
void shouldThrowExceptionForNullBeerName() {
    assertThatThrownBy(() -> BeerValidator.isValidName(null))
        .isInstanceOf(IllegalArgumentException.class);
}
```
→ Run `./gradlew test` → RED (NullPointerException, not IAE). Developer confirms.

Phase 3 (GREEN) — extend code to pass:
```java
public static boolean isValidName(String name) {
    if (name == null) throw new IllegalArgumentException("Name required");
    return !name.isEmpty();
}
```
→ Run `./gradlew test` → GREEN. Developer confirms.

Phase 4 (REFACTOR) — nothing to clean up, move on.

---

**Cycle 3 — example: `"IPA"` → `true`**

Phase 2 (RED) — one new test:
```java
@Test
void shouldReturnTrueForValidName() {
    var result = BeerValidator.isValidName("IPA");
    assertThat(result).isTrue();
}
```
→ Run `./gradlew test` → GREEN immediately (already handled). ✓ No new production code needed.

Phase 4 (REFACTOR):
```
Suggestion: add a message to the IllegalArgumentException?
Approve refactoring?
```

---

## Anti-Patterns (Never Do These)

❌ **Vibe Coding** — Writing code without test validation  
❌ **Big Bang Testing** — Writing all tests at once before any code  
❌ **Batch Testing** — Writing multiple tests before going GREEN on the first  
❌ **Skipping Examples** — Assuming requirements instead of asking  
❌ **Over-Engineering** — Adding features not covered by examples  
❌ **Silent Refactoring** — Changing code without approval  
❌ **Mixing Phases** — Writing code during Phase 2, refactoring during Phase 3  
❌ **Ignoring Test Failure** — Not confirming RED before proceeding to GREEN  

---

## Success Metrics

You've successfully used EXACT TDD when:
1. Every feature has a test written first
2. Developer confirmed RED state before any production code
3. Developer confirmed GREEN state after implementation
4. Code is minimal, no extra "just-in-case" features
5. All tests pass consistently
6. Communication was concise and action-oriented
