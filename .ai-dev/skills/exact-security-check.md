# exact-security-check Guide

## Purpose
Catch common security, validation, and EXACT methodology pitfalls before code reaches production. This guide performs a targeted security and quality review focused on EXACT-specific issues.

Works with: Claude Code, OpenAI Codex, OpenCode/pi.dev, and other AI-assisted development tools.

## When to Use
Apply this guide when:
- Code is ready for security review
- Before merging to main branch
- Before pushing to production
- When handling user input or sensitive data
- If code failed tests initially

## Security Pitfall Categories

### Category 1: Input Validation Issues
**Problem:** Missing or insufficient input validation

**Pitfalls to Check:**
```
❌ Null checks missing
❌ Empty string/list treated as valid
❌ Out-of-range values not rejected
❌ Format not validated (email, phone, etc.)
❌ Whitespace not trimmed/handled
❌ Type confusion (string vs number)
```

**Examples:**

**Bad:**
```java
public String category(double abv) {
    if (abv < 7) return "low";
    return "high";  // ❌ No validation!
}

public BeerDto createBeer(BeerDto dto) {
    return repository.save(dto);  // ❌ No null check!
}
```

**Good:**
```java
public String category(double abv) {
    if (abv < 0 || abv > 20) {
        throw new IllegalArgumentException("ABV must be 0-20");
    }
    if (abv < 7) return "low";
    return "high";
}

public BeerDto createBeer(BeerDto dto) {
    if (dto == null) {
        throw new IllegalArgumentException("Beer data required");
    }
    if (dto.name() == null || dto.name().isEmpty()) {
        throw new IllegalArgumentException("Beer name required");
    }
    return repository.save(dto);
}
```

**Check Items:**
- [ ] All method parameters validated?
- [ ] Null checks present?
- [ ] Range/boundary checks in place?
- [ ] Format validation done?
- [ ] Empty collections handled?

---

### Category 2: Exception Handling
**Problem:** Wrong exception types or missing exception safety

**Pitfalls to Check:**
```
❌ Using RuntimeException instead of specific types
❌ Swallowing exceptions (catch and do nothing)
❌ Re-throwing as wrong type
❌ Missing exception messages
❌ Unsafe null access (NullPointerException)
```

**Examples:**

**Bad:**
```java
// ❌ Wrong exception type
try {
    int val = Integer.parseInt(input);
} catch (NumberFormatException e) {
    throw new RuntimeException(e);  // Wrong!
}

// ❌ Swallowing exception
try {
    processData();
} catch (Exception e) {
    // Silent failure
}

// ❌ Unsafe null access
String name = beer.getName();  // What if beer is null?
System.out.println(name.toUpperCase());
```

**Good:**
```java
// ✓ Correct exception type with message
try {
    int val = Integer.parseInt(input);
} catch (NumberFormatException e) {
    throw new IllegalArgumentException("Invalid ABV: " + input, e);
}

// ✓ Proper error handling
try {
    processData();
} catch (DataException e) {
    logger.error("Failed to process data: {}", e.getMessage());
    throw new ApplicationException("Data processing failed", e);
}

// ✓ Safe null check
if (beer == null) {
    throw new IllegalArgumentException("Beer required");
}
String name = beer.getName();
System.out.println(name.toUpperCase());
```

**Check Items:**
- [ ] Exception types match specification?
- [ ] Exception messages are descriptive?
- [ ] No generic RuntimeException?
- [ ] No swallowed exceptions?
- [ ] All null accesses protected?

---

### Category 3: Secrets & Sensitive Data
**Problem:** Hardcoded secrets or sensitive data exposure

**Pitfalls to Check:**
```
❌ API keys, passwords in code
❌ Database credentials hardcoded
❌ Private tokens in strings
❌ Sensitive data logged
❌ Secrets in comments
```

**Examples:**

