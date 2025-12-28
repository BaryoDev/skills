---
name: baryo-packaging
description: Standardizes the BaryoDev manual packaging process for NuGet and NPM, ensuring correct metadata and version control.
---

# BaryoDev Packaging Protocol

You are the Release Manager. We prefer **Assisted Manual Publishing** over black-box CI/CD for final releases to ensure quality and accountability.

## NuGet Best Practices

### 1. Metadata (.csproj)
Every public package MUST have:
```xml
<PropertyGroup>
    <Authors>BaryoDev</Authors>
    <Company>BaryoDev</Company>
    <Copyright>Copyright (c) BaryoDev 2024</Copyright>
    <PackageProjectUrl>https://github.com/BaryoDev/[RepoName]</PackageProjectUrl>
    <RepositoryUrl>https://github.com/BaryoDev/[RepoName]</RepositoryUrl>
    <PackageLicenseExpression>MPL-2.0</PackageLicenseExpression>
    <PackageIcon>icon.png</PackageIcon>
    <PackageReadmeFile>README.md</PackageReadmeFile>
    <!-- Deterministic Builds -->
    <EmbedUntrackedSources>true</EmbedUntrackedSources>
    <IncludeSymbols>true</IncludeSymbols>
    <SymbolPackageFormat>snupkg</SymbolPackageFormat>
</PropertyGroup>
```

### 2. Versioning Strategy
-   We use **SemVer 2.0**.
-   **Draft/Pre-release**: `1.0.0-beta.1`, `1.0.0-rc.1`
-   **Do not auto-increment** on every commit. Version bumps are a deliberate human decision.

## Manual Release Workflow

### Step 1: Clean & Build
Ensure a clean slate. No cached artifacts.
```bash
dotnet clean -c Release
dotnet build -c Release
```

### Step 2: Test
Must pass 100% of tests.
```bash
dotnet test -c Release --no-build
```

### Step 3: Pack
```bash
dotnet pack -c Release -o ./artifacts /p:Version=[VERSION]
```

### Step 4: Verify
Inspect the generated `.nupkg` (using NuGet Package Explorer or zip tool) to ensure:
-   `README.md` is present.
-   `LICENSE` is present.
-   DLLs are optimised.

### Step 5: Push
```bash
dotnet nuget push ./artifacts/*.nupkg -s https://api.nuget.org/v3/index.json -k [API_KEY]
```
