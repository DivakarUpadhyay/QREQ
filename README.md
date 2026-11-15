<img width="546" height="546" alt="image" src="https://github.com/user-attachments/assets/863d3708-77a3-42a1-a355-eedb1ee22b66" />

<img width="602" height="314" alt="image" src="https://github.com/user-attachments/assets/4cb57618-b4f3-4037-b665-86ea981229d9" />

# Project Structure

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
│   │
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
│   │
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
│   │
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
│
└── tests/
    ├── DataCenter.UnitTests/
    └── DataCenter.IntegrationTests/
```

This keeps the project structure clean and easy to read.
