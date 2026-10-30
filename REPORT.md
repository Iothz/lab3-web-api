# Lab 3 Web API -- Project Report

## What I specified

The semantic properties of each endpoint are:
- POST /employees
  - Safety: Not safe (creates a new employee)
  - Idempotency: Not idempotent (multiple calls create multiple employees with different IDs)
- GET /employees/{id}
  - Safety: Safe (retrieves employee data without modifying it)
  - Idempotency: Idempotent (multiple calls return the same employee data)
- PUT /employees/{id}
  - Safety: Not safe (updates employee data)
  - Idempotency: Idempotent (multiple calls with the same data result in the same state)
- DELETE /employees/{id}
  - Safety: Not safe (deletes an employee)
  - Idempotency: Idempotent (multiple calls result in the same state, employee is deleted)
- GET /employees
  - Safety: Safe (retrieves a list of employees without modifying it)
  - Idempotency: Idempotent (multiple calls return the same list of employees)

Pass criteria
    - Every test must pass the `./gradlew test` command with exact Mockk verification of interaction counts and no calls to forbidden methods.
## What I changed

Edited the following files:
- `src/test/kotlin/es/unizar/webeng/lab3/ControllerTest.kt`
  - Replaced all TODO's with the blocks proposed.
  - Added required imports: `io.mockk.every`, `io.mockk.justRun`,`io.mockk.verify` and `java.util.Optional`.

## Technical decisions

[Choices you own: which MockK block you used for each method, what you rejected, and why a repeated call does or does not change state.]

The following table shows the blocks used in `ControllerTests.kt`:

| Block | Type | Test in `ControllerTests.kt` |
|---|---|---|
| **1.** `Setup: save returns a new id on each call` | Setup | Test 1: `POST is not safe and not idempotent` |
| **2.** `Setup: findById returns an employee for id 1` | Setup | Test 2: `GET is safe and idempotent` |
| **3.** `Setup: findById returns empty for id 2` | Setup | Test 2: `GET is safe and idempotent` |
| **4.** `Setup: findById is empty, then returns the employee` | Setup | Test 3: `PUT is idempotent but not safe` |
| **5.** `Setup: save returns the employee at id 1` | Setup | Test 3: `PUT is idempotent but not safe` |
| **6.** `Setup: deleteById may be called` (`justRun`) | Setup | Test 4: `DELETE is idempotent but not safe` |
| **7.** `Setup: findAll returns one employee` | Setup | Test 5: `GET collection is safe and idempotent` |
| **8.** `Verify: both saves discard the client id` + `no read or delete` | Verify | Test 1: `POST is not safe and not idempotent` |
| **9.** `Verify: findById(1)` y `(2)` twice + `no modifying methods` | Verify | Test 2: `GET is safe and idempotent` |
| **10.** `Verify: both saves store the same employee` + `no delete or list` | Verify | Test 3: `PUT is idempotent but not safe` |
| **11.** `Verify: deleteById was called twice, and nothing else` | Verify | Test 4: `DELETE is idempotent but not safe` |
| **12.** `Verify: findAll was called twice` + `no write or read one id` | Verify | Test 5: `GET collection is safe and idempotent` |

Explanation of different technical decisions:
- In verify for POST, specific arguments (e.g., `Employee("Mary", "Manager", null)`) are used instead of generics like `any()`, to verify that the controller wipes the ID's provided by the client.
- To model state transition for `PUT` and `POST` chained answers with `answers{...}` and `andThenAnswer{...}` are used, to simulate the behavior of the repository returning different results on subsequent calls.
- For `DELETE`, `justRun` is used to indicate that the method can be called without returning a value, using `every {...} return Unit` would have the same effect but is less idiomatic.

## How I verified

Ran `./gradlew check` after every test implementation with the following tests failing with expected `NotImplementedError` on `TODO`

After implementing all tests, `ktlintTestSourceSetCheck` failed due to import ordering (java.util.Optional was not placed at the end). Ran `./gradlew ktlintFormat`, adjusted imports, and re-ran `./gradlew check` resulting in BUILD SUCCESSFUL (all 5 unit tests passed and all Ktlint tasks succeeded).


## AI disclosure

- **Tools / skills:** Antigravity agent using Gemini 3.8 Flash (Google AI), guided by repository instructions in AGENTS.md.
- **Purpose:** Guided step-by-step test implementation, automatic check execution (./gradlew check), Ktlint formatting debugging, and conceptual validation of HTTP safety and idempotency.
- **Representative prompts:** 
  - "Siguiendo la guia comienza a implementar el punto 1"
  - "¿Cómo funciona verify en Mockk y cómo se diferencia de ejecutar el método?"
  - "Verifica que pasen todos los test"
- **Affected files/sections:** `src/test/kotlin/es/unizar/webeng/lab3/ControllerTests.kt` (imports, setups, and verifications).
- **Validation steps:** Ran ./gradlew check after each step, verified Ktlint compliance, and asked the agent for explanations of verify vs execution semantics to ensure complete comprehension.
- **Citations:** Stubbing and verification blocks adapted from docs/GUIDE.md and RFC 9110 definitions.
- **Human-reviewed:** Supervised workflow execution step-by-step, enforced constraints from AGENTS.md, reviewed import reordering caused by Ktlint, and validated the theoretical justification of HTTP semantics.