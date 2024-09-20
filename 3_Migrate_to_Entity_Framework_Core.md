# Migrate to EF Core

## Create new DataProvider Project
Create a new .NET 8 Class Library project and name it `WhoOwesWhat.DataProvider.Net8`

## Add Entity Framework Core
Add the Nuget package `Microsoft.EntityFrameworkCore.SqlServer`
![Add EF Core SqlServer Nuget package](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/ef-core-migration-images/ef-core-nuget-package.png)

## Move entities
Move the Entity folder from `WhoOwesWhat.DataProvider` over to the newly created `WhoOwesWhat.DataProvider.Net8` project

Run `Sync namespaces` in Visual Studio.
IMAGE-sync namespaces


## Rewrite entities to .NET 8
Rewrite the classes in the Entity folder so that they conform to .NET 8 and EF Core libraries.

key aspects to keep in mind: 
1. In .NET 8 it is recommended to use the `required` parameter on the object properties, rather than using [DataAnnotations].
2. We'll switch to reference to the primary key instead of using a code defined entity id. See example in the code below where PersonId has been removed and replaced by

```csharp
[ForeignKey("PersonId")]
public required Person Person { get; set; }
```
and the virtual property for `Person` have been removed.
 
```csharp
[Table("UserCredential")]
[PrimaryKey("PersonId")]
public class UserCredential
{
    public required string Email { get; set; }
    public required string Username { get; set; }
    public required string PasswordHash { get; set; }

    [ForeignKey("PersonId")]
    public required Person Person { get; set; }
}
```



## Create the DbContext
Create the .NET 8 `WhoOwesWhatContext` class. Start with the `DbSet`'s. Keep relations to Mapping Attributes in the entities where that is applicable. Add missing relations or more complex relations later when needed.  

## Inject the DbContext
Inject the newly created .NET 8 DbContext to the `Program.cs` file in `WhoOwesWhat.Service.Net8`

```csharp
builder.Services.AddDbContext<WhoOwesWhatContext>(options =>
{
	options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"));
});
```

## Rewrite code to .NET 8
Start in the `UserController` and focus on `AuthenticateUser`. Follow code into repositories, queries and commands and rewrite. Create new class library projects where necessary. 

### Dependency Injection
Remember to inject new repositories and queries into `Program.cs` in `WhoOwesWhat.Service.Net8`

```csharp
services.AddScoped<IUserRepository, UserRepository>()
```

### Guard clauses
Replace custom Guard clauses with built in `ArgumentException.ThrowIfNullOrEmpty(exampleString)` where applicable.

### LINQ
Rewrite queries to the database to use the new DbSets and LINQ Queries
```csharp
var userCredential = await _DbContext.UserCredentials.Include(u => u.Person).SingleOrDefaultAsync(a => a.Username == username);
```

### Async Controllers
Make the endpoints in the controllers async to improve performance and scalability.
```csharp
public async Task<AuthenticateUserResponse> AuthenticateUser(AuthenticateUserRequest request)
```