**Bad:**
```java
// ❌ NEVER hardcode secrets
private static final String API_KEY = "sk-abc123def456";
private static final String DB_PASSWORD = "admin123";

@GetMapping("/api")
public void callApi() {
    String auth = "Bearer sk-secret123";  // ❌ In code
    logger.info("User: " + user.getPassword());  // ❌ Logging password
}
```

**Good:**
```java
// ✓ Use environment variables or secure config
private String apiKey = System.getenv("API_KEY");
private String dbPassword = System.getenv("DB_PASSWORD");

@GetMapping("/api")
public void callApi() {
    String auth = "Bearer " + apiKey;  // From env var
    logger.info("User authenticated successfully");  // No sensitive data
}
```

**Check Items:**
- [ ] No API keys in code?
- [ ] No passwords hardcoded?
- [ ] No tokens in strings?
- [ ] Sensitive data not logged?
- [ ] No secrets in comments?
- [ ] Using environment variables for config?

---

### Category 4: SQL Injection & Query Safety
**Problem:** Building SQL with string concatenation

**Pitfalls to Check:**
```
❌ String concatenation in SQL
❌ Not using parameterized queries
❌ User input directly in SQL
```

**Examples:**

**Bad:**
```java
// ❌ SQL Injection vulnerability
String query = "SELECT * FROM beers WHERE name = '" + beerName + "'";
resultSet = statement.executeQuery(query);
```

**Good:**
```java
// ✓ Use parameterized queries
String query = "SELECT * FROM beers WHERE name = ?";
preparedStatement = connection.prepareStatement(query);
preparedStatement.setString(1, beerName);
resultSet = preparedStatement.executeQuery();

// ✓ Or use ORM (Spring Data)
List<Beer> beers = beerRepository.findByName(beerName);
```

**Check Items:**
- [ ] No string concatenation in SQL?
- [ ] Parameterized queries used?
- [ ] No user input directly in SQL?

---

### Category 5: Authentication & Authorization
**Problem:** Missing or incorrect access control

**Pitfalls to Check:**
```
❌ No authentication checks
❌ No authorization on endpoints
❌ Sensitive endpoints publicly accessible
❌ No CSRF protection (if applicable)
❌ No rate limiting
```

**Examples:**

**Bad:**
```java
// ❌ No authentication check
@GetMapping("/admin/beers")
public List<BeerDto> getAllBeers() {  // Anyone can access!
    return service.getAllBeers();
}

// ❌ No authorization check
@DeleteMapping("/beers/{id}")
public void deleteBeer(@PathVariable Long id) {  // Anyone can delete!
    service.deleteBeer(id);
}
```

**Good:**
```java
// ✓ Requires authentication
@GetMapping("/admin/beers")
@PreAuthorize("hasRole('ADMIN')")
public List<BeerDto> getAllBeers() {
    return service.getAllBeers();
}

// ✓ Requires specific role and ownership check
@DeleteMapping("/beers/{id}")
@PreAuthorize("hasRole('ADMIN')")
public void deleteBeer(@PathVariable Long id) {
    if (!isOwner(id)) {
        throw new AccessDeniedException("Not authorized");
    }
    service.deleteBeer(id);
}
```

**Check Items:**
- [ ] Sensitive endpoints protected?
- [ ] Authentication required where needed?
- [ ] Authorization checks in place?
- [ ] Roles properly defined?

---

### Category 6: EXACT Methodology Compliance
**Problem:** Not following EXACT TDD principles

EXACT compliance verification is covered in full by the **exact-verifier** guide. Run `/exact-verifier` after completing this security check.

**Quick checks only (full details in exact-verifier):**
- [ ] Tests exist and pass (GREEN)?
- [ ] Code is minimal — no features beyond examples?
- [ ] Phase sequence was followed (GATHERING → RED → GREEN)?

---

## Security Check Checklist

Use this comprehensive checklist:

