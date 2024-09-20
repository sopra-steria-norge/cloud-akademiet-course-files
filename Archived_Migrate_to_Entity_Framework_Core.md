# Migrate to EF Core

## We've already done the following steps in order to be able to time box this workshop within a reasonable time frame: 
Created a new class library `WhoOwesWhat.Dataprovider.Net8`: 
- Added project references from this project to `WhoOwesWhat.Domain.DTO`
- Refactored all entities and added these to the folder `Entities`
- Added a new folder `PersonEntity` with a new class `PersonQuery.cs` and its associated interface (both in the same folder for simplicity). Also added the method `GetPersonByUsername` to this class
- Added a new folder `UserCredentialEntity` with a new class `UserCredentialQuery.cs` and its associated interface (both in the same folder for simplicity). Also added the method `GetUserCredential` to this class
- Added a new DbContext `WhoOwesWhatContext.cs` with method `OnModelCreating(...)` and basic funtionality that need to be updated in this workshop

Created a new class libray `WhoOwesWhat.Domain.Net8`:
- Added project references from this project to `WhoOwesWhat.Domain.DTO` and `WhoOwesWhat.DataProvider.Net8`
- Added the class `HashUtils.cs`
- Added the class `UserRepository.cs` and its associated interface (both in the same folder for simplicity). Also added the method `AuthenticateUser.cs`


Refactored parts of `WhoOwesWhat.Service.Net8`:
- Added project references from this project to `WhoOwesWhat.Domain.Net8` and `WhoOwesWhat.DataProvider.Net8`
- Added required services to dependency container via `AddControllers(...)` in `Program.cs`
- Refactored dependecies for controllers and made the endpoints `Async()`


## Create new DataProvider Project
Create a new .NET 8 Class Library project and name it `WhoOwesWhat.DataProvider.Net8`

## Add Entity Framework Core
Add the Nuget package `Microsoft.EntityFrameworkCore.SqlServer`
![Add EF Core SqlServer Nuget package](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/ef-core-migration-images/ef-core-nuget-package.png)

## Move entities
Move the Entity folder from `WhoOwesWhat.DataProvider` over to the newly created `WhoOwesWhat.DataProvider.Net8` project

Run `Sync namespaces` in Visual Studio.

![Sync name spaces](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/ef-core-migration-images/entity_sync_namespaces.png)


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

3. Using .NET 8 syntax for namespaces, i.e. removing braces `{}` and replace with `;`.

Refactored entities:

```csharp
using System.ComponentModel.DataAnnotations.Schema;

namespace WhoOwesWhat.DataProvider.Net8.Entities;

[Table("Error")]
public class Error
{
    public int ErrorId { get; set; }

    public DateTime Created { get; set; }

    public string? Message { get; set; }

    public string? ErrorJson { get; set; }
}
```

```csharp
using System.ComponentModel.DataAnnotations.Schema;

namespace WhoOwesWhat.DataProvider.Net8.Entities;

[Table("Friend")]
public class Friend
{
    public int FriendId { get; set; }

    public bool IsDeleted { get; set; }

    [ForeignKey("Person_PersonId")]
    public virtual required Person Person { get; set; }

    [ForeignKey("Owner_PersonId")]
    public virtual required Person Owner { get; set; }
}
```

```csharp
using System.ComponentModel.DataAnnotations.Schema;

namespace WhoOwesWhat.DataProvider.Net8.Entities;

[Table("Friendrequest")]
public class Friendrequest
{
    public int FriendrequestId { get; set; }

    [ForeignKey("RequesterPerson_PersonId")]
    public virtual required Person PersonRequested { get; set; }

    [ForeignKey("PersonRequested_PersonId")]
    public virtual required Person RequesterPerson { get; set; }
}
```

```csharp
using System.ComponentModel.DataAnnotations.Schema;

namespace WhoOwesWhat.DataProvider.Net8.Entities;

[Table("Group")]
public class Group
{
    public int GroupId { get; set; }
    public Guid GroupGuid { get; set; }
    public string? Name { get; set; }
    public bool IsDeleted { get; set; }
    public DateTime VersionUpdated { get; set; }


    public virtual ICollection<Post>? Posts { get; set; }

    public virtual Person? CreatedBy { get; set; }
}
```

```csharp
using System.ComponentModel.DataAnnotations.Schema;

namespace WhoOwesWhat.DataProvider.Net8.Entities;

[Table("Person")]
public class Person
{
    public int PersonId { get; set; }

    public required Guid PersonGuid { get; set; }

    public required string Displayname { get; set; }

    public string? Mobile { get; set; }
    public bool IsDeleted { get; set; }
}
```

