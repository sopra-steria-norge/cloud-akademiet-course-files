# Migrate to native .NET Core Dependency Injection - HOW TO

### Create new project for .NET 8 (ASP.NET Core Web API)

![ASP.NET Core Web API](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/dependency-injection-images/ASP.NETCoreWebAPI.png)
![ASP.NET Core Web API Configuration](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/dependency-injection-images/ASP.NETCoreWebAPI_configuration.png)

Remove the auto-generated files related to WeatherController. 

### Move Controllers
Move `UserController.cs` from the `WhoOwesWhat.Service.Controller` project to the `Controllers` folder in the `WhoOwesWhat.Service.Net8` project. Due to time constraints, we will focus on just this controller.

### Add Necessary Dependencies
Add all the required project dependencies to ensure the controllers can run in the new project. Check the using statements at the top of `UserController.cs` to see what projects are needed.

![Project dependencies](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/dependency-injection-images/project-dependencies.png)

### Sync Namespaces
Right click the project file and click `Sync Namespaces` to automatically update the namespaces.

![Sync namespaces](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/dependency-injection-images/sync-namespaces.png) 

### Remove Interface
Remove the interface from the controller (they are no longer needed).

### Extend `ControllerBase`
Ensure that the controller extend from `ControllerBase`.

### Add OpenAPI3 Decorators
For OpenAPI3, add the necessary decorators to the controller classes, like so:
```csharp
[ApiController]
[Route("controller")]
```

### Add `HttpPost` and `HttpGet` Decorators
Add `HttpPost` and `HttpGet` decorators to the relevant methods in the controllers. For example:
```csharp
[HttpPost(nameof(CreateUser))]
```

### Configure Dependency Injection for Repositories
Add Dependency Injection for required repositories in the `Program.cs` file, for example:
```csharp
builder.Services.AddScoped<IUserRepository, UserRepository>();
```

You may need to add additional project dependencies.
Build and run the project regularly to identify missing dependencies. 
Use the error messages to determine which dependencies need to be added.

Here is a finished class with a ServiceCollection method to add all necessary services

```csharp
public static class ServiceCollectionExtensions
{
    public static IServiceCollection AddServices(this IServiceCollection services)
    {
        return services
        .AddScoped<IUserRepositoryLogic, UserRepositoryLogic>()
        .AddScoped<IUserRepository, UserRepository>()
        .AddScoped<IUserCredentialCommand, UserCredentialCommand>()
        .AddScoped<IUserCredentialDataProviderLogic, UserCredentialDataProviderLogic>()
        .AddScoped<IUserCredentialContext, UserCredentialContext>()
        .AddScoped<IUserCredentialQuery, UserCredentialQuery>()
        .AddScoped<IPersonRepositoryLogic, PersonRepositoryLogic>()
        .AddScoped<IPersonCommand, PersonCommand>()
        .AddScoped<IPersonQuery, PersonQuery>()
        .AddScoped<IPersonDataProviderLogic, PersonDataProviderLogic>()
        .AddScoped<IPersonContext, PersonContext>()
        .AddScoped<IHashUtils, HashUtils>()
        .AddScoped<IWhoOwesWhatContext, WhoOwesWhatContext>()

        .AddScoped<ISyncGroupQuery, SyncGroupQuery>()
        .AddScoped<ISyncGroupCommand, SyncGroupCommand>()
        .AddScoped<ISyncGroupsCommand, SyncGroupsCommand>()
        .AddScoped<ISyncControllerLogic, SyncControllerLogic>()
        .AddScoped<IGroupCommand, GroupCommand>()
        .AddScoped<IGroupDataProviderLogic, GroupDataProviderLogic>()
        .AddScoped<IGroupQuery, GroupQuery>()
        .AddScoped<IGroupContext, GroupContext>()
        .AddScoped<ITransactionCommand, TransactionCommand>()
        .AddScoped<ITransactionDataProviderLogic, TransactionDataProviderLogic>()
        .AddScoped<ITransactionContext, TransactionContext>()
        .AddScoped<ITransactionQuery, TransactionQuery>()
        .AddScoped<IPostContext, PostContext>()
        .AddScoped<IPostCommand, PostCommand>()
        .AddScoped<IPostQuery, PostQuery>()
        .AddScoped<IPostDataProviderLogic, PostDataProviderLogic>()
        .AddScoped<ISyncPostCommand, SyncPostCommand>()
        .AddScoped<ISyncPostsCommand, SyncPostsCommand>()
        .AddScoped<ISyncPostQuery, SyncPostQuery>()
        .AddScoped<IDeleteFriendCommand, DeleteFriendCommand>()
        .AddScoped<IFriendRepositoryLogic, FriendRepositoryLogic>()
        .AddScoped<IFriendQuery, FriendQuery>()
        .AddScoped<IFriendDataProviderLogic, FriendDataProviderLogic>()
        .AddScoped<IFriendCommand, FriendCommand>()
        .AddScoped<IFriendRepository, FriendRepository>()

        .AddScoped<IAcceptFriendrequestLogic, AcceptFriendrequestLogic>()
        .AddScoped<IFriendrequestQuery, FriendrequestQuery>()
        .AddScoped<IFriendrequestCommand, FriendrequestCommand>()
        .AddScoped<IFriendrequestRepository, FriendrequestRepository>();
    }
}
```

Add it to the `Program.cs` file like this:
```csharp
builder.Services.AddServices();
```

### Install Entity Framework Dependency
Install the existing version of Entity Framework without upgrading. We will upgrade / migrate to `Entity Framework Core` later in the course.
Easiest way is when you have added the `WhoOwesWhatContext` and get a red line underneath, right click it and select `Quick Actions and Refactorings...` or `CTRL + .` when the cursor is on the line.
Then select `Install package 'Entity Framework'` and `Use local version '6.5.1'`.

![Install Entity Framework](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/dependency-injection-images/install-entity-framework.png) 

### Install Log4Net Dependency
Install the `Microsoft.Extensions.Logging.Log4Net.AspNetCore` package (rest assured, we will replace Log4Net later). Right click the project and select `Manage NuGet Packages...`.

![Manage NuGet Packages](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/dependency-injection-images/manage-nuget-packages.png) 
![Install Log4Net](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/dependency-injection-images/install-log4net.png) 


### Copy `log4net.config`
Copy the `log4net.config` file to the new project, `WhoOwesWhat.Service.Net8`.

### Configure Log4Net for .NET 8
To enable Log4Net to work with .NET 8, add the following lines to `Program.cs`:
```csharp
XmlConfigurator.Configure(new FileInfo("log4net.config"));
builder.Logging.AddLog4Net();
builder.Services.AddSingleton<log4net.ILog>(provider => log4net.LogManager.GetLogger(typeof(Program)));
```

### Swagger
Due to a Swagger error we need to add the following code snippet in `Program.cs`:
```csharp
// Add this to avoid errors in the Swagger UI
// https://github.com/swagger-api/swagger-ui/issues/7911
builder.Services.AddSwaggerGen(options =>
{
  options.CustomSchemaIds(s => s.FullName?.Replace("+", "."));
});
```

### Run the project
Set the new .NET 8 Service project as the startup project and run it. If it does not take you to the Swagger web page, add `/swagger` to the url. Try to call one of the endpoints in `UserController`, you should get a `SqlError`.
