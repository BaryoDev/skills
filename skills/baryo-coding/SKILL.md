---
name: baryo-coding
description: Enforces BaryoDev's strict high-performance, zero-dependency, and allocation-free coding standards. Use this for all library development.
---

# BaryoDev Coding Standard

You are acting as the Core Architect for BaryoDev. Your goal is to write code that fits the "Baryo Way".

## Modes of Operation

Determine if you are writing a **Library** (like `Carom`, `Verdict`, `Mapsicle`) or an **Application** (like `barakoCMS`).

### Mode A: Library (Performance First)
**Philosophy**: "Zero Allocations, Zero Dependencies, Zero Bloat."
-   **Structs**: Use `readonly struct` for configuration and state. (Example: `Bounce` struct in Carom).
-   **Loops**: Use `for` loops. **NEVER** use LINQ (`Where`, `Select`) in hot paths.
-   **API Design**: Use `static` methods over class instantiation where possible to reduce GC pressure.
-   **Fluent Builders**: Implement Fluent APIs that mutate structs or configuration objects without allocating.
-   **Async/Sync**: Manually mirror `Async` and `Sync` methods. Do NOT use `Task.Run` wrappers.

#### A.1 Meta-Programming & Reflection (Mapsicle Style)
-   **Expression Trees**: Use `Expression<T>` and `Compile()` to generate high-performance delegates at runtime. Cache them in `ConcurrentDictionary`.
-   **Avoid Raw Reflection**: Do not use `PropertyInfo.GetValue()` in hot loop paths. Compile it once.
-   **No Emit**: Avoid `System.Reflection.Emit` (ILGenerator) unless absolutely necessary, as it breaks generic AOT/trimming more easily than Expressions.
-   **Dictionary fallback**: Provide reflection-based fallbacks only for dynamic scenarios (`ToDictionary`), but warn about performance.

### Mode B: Application (Maintainability First)
**Philosophy**: "Vertical Slices, Observability, Cleanliness."
-   **Architecture**: Use **Vertical Slice Architecture**. Group code by Feature (`Features/Auth`, `Features/Content`) not by layer.
-   **Clean Start**: Keep `Program.cs` minimal. Move setup to `Extensions/ServiceCollectionExtensions.cs`.
-   **Observability**: ALWAYs wire up Serilog, Prometheus Metrics (`UseHttpMetrics`), and HealthChecks.
-   **Fail Fast**: Validate critical configuration (e.g., DB strings) at startup and crash if missing.

## Universal Rules
1.  **Nullable**: `<Nullable>enable</Nullable>` is mandatory.
2.  **Comments**: Use `/// <summary>` XML comments for all public members.
3.  **Naming**: Prefer creative, metaphorical names over generic ones (e.g., `Bounce`, `Shot` instead of `RetryPolicy`, `Execute`).
4.  **No Exceptions for Flow**: Return `Results` (Success/Failed) instead of throwing exceptions for validation errors.
