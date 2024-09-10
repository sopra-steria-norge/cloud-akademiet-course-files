# Migrate to native .NET Core Dependency Injection - HOW TO

### Create new project for .NET 8 (ASP.NET Core Web API)

![ASP.NET Core Web API](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/dependency-injection-images/ASP.NETCoreWebAPI.png)
![ASP.NET Core Web API Configuration](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/dependency-injection-images/ASP.NETCoreWebAPI_configuration.png)

- Remove the auto-generated files related to WeatherController. 

### Move Controllers
Move the controllers from the `WhoOwesWhat.Service.Controller` project to the `Controllers` folder in the `WhoOwesWhat.Service.Net8` project.

### Add Necessary Dependencies
Add all the required project dependencies to ensure the controllers can run in the new project.

### Remove Interfaces
Remove the interfaces from the controllers (they are no longer needed).

### Extend `ControllerBase`
Ensure that all controllers extend from `ControllerBase`.

### Add OpenAPI3 Decorators
- For OpenAPI3, add the necessary decorators to the controller classes, such as:
  ```csharp
  [ApiController]
  [Route("controller")]
  
### Add `HttpPost` and `HttpGet` Decorators
- Add `HttpPost` and `HttpGet` decorators to the relevant methods in the controllers. For example:
  ```csharp
  [HttpPost("CreateUser")]

### Configure Dependency Injection for Repositories
- Add Dependency Injection for required repositories in the `Program.cs` file, for example:
  ```csharp
  builder.Services.AddScoped<IUserRepository, UserRepository>();
You may need to add additional project dependencies.
Build and run the project regularly to identify missing dependencies. 
Use the error messages to determine which dependencies need to be added.

### Install Entity Framework Dependency
Install the existing version of Entity Framework without upgrading. We will upgrade / migrate to `Entity Framework Core` later in the course.

### Install Log4Net Dependency
Install the `Microsoft.Extensions.Logging.Log4Net.AspNetCore` package (rest assured, we will replace Log4Net later).

### Copy `log4net.config`
Copy the `log4net.config` file to the new project, `WhoOwesWhat.Service.Net8`.

### Configure Log4Net for .NET 8
- To enable Log4Net to work with .NET 8, add the following lines to `Program.cs`:
  ```csharp
  XmlConfigurator.Configure(new FileInfo("log4net.config"));
  builder.Logging.AddLog4Net();
  builder.Services.AddSingleton<log4net.ILog>(provider => log4net.LogManager.GetLogger(typeof(Program)));
