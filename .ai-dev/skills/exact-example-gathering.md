# exact-example-gathering Guide

## Purpose
Help extract concrete, unambiguous examples before any code is written. This guide helps gather the clarity needed for EXACT TDD Phase 1 (GATHERING).

Works with: Claude Code, OpenAI Codex, OpenCode/pi.dev, and other AI-assisted development tools.

## When to Use
Apply this guide when:
- Starting a new feature
- Requirements are vague or incomplete
- Developer says "add a feature but" without concrete cases
- You need to clarify edge cases before proceeding

## The Gathering Process

### Step 1: Understand the Domain Concept
**Ask:** "What are we building? (1-2 sentences)"

**Example Answers:**
- "A method that rates beers on quality"
- "An endpoint that returns all IPAs in stock"
- "A validator for beer names"

### Step 2: Get Concrete Input-Output Examples
**Ask:** "Give me 2-3 concrete examples: input → expected output"

**EXACT Format:**
```
Example 1:
  Input:  [what goes in?]
  Output: [what should come back?]

Example 2:
  Input:  [different case]
  Output: [expected result]

Example 3:
  Input:  [edge case]
  Output: [expected result]
```

**Good Examples (Concrete):**
```
Example 1:
  Input:  Beer("IPA", 6.5, "American")
  Output: Rating = 8/10

Example 2:
  Input:  Beer("Pilsner", 5.0, "Czech")
  Output: Rating = 7/10

Example 3:
  Input:  Beer(null, 5.0, "Czech")
  Output: IllegalArgumentException("Name required")
```

**Bad Examples (Too Vague):**
```
❌ Input: a beer → Output: its rating
❌ Input: something valid → Output: something good
❌ Input: a request → Output: data (unclear what data)
```

### Step 3: Clarify Error Cases & Edge Cases
**Ask:** "What happens in these edge cases?"

**Common Edge Cases to Ask About:**
- What if input is `null`?
- What if input is empty (`""`, `[]`, empty list)?
- What if input is negative or out of range?
- What if a required field is missing?
- What if there's a duplicate?
- What happens with very large inputs?
- What's the behavior for boundary values?

**Example Questions:**
```
Q: "If beer name is null, should we throw an exception or return a default rating?"
Q: "If the list of beers is empty, should we return an empty list or throw?"
Q: "What's the maximum ABV value allowed?"
```

### Step 4: Clarify Success vs Failure Paths
**Ask:** "What should happen if...?"

- **Happy Path:** Normal expected behavior (covered in examples)
- **Error Paths:** Invalid inputs, exceptions, null handling
- **Edge Cases:** Boundary conditions, empty collections, special values

**Example:**
```
Happy Path:
  Input: Beer with valid name → Output: Created successfully

Error Path 1:
  Input: Beer with null name → Output: Throw IllegalArgumentException

Error Path 2:
  Input: Beer with empty name → Output: Return false / reject
```

### Step 5: Clarify Return Types & Exceptions
**Ask:** "What's the return type and exceptions?"

- Does the method return a primitive, object, collection, or void?
- Should it throw checked or unchecked exceptions?
- What exceptions specifically?
- What are the exception messages?

**Example Answers:**
```
Method: boolean isValidBeer(Beer beer)
Throws: IllegalArgumentException if beer.name is null with message "Beer name required"

Method: List<Beer> getBeersInStock()
Returns: List<Beer> (never null, empty list if no stock)
Throws: None

Method: void addBeer(Beer beer)
Throws: IllegalArgumentException if beer.name is empty
```

### Step 6: Clarify Constraints & Requirements
**Ask:** "Any constraints I should know about?"

- Business logic constraints (e.g., "ABV must be between 0-15%")
- Performance requirements?
- Immutability requirements?
- Thread-safety needed?
- Data format requirements (e.g., "name must be alphanumeric")?

**Example Answers:**
```
- Beer names must be non-empty and start with a letter
- ABV must be between 0% and 15%
- The list should be sorted by name
- The operation must be idempotent
```

---

## Question Checklist

Use this to ensure you've gathered enough clarity:

- [ ] **Domain:** What are we building? (1-2 sentences)
- [ ] **Examples:** 2-3 concrete input → output cases?
- [ ] **Null/Empty:** What happens with null or empty inputs?
- [ ] **Negative:** What if input is out of range or invalid?
- [ ] **Errors:** What exceptions are thrown and with what messages?
- [ ] **Return Types:** What does the method return?
- [ ] **Collections:** If returning a list, what about empty cases?
- [ ] **Constraints:** Any business logic constraints?

