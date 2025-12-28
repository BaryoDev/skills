---
name: baryo-architecture
description: Defines the standard architectural patterns for BaryoDev applications, focusing on Vertical Slices, Observability, and Clean Startup.
---

# BaryoDev Architecture Standard

You are the Lead System Designer. When building **Applications** (Web APIs, Services), you follow this strict architectural blueprint.

## 1. Vertical Slice Architecture
We do **not** use "Clean Architecture" layers (Controller -> Service -> Repository) horizontally. We use Vertical Slices.

**Folder Structure**:
```
src/ProjectName/
├── Features/
│   ├── Auth/
│   │   ├── Login.cs
│   │   ├── Register.cs
│   │   └── AuthEvents.cs
│   ├── Workflows/
│   │   ├── CreateWorkflow.cs
│   │   └── ExecuteWorkflow.cs
├── Infrastructure/      (Shared concerns: DbContext, EmailSender)
└── Program.cs
```

## 2. Startup & Observability
Every BaryoDev application MUST be observable from Day 1.

**Mandatory Middleware**:
1.  **Serilog**: Wired to Console and File. Structured logging only.
2.  **Prometheus**: `app.UseHttpMetrics()` and `app.MapMetrics()`.
3.  **HealthChecks**: `app.MapHealthChecks("/health")`.

**Fail Fast Strategy**:
Do not allow the app to start if dependencies are missing. Check connections/vars in `Program.cs`.

```csharp
// Example: Fail Fast
var dbUrl = Environment.GetEnvironmentVariable("DATABASE_URL");
if (string.IsNullOrEmpty(dbUrl)) 
{
    Log.Fatal("DATABASE_URL is missing!");
    throw new Exception("Config Error");
}
```

## 3. Deployment Ready
-   **Docker**: Always include a `Dockerfile` and `docker-compose.yml`.
-   **CI/CD**: Assume GitHub Actions.
-   **Seeding**: If data is needed, run a `DataSeeder` task in the background after startup, do not block the main thread.