```csharp
using System.ComponentModel.DataAnnotations.Schema;

namespace WhoOwesWhat.DataProvider.Net8.Entities;

[Table("Post")]
public class Post
{ 
    public int PostId { get; set; }

    public Guid PostGuid { get; set; }
    public DateTime PurchaseDate { get; set; }
    public string? Description { get; set; }
    public string? TotalCost { get; set; }
    public string? Iso4217CurrencyCode { get; set; }
    public int Version { get; set; }
    public DateTime VersionUpdated { get; set; }
    public bool IsDeleted { get; set; }
    public string? Comment { get; set; }

    public DateTime LastUpdated { get; set; }
    public DateTime Created { get; set; }

    public virtual Group? Group { get; set; }
    public virtual Person? LastUpdatedBy { get; set; }
    public virtual Person? CreatedBy { get; set; }

    public virtual ICollection<Transaction>? Transactions { get; set; }
}
```

```csharp
using System.ComponentModel.DataAnnotations.Schema;

namespace WhoOwesWhat.DataProvider.Net8.Entities;

[Table("Transaction")]
public abstract class Transaction
{
    public int TransactionId { get; set; }
    public bool IsAmountSetManually { get; set; }
    public string? Amount { get; set; }

    public virtual Post? Post { get; set; }
    public virtual Person? Person { get; set; }
}

public class Payer : Transaction
{
}    

public class Consumer : Transaction
{
}
```


```csharp
using Microsoft.EntityFrameworkCore;
using System.ComponentModel.DataAnnotations.Schema;

namespace WhoOwesWhat.DataProvider.Net8.Entities;

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



## Create a new DbContext for your .NET 8 class library
Create the .NET 8 `WhoOwesWhatContext` class. Start with the `DbSet`'s. Keep relations to Mapping Attributes in the entities where that is applicable. Add missing relations or more complex relations later when needed. 

1. Create WhoOwesWhatContext.cs in the new .Net8 project and copy & paste the DbSets from the .NET Framework 4.8.1 project.
2. Make sure to add `Payer` and `Consumer` to the `OnModelBuilder` method so that these classes can reference the abstract class (entity) `Transaction`.

Your new Context should look something like this: 
```csharp
using Microsoft.EntityFrameworkCore;
using WhoOwesWhat.DataProvider.Net8.Entities;

namespace WhoOwesWhat.DataProvider.Net8;

public class WhoOwesWhatContext(DbContextOptions<WhoOwesWhatContext> options) : DbContext(options)
{
    public DbSet<UserCredential> UserCredentials { get; set; }
    public DbSet<Person> People { get; set; }
    public DbSet<Friend> Friends { get; set; }
    public DbSet<Friendrequest> FriendRequests { get; set; }
    public DbSet<Error> Errors { get; set; }
    public DbSet<Post> Posts { get; set; }
    public DbSet<Group> Groups { get; set; }
    public DbSet<Transaction> Transactions { get; set; }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        builder.Entity<Payer>();
        builder.Entity<Consumer>();

        base.OnModelCreating(builder);
    }
}
```

## Install required Nuget-packages for Entity Framework Core and delete the old dependency to Entity Framework 6.5.1 to your project `WhoOwesWhat.Service.Net8`

Install: 

`Microsoft.ENtityFrameworkCore` and `Microsoft.EntityFrameworkCore.SqlServer`

Remove: 

`EntityFramework (6.5.1`

## Add project reference to your new class library `WhoOwesWhat.DataProvider.Net8`
Make sure your Web API `WhoOwesWhat.Service.Net8` references your new class library. You can now `Unload` the old class library `WhoOwesWhat.DataProvider`.

!IMAGE!

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

1. Create new class library `WhoOwesWhat.Domain.Net8`.
2. Copy & paste `AuthenticateUser` method code from the old `Domain` project in the class `UserRepository`.
3. Add Project reference from `WhoOwesWhat.Domain.Net8` to `WhoOwesWhat.Domain.DTO` to use the already defined data transfer objects (DTO's) / classes.
4. Add folder `UserCredentialEntity` to the class library `WhoOwesWhat.DataProvider.Net8` and add a new class `UserCredentialQuery`.
5. Copy & paste `GetUserCrential` method code from the old `Dataprovider` project in the class `UserCredentialQuery`.
6. Add Project reference from `WhoOwesWhat.DataProvider.Net8` to `WhoOwesWhat.Domain.DTO` to use the already defined data transfer objects (DTO's) / classes `UserCredential`.
7. Inject your new `WhoOwesWhatContext` in your class `UserCredentialQuery`. 

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
