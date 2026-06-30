# exact-stack-java — Java 21 / Spring Boot / Gradle

Stack reference for the EXACT workflow. Import this alongside the methodology guides.

Works with: Claude Code, OpenAI Codex, OpenCode/pi.dev, and other AI-assisted development tools.

## Technical Stack

- **Language:** Java 21 (records, pattern matching, `var`, sealed classes)
- **Framework:** Spring Boot 3.x (clean architecture, separation of concerns)
- **Build Tool:** Gradle
- **Testing:** JUnit 5 + AssertJ (`assertThat(...)`)
- **Lombok:** NO — use native Java 21 records for maximum transparency

## Project Structure

```
src/main/java/com/example/beerapp/
├── BeerAppApplication.java
├── controller/
├── service/
├── model/
└── dto/

src/test/java/com/example/beerapp/
└── [mirrors src/main/java structure]

src/main/resources/
└── application.properties
```

## Common Commands

```bash
# Run all tests
./gradlew test

# Run single test class
./gradlew test --tests BeerServiceTests

# Run single test method
./gradlew test --tests BeerServiceTests.shouldValidateEmptyName

# Run with output
./gradlew test --info

# Build (skip tests)
./gradlew build -x test

# Run app → http://localhost:8080
./gradlew bootRun
```

## Test Code Patterns

### Simple Unit Test (no Spring context)
```java
import org.junit.jupiter.api.Test;
import static org.assertj.core.api.Assertions.*;

class BeerValidatorTests {

    @Test
    void shouldReturnTrueWhenNameIsValid() {
        var result = BeerValidator.isValid("IPA");
        assertThat(result).isTrue();
    }

    @Test
    void shouldThrowExceptionWhenNameIsNull() {
        assertThatThrownBy(() -> BeerValidator.isValid(null))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessage("Name required");
    }
}
```

### Spring Service Test (@SpringBootTest)
```java
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.beans.factory.annotation.Autowired;

@SpringBootTest
class BeerServiceTests {

    @Autowired
    private BeerService beerService;

    @Test
    void shouldCreateBeerWhenInputIsValid() {
        var dto = new BeerDto("IPA", 6.5, "American");
        var result = beerService.createBeer(dto);
        assertThat(result.name()).isEqualTo("IPA");
    }
}
```

### Controller Slice Test (@WebMvcTest)
```java
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.web.servlet.MockMvc;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(BeerController.class)
class BeerControllerTests {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private BeerService beerService;

    @Test
    void shouldReturnOkWhenGettingBeers() throws Exception {
        mockMvc.perform(get("/beers"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$").isArray());
    }
}
```

## Production Code Patterns

### Record (DTO)
```java
public record BeerDto(String name, double abv, String style) {}
```

### Service
```java
@Service
public class BeerService {
    public boolean isValidName(String name) {
        if (name == null) throw new IllegalArgumentException("Name required");
        return !name.isEmpty();
    }
}
```

### REST Controller
```java
@RestController
@RequestMapping("/beers")
public class BeerController {
    private final BeerService beerService;

    public BeerController(BeerService beerService) {
        this.beerService = beerService;
    }

    @GetMapping
    public List<BeerDto> getBeers() {
        return beerService.getAllBeers();
    }
}
```

## AssertJ Quick Reference

```java
// Primitives
assertThat(value).isEqualTo(5);
assertThat(value).isPositive();
assertThat(value).isBetween(0, 100);

// Strings
assertThat(name).isEqualTo("Beer");
assertThat(name).contains("IPA");
assertThat(name).isNotBlank();

// Collections
assertThat(beers).isEmpty();
assertThat(beers).hasSize(3);
assertThat(beers).allMatch(b -> b.abv() > 0);

// Exceptions
assertThatThrownBy(() -> service.method())
    .isInstanceOf(IllegalArgumentException.class)
    .hasMessage("Name required");

// Optional
assertThat(optional).isPresent();
assertThat(optional).contains(expected);
```

## Test File Naming

```
Source: src/main/java/.../BeerService.java
Test:   src/test/java/.../BeerServiceTests.java
```
