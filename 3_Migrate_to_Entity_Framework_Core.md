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
