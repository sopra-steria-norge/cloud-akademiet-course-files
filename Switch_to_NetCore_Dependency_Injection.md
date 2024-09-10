# Migrate to native .NET Core Dependency Injection - HOW TO

### Create new project for .NET 8 (ASP.NET Core Web API)

...[Image - ASP net core api]

...[image - config project]

- Remove the auto-generated files related to WeatherController. 


### 1. Move Controllers
- Move the controllers from the `WhoOwesWhat.Service.Controller` project to the `Controllers` folder in the `WhoOwesWhat.Service.Net8` project.

### 2. Add Necessary Dependencies
- Add all the required project dependencies to ensure the controllers can run in the new project.

### 3. Remove Interfaces
- Remove the interfaces from the controllers (they are no longer needed).

### 4. Extend `ControllerBase`
- Ensure that all controllers extend from `ControllerBase`.

### 5. Add OpenAPI3 Decorators
- For OpenAPI3, add the necessary decorators to the controller classes, such as:
  ```csharp
  [ApiController]
  [Route("controller")]
  
### 6. Add `HttpPost` and `HttpGet` Decorators
- Add `HttpPost` and `HttpGet` decorators to the relevant methods in the controllers. For example:
  ```csharp
  [HttpPost("CreateUser")]

### 7. Configure Dependency Injection for Repositories
- Add Dependency Injection for required repositories in the `Program.cs` file, for example:
  ```csharp
  builder.Services.AddScoped<IUserRepository, UserRepository>();
- You may need to add additional project dependencies.
- Build and run the project regularly to identify missing dependencies. 
- Use the error messages to determine which dependencies need to be added.

### 8. Install Entity Framework Dependency
- Install the existing version of Entity Framework without upgrading. We will upgrade / migrate to `Entity Framework Core` later in the course.

### 9. Install Log4Net Dependency
- Install the `Microsoft.Extensions.Logging.Log4Net.AspNetCore` package (rest assured, we will replace Log4Net later).

### 10. Copy `log4net.config`
- Copy the `log4net.config` file to the new project, `WhoOwesWhat.Service.Net8`.

### 11. Configure Log4Net for .NET 8
- To enable Log4Net to work with .NET 8, add the following lines to `Program.cs`:
  ```csharp
  XmlConfigurator.Configure(new FileInfo("log4net.config"));
  builder.Logging.AddLog4Net();
  builder.Services.AddSingleton<log4net.ILog>(provider => log4net.LogManager.GetLogger(typeof(Program)));


Må også installere dependency for Entity Framework. Legger til allerede eksisterende versjon istedenfor å oppgradere. (Kan oppgradere denne senere)
Må installere Microsoft.Extensions.Logging.Log4Net.AspNetCore. (Vi erstatter log4net senere)
Kopier over log4net.config
For å få log4net til å fungere med dotnet 8 må man legge til disse linjene Program.cs XmlConfigurator.Configure(new FileInfo("log4net.config")); builder.Logging.AddLog4Net(); builder.Services.AddSingleton<log4net.ILog>(provider => log4net.LogManager.GetLogger(typeof(Program)));
Fjern gammelt Service og Controller prosjekt.


