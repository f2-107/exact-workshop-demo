# exact-live-demo Guide

## Purpose
Optimize code for live demonstrations: clear, readable, explainable, with minimal magic and maximum transparency. This guide focuses on presentation quality and developer communication.

Works with: Claude Code, OpenAI Codex, OpenCode/pi.dev, and other AI-assisted development tools.

## When to Use
Apply this guide when:
- Presenting code to an audience (live demo, presentation, walkthrough)
- Code needs to be understood quickly (limited time)
- Explaining implementation choices to stakeholders
- Preparing code for review or handoff
- Recording video tutorials

## Live Demo Principles

### 1. Clarity Over Cleverness
**Rule:** Write code that's easy to understand, not code that's clever.

**Bad (Clever):**
```java
// Confusing one-liner with hidden logic
public String category(double abv) {
    return new String[]{"low", "medium", "high"}[(int)(abv / 7)]
        .equals("high") ? "high" : abv < 7 ? "low" : "medium";
}
```

**Good (Clear):**
```java
// Easy to follow for someone seeing it for the first time
public String category(double abv) {
    if (abv < 7) return "low";
    if (abv < 14) return "medium";
    return "high";
}
```

### 2. Explicit Over Implicit
**Rule:** Make assumptions and logic explicit, not implicit.

**Bad (Implicit):**
```java
// What's the magic number 7? Why?
public String category(double abv) {
    return abv < 7 ? "low" : "high";
}
```

**Good (Explicit):**
```java
// Clear boundary values and logic
private static final double LOW_THRESHOLD = 7.0;
private static final double MEDIUM_THRESHOLD = 14.0;

public String category(double abv) {
    if (abv < LOW_THRESHOLD) return "low";
    if (abv < MEDIUM_THRESHOLD) return "medium";
    return "high";
}
```

### 3. Visible Dependencies
**Rule:** Show what the code depends on; avoid hidden imports or side effects.

**Bad (Hidden):**
```java
// Where does logger come from? Is there side effect?
@GetMapping("/beers")
public List<BeerDto> getBeers() {
    logger.info("Fetching beers");  // Hidden dependency
    return service.getAll();  // Hidden side effect
}
```

**Good (Visible):**
```java
// Clear what's being called and why
@GetMapping("/beers")
public List<BeerDto> getBeers() {
    return beerService.getAllBeers();
}
```

### 4. Readable Names Over Short Names
**Rule:** Use full, descriptive names that explain intent.

**Bad (Cryptic):**
```java
double calcAbvCat(double a) {  // What's "a"? What does it do?
    if (a < 7) return 1;
    return 0;
}
```

**Good (Readable):**
```java
String getAbvCategory(double abv) {
    if (abv < MEDIUM_ABV_THRESHOLD) return "low";
    return "high";
}
```

### 5. Linear Flow Over Nested Logic
**Rule:** Keep code reading top-to-bottom; avoid deep nesting.

**Bad (Nested):**
```java
// Hard to follow nested conditions
public void processBeers(List<Beer> beers) {
    if (beers != null) {
        if (!beers.isEmpty()) {
            for (Beer beer : beers) {
                if (beer.isValid()) {
                    if (beer.abv() > 0) {
                        // Do something...
                    }
                }
            }
        }
    }
}
```

**Good (Linear):**
```java
// Clear, top-to-bottom flow
public void processBeers(List<Beer> beers) {
    if (beers == null || beers.isEmpty()) return;
    
    for (Beer beer : beers) {
        if (!beer.isValid()) continue;
        if (beer.abv() <= 0) continue;
        
        // Do something...
    }
}
```

---

## Live Demo Code Patterns

### Pattern 1: Clear Method Signatures
**In a demo, signatures should tell the story.**

```java
// ✓ Good — signature is self-documenting
public class BeerValidator {
    public void validateBeerName(String name) {
        if (name == null) {
            throw new IllegalArgumentException("Beer name required");
        }
        if (name.isEmpty()) {
            throw new IllegalArgumentException("Beer name cannot be empty");
        }
    }
}
```

### Pattern 2: Obvious Entry Points
**Make it clear where the feature starts.**

```java
// ✓ Good — @RestController is obvious entry point
@RestController
@RequestMapping("/beers")
public class BeerController {
    
    @GetMapping
    public List<BeerDto> getAllBeers() {
        return beerService.getAllBeers();
    }
    
    @PostMapping
    public BeerDto createBeer(@RequestBody BeerDto beerDto) {
        return beerService.createBeer(beerDto);
    }
}
```

### Pattern 3: Named Constants Over Magic Numbers
**Every number should have a name explaining its purpose.**

```java
// ✓ Good — constants explain the "why"
public class BeerAbvCategories {
    private static final double LOW_ABV_MAX = 7.0;      // < 7% = low
    private static final double MEDIUM_ABV_MAX = 14.0;  // 7-14% = medium
    private static final double MIN_ABV = 0.0;
    private static final double MAX_ABV = 20.0;
    
    public String category(double abv) {
        validateAbv(abv);
        if (abv < LOW_ABV_MAX) return "low";
        if (abv < MEDIUM_ABV_MAX) return "medium";
        return "high";
    }
    
    private void validateAbv(double abv) {
        if (abv < MIN_ABV || abv > MAX_ABV) {
            throw new IllegalArgumentException(
                "ABV must be between " + MIN_ABV + "% and " + MAX_ABV + "%"
            );
        }
    }
}
```

