# Console Job Migration Instructions: .NET Framework 4.8 to .NET 9

**Purpose:** This document provides step-by-step instructions for LLMs and developers to migrate console job applications from .NET Framework 4.8 to .NET 9 using the Orchestra Clean Architecture pattern.

## ## LLM Instruction Header

> **IMPORTANT FOR AI ASSISTANTS:** When a user asks about migrating .NET Framework 4.8 console jobs to .NET 9, you MUST follow the guidelines in this document. This includes:

1.  **Architecture:** Follow the Clean Architecture pattern (Domain -> Application -> Infrastructure -> ConsoleApp)
2.  **Project Structure:** Create files in the correct layer folders as specified in this guide
3.  **Code Patterns:** Use the .NET 9 patterns shown in the "Code Patterns & Transformations" section
4.  **Configuration:** Convert app.config to appsettings.json using the Options pattern
5.  **Testing:** Use xUnit with Moq and FluentAssertions
6.  **Dependencies:** Use NuGet with centralized package management (Directory.Packages.props)

**Reference this file:** `src/HeaderUpdate.ConsoleApp/migration-instructions.md`

## ## Table of Contents

1.  [Architecture Overview](#architecture-overview)
2.  [Project Structure](#project-structure)
3.  [Migration Checklist](#migration-checklist)
4.  [Layer-by-Layer Migration Guide](#layer-by-layer-migration-guide)
5.  [Code Patterns & Transformations](#code-patterns-transformations)
6.  [Configuration Migration](#configuration-migration)
7.  [Dependency Injection Setup](#dependency-injection-setup)
8.  [Testing Migration](#testing-migration)
9.  [Common Issues & Solutions](#common-issues-solutions)

---

## ## Architecture Overview

This solution follows **"Clean Architecture (Domain-Driven Design)"** with four distinct layers:

```text
       Presentation Layer
       (ProjectName).ConsoleApp
       (Entry Point - Program.cs)

               |
               | depends on
               v
     Infrastructure Layer
    (ProjectName).Infrastructure
(External Services, Health Checks, Messaging, Jobs)

               |
               | depends on
               v
      Application Layer
     (ProjectName).Application
(Business Logic, Contracts/Interfaces, Services)

               |
               | depends on
               v
         Domain Layer
       (ProjectName).Domain
(Entities, Exceptions, Core Models - NO DEPENDENCIES)
```

### ### Key Principles
1.  **Domain Layer:** No external dependencies. Contains entities, value objects, domain exceptions, and repository interfaces.
2.  **Application Layer:** Depends only on Domain. Contains Business logic, service interfaces (contracts), and orchestration.
3.  **Infrastructure Layer:** Depends on Application. Implements external integrations, messaging, health checks, and jobs.
4.  **Presentation Layer:** Depends on Application and Infrastructure. Entry point with DI configuration.

---

## ## Project Structure

When migrating, create/maintain this folder structure:

```text
{SolutionName}/
├── src/
│   ├── {ProjectName}.Domain/
│   │   ├── {ProjectName}.Domain.csproj
│   │   ├── Entities/             # Domain entities
│   │   ├── Exceptions/           # Custom domain exceptions
│   │   ├── ValueObjects/         # Value objects
│   │   └── Repositories/         # Repository interfaces (optional)
│   ├── {ProjectName}.Application/
│   │   ├── {ProjectName}.Application.csproj
│   │   ├── Constants/            # Configuration keys, default values, messages
│   │   ├── Contracts/            # Service interfaces (e.g., IMyService)
│   │   ├── Extensions/           # ServiceCollectionExtensions.cs
│   │   ├── Services/             # Business logic implementations
│   │   └── DTOs/                 # Data transfer objects
│   ├── {ProjectName}.Infrastructure/
│   │   ├── {ProjectName}.Infrastructure.csproj
│   │   ├── Constants/            # Infrastructure-specific constants
│   │   ├── Extensions/           # ServiceCollectionExtensions.cs
│   │   ├── HealthChecks/         # Custom health checks
│   │   ├── Jobs/                 # Background job implementations
│   │   ├── QuartzJobs/           # Quartz scheduled jobs
│   │   ├── Messaging/            # Kafka/messaging implementations
│   │   └── Models/               # Configuration/settings models
│   └── {ProjectName}.ConsoleApp/
│       ├── {ProjectName}.ConsoleApp.csproj
│       ├── Program.cs            # Entry point with Host configuration
│       ├── Properties/
│       │   └── launchSettings.json
│       └── appsettings.json      # Configuration file
└── Tests/
    ├── {ProjectName}.Domain.Tests/
    ├── {ProjectName}.Application.Tests/
    ├── {ProjectName}.Infrastructure.Tests/
    ├── {ProjectName}.ConsoleApp.Tests/
    └── {ProjectName}.BDD.Tests/   # BDD/Integration tests
```

---

## ## Migration Checklist

### ### Pre-Migration
- [ ] Identify all NuGet packages and their .NET 9 equivalents
- [ ] Document existing configuration (app.config, web.config)
- [ ] List all external dependencies (databases, APIs, message queues)
- [ ] Catalog scheduled jobs/timers and their triggers
- [ ] Review logging implementation for migration to Microsoft.Extensions.Logging

### ### Project Setup
- [ ] Create solution structure with Clean Architecture layers
- [ ] Configure `Directory.Packages.props` for centralized package management
- [ ] Set up project references following dependency rules
- [ ] Configure `build.props` for common build settings

### ### Code Migration
- [ ] Migrate domain entities and exceptions to Domain layer
- [ ] Migrate business logic and interfaces to Application layer
- [ ] Migrate external integrations to Infrastructure layer
- [ ] Set up Program.cs with Generic Host pattern
- [ ] Configure dependency injection in ServiceCollectionExtensions

### ### Configuration
- [ ] Convert app.config/web.config to appsettings.json
- [ ] Set up environment-specific configuration files
- [ ] Configure Options pattern for strongly-typed settings
- [ ] Migrate connection strings and secrets

### ### Testing
- [ ] Set up xUnit test projects
- [ ] Migrate existing tests to xUnit syntax
- [ ] Configure Moq for mocking
- [ ] Set up BDD tests with Reqroll (if applicable)

---

## ## Layer-by-Layer Migration Guide

### ### 1. Domain Layer (`{ProjectName}.Domain.csproj`)
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
  </PropertyGroup>
  <!-- NO PackageReference or ProjectReference allowed -->
</Project>
```

**Migration Steps:**
1.  Create custom exceptions inheriting from `Exception`
2.  Move domain entities (persistence-ignorant)
3.  Define repository interfaces (if using Repository pattern)
4.  No external NuGet packages allowed in this layer

**Example Exception Migration:**
```csharp
// .NET Framework 4.8 (Before)
public class CustomException : ApplicationException
{
    public CustomException(string message) : base(message) { }
}

// .NET 9 (After)
namespace {ProjectName}.Domain.Exceptions;

public class CustomException : Exception
{
    public CustomException() { }
    public CustomException(string message) : base(message) { }
    public CustomException(string message, Exception inner) : base(message, inner) { }
}
```

### ### 2. Application Layer (`{ProjectName}.Application.csproj`)
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="WellsFargo.Orchestra.MetaPackage" />
  </ItemGroup>
  <ItemGroup>
    <ProjectReference Include="..\..\{ProjectName}.Domain\{ProjectName}.Domain.csproj" />
  </ItemGroup>
</Project>
```

**Migration Steps:**
1.  Define service interfaces in `Contracts/` folder
2.  Move business logic to `Services/` folder
3.  Create constants for configuration sections and messages
4.  Create `ServiceCollectionExtensions` for DI registration

**ServiceCollectionExtensions Pattern:**
```csharp
using Microsoft.Extensions.DependencyInjection;

namespace {ProjectName}.Application.Extensions;

public static class ServiceCollectionExtensions
{
    public static IServiceCollection RegisterApplicationServices(this IServiceCollection services)
    {
        // Register application layer services
        // services.AddScoped<IMyService, MyService>();
        return services;
    }
}
```

### ### 3. Infrastructure Layer (`{ProjectName}.Infrastructure.csproj`)
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="WellsFargo.Orchestra.MetaPackage" />
  </ItemGroup>
  <ItemGroup>
    <ProjectReference Include="..\..\{ProjectName}.Application\{ProjectName}.Application.csproj" />
  </ItemGroup>
</Project>
```

**Migration Steps:**
1.  Implement Application layer interfaces
2.  Create health checks for external dependencies
3.  Migrate scheduled jobs to Quartz.NET pattern
4.  Set up messaging/Kafka producers and consumers
5.  Create configuration models for settings

**ServiceCollectionExtensions Pattern:**
```csharp
using Microsoft.Extensions.DependencyInjection;

namespace {ProjectName}.Infrastructure.Extensions;

public static class ServiceCollectionExtensions
{
    public static IServiceCollection RegisterInfrastructureServices(this IServiceCollection services)
    {
        // Register infrastructure layer services
        // services.AddScoped<IExternalService, ExternalServiceImplementation>();
        // services.AddHealthChecks().AddCheck<MyHealthCheck>("my-health-check");
        return services;
    }
}
```

### ### 4. Console Application Layer (`{ProjectName}.ConsoleApp.csproj`)
```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="WellsFargo.Orchestra.MetaPackage" />
  </ItemGroup>
  <ItemGroup>
    <ProjectReference Include="..\..\{ProjectName}.Application\{ProjectName}.Application.csproj" />
    <ProjectReference Include="..\..\{ProjectName}.Infrastructure\{ProjectName}.Infrastructure.csproj" />
  </ItemGroup>
  <ItemGroup>
    <Content Update="appsettings.json">
      <CopyToOutputDirectory>Always</CopyToOutputDirectory>
    </Content>
  </ItemGroup>
  <ItemGroup>
    <AssemblyMetadata Include="Orchestra:FrameworkType" Value="Console" />
  </ItemGroup>
</Project>
```

**Program.cs Pattern:**
```csharp
using {ProjectName}.Application.Extensions;
using {ProjectName}.Infrastructure.Extensions;
using WellsFargo.Orchestra.Core.Extensions;
using WellsFargo.Logging.Core.Extensions;

namespace {ProjectName}.ConsoleApp;

partial class Program
{
    static Program() { }
    protected Program() { }

    public static void Main()
    {
        Console.WriteLine("Starting the application...");

        var host = CreateHostBuilder().Build();
        host.RunAsync();

        Console.WriteLine("Application started. Press any key to exit.");
        Console.Read();
    }

    static IHostBuilder CreateHostBuilder() =>
        Host.CreateDefaultBuilder()
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var env = hostingContext.HostingEnvironment;
                config.AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true, reloadOnChange: true)
                      .SetBasePath(Environment.CurrentDirectory);
            })
            .AddOrchestraLogging()
            .ConfigureServices((hostContext, services) =>
            {
                services.RegisterApplicationServices();
                services.RegisterInfrastructureServices();
            });
}
// Required for BDD testing
public partial class Program { }
```

---

## ## Code Patterns & Transformations

### ### Common .NET Framework to .NET 9 Transformations


| .NET Framework 4.8 | .NET 9 Equivalent |
| :--- | :--- |
| `ConfigurationManager.AppSettings["key"]` | `IConfiguration["key"]` or Options pattern |
| `ConfigurationManager.ConnectionStrings["name"]` | `IConfiguration.GetConnectionString("name")` |
| `Log4net` / `NLog` (direct) | `Microsoft.Extensions.Logging.ILogger<T>` |
| `System.Net.Http.HttpClient` | `IHttpClientFactory` |
| `Timer` / `System.Threading.Timer` | `Quartz.NET` 'Live' or 'BackgroundService' |
| `static` singletons | Dependency Injection with `AddSingleton<T>()` |
| `app.config` / `web.config` | `appsettings.json` with Options pattern |

### ### Logging Migration
```csharp
// .NET Framework 4.8 (Before)
private static readonly ILog _logger = LogManager.GetLogger(typeof(MyClass));
_logger.Info("Processing started");
_logger.Error("Error occurred", exception);

// .NET 9 (After)
public class MyClass
{
    private readonly ILogger<MyClass> _logger;

    public MyClass(ILogger<MyClass> logger)
    {
        _logger = logger;
    }

    public void Process()
    {
        _logger.LogInformation("Processing started");
        _logger.LogError(exception, "Error occurred");
    }
}
```

### ### Configuration Migration
```csharp
// .NET Framework 4.8 (Before - app.config)
var setting = ConfigurationManager.AppSettings["MySetting"];
var connectionString = ConfigurationManager.ConnectionStrings["MyDb"].ConnectionString;

// .NET 9 (After - Options Pattern)
// 1. Create settings model
public class MySettings
{
    public string MySetting { get; set; } = string.Empty;
    public string ConnectionString { get; set; } = string.Empty;
}

// 2. Register in DI
services.Configure<MySettings>(configuration.GetSection("MySettings"));

// 3. Inject and use
public class MyService
{
    private readonly MySettings _settings;

    public MyService(IOptions<MySettings> options)
    {
        _settings = options.Value;
    }
}
```

### ### Scheduled Job Migration (Timer to Quartz)
```csharp
// .NET Framework 4.8 (Before)
private Timer _timer;
public void Start()
{
    _timer = new Timer(DoWork, null, TimeSpan.Zero, TimeSpan.FromMinutes(5));
}

// .NET 9 (After - Quartz Job)
public class MyScheduledJob : IJob
{
    private readonly ILogger<MyScheduledJob> _logger;
    private readonly IMyService _service;

    public MyScheduledJob(ILogger<MyScheduledJob> logger, IMyService service)
    {
        _logger = logger;
        _service = service;
    }

    public async Task Execute(IJobExecutionContext context)
    {
        _logger.LogInformation("Job started at {Time}", DateTime.UtcNow);
        await _service.DoWorkAsync();
        _logger.LogInformation("Job completed at {Time}", DateTime.UtcNow);
    }
}

// Register in appsettings.json
// "QuartzSchedule": { "MyJobTriggerTime": "0 0/5 * 1/1 * ? * " }
```

### ### HttpClient Migration
```csharp
// .NET Framework 4.8 (Before)
using (var client = new HttpClient())
{
    client.BaseAddress = new Uri("https://api.example.com");
    var response = await client.GetAsync("/endpoint");
}

// .NET 9 (After - Using IHttpClientFactory)
// 1. Register in DI
services.AddHttpClient("MyApiClient", client =>
{
    client.BaseAddress = new Uri("https://api.example.com");
});

// 2. Inject and use
public class MyApiService
{
    private readonly IHttpClientFactory _httpClientFactory;

    public MyApiService(IHttpClientFactory httpClientFactory)
    {
        _httpClientFactory = httpClientFactory;
    }

    public async Task<string> GetDataAsync()
    {
        var client = _httpClientFactory.CreateClient("MyApiClient");
        var response = await client.GetAsync("/endpoint");
        return await response.Content.ReadAsStringAsync();
    }
}
```

---

## ## Configuration Migration

### ### app.config to appsettings.json Conversion

**Before (app.config):**
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <appSettings>
    <add key="ServiceEndpoint" value="https://api.example.com" />
    <add key="MaxRetries" value="3" />
    <add key="TimeoutSeconds" value="30" />
  </appSettings>
  <connectionStrings>
    <add name="DefaultConnection" connectionString="Server=...;Database=..." />
  </connectionStrings>
</configuration>
```

**After (appsettings.json):**
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning"
    }
  },
  "ServiceSettings": {
    "ServiceEndpoint": "https://api.example.com",
    "MaxRetries": 3,
    "TimeoutSeconds": 30
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=...;Database=..."
  },
  "QuartzSchedule": {
    "JobTriggerTime": "0 0/5 * 1/1 * ? *"
  },
  "FeatureManagement": {
    "HealthChecks": true,
    "EnterpriseLoggingEnabled": true
  },
  "TraceSettings": {
    "ApplicationId": "your-app-id",
    "ObservabilitySupportEnabled": true
  }
}
```

### ### Environment-Specific Configuration
Create separate files for each environment:
- `appsettings.json` (base configuration)
- `appsettings.Development.json`
- `appsettings.Staging.json`
- `appsettings.Production.json`

---

## ## Dependency Injection Setup

### ### Registration Order in Program.cs
```csharp
.ConfigureServices((hostContext, services) =>
{
    var configuration = hostContext.Configuration;

    // 1. Register Options/Settings
    services.Configure<MySettings>(configuration.GetSection("MySettings"));

    // 2. Register Application Layer Services
    services.RegisterApplicationServices();

    // 3. Register Infrastructure Layer Services
    services.RegisterInfrastructureServices();

    // 4. Register Hosted Services (if any)
    services.AddHostedService<MyBackgroundService>();

    // 5. Register Health Checks
    services.AddHealthChecks()
            .AddCheck<DatabaseHealthCheck>("database");
});
```

### ### Service Lifetime Guidelines


| Lifetime | Use Case |
| :--- | :--- |
| **Singleton** | Stateless services, caches, configuration |
| **Scoped** | Per-request/per-operation services, DbContext |
| **Transient** | Lightweight, stateless services |

---

## ## Testing Migration

### ### Test Project Setup
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <IsPackable>false</IsPackable>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" />
    <PackageReference Include="xunit" />
    <PackageReference Include="xunit.runner.visualstudio" />
    <PackageReference Include="Moq" />
    <PackageReference Include="FluentAssertions" />
  </ItemGroup>
  <ItemGroup>
    <ProjectReference Include="..\..\src\{ProjectName}.Application\{ProjectName}.Application.csproj" />
  </ItemGroup>
</Project>
```


### ### Test Pattern Migration
```csharp
// .NET Framework 4.8 (MSTest - Before)
[TestClass]
public class MyServiceTests
{
    [TestMethod]
    public void MyMethod_WhenCalled_ShouldReturnExpectedResult()
    {
        // Arrange, Act, Assert
    }

    [TestMethod]
    [ExpectedException(typeof(ArgumentException))]
    public void MyMethod_WithInvalidInput_ShouldThrowException()
    {
        // Test code
    }
}

// .NET 9 (xUnit - After)
public class MyServiceTests
{
    private readonly Mock<ILogger<MyService>> _loggerMock;
    private readonly Mock<IDependency> _dependencyMock;
    private readonly MyService _sut;

    public MyServiceTests()
    {
        _loggerMock = new Mock<ILogger<MyService>>();
        _dependencyMock = new Mock<IDependency>();
        _sut = new MyService(_loggerMock.Object, _dependencyMock.Object);
    }

    [Fact]
    public void MyMethod_WhenCalled_ShouldReturnExpectedResult()
    {
        // Arrange
        _dependencyMock.Setup(x => x.GetData()).Returns("test");

        // Act
        var result = _sut.MyMethod();

        // Assert
        result.Should().Be("expected");
    }

    [Fact]
    public void MyMethod_WithInvalidInput_ShouldThrowArgumentException()
    {
        // Arrange
        var invalidInput = "";

        // Act & Assert
        Assert.Throws<ArgumentException>(() => _sut.MyMethod(invalidInput));
    }

    [Theory]
    [InlineData("input1", "output1")]
    [InlineData("input2", "output2")]
    public void MyMethod_WithVariousInputs_ShouldReturnCorrectOutput(string input, string expected)
    {
        var result = _sut.Process(input);
        result.Should().Be(expected);
    }
}
```

### ### Test Attribute Mapping


| MSTest | xUnit |
| :--- | :--- |
| `[TestClass]` | (not needed) |
| `[TestMethod]` | `[Fact]` |
| `[DataRow]` | `[InlineData]` with `[Theory]` |
| `[TestInitialize]` | Constructor |
| `[TestCleanup]` | `IDisposable.Dispose()` |
| `[ExpectedException]` | `Assert.Throws<T>()` |

---

## ## Common Issues & Solutions

### ### Issue 1: Missing Assembly References
**Symptom:** "The type or namespace name 'X' could not be found"
**Solution:** Add the appropriate NuGet package using centralized package management in `Directory.Packages.props`.

### ### Issue 2: Configuration Not Loading
**Symptom:** Configuration values are null or default
**Solution:**
1.  Ensure `appsettings.json` has "CopyToOutputDirectory" set to "Always"
2.  Verify configuration section names match exactly
3.  Check that Options pattern is correctly registered

### ### Issue 3: Dependency Injection Errors
**Symptom:** "Unable to resolve service for type 'X'"
**Solution:**
1.  Ensure service is registered in `ServiceCollectionExtensions`
2.  Check registration lifetime (Singleton, Scoped, Transient)
3.  Verify all dependencies of the service are also registered

### ### Issue 4: Static Methods/Classes
**Symptom:** Cannot inject dependencies into static classes
**Solution:** Refactor static classes to instance classes with constructor injection.

```csharp
// Before (static)
public static class Helper
{
    public static void DoWork() { }
}

// After (instance with DI)
public interface IHelper
{
    void DoWork();
}

public class Helper : IHelper
{
    private readonly ILogger<Helper> _logger;

    public Helper(ILogger<Helper> logger)
    {
        _logger = logger;
    }

    public void DoWork() { }
}
```

### ### Issue 5: Timer/Scheduled Jobs Not Running
**Symptom:** Background jobs don't execute
**Solution:**
1.  Use Quartz.NET for scheduled jobs
2.  Register jobs properly with trigger expressions
3.  Ensure host keeps running (`host.RunAsync()` not `host.Run()` for non-blocking)

### ### Issue 6: Async/Await Migration
**Symptom:** Deadlocks or synchronous code blocking
**Solution:** Migrate to async patterns throughout the call chain.

```csharp
// Before (sync over async - BAD)
var result = SomeAsyncMethod().Result;

// After (proper async)
var result = await SomeAsyncMethod();
```

### ### Issue 7: Package Source Mapping Errors
**Symptom:** NuGet packages not found
**Solution:** Configure package source mapping in `nuget.config`.

```xml
<packageSourceMapping>
  <packageSource key="nuget.org">
    <package pattern="*" />
  </packageSource>
  <packageSource key="WellsFargo">
    <package pattern="WellsFargo.*" />
  </packageSource>
</packageSourceMapping>
```

---

## ## Quick Reference: Cron Expression Format

For Quartz.NET scheduled jobs:

```text
* * * * * * *

| | | | | | |
| | | | | | +-- year (optional)
| | | | | +---- day of week (0-6 or SUN-SAT)
| | | | +------ month (1-12)
| | | +-------- day of month (1-31)
| | +---------- hour (0-23)
| +------------ minute (0-59)
+-------------- second (0-59)
```

**Examples:
0 0/5 * * * ? - Every 5 minutes
0 0 8 * * ? - Every day at 8:00 AM
0 0 0/1 * * ? - Every hour (pp. 27-28)
## Additional Resources
Orchestra .NET Documentation
Clean Architecture by Microsoft
.NET 9 Migration Guide
Quartz.NET Documentation
xUnit Documentation (pp. 27-28)
## Orchestra Framework Support
For help on Orchestra frameworks use following resources:
Refer to project README.md
Orchestra AI Agent (@orchestra) (hop.wf.com)
Access Orchestra Knowledge Base (p. 28)
## Console Job Migration Context
When working on console job migrations from .NET Framework 4.8 to .NET 9, you MUST:
Read and follow the instructions in #file:src/HeaderUpdate.ConsoleApp/migration-instructions.md
Follow Clean Architecture pattern (Domain -> Application -> Infrastructure + ConsoleApp)
Use the code patterns and transformations specified in the migration instructions
Ensure all new code follows the project structure defined in migration guide (p. 28)
#File:README.md
#file:src/HeaderUpdate.ConsoleApp/migration-instructions.md (p. 28)