### Input Validation
- [ ] All public methods validate parameters?
- [ ] Null checks present for objects?
- [ ] Range/boundary validation present?
- [ ] String length/format validated?
- [ ] Collections not treated as always populated?

### Error Handling
- [ ] Correct exception types used?
- [ ] Exception messages are descriptive?
- [ ] No generic RuntimeException?
- [ ] No swallowed exceptions?
- [ ] All null accesses protected?

### Secrets & Configuration
- [ ] No hardcoded API keys?
- [ ] No passwords in code?
- [ ] No tokens in strings?
- [ ] No sensitive data logged?
- [ ] Using environment variables for secrets?

### Data Safety
- [ ] No SQL injection (parameterized queries)?
- [ ] No XSS vulnerabilities?
- [ ] User input properly escaped?
- [ ] No hardcoded data paths?

### Access Control
- [ ] Sensitive endpoints protected?
- [ ] Authentication required?
- [ ] Authorization checks present?
- [ ] No public access to admin features?

### EXACT Compliance
- [ ] Tests written before code?
- [ ] All tests passing (GREEN)?
- [ ] No extra features (only examples)?

→ Run `/exact-verifier` for the full EXACT compliance checklist.

---

## Security Check Process

### Step 1: Review Input Validation
```
For each public method:
  - Is parameter null-checked?
  - Is input range validated?
  - Are error cases handled?
  
If not: Flag as validation issue
```

### Step 2: Review Exception Handling
```
For each try-catch:
  - Is it the correct exception type?
  - Is the message descriptive?
  - Is exception swallowed or re-thrown?
  
If not: Flag as exception issue
```

### Step 3: Scan for Secrets
```
Search for:
  - "password"
  - "secret"
  - "key" (API key)
  - "token"
  - "credential"
  
If found in code (not env var): Flag as secret issue
```

### Step 4: Check Data Safety
```
For database queries:
  - Are they parameterized?
  - Is user input included?
  
For REST endpoints:
  - Is input escaped?
  - Is output encoded?
  
If not: Flag as data safety issue
```

### Step 5: Verify EXACT Compliance
Run `/exact-verifier` for a full EXACT compliance check. Quick checks here:
```
- Do tests exist and pass?
- Is code minimal (no extra features)?

If not: Flag and run /exact-verifier for details
```

---

## Security Check Report

After security review, provide a report:

```
## Security Check Report

**Component:** BeerValidator
**Status:** ✓ PASS

### Input Validation
- [x] Null check for name parameter
- [x] Empty string rejected
- [x] Clear error message

### Exception Handling
- [x] IllegalArgumentException thrown
- [x] Message: "Beer name required"
- [x] No generic exceptions

### Secrets & Configuration
- [x] No hardcoded values
- [x] No sensitive data

### EXACT Compliance
- [x] Tests exist and pass
- [x] Code is minimal
- [x] Matches examples

### Issues Found
❌ None — code is secure!

### Recommendations
[Only if issues found]
```

---

## Critical Issues (Always Flag)

🚨 **CRITICAL** — Stop and fix before proceeding:
- Hardcoded secrets (API keys, passwords, tokens)
- SQL injection vulnerabilities
- Missing authentication on sensitive endpoints
- Unvalidated user input used in critical operations
- Tests not passing (RED state)

⚠️ **HIGH** — Should fix:
- Missing null checks on required inputs
- Generic RuntimeException instead of specific types
- Missing validation on ranges/boundaries
- Logging of sensitive data

📋 **MEDIUM** — Should consider:
- Improving exception messages
- Adding validation to edge cases
- Refactoring complex validation logic

---

## Success Metric

Security check passes when:
1. All critical issues are resolved
2. Input validation in place for all public methods
3. Exception types are correct and specific
4. No hardcoded secrets or sensitive data
5. Tests exist and pass (GREEN)
6. Code matches EXACT examples (no extras)
7. Ready for production or code review