### Pattern 4: Minimal Comments (Only "Why")
**Java 21 records and clear names eliminate the need for most comments.**

```java
// ✓ Good — name explains "what", code is clear, comment explains "why"
public record BeerDto(String name, double abv, String style) {
    public BeerDto {
        if (name == null || name.isEmpty()) {
            throw new IllegalArgumentException("Name required");
        }
        // Normalize whitespace to ensure consistent storage
        // (Required for API compatibility with older clients)
        name = name.trim();
    }
}
```

### Pattern 5: Separation of Concerns
**Each class/method has one clear responsibility.**

```java
// ✓ Good — clear separation
public class BeerValidator {
    public void validate(BeerDto beerDto) {
        validateName(beerDto.name());
        validateAbv(beerDto.abv());
    }
    
    private void validateName(String name) { /* ... */ }
    private void validateAbv(double abv) { /* ... */ }
}

public class BeerService {
    public BeerDto createBeer(BeerDto beerDto) {
        validator.validate(beerDto);
        return repository.save(beerDto);
    }
}

@RestController
public class BeerController {
    @PostMapping("/beers")
    public BeerDto createBeer(@RequestBody BeerDto beerDto) {
        return service.createBeer(beerDto);
    }
}
```

---

## Live Demo Communication Rules

### Rule 1: Explain the Path, Not the Details
**What to say:**
```
"We're building a method that categorizes beers by ABV. 
It validates input between 0-20%, then returns low/medium/high."
```

**Don't say:**
```
"We're using a lambda with a stream-based approach that 
filters with a predicate and maps to a categorization function..."
```

### Rule 2: Show Example Input → Output
**Before diving into code, show:**
```
Example:
  Input: 3.5% ABV → Output: "low"
  Input: 5.5% ABV → Output: "medium"
  Input: 8.5% ABV → Output: "high"
```

### Rule 3: Walk Through One Example in Code
**Pick the simplest example and trace through it:**
```
"Let me trace through the first example: ABV = 3.5
→ We call validateAbv(3.5)
→ 3.5 is between 0-20? Yes ✓
→ 3.5 < 7.0? Yes → return 'low' ✓"
```

### Rule 4: Highlight Key Decisions
**Point out the important parts:**
```
"Notice we validate input first—this ensures we never 
process invalid data. If ABV is negative or over 20, 
we throw an exception immediately."
```

### Rule 5: Show Tests, Not Just Code
**In a demo, tests are proof:**
```
"Here are the tests that validate this feature:
- Test 1: ABV=3.5 → 'low' ✓
- Test 2: ABV=5.5 → 'medium' ✓
- Test 3: ABV=8.5 → 'high' ✓
- Test 4: ABV=-1 → throws exception ✓

All tests pass, so the feature works as promised."
```

---

## Demo Code Checklist

Before presenting code, verify:

- [ ] **Names:** All classes, methods, variables have clear, descriptive names?
- [ ] **Magic numbers:** All constants have explanatory names?
- [ ] **Flow:** Code reads top-to-bottom without deep nesting?
- [ ] **Dependencies:** Clear what each method depends on?
- [ ] **Comments:** Only "why" comments present (no "what" comments)?
- [ ] **Errors:** Clear error messages for all validation?
- [ ] **Examples:** Can show concrete input/output?
- [ ] **Tests:** Have tests ready to show as proof?

---

## Demo Presentation Flow

### 1. Show the Problem (30 seconds)
```
"We need to categorize beers by their ABV (alcohol content):
- Low ABV: 0-7%
- Medium ABV: 7-14%
- High ABV: 14%+"
```

### 2. Show Test Examples (1 minute)
```
"Here are the tests showing what we expect:
  3.5% → 'low'
  5.5% → 'medium'
  8.5% → 'high'
  -1% → throw exception
```

### 3. Show Implementation (2-3 minutes)
```
"The code is straightforward:
1. Validate input (must be 0-20%)
2. Compare against thresholds (7% and 14%)
3. Return the category"
```

### 4. Run Tests (30 seconds)
```
./gradlew test --tests BeerAbvCategoryTests
```
Show all tests passing (GREEN).

### 5. Show Live Example (1 minute)
Boot the app and demonstrate:
```
curl http://localhost:8080/beers/3.5/category
→ "low"
```

---

## Anti-Patterns for Live Demos

❌ **Magic numbers** — Unexplained constants  
❌ **Deep nesting** — Hard to follow logic  
❌ **Cryptic names** — Variables like `x`, `tmp`, `a`  
❌ **Hidden dependencies** — Implicit side effects  
❌ **Walls of text comments** — Over-documented code  
❌ **No examples** — Theory without concrete cases  
❌ **Skipping tests** — Showing code without proof  
❌ **Over-explanation** — Explaining obvious code  

---

## Success Metric

Code is ready for live demo when:
1. Names are clear and descriptive
2. Logic flows top-to-bottom (easy to follow)
3. All magic numbers are named constants
4. Comments explain "why", not "what"
5. Tests demonstrate correct behavior
6. Can explain any method in 30 seconds
7. Audience can understand the code on first read
