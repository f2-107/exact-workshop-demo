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
**Goal:** Write minimal failing test code that validates the examples.

**AI Actions:**
1. Create test file (if needed) in `src/test/java/com/example/beerapp/`
2. Write ONLY test code covering the provided examples
3. Use JUnit 5 annotations: `@Test`, `@SpringBootTest`, `@WebMvcTest`, etc.
4. Use AssertJ assertions: `assertThat(...).isEqualTo(...)`, `.contains()`, `.isEmpty()`, etc.
5. Keep tests focused — one behaviour per test, assert everything relevant to it
6. DO NOT write any production code

**Code Patterns:**

**Simple Unit Test (no Spring context):**
```java
@Test
void shouldReturnTrueWhenCondition() {
    // Arrange
    var input = ...;
    
    // Act
    var result = service.method(input);
    
    // Assert
    assertThat(result).isTrue();
}
```

**Integration Test (with @SpringBootTest):**
```java
@SpringBootTest
class BeerControllerTests {
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void shouldReturnBeersWhenGETBeers() {
        var response = restTemplate.getForEntity("/beers", String.class);
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    }
}
```

**Controller Slice Test (with @WebMvcTest):**
```java
@WebMvcTest(BeerController.class)
class BeerControllerTests {
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private BeerService beerService;
    
    @Test
    void shouldReturnBeersWhenGETBeers() throws Exception {
        mockMvc.perform(get("/beers"))
            .andExpect(status().isOk());
    }
}
```

**Next Step:**
```
→ Run: ./gradlew test
Expected: Tests fail (RED state)
```

**When to Proceed:** Developer confirms tests fail and shows RED output.

---

### Phase 3: AI-Collaborative (GREEN)
**Goal:** Write ONLY minimum production code to pass the failing test.

**Rules:**
1. Write production code in `src/main/java/com/example/beerapp/`
2. Write ONLY what's needed to pass the test—no extra features
3. Use Java 21 features: records, pattern matching, `var` where appropriate
4. Avoid Lombok (use native Java 21 records for live demo clarity)
5. No over-engineering, no "just-in-case" code
6. Handle examples provided; ignore hypotheticals

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
→ Run: ./gradlew test
Expected: Tests pass (GREEN state)
```

**When to Proceed:** Developer confirms tests pass and shows GREEN output.

---

### Phase 4: Refactor (IMPROVE)
**Goal:** Suggest code cleanups ONLY after developer approval.

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
- ❌ DO NOT suggest refactoring during GREEN phase
- ❌ DO NOT assume examples; always ask explicitly
- ✅ Always gather examples first
- ✅ Always write tests before code
- ✅ Always let developer verify RED and GREEN

### Communication Style (Live Demo Optimized)
- **No prose walls** — Keep explanations short and punchy
- **Code-first** — Show code immediately, not explanations
- **Next step clear** — Always end with explicit next action
- **Ask before acting** — Never refactor without approval

### Test Patterns (AssertJ)
```java
// Numbers
assertThat(value).isEqualTo(5);
assertThat(value).isPositive();
assertThat(value).isBetween(0, 100);

// Strings
assertThat(name).isEqualTo("Beer");
assertThat(name).contains("IPA");
assertThat(name).startsWith("Craft");

// Collections
assertThat(beers).isEmpty();
assertThat(beers).hasSize(3);
assertThat(beers).contains(expected);

// Exceptions
assertThatThrownBy(() -> service.method())
    .isInstanceOf(IllegalArgumentException.class)
    .hasMessage("Beer name required");

// Booleans
assertThat(isValid).isTrue();
```

### Common Gradle Commands
```bash
# Run all tests
./gradlew test

# Run single test class
./gradlew test --tests BeerServiceTests

# Run single test method
./gradlew test --tests BeerServiceTests.shouldValidateEmptyName

# Run with output (helpful for debugging)
./gradlew test --info

# Build without tests
./gradlew build -x test

# Run the app
./gradlew bootRun
```

---

## Checklist: Did You Follow EXACT?

- [ ] **Phase 1 (GATHERING):** Asked for concrete example, clarified edge cases
- [ ] **Phase 2 (RED):** Wrote minimal test code covering examples; developer ran `./gradlew test` and confirmed failure
- [ ] **Phase 3 (GREEN):** Wrote minimum production code; developer ran `./gradlew test` and confirmed pass
- [ ] **Phase 4 (REFACTOR):** Suggested improvements (if any); got approval before applying
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

**Phase 2 (RED):**
```java
@Test
void shouldReturnFalseForEmptyBeerName() {
    var result = BeerValidator.isValidName("");
    assertThat(result).isFalse();
}

@Test
void shouldThrowExceptionForNullBeerName() {
    assertThatThrownBy(() -> BeerValidator.isValidName(null))
        .isInstanceOf(IllegalArgumentException.class);
}

@Test
void shouldReturnTrueForValidName() {
    var result = BeerValidator.isValidName("IPA");
    assertThat(result).isTrue();
}
```
→ Run `./gradlew test` to see RED failures.

**Phase 3 (GREEN):**
```java
public class BeerValidator {
    public static boolean isValidName(String name) {
        if (name == null) throw new IllegalArgumentException("Name required");
        return !name.isEmpty();
    }
}
```
→ Run `./gradlew test` to confirm GREEN (all pass).

**Phase 4 (REFACTOR):**
```
Suggestion: Extract to a more expressive method name? 
Or add a message to the exception?
Approve refactoring?
```

---

## Anti-Patterns (Never Do These)

❌ **Vibe Coding** — Writing code without test validation  
❌ **Big Bang Testing** — Writing all tests at once before any code  
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
