<img width="546" height="546" alt="image" src="https://github.com/user-attachments/assets/863d3708-77a3-42a1-a355-eedb1ee22b66" />

<img width="602" height="314" alt="image" src="https://github.com/user-attachments/assets/4cb57618-b4f3-4037-b665-86ea981229d9" />


<img width="1910" height="902" alt="image" src="https://github.com/user-attachments/assets/178e30dd-4702-466c-9391-93ff9a3e5ca0" />

https://codegenapp.vercel.app/
https://factoryos-alpha.vercel.app/login
https://divakar-racktest-demo.pages.dev/login.html
https://github.com/DivakarUpadhyay/QREQ/wiki



# Data Center Solution

A Clean Architecture example for managing data-center servers, tests, firmware, and racks. The tree below shows the complete proposed structure; the code samples that follow are illustrative starting points, not a claim that these files already exist in the repository.

## Project Structure

```text
DataCenterSolution/
├── DataCenterSolution.sln
├── src/
│   ├── DataCenter.API/
│   │   ├── Controllers/
│   │   │   ├── ServersController.cs
│   │   │   ├── ServerTestsController.cs
│   │   │   ├── FirmwareController.cs
│   │   │   └── RacksController.cs
│   │   ├── Middleware/
│   │   │   └── ExceptionMiddleware.cs
│   │   ├── Extensions/
│   │   ├── appsettings.json
│   │   └── Program.cs
│   ├── DataCenter.Application/
│   │   ├── Interfaces/
│   │   │   ├── IServerRepository.cs
│   │   │   ├── ITestRepository.cs
│   │   │   ├── IRedfishService.cs
│   │   │   └── IMesService.cs
│   │   ├── Features/
│   │   │   ├── Servers/
│   │   │   │   ├── Commands/
│   │   │   │   ├── Queries/
│   │   │   │   └── DTOs/
│   │   │   ├── ServerTesting/
│   │   │   │   ├── StartServerTest.cs
│   │   │   │   └── GetTestResult.cs
│   │   │   ├── Firmware/
│   │   │   └── RackManagement/
│   │   └── DependencyInjection.cs
│   ├── DataCenter.Domain/
│   │   ├── Entities/
│   │   │   ├── Server.cs
│   │   │   ├── Rack.cs
│   │   │   ├── TestResult.cs
│   │   │   └── Firmware.cs
│   │   ├── Enums/
│   │   │   └── TestStatus.cs
│   │   ├── Exceptions/
│   │   └── ValueObjects/
│   └── DataCenter.Infrastructure/
│       ├── Persistence/
│       │   ├── DbConnectionFactory.cs
│       │   └── Repositories/
│       │       ├── ServerRepository.cs
│       │       └── TestRepository.cs
│       ├── ExternalServices/
│       │   ├── RedfishService.cs
│       │   └── MesService.cs
│       └── DependencyInjection.cs
└── tests/
    ├── DataCenter.UnitTests/
    └── DataCenter.IntegrationTests/
```
namespace DataCenter.Domain.Entities;

```
public record CreateServerCommand(
    string SerialNumber,
    string Model,
    string BmcIpAddress
);
```

```
public class CreateServerHandler
{
    private readonly IServerRepository _repository;

    public CreateServerHandler(
        IServerRepository repository)
    {
        _repository = repository;
    }

    public async Task<int> HandleAsync(
        CreateServerCommand command,
        CancellationToken ct)
    {
        var server = new Server
        {
            SerialNumber = command.SerialNumber,
            Model = command.Model,
            BmcIpAddress = command.BmcIpAddress
        };

        return await _repository.AddAsync(server, ct);
    }
}
```

```
public class Server
{
    public int Id { get; set; }
    public string SerialNumber { get; set; } = "";
    public string Model { get; set; } = "";
    public string BmcIpAddress { get; set; } = "";
}
```

public interface IRedfishService
{
    Task<ServerHealthDto> GetHealthAsync(
        string serverId,
        CancellationToken ct);
}

public record ServerHealthDto(
    string PowerState,
    string Health);

    public class StartServerTest
{
    private readonly IRedfishService _redfish;

    public StartServerTest(IRedfishService redfish)
    {
        _redfish = redfish;
    }

    public async Task<ServerHealthDto> ExecuteAsync(
        string serverId,
        CancellationToken ct)
    {
        return await _redfish.GetHealthAsync(
            serverId, ct);
    }
}

using System.Net.Http.Json;

public class RedfishService : IRedfishService
{
    private readonly HttpClient _client;

    public RedfishService(HttpClient client)
    {
        _client = client;
    }

