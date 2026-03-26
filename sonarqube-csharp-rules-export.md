# C# Unit Testing Prompt for xUnit and Moq

## Objective
Generate comprehensive unit test cases for a C# application using **xUnit** and **Moq** frameworks to achieve **minimum 80% code coverage** for SonarQube compliance.

## Instructions

### 1. Test Structure Requirements
- Use **xUnit** as the testing framework
- Implement **Moq** for mocking dependencies and external services
- Follow **AAA pattern** (Arrange, Act, Assert) in all test methods
- Use descriptive test method names following the pattern: `MethodName_Scenario_ExpectedResult`

### 2. Coverage Requirements
- Target **minimum 80% code coverage** for SonarQube
- Cover all public methods and properties
- Include both **positive and negative test scenarios**
- Test **edge cases and boundary conditions**
- Cover **exception handling paths**

### 3. Test Categories to Include
- **Happy path scenarios**: normal execution flows
- **Edge cases**: boundary values, empty collections, null inputs
- **Error scenarios**: invalid inputs, exceptions
- **Business logic validation**
- **Integration points** with mocked dependencies

### 4. Moq Implementation Guidelines
- Mock all external dependencies (databases, APIs, file systems)
- Use `Mock.Setup()` to configure method returns
- Verify method calls using `Mock.Verify()`
- Use `It.Is<T>()` and `It.IsAny<T>()` for parameter matching
- Mock async methods appropriately

### 5. Best Practices
- One assertion per test method when possible
- Use meaningful test data and avoid magic numbers
- Implement test fixtures for complex setup using `IClassFixture<T>`
- Use `[Theory]` and `[InlineData]` for parameterized tests
- Group related tests in the same test class
- Use proper disposal patterns for resources

### 6. Code Quality Standards
- Ensure tests are **isolated and independent**
- Make tests **deterministic and repeatable**
- Keep test methods **simple and focused**
- Use **clear and descriptive assertions**
- Add **XML documentation** for complex test scenarios

## Output Format
For each class under test, provide:
1. Complete test class with proper namespace and using statements
2. Mock setup in constructor or test method
3. All test methods covering the specified scenarios
4. Inline comments explaining complex test logic
5. Coverage analysis summary

## Example Request Format
When requesting tests, provide:
- Source code of the class/method to test
- Dependencies and interfaces used
- Any specific business rules or constraints
- Expected input/output examples

---

*Ensure all generated tests compile successfully and follow C# coding conventions.*
