# exact-verifier Guide

## Purpose
Verify that code changes actually match the concrete examples provided in the gathering phase. Catch common pitfalls early (security, validation, edge cases).

Works with: Claude Code, OpenAI Codex, OpenCode/pi.dev, and other AI-assisted development tools.

## When to Use
Apply this guide when:
- Implementation is complete (GREEN phase)
- Tests pass and you want to verify quality
- Before refactoring
- Before marking a feature complete
- When code looks correct but might have subtle bugs

## What the Verifier Checks

### 1. Examples Verification
**Does the code actually match the concrete examples?**

- [ ] Example 1: Input → Correct output? (Run code mentally or trace)
- [ ] Example 2: Different input → Correct output?
- [ ] Example 3: Edge case → Correct output?
- [ ] Boundary cases handled exactly as specified?

**Questions to Ask:**
```
Example said: ABV=4.0 → "medium"
Does code do: if (abv >= 4.0 && abv < 7.0) return "medium"? ✓
```

### 2. Error Handling Verification
**Are all error cases handled correctly?**

- [ ] Null inputs: Are they caught? Do they throw the right exception?
- [ ] Empty collections: Are they handled? (return empty list or throw?)
- [ ] Out of range: Does code validate boundaries?
- [ ] Invalid inputs: Does code reject with clear messages?
- [ ] Exception types: Are they the ones specified in examples?

**Common Pitfalls:**
```
❌ Throwing NullPointerException instead of IllegalArgumentException
❌ Accepting null when it should be rejected
❌ Not validating input ranges
❌ Silently returning null instead of throwing
```

### 3. Security Verification
**Are there any security issues?**

- [ ] No hardcoded secrets (API keys, passwords, tokens)?
- [ ] No SQL injection vulnerabilities?
- [ ] Input properly validated before use?
- [ ] No sensitive data logged?
- [ ] Authentication/authorization in place (if needed)?

**Common Pitfalls:**
```
❌ Building SQL with string concatenation
❌ Storing plaintext passwords
❌ Logging user input without sanitization
❌ Missing validation on REST endpoint inputs
```

### 4. Validation Verification
**Is all input validation in place?**

- [ ] Null checks where specified?
- [ ] Length/size checks (min/max)?
- [ ] Range checks (ABV between 0-20)?
- [ ] Format checks (email, phone, etc.)?
- [ ] Whitespace handling (trim, empty string)?

**Example:**
```java
// ❌ Bad — accepts invalid input
public String category(double abv) {
    if (abv < 7) return "low";
    return "high";
}

// ✓ Good — validates input
public String category(double abv) {
    if (abv < 0 || abv > 20) {
        throw new IllegalArgumentException("ABV must be 0-20");
    }
    if (abv < 7) return "low";
    return "high";
}
```

### 5. Consistency Verification
**Is the code consistent with other similar code?**

- [ ] Naming conventions followed? (camelCase, UPPER_CASE)
- [ ] Similar patterns used (same as other validators, controllers, etc.)?
- [ ] Exception handling style consistent?
- [ ] Comments/documentation style consistent?

### 6. Test Coverage Verification
**Do all examples have passing tests?**

- [ ] Test for example 1? ✓
- [ ] Test for example 2? ✓
- [ ] Test for edge case? ✓
- [ ] Test for error case? ✓
- [ ] All tests pass (GREEN)? ✓

---

## Verification Checklist

Use this before declaring a feature complete:

### Phase Compliance
- [ ] Was example gathering done (GATHERING phase)?
- [ ] Were tests written first (RED phase)?
- [ ] Is code minimal (GREEN phase — only what passes tests)?
- [ ] Are refactoring suggestions awaiting approval (REFACTOR phase)?

### Examples & Behavior
- [ ] All concrete examples produce correct output?
- [ ] All edge cases handled as specified?
- [ ] All error cases throw correct exceptions?
- [ ] No additional features added (EXACT rule)?

### Quality & Safety
- [ ] No hardcoded secrets or sensitive data?
- [ ] Input validation in place?
- [ ] Exception messages clear and helpful?
- [ ] No null pointer exceptions on valid edge cases?
- [ ] No SQL injection or XSS vulnerabilities?

### Testing & Coverage
- [ ] All tests pass (GREEN state)?
- [ ] One test per example?
- [ ] Test names clearly describe what they test?
- [ ] Tests cover happy path AND error paths?

### Code Quality
- [ ] Naming is clear and follows conventions?
- [ ] Code is minimal, no over-engineering?
- [ ] No duplicate code (copy-paste)?
- [ ] Consistent with codebase style?
- [ ] Comments only where WHY is non-obvious?

---

## Common Pitfalls to Catch

### Pitfall 1: Accepting Invalid Input
**Problem:** Code doesn't validate what the examples said should be rejected.