    public async Task<ServerHealthDto> GetHealthAsync(
        string serverId,
        CancellationToken ct)
    {
        var result = await _client.GetFromJsonAsync<
            RedfishResponse>(
            $"/redfish/v1/Systems/{serverId}", ct);

        return new ServerHealthDto(
            result?.PowerState ?? "Unknown",
            result?.Status?.Health ?? "Unknown");
    }
}

public record RedfishResponse(
    string PowerState,
    RedfishStatus Status);

public record RedfishStatus(string Health);

[ApiController]
[Route("api/server-tests")]
public class ServerTestsController : ControllerBase
{
    private readonly StartServerTest _service;

    public ServerTestsController(
        StartServerTest service)
    {
        _service = service;
    }

    [HttpGet("{serverId}/health")]
    public async Task<IActionResult> GetHealth(
        string serverId,
        CancellationToken ct)
    {
        var result = await _service.ExecuteAsync(
            serverId, ct);

        return Ok(result);
    }
}

## Example file contents

The examples below show the intended responsibility and a simple implementation shape for **every file listed in the tree**. Add the required NuGet packages (for example, `Microsoft.AspNetCore.OpenApi`, `Microsoft.Data.SqlClient`, and a test framework) to the corresponding projects. Connection, Redfish, and MES details should be adapted to your environment.

### `DataCenterSolution.sln`

A solution file is generated and maintained by the .NET CLI or Visual Studio. It references the four source projects and test projects. Create it with commands such as:

```bash
dotnet new sln -n DataCenterSolution
dotnet sln add src/DataCenter.API/DataCenter.API.csproj
dotnet sln add src/DataCenter.Application/DataCenter.Application.csproj
dotnet sln add src/DataCenter.Domain/DataCenter.Domain.csproj
dotnet sln add src/DataCenter.Infrastructure/DataCenter.Infrastructure.csproj
dotnet sln add tests/DataCenter.UnitTests/DataCenter.UnitTests.csproj
dotnet sln add tests/DataCenter.IntegrationTests/DataCenter.IntegrationTests.csproj
```

### `src/DataCenter.API/Controllers/ServersController.cs`

```csharp
using DataCenter.Application.Interfaces;
using Microsoft.AspNetCore.Mvc;

namespace DataCenter.API.Controllers;

[ApiController]
[Route("api/servers")]
public sealed class ServersController(IServerRepository servers) : ControllerBase
{
    [HttpGet]
    public async Task<IActionResult> GetAll(CancellationToken cancellationToken) =>
        Ok(await servers.GetAllAsync(cancellationToken));
}
```

### `src/DataCenter.API/Controllers/ServerTestsController.cs`

```csharp
using DataCenter.Application.Features.ServerTesting;
using Microsoft.AspNetCore.Mvc;

namespace DataCenter.API.Controllers;

[ApiController]
[Route("api/server-tests")]
public sealed class ServerTestsController(
    StartServerTestHandler startTest,
    GetTestResultHandler getResult) : ControllerBase
{
    [HttpPost("{serverId:guid}")]
    public async Task<IActionResult> Start(Guid serverId, CancellationToken cancellationToken) =>
        Accepted(await startTest.HandleAsync(serverId, cancellationToken));

    [HttpGet("{testId:guid}")]
    public async Task<IActionResult> GetResult(Guid testId, CancellationToken cancellationToken)
    {
        var result = await getResult.HandleAsync(testId, cancellationToken);
        return result is null ? NotFound() : Ok(result);
    }
}
```

### `src/DataCenter.API/Controllers/FirmwareController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;

namespace DataCenter.API.Controllers;

[ApiController]
[Route("api/firmware")]
public sealed class FirmwareController : ControllerBase
{
    [HttpGet]
    public IActionResult GetFirmwareCatalog() =>
        Ok(new[] { new { Version = "1.0.0", Model = "ExampleModel" } });
}
```

Replace the sample response with an Application use case and repository when implementing firmware management.

### `src/DataCenter.API/Controllers/RacksController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;

namespace DataCenter.API.Controllers;

[ApiController]
[Route("api/racks")]
public sealed class RacksController : ControllerBase
{
    [HttpGet]
    public IActionResult GetRacks() => Ok(Array.Empty<object>());
}
```

Replace the placeholder with a Rack Management query/use case.

### `src/DataCenter.API/Middleware/ExceptionMiddleware.cs`

```csharp
using System.Net;
using System.Text.Json;

namespace DataCenter.API.Middleware;