---

## What Makes a Good Example

### ✅ Good Examples Are:
- **Concrete** — "name = 'IPA'" not "a beer name"
- **Specific** — "returns 8" not "returns a number"
- **Testable** — Can be directly translated to assertions
- **Minimal** — Only what's needed to understand behavior
- **Complete** — Input and output both stated clearly

### ❌ Bad Examples Are:
- **Vague** — "handles beer data"
- **Abstract** — "returns something valid"
- **Hypothetical** — "what if we need X in the future?"
- **Incomplete** — Only input stated, not expected output
- **Ambiguous** — "It should work like beer validation"

---

## Common Scenarios & Questions

### Scenario 1: Adding a Validation Method
```
Developer: "Add validation for beer names."

Questions:
1. What makes a beer name valid? (Minimum length? Characters allowed?)
2. Give me examples:
   - "IPA" → valid?
   - "" → invalid?
   - "Beer123" → valid?
3. What if name is null?
4. Should it throw or return false for invalid names?
```

### Scenario 2: Adding a REST Endpoint
```
Developer: "Create a GET endpoint for beers."

Questions:
1. What does the endpoint return?
2. Examples:
   - GET /beers → returns what structure?
   - GET /beers?type=IPA → filtered list?
   - GET /beers (empty database) → empty list or 404?
3. What HTTP status codes? (200 OK, 404 Not Found, etc.)
4. What if an error occurs? (500?)
```

### Scenario 3: Adding a Search/Filter Function
```
Developer: "Filter beers by style."

Questions:
1. Which styles are valid?
2. Examples:
   - Filter style="IPA", beers=[IPA, Pilsner, IPA] → [IPA, IPA]?
   - Filter style="Lager", beers=[] → []?
   - Filter style=null → throw or return all?
3. Is search case-sensitive?
4. What if style doesn't exist in database?
```

---

## Example Gathering Session (Full)

**Developer:** "Add a method to calculate beer ABV category (low, medium, high)."

**Phase 1 (Your Questions):**
```
Q1: "What makes each category? What ABV ranges are low/medium/high?"
A1: "Low: 0-7%, Medium: 7-14%, High: 14%+"

Q2: "Give me concrete examples:"
A2: 
   Input: 3.5 → Output: "low"
   Input: 10.0 → Output: "medium"
   Input: 15.0 → Output: "high"

Q3: "What if ABV is negative or over 20%?"
A3: "Throw IllegalArgumentException for invalid ABV"

Q4: "What about boundary cases? ABV = exactly 7.0?"
A4: "7.0 is medium (inclusive on lower bound)"

Q5: "Return type and exceptions?"
A5: "Returns String (one of: 'low', 'medium', 'high')"
    "Throws IllegalArgumentException if ABV < 0 or > 20"
```

**You've Now Gathered:** Enough clarity to write a perfect test!

---

## Output: Gathering Summary

After gathering, provide a summary for the developer:

```
## Gathering Summary

**Feature:** Calculate beer ABV category

**Examples:**
1. ABV = 3.5 → "low"
2. ABV = 10.0 → "medium"
3. ABV = 15.0 → "high"

**Edge Cases:**
- ABV < 0 → throw IllegalArgumentException
- ABV > 20 → throw IllegalArgumentException
- ABV = 7.0 (boundary) → "medium"

**Return Type:** String
**Exceptions:** IllegalArgumentException(message)

**Ready for testing?** ✓ Yes, let's write tests!
```

Then proceed to exact-tdd Phase 2 (RED).

---

## Anti-Patterns (Never Do These)

❌ **Assume requirements** — "I think this means..."  
❌ **Ask yes/no questions** — "Should it validate?" (ask "how" not "should")  
❌ **Accept vague answers** — "It should handle errors" (which errors? how?)  
❌ **Multiple questions at once** — Ask one, listen, ask next  
❌ **Skip edge cases** — "Let's handle null later"  
❌ **Design discussions** — Gathering is about EXAMPLES, not architecture  
❌ **Assume happy path only** — Always ask about errors  

---

## Success Metric

You've gathered successfully when:
1. Developer has given 2-3+ concrete input → output examples
2. Edge cases (null, empty, out of range) are clarified
3. Error cases and exceptions are defined
4. Return types and method signatures are clear
5. Developer says "Yes, I understand what needs to be tested"

Then hand off to `/exact-tdd` Phase 2 (RED).
