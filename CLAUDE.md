# EXACT Coding Skills — Starter Kit

This repository is a **copy-paste starter kit** for AI-assisted, test-driven development using the EXACT methodology. Drop it into any project and pick the stack that matches.

**Methodology:** EXACT (Example-guided, AI-Collaborative, Test-driven) — see [`.ai-dev/ai-instructions.md`](.ai-dev/ai-instructions.md)

## How to Use This Kit

1. Copy the files below into your project root
2. Pick your stack and configure CLAUDE.md (see below)
3. Open your AI tool and start with the EXACT workflow

### Files to Copy

```
.ai-dev/
├── ai-instructions.md          ← Core EXACT methodology (language-agnostic)
└── skills/
├── exact-tdd.md                ← Red-Green-Refactor workflow guide
├── exact-example-gathering.md  ← Gathering concrete examples
├── exact-test-writer.md        ← Generating minimal test code
├── exact-verifier.md           ← Verifying code matches examples
├── exact-security-check.md     ← Security & quality review
├── exact-live-demo.md          ← Optimizing for presentations
│
├── exact-stack-java.md         ← Stack: Java 21 / Spring Boot / Gradle
├── exact-stack-kotlin.md       ← Stack: Kotlin / Spring Boot / Gradle
└── exact-stack-angular.md      ← Stack: Angular / TypeScript / Jest
```

## CLAUDE.md — Configure for Your Stack

Copy and adapt this block into your project's `CLAUDE.md`:

### Java 21 / Spring Boot

```markdown
## Quick Start

Spring Boot 3.x web app (Gradle).

**Stack:** Java 21, Spring Boot 3.x, Gradle, JUnit 5 + AssertJ, no Lombok

**Methodology:** EXACT — see [`.ai-dev/ai-instructions.md`](.ai-dev/ai-instructions.md) and `.ai-dev/skills/exact-stack-java.md`

## Common Commands

    ./gradlew test                      # Run all tests
    ./gradlew test --tests MyClass      # Run single test class
    ./gradlew bootRun                   # Run app → http://localhost:8080

## Important Rules

1. Never skip EXACT workflow — tests first, code second
2. Always ask for concrete examples before coding
3. Minimal production code — no extra features
4. No Lombok — use Java 21 records
5. Use AssertJ (`assertThat(...)` not `assertEquals(...)`)
6. Keep explanations short (live demo optimized)
```

### Kotlin / Spring Boot

```markdown
## Quick Start

Spring Boot 3.x web app (Gradle, Kotlin DSL).

**Stack:** Kotlin, Spring Boot 3.x, Gradle, JUnit 5 + AssertJ

**Methodology:** EXACT — see [`.ai-dev/ai-instructions.md`](.ai-dev/ai-instructions.md) and `.ai-dev/skills/exact-stack-kotlin.md`

## Common Commands

    ./gradlew test                      # Run all tests
    ./gradlew test --tests MyClass      # Run single test class
    ./gradlew bootRun                   # Run app → http://localhost:8080

## Important Rules

1. Never skip EXACT workflow — tests first, code second
2. Always ask for concrete examples before coding
3. Minimal production code — no extra features
4. Use data classes for DTOs, not annotations
5. Use AssertJ (`assertThat(...)` not `assertEquals(...)`)
6. Use backtick test names: `fun \`should return low when abv is low\`()`
```

### Angular / TypeScript

```markdown
## Quick Start

Angular 17+ web app (standalone components).

**Stack:** TypeScript (strict), Angular 17+, Jest, Testing Library

**Methodology:** EXACT — see [`.ai-dev/ai-instructions.md`](.ai-dev/ai-instructions.md) and `.ai-dev/skills/exact-stack-angular.md`

## Common Commands

    npx jest                            # Run all tests
    npx jest beer.service.spec.ts       # Run single test file
    npx jest --watch                    # Watch mode
    ng serve                            # Run app → http://localhost:4200

## Important Rules

1. Never skip EXACT workflow — tests first, code second
2. Always ask for concrete examples before coding
3. Minimal production code — no extra features
4. No `any` — use explicit types or `unknown`
5. Standalone components only (no NgModules unless forced)
6. Keep explanations short (live demo optimized)
```

## Available Skills

Use these with `/exact-tdd`, `/exact-example-gathering`, etc. in your AI tool:

| Skill | When to Use |
|-------|-------------|
| `exact-tdd` | Full workflow enforcement (Red-Green-Refactor) |
| `exact-example-gathering` | Extract concrete examples before coding |
| `exact-test-writer` | Generate minimal failing tests |
| `exact-verifier` | Verify code matches examples |
| `exact-security-check` | Security & quality review before merge |
| `exact-live-demo` | Prepare code for presentations |

## EXACT Workflow Summary

```
1. GATHER   → Ask for concrete input → output examples
2. RED      → Write minimal failing test; run tests → confirm failure
3. GREEN    → Write minimum production code; run tests → confirm pass
4. REFACTOR → Suggest improvements; get approval before applying
```

Never skip phases. Never write code before a failing test exists.

## See Also

- **[`.ai-dev/ai-instructions.md`](.ai-dev/ai-instructions.md)** — Full EXACT methodology for AI tools
- **[`.ai-dev/skills/`](.ai-dev/skills/)** — All workflow and stack guides