public sealed class ExceptionMiddleware(RequestDelegate next, ILogger<ExceptionMiddleware> logger)
{
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await next(context);
        }
        catch (Exception exception)
        {
            logger.LogError(exception, "Unhandled request error");
            context.Response.StatusCode = (int)HttpStatusCode.InternalServerError;
            context.Response.ContentType = "application/json";
            await context.Response.WriteAsync(JsonSerializer.Serialize(new
            {
                error = "An unexpected error occurred."
            }));
        }
    }
}
```

### `src/DataCenter.API/Extensions/`

This is a folder, not a file. Put API-specific registration or middleware extension methods here, for example `WebApplicationExtensions.cs` with a `UseApiDefaults` extension. No such file was listed in the supplied tree.

### `src/DataCenter.API/appsettings.json`

```json
{
  "ConnectionStrings": {
    "DataCenter": "Server=localhost;Database=DataCenter;Trusted_Connection=True;TrustServerCertificate=True"
  },
  "Redfish": {
    "BaseUrl": "https://bmc.example.local/",
    "TimeoutSeconds": 30
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

Do not put real passwords, tokens, or production secrets in this file; use user secrets or a secure configuration provider.

### `src/DataCenter.API/Program.cs`

```csharp
using DataCenter.API.Middleware;
using DataCenter.Application;
using DataCenter.Infrastructure;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
builder.Services.AddApplication();
builder.Services.AddInfrastructure(builder.Configuration);

var app = builder.Build();

app.UseMiddleware<ExceptionMiddleware>();
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
app.UseHttpsRedirection();
app.MapControllers();
app.Run();
```

### `src/DataCenter.Application/Interfaces/IServerRepository.cs`

```csharp
using DataCenter.Domain.Entities;

namespace DataCenter.Application.Interfaces;

public interface IServerRepository
{
    Task<Server?> GetByIdAsync(Guid id, CancellationToken cancellationToken);
    Task<IReadOnlyList<Server>> GetAllAsync(CancellationToken cancellationToken);
    Task AddAsync(Server server, CancellationToken cancellationToken);
}
```

### `src/DataCenter.Application/Interfaces/ITestRepository.cs`

```csharp
using DataCenter.Domain.Entities;

namespace DataCenter.Application.Interfaces;

public interface ITestRepository
{
    Task<TestResult?> GetByIdAsync(Guid id, CancellationToken cancellationToken);
    Task AddAsync(TestResult result, CancellationToken cancellationToken);
    Task UpdateAsync(TestResult result, CancellationToken cancellationToken);
}
```

### `src/DataCenter.Application/Interfaces/IRedfishService.cs`

```csharp
namespace DataCenter.Application.Interfaces;

public interface IRedfishService
{
    Task<bool> IsServerReachableAsync(Guid serverId, CancellationToken cancellationToken);
}
```

### `src/DataCenter.Application/Interfaces/IMesService.cs`

```csharp
namespace DataCenter.Application.Interfaces;

public interface IMesService
{
    Task NotifyTestCompletedAsync(Guid serverId, Guid testId, CancellationToken cancellationToken);
}
```

### `src/DataCenter.Application/Features/Servers/Commands/`, `Queries/`, and `DTOs/`

These are folders, not files. Put server write requests in `Commands/`, read requests in `Queries/`, and API/application data-transfer records in `DTOs/`. For example, a future `GetServerDto.cs` could define `public sealed record GetServerDto(Guid Id, string Name, string Address);`. No individual files in these folders were listed in the supplied tree.

### `src/DataCenter.Application/Features/ServerTesting/StartServerTest.cs`

```csharp
using DataCenter.Application.Interfaces;
using DataCenter.Domain.Entities;

namespace DataCenter.Application.Features.ServerTesting;

public sealed class StartServerTestHandler(
    IServerRepository servers,
    ITestRepository tests,
    IRedfishService redfish)
{
    public async Task<Guid> HandleAsync(Guid serverId, CancellationToken cancellationToken)
    {
        var server = await servers.GetByIdAsync(serverId, cancellationToken)
            ?? throw new KeyNotFoundException($"Server {serverId} was not found.");

        var reachable = await redfish.IsServerReachableAsync(serverId, cancellationToken);
        var result = TestResult.Start(server.Id, reachable);
        await tests.AddAsync(result, cancellationToken);
        return result.Id;
    }
}
```

### `src/DataCenter.Application/Features/ServerTesting/GetTestResult.cs`

```csharp
using DataCenter.Application.Interfaces;
using DataCenter.Domain.Entities;

namespace DataCenter.Application.Features.ServerTesting;

public sealed class GetTestResultHandler(ITestRepository tests)
{
    public Task<TestResult?> HandleAsync(Guid testId, CancellationToken cancellationToken) =>
        tests.GetByIdAsync(testId, cancellationToken);
}
```

### `src/DataCenter.Application/Features/Firmware/` and `RackManagement/`

These are folders with no individual files named in the supplied tree. Add feature-specific commands, queries, handlers, validators, and DTOs here as those use cases are implemented.

### `src/DataCenter.Application/DependencyInjection.cs`

```csharp
using Microsoft.Extensions.DependencyInjection;

namespace DataCenter.Application;

public static class DependencyInjection
{
    public static IServiceCollection AddApplication(this IServiceCollection services)
    {
        services.AddScoped<Features.ServerTesting.StartServerTestHandler>();
        services.AddScoped<Features.ServerTesting.GetTestResultHandler>();
        return services;
    }
}
```

### `src/DataCenter.Domain/Entities/Server.cs`

```csharp
namespace DataCenter.Domain.Entities;

public sealed class Server
{
    public Guid Id { get; private set; } = Guid.NewGuid();
    public string Name { get; private set; }
    public string Address { get; private set; }
    public Guid? RackId { get; private set; }

    public Server(string name, string address, Guid? rackId = null)
    {
        if (string.IsNullOrWhiteSpace(name)) throw new ArgumentException("Name is required.");
        if (string.IsNullOrWhiteSpace(address)) throw new ArgumentException("Address is required.");
        Name = name;
        Address = address;
        RackId = rackId;
    }
}
```

### `src/DataCenter.Domain/Entities/Rack.cs`

```csharp
namespace DataCenter.Domain.Entities;

public sealed class Rack
{
    public Guid Id { get; private set; } = Guid.NewGuid();
    public string Name { get; private set; }
    public string Location { get; private set; }

    public Rack(string name, string location)
    {
        Name = string.IsNullOrWhiteSpace(name) ? throw new ArgumentException("Name is required.") : name;
        Location = string.IsNullOrWhiteSpace(location) ? throw new ArgumentException("Location is required.") : location;
    }
}
```

### `src/DataCenter.Domain/Entities/TestResult.cs`

```csharp
using DataCenter.Domain.Enums;

namespace DataCenter.Domain.Entities;

public sealed class TestResult
{
    public Guid Id { get; private set; } = Guid.NewGuid();
    public Guid ServerId { get; private set; }
    public TestStatus Status { get; private set; }
    public DateTimeOffset StartedAt { get; private set; } = DateTimeOffset.UtcNow;

    private TestResult(Guid serverId, TestStatus status) => (ServerId, Status) = (serverId, status);

    public static TestResult Start(Guid serverId, bool reachable) =>
        new(serverId, reachable ? TestStatus.Passed : TestStatus.Failed);
}
```

### `src/DataCenter.Domain/Entities/Firmware.cs`

```csharp
namespace DataCenter.Domain.Entities;

public sealed class Firmware
{
    public Guid Id { get; private set; } = Guid.NewGuid();
    public string Version { get; private set; }
    public string Model { get; private set; }

    public Firmware(string version, string model)
    {
        Version = string.IsNullOrWhiteSpace(version) ? throw new ArgumentException("Version is required.") : version;
        Model = string.IsNullOrWhiteSpace(model) ? throw new ArgumentException("Model is required.") : model;
    }
}
```

### `src/DataCenter.Domain/Enums/TestStatus.cs`

```csharp
namespace DataCenter.Domain.Enums;

public enum TestStatus
{
    Pending,
    Passed,
    Failed
}
```

### `src/DataCenter.Domain/Exceptions/` and `ValueObjects/`

These are currently empty folders in the supplied tree. Domain-specific exceptions (for example, `ServerNotFoundException.cs`) and immutable value objects (for example, `IpAddress.cs`) belong here when needed.

### `src/DataCenter.Infrastructure/Persistence/DbConnectionFactory.cs`

```csharp
using Microsoft.Data.SqlClient;

namespace DataCenter.Infrastructure.Persistence;

public sealed class DbConnectionFactory(string connectionString)
{
    public SqlConnection CreateConnection() => new(connectionString);
}
```

### `src/DataCenter.Infrastructure/Persistence/Repositories/ServerRepository.cs`

```csharp
using DataCenter.Application.Interfaces;
using DataCenter.Domain.Entities;
using DataCenter.Infrastructure.Persistence;

namespace DataCenter.Infrastructure.Persistence.Repositories;

// Replace the illustrative in-memory storage with database queries for production.
public sealed class ServerRepository(DbConnectionFactory connectionFactory) : IServerRepository
{
    private static readonly List<Server> Items = [];

    public Task<Server?> GetByIdAsync(Guid id, CancellationToken cancellationToken) =>
        Task.FromResult(Items.SingleOrDefault(server => server.Id == id));

    public Task<IReadOnlyList<Server>> GetAllAsync(CancellationToken cancellationToken) =>
        Task.FromResult<IReadOnlyList<Server>>(Items.ToArray());

    public Task AddAsync(Server server, CancellationToken cancellationToken)
    {
        Items.Add(server);
        return Task.CompletedTask;
    }
}
```

The `connectionFactory` parameter indicates where database access is provided; this minimal sample keeps storage in memory so the repository contract is easy to understand.

### `src/DataCenter.Infrastructure/Persistence/Repositories/TestRepository.cs`

```csharp
using DataCenter.Application.Interfaces;
using DataCenter.Domain.Entities;

namespace DataCenter.Infrastructure.Persistence.Repositories;

// Replace the illustrative in-memory storage with database queries for production.
public sealed class TestRepository : ITestRepository
{
    private readonly Dictionary<Guid, TestResult> _items = [];

    public Task<TestResult?> GetByIdAsync(Guid id, CancellationToken cancellationToken) =>
        Task.FromResult(_items.GetValueOrDefault(id));

    public Task AddAsync(TestResult result, CancellationToken cancellationToken)
    {
        _items.Add(result.Id, result);
        return Task.CompletedTask;
    }

    public Task UpdateAsync(TestResult result, CancellationToken cancellationToken)
    {
        _items[result.Id] = result;
        return Task.CompletedTask;
    }
}
```

### `src/DataCenter.Infrastructure/ExternalServices/RedfishService.cs`

```csharp
using DataCenter.Application.Interfaces;

namespace DataCenter.Infrastructure.ExternalServices;

public sealed class RedfishService(HttpClient httpClient) : IRedfishService
{
    public async Task<bool> IsServerReachableAsync(Guid serverId, CancellationToken cancellationToken)
    {
        using var response = await httpClient.GetAsync($"redfish/v1/Systems/{serverId}", cancellationToken);
        return response.IsSuccessStatusCode;
    }
}
```

Configure the real Redfish endpoint and authentication securely; server IDs may need to map to configured BMC addresses.

### `src/DataCenter.Infrastructure/ExternalServices/MesService.cs`

```csharp
using System.Net.Http.Json;
using DataCenter.Application.Interfaces;

namespace DataCenter.Infrastructure.ExternalServices;

public sealed class MesService(HttpClient httpClient) : IMesService
{
    public async Task NotifyTestCompletedAsync(Guid serverId, Guid testId, CancellationToken cancellationToken)
    {
        using var response = await httpClient.PostAsJsonAsync(
            "api/test-results", new { ServerId = serverId, TestId = testId }, cancellationToken);
        response.EnsureSuccessStatusCode();
    }
}
```

### `src/DataCenter.Infrastructure/DependencyInjection.cs`

```csharp
using DataCenter.Application.Interfaces;
using DataCenter.Infrastructure.ExternalServices;
using DataCenter.Infrastructure.Persistence;
using DataCenter.Infrastructure.Persistence.Repositories;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace DataCenter.Infrastructure;

public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services, IConfiguration configuration)
    {
        var connectionString = configuration.GetConnectionString("DataCenter")
            ?? throw new InvalidOperationException("Connection string 'DataCenter' is missing.");

        services.AddSingleton(new DbConnectionFactory(connectionString));
        services.AddScoped<IServerRepository, ServerRepository>();
        services.AddScoped<ITestRepository, TestRepository>();
        services.AddHttpClient<IRedfishService, RedfishService>(client =>
            client.BaseAddress = new Uri(configuration["Redfish:BaseUrl"]!));
        services.AddHttpClient<IMesService, MesService>(client =>
            client.BaseAddress = new Uri(configuration["Mes:BaseUrl"] ?? "https://mes.example.local/"));
        return services;
    }
}
```

### `tests/DataCenter.UnitTests/` and `tests/DataCenter.IntegrationTests/`

<img width="527" height="264" alt="image" src="https://github.com/user-attachments/assets/a1848b9f-7e08-442b-bbd4-13f0d3ecc716" />


These are project folders, not individual files in the supplied tree. Add test project files (`.csproj`) and test classes. Unit tests should cover domain rules and Application handlers with mocked interfaces; integration tests should exercise API and persistence behavior against test configuration.
