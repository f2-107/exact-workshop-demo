# exact-test-writer Guide

## Purpose
Generate minimal, correct test code from concrete examples. This guide focuses on Phase 2 (RED) — writing failing tests that drive implementation.

Works with: Claude Code, OpenAI Codex, OpenCode/pi.dev, and other AI-assisted development tools.

## When to Use
Apply this guide when:
- You have gathered concrete examples (from exact-example-gathering)
- You need to write test code
- Examples are clear and ready to be tested
- Ready to move from GATHERING → RED phase

## Input Requirements

Before calling this skill, you MUST have:
1. ✓ Concrete examples (input → output)
2. ✓ Edge cases clarified (null, empty, boundaries)
3. ✓ Exception behavior defined
4. ✓ Return types and method signatures known

## Test Writing Philosophy

### Core Rules
1. **Test = Executable Example** — Each test is one example from gathering
2. **Focused Assertions** — Each test covers one behaviour; assert everything relevant to that behaviour
3. **Minimal Setup** — Only arrange what's needed for the example
4. **Clear Names** — Test name describes the example being tested
5. **No Production Code** — Tests only, no implementation

### Test Naming Convention
```
shouldReturn[ExpectedValue]When[Condition]
shouldThrow[ExceptionType]When[Condition]
should[Behavior]For[Scenario]
```

**Examples:**
```java
shouldReturnLowWhenAbvIsThreePointFive()
shouldThrowIllegalArgumentExceptionWhenAbvIsNegative()
shouldReturnEmptyListWhenNoBeerInStock()
```

---

## Test Code Patterns by Type

### Pattern 1: Simple Unit Test (No Spring)
**When to use:** Testing a utility, service method, or pure function

```java
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.*;

class BeerValidatorTests {
    
    @Test
    void shouldReturnTrueWhenNameIsValid() {
        // Arrange
        var name = "IPA";
        
        // Act
        var result = BeerValidator.isValid(name);
        
        // Assert
        assertThat(result).isTrue();
    }
    
    @Test
    void shouldReturnFalseWhenNameIsEmpty() {
        // Arrange
        var name = "";
        
        // Act
        var result = BeerValidator.isValid(name);
        
        // Assert
        assertThat(result).isFalse();
    }
    
    @Test
    void shouldThrowExceptionWhenNameIsNull() {
        assertThatThrownBy(() -> BeerValidator.isValid(null))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessage("Name required");
    }
}
```

### Pattern 2: Spring Service Test (@SpringBootTest)
**When to use:** Testing services that use @Autowired dependencies

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import static org.assertj.core.api.Assertions.*;

@SpringBootTest
class BeerServiceTests {
    
    @Autowired
    private BeerService beerService;
    
    @Test
    void shouldCreateBeerWhenInputIsValid() {
        // Arrange
        var beerDto = new BeerDto("IPA", 6.5, "American");
        
        // Act
        var result = beerService.createBeer(beerDto);
        
        // Assert
        assertThat(result.name()).isEqualTo("IPA");
        assertThat(result.abv()).isEqualTo(6.5);
    }
    
    @Test
    void shouldThrowExceptionWhenNameIsNull() {
        var invalidBeer = new BeerDto(null, 5.0, "Lager");
        
        assertThatThrownBy(() -> beerService.createBeer(invalidBeer))
            .isInstanceOf(IllegalArgumentException.class);
    }
}
```

### Pattern 3: Spring Controller Test (@WebMvcTest)
**When to use:** Testing REST endpoints

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.web.servlet.MockMvc;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(BeerController.class)
class BeerControllerTests {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void shouldReturnOkWhenGettingBeers() throws Exception {
        mockMvc.perform(get("/beers"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$").isArray());
    }
    
    @Test
    void shouldReturnIPAsWhenFilteringByType() throws Exception {
        mockMvc.perform(get("/beers?type=IPA"))
            .andExpect(status().isOk());
    }
}
```

### Pattern 4: Testing Collections
**When to use:** Testing methods that return lists

```java
@Test
void shouldReturnIPAsWhenFilteringByType() {
    // Arrange
    var beers = List.of(
        new Beer("IPA", 6.5),
        new Beer("Pilsner", 5.0),
        new Beer("IPA", 7.0)
    );
    var filter = new BeerFilter("IPA");
    
    // Act
    var result = filter.apply(beers);
    
    // Assert
    assertThat(result)
        .hasSize(2)
        .allMatch(b -> b.type().equals("IPA"));
}

@Test
void shouldReturnEmptyListWhenNoBeersMatch() {
    var beers = List.of(new Beer("Pilsner", 5.0));
    var filter = new BeerFilter("IPA");
    
    var result = filter.apply(beers);
    
    assertThat(result).isEmpty();
}
```

### Pattern 5: Testing Validation
**When to use:** Testing validation logic

```java
@Test
void shouldAcceptValidAbvRange() {
    var validator = new BeerValidator();
    
    // Boundary test: exactly 0%
    assertThat(validator.isValidAbv(0.0)).isTrue();
    
    // Boundary test: exactly 20%
    assertThat(validator.isValidAbv(20.0)).isTrue();
    
    // Normal case: in range
    assertThat(validator.isValidAbv(5.5)).isTrue();
}

@Test
void shouldRejectInvalidAbvRange() {
    var validator = new BeerValidator();
    
    assertThatThrownBy(() -> validator.validateAbv(-0.1))
        .isInstanceOf(IllegalArgumentException.class)
        .hasMessage("ABV must be between 0 and 20");
}
```

---

## AssertJ Assertion Reference