**Example:**
```java
// Gathering said: null → throw IllegalArgumentException
// But code does:
public String getCategory(Double abv) {
    if (abv == null) return "unknown";  // ❌ Wrong!
}

// Should be:
public String getCategory(Double abv) {
    if (abv == null) throw new IllegalArgumentException("ABV required");  // ✓
}
```

### Pitfall 2: Wrong Exception Type
**Problem:** Throwing general Exception instead of specified type.

**Example:**
```java
// Gathering said: IllegalArgumentException
// But code throws:
try {
    int val = Integer.parseInt(input);
} catch (NumberFormatException e) {
    throw e;  // ❌ Wrong exception type!
}

// Should be:
try {
    int val = Integer.parseInt(input);
} catch (NumberFormatException e) {
    throw new IllegalArgumentException("Invalid ABV: " + input);  // ✓
}
```

### Pitfall 3: Missing Boundary Checks
**Problem:** Doesn't validate boundaries specified in examples.

**Example:**
```java
// Gathering said: ABV must be 0-20
// But code does:
public String category(double abv) {
    if (abv < 7) return "low";  // ❌ No validation!
    return "high";
}

// Should be:
public String category(double abv) {
    if (abv < 0 || abv > 20) {
        throw new IllegalArgumentException("ABV must be 0-20");
    }
    if (abv < 7) return "low";
    return "high";
}
```

### Pitfall 4: Silent Failures Instead of Exceptions
**Problem:** Code returns null/default instead of throwing when it should.

**Example:**
```java
// Gathering said: throw exception for null name
// But code does:
public String getName(Beer beer) {
    if (beer.name() == null) return "";  // ❌ Silent failure!
}

// Should be:
public String getName(Beer beer) {
    if (beer.name() == null) {
        throw new IllegalArgumentException("Beer name required");
    }
    return beer.name();
}
```

### Pitfall 5: Hardcoded Secrets or Sensitive Data
**Problem:** API keys, passwords, or sensitive data in code.

**Example:**
```java
// ❌ Never hardcode secrets
private static final String API_KEY = "sk-1234567890";

// ✓ Use environment variables or config
private String apiKey = System.getenv("API_KEY");
```

### Pitfall 6: Over-Engineering (Adding Unlisted Features)
**Problem:** Code does more than the examples specified.

**Example:**
```java
// Gathering said: simple validator for beer name
// But code does:
public class BeerValidator {
    public boolean isValid(String name) {
        // ❌ Added extra features not in examples:
        // - Check if name exists in database
        // - Log all validation attempts
        // - Email alerts on invalid input
        // - Caching mechanism
    }
}

// Should be minimal:
public class BeerValidator {
    public boolean isValid(String name) {  // ✓ Only what examples need
        if (name == null) throw new IllegalArgumentException("Name required");
        return !name.isEmpty();
    }
}
```

---

## Verification Process

### Step 1: Read the Specification
Go back to the examples and clarifications from the gathering phase.

### Step 2: Trace Through Examples
Mentally execute the code with each example input:
```
Example 1: Input = 3.5
  → category(3.5)
  → abv is between 0-20 ✓
  → abv < 7 → return "low" ✓
  Result: "low" ✓ MATCHES EXPECTED

Example 2: Input = 5.5
  → category(5.5)
  → abv is between 0-20 ✓
  → abv >= 7? No → return "medium" ✓
  Result: "medium" ✓ MATCHES EXPECTED
```

### Step 3: Check Error Cases
```
Example (Error): Input = -1
  → category(-1)
  → abv < 0 → throw IllegalArgumentException ✓
  Result: Exception thrown ✓ MATCHES EXPECTED
```

### Step 4: Run Tests
```bash
./gradlew test
```

All tests should PASS (GREEN). If any fail, code doesn't match examples.

### Step 5: Check for Pitfalls
Look for common security/validation issues using the checklist.

### Step 6: Verify No Over-Engineering
Does the code do ONLY what the examples specified? Nothing more?

---

## Verification Report Template

After verification, provide a report:

```
## Verification Report

**Feature:** Calculate beer ABV category
**Status:** ✓ VERIFIED

### Examples
- [x] Input 3.5 → "low" ✓
- [x] Input 5.5 → "medium" ✓
- [x] Input 8.5 → "high" ✓
- [x] Input -1 → IllegalArgumentException ✓

### Test Coverage
- [x] All 4 tests passing (GREEN)
- [x] One test per example
- [x] Error cases covered

### Quality Checks
- [x] No hardcoded values
- [x] Input validated (ABV range 0-20)
- [x] Clear exception messages
- [x] Minimal code (no extra features)
- [x] Follows codebase style

### Issues Found
❌ None — code is ready!

### Refactor Suggestions (optional)
[Only if you have suggestions for REFACTOR phase]
```

---

## Success Metric

Verification passes when:
1. All concrete examples produce correct output
2. All error cases throw correct exceptions
3. All tests pass (GREEN)
4. No security/validation pitfalls found
5. Code is minimal (nothing added beyond examples)
6. Ready to proceed with optional REFACTOR phase
