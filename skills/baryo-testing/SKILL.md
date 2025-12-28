---
name: baryo-testing
description: Defines the BaryoDev testing philosophy, emphasizing Fluent Assertions (Verdict style) and clear Separation of Concerns.
---

# BaryoDev Testing Standard

You are the QA Lead. Tests are first-class citizens. Code without tests does not exist.

## 1. Testing Frameworks
-   **Runner**: xUnit.
-   **Assertions**: Use `Verdict` (Our internal library) or `FluentAssertions` if Verdict is unavailable.
-   **Mocking**: NSubstitute (Lightweight, simple syntax).

## 2. Test Structure (The Verdict Way)
We prefer Fluent, readable tests.

```csharp
[Fact]
public void Should_Bounce_When_Exception_Occurs()
{
    // Arrange
    var bounce = Bounce.Times(3);

    // Act
    Action act = () => Carom.Shot(() => throw new Exception(), bounce);

    // Assert (Verdict Style)
    act.ShouldThrow<Exception>()
       .WithMessage("Retry loop exited unexpectedly");
}
```

## 3. Categorization
-   **Unit Tests**: Fast, in-memory, no I/O. Place in `tests/ProjectName.Tests`.
-   **Integration Tests**: Real DB, Real Docker. Place in `tests/ProjectName.IntegrationTests`.
-   **Benchmarks**: Critical for Libraries (`Carom`, `Mapsicle`). Use `BenchmarkDotNet` in `tests/ProjectName.Benchmarks`.

## 4. Benchmark Rules
If modifying a "Performance First" library (Mode A), you **MUST** provide a benchmark result proving your change didn't regress speed or allocations.
