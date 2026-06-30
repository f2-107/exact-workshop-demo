# exact-stack-kotlin — Kotlin / Spring Boot / Gradle

Stack reference for the EXACT workflow. Import this alongside the methodology guides.

Works with: Claude Code, OpenAI Codex, OpenCode/pi.dev, and other AI-assisted development tools.

## Technical Stack

- **Language:** Kotlin (data classes, extension functions, null safety, coroutines where needed)
- **Framework:** Spring Boot 3.x (clean architecture, separation of concerns)
- **Build Tool:** Gradle (Kotlin DSL: `build.gradle.kts`)
- **Testing:** JUnit 5 + AssertJ (`assertThat(...)`)
- **No annotation processors** that obscure behavior — prefer explicit code for demo clarity

## Project Structure

```
src/main/kotlin/com/example/beerapp/
├── BeerAppApplication.kt
├── controller/
├── service/
├── model/
└── dto/

src/test/kotlin/com/example/beerapp/
└── [mirrors src/main/kotlin structure]

src/main/resources/
└── application.properties

build.gradle.kts
```

## Common Commands

```bash
# Run all tests
./gradlew test

# Run single test class
./gradlew test --tests BeerServiceTests

# Run single test method
./gradlew test --tests "BeerServiceTests.should validate empty name"

# Run with output
./gradlew test --info

# Build (skip tests)
./gradlew build -x test

# Run app → http://localhost:8080
./gradlew bootRun
```

## Test Code Patterns

### Simple Unit Test (no Spring context)
```kotlin
import org.assertj.core.api.Assertions.*
import org.junit.jupiter.api.Test

class BeerValidatorTests {

    @Test
    fun `should return true when name is valid`() {
        val result = BeerValidator.isValid("IPA")
        assertThat(result).isTrue()
    }

    @Test
    fun `should throw exception when name is null`() {
        assertThatThrownBy { BeerValidator.isValid(null) }
            .isInstanceOf(IllegalArgumentException::class.java)
            .hasMessage("Name required")
    }
}
```

### Spring Service Test (@SpringBootTest)
```kotlin
import org.springframework.boot.test.context.SpringBootTest
import org.springframework.beans.factory.annotation.Autowired

@SpringBootTest
class BeerServiceTests {

    @Autowired
    private lateinit var beerService: BeerService

    @Test
    fun `should create beer when input is valid`() {
        val dto = BeerDto(name = "IPA", abv = 6.5, style = "American")
        val result = beerService.createBeer(dto)
        assertThat(result.name).isEqualTo("IPA")
    }
}
```

### Controller Slice Test (@WebMvcTest)
```kotlin
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest
import org.springframework.test.web.servlet.MockMvc
import org.springframework.test.web.servlet.get

@WebMvcTest(BeerController::class)
class BeerControllerTests {

    @Autowired
    private lateinit var mockMvc: MockMvc

    @MockBean
    private lateinit var beerService: BeerService

    @Test
    fun `should return ok when getting beers`() {
        mockMvc.get("/beers")
            .andExpect { status { isOk() } }
            .andExpect { jsonPath("$") { isArray() } }
    }
}
```

## Production Code Patterns

### Data Class (DTO)
```kotlin
data class BeerDto(
    val name: String,
    val abv: Double,
    val style: String
)
```

### Service
```kotlin
@Service
class BeerService {
    fun isValidName(name: String?): Boolean {
        requireNotNull(name) { "Name required" }
        return name.isNotEmpty()
    }
}
```

### REST Controller
```kotlin
@RestController
@RequestMapping("/beers")
class BeerController(private val beerService: BeerService) {

    @GetMapping
    fun getBeers(): List<BeerDto> = beerService.getAllBeers()
}
```

## Kotlin-Specific Tips for EXACT

- **Data classes** replace Java records — use for all DTOs
- **Null safety** replaces null checks: `String?` vs `String`
- **require / requireNotNull** throw `IllegalArgumentException` — perfect for validations:
  ```kotlin
  require(abv in 0.0..20.0) { "ABV must be 0-20" }
  requireNotNull(name) { "Name required" }
  ```
- **Named arguments** make test setup readable:
  ```kotlin
  val dto = BeerDto(name = "IPA", abv = 6.5, style = "American")
  ```
- **Backtick test names** make test output readable in reports

## AssertJ Quick Reference (Kotlin)

```kotlin
// Primitives
assertThat(value).isEqualTo(5)
assertThat(value).isPositive()
assertThat(value).isBetween(0, 100)

// Strings
assertThat(name).isEqualTo("Beer")
assertThat(name).contains("IPA")
assertThat(name).isNotBlank()

// Collections
assertThat(beers).isEmpty()
assertThat(beers).hasSize(3)
assertThat(beers).allMatch { it.abv > 0 }

// Exceptions
assertThatThrownBy { service.method() }
    .isInstanceOf(IllegalArgumentException::class.java)
    .hasMessage("Name required")
```

## Test File Naming

```
Source: src/main/kotlin/.../BeerService.kt
Test:   src/test/kotlin/.../BeerServiceTests.kt
```