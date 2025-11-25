# Spring Boot Unit Testing Workshop – Baseline

This repository is the minimalist baseline for a hands‑on Spring Boot unit testing workshop. It intentionally contains only build artifacts (compiled classes and historic Surefire test reports) so participants can practice creating the source code, tests, and Maven configuration from scratch or import their own existing `pom.xml` and source tree.

## Purpose
- Provide a clean starting point with no distractions.
- Encourage Test Driven Development (TDD) by writing failing tests first.
- Practice high‑quality unit tests around Spring components (controllers, services, repositories) using JUnit 5, Mockito, AssertJ, and Spring Boot test support.

## What You Will Build
During the workshop you will typically implement:
- A simple banking domain (e.g. savings rate calculator, health/status endpoint).
- Focused unit tests for pure logic (no Spring context needed).
- Slice tests (e.g. `@WebMvcTest`) for controllers.
- Tests using mocks/stubs for external dependencies.

## Current State of This Baseline
Files present are mainly compiled output directories such as `classes/`, `test-classes/`, and historical reports under `surefire-reports/`. A `pom.xml` and `src/` tree are intentionally absent so you can create them yourself.

If you prefer starting from an already scaffolded Spring Boot project you can generate one at https://start.spring.io and then copy exercises from this workshop scenario.

## Prerequisites
- Java 17+ (adjust if your training uses a different LTS).
- Maven 3.9+ (or use Gradle if agreed; examples below use Maven).
- Git.
- An IDE (IntelliJ IDEA, VS Code with Java extensions, or Eclipse).

Verify versions:
```bash
java -version
mvn -v
```

## Step 1: Create Maven Project Skeleton
Add a `pom.xml` at the repository root. Example minimal Spring Boot setup:
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
				 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.bank</groupId>
	<artifactId>starter</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>spring-boot-unit-testing-workshop</name>
	<description>Workshop project for mastering Spring Boot unit tests</description>
	<properties>
		<java.version>17</java.version>
		<spring.boot.version>3.2.0</spring.boot.version>
	</properties>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-dependencies</artifactId>
				<version>${spring.boot.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<!-- Testing Stack -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<!-- Includes JUnit 5, AssertJ, Mockito, Spring Test -->
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>
```

## Step 2: Add Source Structure
Create the conventional directories:
```
src/
	main/
		java/
			com/bank/starter/   (production code)
	test/
		java/
			com/bank/starter/   (test code)
```

Example starter classes (to be created by participants):
```java
// src/main/java/com/bank/starter/HealthController.java
package com.bank.starter;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HealthController {
	@GetMapping("/health")
	public String health() {
		return "OK";
	}
}
```
```java
// src/main/java/com/bank/starter/SavingsRateCalculator.java
package com.bank.starter;

public class SavingsRateCalculator {
	public double rateForBalance(double balance) {
		if (balance < 0) throw new IllegalArgumentException("Negative balance");
		if (balance < 1000) return 0.01;
		if (balance < 10000) return 0.015;
		return 0.02;
	}
}
```

## Step 3: Write Your First Tests
Illustrative unit test examples:
```java
// src/test/java/com/bank/starter/SavingsRateCalculatorTest.java
package com.bank.starter;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class SavingsRateCalculatorTest {
	private final SavingsRateCalculator calc = new SavingsRateCalculator();

	@Test void smallBalanceHasOnePercent() {
		assertEquals(0.01, calc.rateForBalance(500));
	}
}
```

```java
// src/test/java/com/bank/starter/HealthControllerTest.java
package com.bank.starter;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(HealthController.class)
class HealthControllerTest {
	@Autowired MockMvc mvc;

	@Test void healthEndpointReturnsOk() throws Exception {
		mvc.perform(get("/health"))
			 .andExpect(status().isOk())
			 .andExpect(content().string("OK"));
	}
}
```

## Running Tests
After adding `pom.xml` and sources:
```bash
mvn clean test
```
The generated reports will appear under `target/surefire-reports/`. Compare them with the historical ones currently in `surefire-reports/` for learning purposes.

## Suggested Exercise Roadmap
1. Red/Green/Refactor on `SavingsRateCalculator` edge cases.
2. Parameterized tests for multiple balance tiers.
3. Introduce a dependency (e.g. interest policy provider) and mock it with Mockito.
4. Controller slice test with error handling.
5. Validation logic and testing constraint violations.
6. Refactor tests for readability (Arrange/Act/Assert pattern).
7. Measure coverage (optional) and discuss limitations.

## Testing Guidelines
- Keep unit tests fast and isolated; avoid the Spring context unless necessary.
- Name tests clearly, prefer descriptive method names over comments.
- Assert one logical behavior per test.
- Use parameterized tests for data permutations.
- Use mocks only where interaction behavior matters.

## Common Pitfalls
- Overusing Spring context for simple logic tests (slows feedback loop).
- Asserting too many behaviors in one test.
- Mocking value objects (prefer real instances).
- Ignoring readability—tests are executable specifications.

## Useful References
- Spring Boot Docs: https://docs.spring.io/spring-boot/docs/current/reference/html/
- JUnit 5 User Guide: https://junit.org/junit5/docs/current/user-guide/
- Mockito: https://site.mockito.org/
- AssertJ: https://assertj.github.io/doc/

## Contributing
During the workshop, create feature branches per exercise (e.g. `exercise/parameterized-tests`) and open pull requests for review/discussion if using a shared org.

## License
No explicit license provided yet. For broader reuse, add an OSS license (e.g. MIT or Apache 2.0).

---
Happy testing! Build understanding first; code coverage follows.