### For Primitives & Objects
```java
assertThat(value).isEqualTo(expected);
assertThat(value).isNotEqualTo(other);
assertThat(value).isNull();
assertThat(value).isNotNull();
assertThat(value).isSameAs(other);
assertThat(value).isInstanceOf(String.class);
```

### For Numbers
```java
assertThat(number).isPositive();
assertThat(number).isNegative();
assertThat(number).isZero();
assertThat(number).isBetween(0, 100);
assertThat(number).isGreaterThan(5);
assertThat(number).isLessThanOrEqualTo(10);
```

### For Strings
```java
assertThat(name).isEqualTo("IPA");
assertThat(name).contains("IPA");
assertThat(name).startsWith("Craft");
assertThat(name).endsWith("Ale");
assertThat(name).isEmpty();
assertThat(name).isNotBlank();
```

### For Collections
```java
assertThat(beers).isEmpty();
assertThat(beers).isNotEmpty();
assertThat(beers).hasSize(3);
assertThat(beers).contains(expected1, expected2);
assertThat(beers).containsExactly(item1, item2);
assertThat(beers).containsExactlyInAnyOrder(item1, item2);
assertThat(beers).allMatch(b -> b.abv() > 0);
assertThat(beers).anyMatch(b -> b.type().equals("IPA"));
```

### For Exceptions
```java
assertThatThrownBy(() -> service.method())
    .isInstanceOf(IllegalArgumentException.class)
    .hasMessage("Name required");

assertThatThrownBy(() -> service.method())
    .hasMessageContaining("required");
```

### For Optional
```java
assertThat(optional).isPresent();
assertThat(optional).isEmpty();
assertThat(optional).contains(expected);
```

---

## Test Code Generation Process

### Step 1: Identify Test Type
Determine what kind of test to write based on what's being tested:
- Unit test (simple methods)?
- Service test (with @Autowired)?
- Controller test (REST endpoints)?
- Integration test (full flow)?

### Step 2: Name the Test
Use the naming convention:
```
shouldReturn[ExpectedValue]When[Condition]
shouldThrow[ExceptionType]When[Condition]
```

### Step 3: Write AAA Structure
Always use Arrange-Act-Assert:
1. **Arrange** — Set up test data
2. **Act** — Call the method
3. **Assert** — Verify with AssertJ

### Step 4: Focused Assertions
Each test covers one behaviour. Assert everything relevant to that behaviour — grouping related assertions (e.g. multiple boundary values for the same rule, or multiple fields of a returned object) in one test is fine:
```java
// ✅ Good — one behaviour, multiple assertions that verify it together
@Test
void shouldReturnLowForAllAbvsUnderSevenPercent() {
    assertThat(calculator.category(0.0)).isEqualTo("low");
    assertThat(calculator.category(3.5)).isEqualTo("low");
    assertThat(calculator.category(6.9)).isEqualTo("low");
}

// ✅ Also good — single input, single assertion
@Test
void shouldReturnLowWhenAbvIsThreePointFive() {
    assertThat(calculator.category(3.5)).isEqualTo("low");
}

// ❌ Bad — unrelated behaviours crammed into one test
@Test
void shouldCalculateCategory() {
    assertThat(calculator.category(3.5)).isEqualTo("low");
    assertThat(calculator.category(10.0)).isEqualTo("medium");
    assertThat(calculator.category(15.0)).isEqualTo("high");
    assertThatThrownBy(() -> calculator.category(-1)).isInstanceOf(IllegalArgumentException.class);
}
```

### Step 5: Minimal Setup
Only arrange what's needed:
```java
// ✅ Good — minimal, specific setup
@Test
void shouldReturnValidWhenNameIsNotEmpty() {
    var result = validator.isValid("IPA");
    assertThat(result).isTrue();
}

// ❌ Bad — unnecessary setup
@Test
void shouldValidate() {
    var beer = new Beer("IPA", 6.5, "American IPA", true, false, null);
    var validator = new BeerValidator(new Config(), new Database(), ...);
    var result = validator.validate(beer);
    assertThat(result).isTrue();
}
```

---

## Test File Locations

Place test files in `src/test/java/com/example/beerapp/` following the source structure:

```
src/main/java/com/example/beerapp/
├── BeerService.java
├── BeerValidator.java
└── BeerController.java

src/test/java/com/example/beerapp/
├── BeerServiceTests.java
├── BeerValidatorTests.java
└── BeerControllerTests.java
```

**Test class naming:** `{SourceClass}Tests.java`

---

## Common Mistakes to Avoid

❌ **Unrelated behaviours in one test** — Keep each test focused on a single behaviour  
❌ **Complex setup** — Keep Arrange section minimal  
❌ **Testing implementation** — Test behavior, not how it's done  
❌ **Untestable code** — Avoid new in production code; use dependency injection  
❌ **Missing edge cases** — Write separate tests for each example  
❌ **Silent test failures** — Use specific assertions that fail with clear messages  
❌ **Tests that depend on each other** — Each test should be independent  

---

## Output: Test Code Summary

After writing tests, provide:

```
## Test Code Written

**Test Class:** BeerValidatorTests.java

**Tests Created:**
1. ✓ shouldReturnTrueWhenNameIsValid
2. ✓ shouldReturnFalseWhenNameIsEmpty
3. ✓ shouldThrowExceptionWhenNameIsNull

**Next Step:** Run `./gradlew test` to confirm RED state (tests should fail).
```

---

## Success Metric

Tests are ready when:
1. Each example has its own test method
2. Test names clearly describe what they test
3. All assertions use AssertJ
4. No production code written (only tests)
5. Ready to run `./gradlew test` and see failures (RED)
