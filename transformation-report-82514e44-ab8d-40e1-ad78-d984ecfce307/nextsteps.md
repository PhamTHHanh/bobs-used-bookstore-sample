# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Summary

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore Dependencies

Run a full NuGet package restore to ensure all dependencies are resolved correctly in the new target framework:

```bash
dotnet restore
```

Review the output for any warnings about deprecated or incompatible packages. If any packages reference `netstandard` or older `net4x` targets exclusively, consider finding cross-platform compatible alternatives on [NuGet](https://www.nuget.org).

---

## 2. Build the Solution

Perform a full solution build to confirm there are no compilation issues:

```bash
dotnet build --configuration Release
```

Address any warnings that surface during the build, particularly those related to nullable reference types, obsolete APIs, or platform compatibility analyzers (e.g., `CA1416`).

---

## 3. Run the Unit Tests

Execute the test project to verify that existing logic behaves as expected after migration:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

- Review any failing tests and determine whether failures are due to migration-related changes or pre-existing issues.
- Check that test dependencies such as mocking frameworks (e.g., Moq, NSubstitute) are compatible with the current .NET version.

---

## 4. Validate the Data Layer

The `Bookstore.Data` project likely contains database access logic (e.g., Entity Framework Core). Perform the following checks:

- Confirm that the correct EF Core provider package is referenced (e.g., `Microsoft.EntityFrameworkCore.SqlServer`, `Npgsql.EntityFrameworkCore.PostgreSQL`).
- If using EF Core migrations, verify existing migrations are intact:

```bash
dotnet ef migrations list --project app/Bookstore.Data
```

- Apply migrations to a local or development database to confirm schema compatibility:

```bash
dotnet ef database update --project app/Bookstore.Data
```

---

## 5. Run the Web Application Locally

Start the `Bookstore.Web` project and manually verify core functionality:

```bash
dotnet run --project app/Bookstore.Web --configuration Release
```

- Navigate through the application in a browser and test key user flows (e.g., browsing books, authentication if applicable).
- Check the application logs for any runtime exceptions that would not surface at compile time, such as missing configuration values or unresolved service registrations.
- Confirm that `appsettings.json` and `appsettings.Production.json` contain all required configuration keys, particularly connection strings and any service endpoints.

---

## 6. Review the CDK Project

The `Bookstore.Cdk` project suggests infrastructure is defined in code, likely using AWS CDK for .NET. Verify the following:

- Confirm the `Amazon.CDK` NuGet packages are updated to a version compatible with the target .NET version.
- Synthesize the CDK stack to check for any infrastructure definition errors:

```bash
cdk synth
```

- Review the synthesized CloudFormation template for correctness before any deployment is attempted.

---

## 7. Check for Platform-Specific API Usage

Run the .NET Compatibility Analyzer to identify any remaining platform-specific API calls that may not behave correctly on non-Windows environments:

```bash
dotnet build /p:EnableNETAnalyzers=true /p:AnalysisMode=All
```

Pay particular attention to warnings prefixed with `CA1416` (platform compatibility) and address them by either guarding the calls with `OperatingSystem.IsWindows()` checks or replacing them with cross-platform alternatives.

---

## 8. Validate Configuration and Secrets

- Ensure no secrets or environment-specific values are hardcoded in source files.
- Use `dotnet user-secrets` for local development configuration:

```bash
dotnet user-secrets init --project app/Bookstore.Web
dotnet user-secrets set "ConnectionStrings:Default" "your-local-connection-string" --project app/Bookstore.Web
```

- Confirm that production configuration is sourced from environment variables or a secrets manager rather than committed configuration files.