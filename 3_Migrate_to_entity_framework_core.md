# Migrate to EF Core - HOW TO

## Starting branch
[ef-core-quickstart](https://github.com/sopra-steria-norge/WhoOwesWhat-Net48/tree/ef-core-quickstart)

## Solution branch
[ef-core](https://github.com/sopra-steria-norge/WhoOwesWhat-Net48/tree/ef-core)

## NB! We've already done the following steps in order to be able to time box this workshop within a reasonable time frame: 
Created a new class library `WhoOwesWhat.Dataprovider.Net8`: 
- Added project references from this project to `WhoOwesWhat.Domain.DTO`
- Refactored all entities and added these to the folder `Entities`
- Added a new folder `PersonEntity` with a new class `PersonQuery.cs` and its associated interface (both in the same folder for simplicity). Also added the shell of the method `GetPersonByUsername` to be implemented by you
- Added a new folder `UserCredentialEntity` with a new class `UserCredentialQuery.cs` and its associated interface (both in the same folder for simplicity). Also added the shell of the method `GetUserCredential` to be implemented by you
- Added a new DbContext `WhoOwesWhatContext.cs` with method `OnModelCreating(...)` and basic funtionality that need to be updated in this workshop

Created a new class libray `WhoOwesWhat.Domain.Net8`:
- Added project references from this project to `WhoOwesWhat.Domain.DTO` and `WhoOwesWhat.DataProvider.Net8`
- Added the class `HashUtils.cs`
- Added the class `UserRepository.cs` and its associated interface (both in the same folder for simplicity). Also added the shell of the method `AuthenticateUser.cs` to be implemented by you

Refactored parts of `WhoOwesWhat.Service.Net8`:
- Added project references from this project to `WhoOwesWhat.Domain.Net8` and `WhoOwesWhat.DataProvider.Net8`
- Added required services to dependency container via `AddControllers(...)` in `Program.cs`
- Refactored dependecies for controllers and made the endpoints `Async()`

## Use the built-in `Task list` in Visual Studio to locate the TODO's in your solution
**NB: Requires Visual Studio Professional, if you're using the Community version you can search (Ctrl + F) for 'TODO:'.**

![Task list](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/ef-core-migration-images/task-list.png)

Double click on an element in the list to navigate to the corresponding TODO comment

## TODO: 1. Finish UserCredential Entity
The UserCredential entity contains the credentials of every given user in the solution, such as email, username, the hashed password and a relation to the Person entity.

Double click on TODO 1 in the task list.

Rewrite the class (entity) `UserCredential.cs` in the Entity folder so that they conform to .NET 8 and EF Core libraries.

Key aspects to keep in mind: 
1. In .NET 8 it is recommended to use the `required` parameter on the object properties, rather than using [DataAnnotations].
2. We'll switch to reference to the primary key instead of using a code defined entity id. See example in the code below where PersonId has been removed and replaced by

```csharp
[ForeignKey("PersonId")]
public required Person Person { get; set; }
```
and the virtual property for `Person` have been removed. (We are going to use eager loading instead)

3. Using .NET 8 syntax for namespaces, i.e. removing braces `{}` and replace with `;`.

**Tip: look at the other refactored classes and the old version from the .NET Framework 4.8.1 project for hints on how to implement the new entity.**

<details>
  <summary> Code snippet (spoiler!) </summary>
	
```csharp
using System.ComponentModel.DataAnnotations;
using System.ComponentModel.DataAnnotations.Schema;

namespace WhoOwesWhat.DataProvider.Net8.Entities;

// The Table annotation is needed since in the WhoOwesWhatContext the DbSet is called UserCredentials, but the table in the database is called UserCredential. This should ideally be avoided, but to simplify the modernisation process here it is done like this.
[Table("UserCredential")]
public class UserCredential
{
    [Key]
    public required int PersonId { get; set; }
    public required string Email { get; set; }
    public required string Username { get; set; }
    public required string PasswordHash { get; set; }

    [ForeignKey("PersonId")]
    public required Person Person { get; set; }
}
```
</details>

## TODO: 2. Finish WhoOwesWhatContext
Double click on TODO 2 in the task list.

Finish the .NET 8 `WhoOwesWhatContext` class. Start with the `DbSet`'s. We want to keep the relations and mapping attributes in the entities where this is applicable. Add missing relations or more complex relations later when needed. 

In short: 

1. Copy & paste the `DbSets<...> SomeProperty` from the .NET Framework 4.8.1 project into the WhoOwesWhatContext class.
2. Make sure that the `Payer` and `Consumer` classes are added to the `OnModelCreating` method so that these classes can reference the abstract class (entity) `Transaction`. 

<details>
  <summary> Code snippet (spoiler!) </summary>
	
```csharp
using Microsoft.EntityFrameworkCore;
using WhoOwesWhat.DataProvider.Net8.Entities;

namespace WhoOwesWhat.DataProvider.Net8;

public class WhoOwesWhatContext(DbContextOptions<WhoOwesWhatContext> options) : DbContext(options)
{

    public DbSet<Person> Persons { get; set; }
    public DbSet<UserCredential> UserCredentials { get; set; }
    public DbSet<Friend> Friends { get; set; }
    public DbSet<Friendrequest> Friendrequests { get; set; }
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
</details>

## TODO: 3. Inject DbContext	
Double click on TODO 3 in the task list.

Inject the .NET 8 DbContext we created in the previous task into the `Program.cs` file in `WhoOwesWhat.Service.Net8`

<details>
  <summary> Code snippet (spoiler!) </summary>
	
```csharp
builder.Services.AddDbContext<WhoOwesWhatContext>(options =>
{
	options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"));
});
```
This code snippet injects our `WhoOwesWhatContext` by using `builder.Services.AddDbContext<>()`. `options.UseSqlServer` tells it that we want to connect this DbContext to our SqlServer using the ConnectionString in appsettings.json 

</details>

## TODO: 4. Implement AuthenticateUser
Double click on TODO 4 in the task list.

Tip 1: Get inspiration from the old .NET Framework 4.8.1 project `WhoOwesWhat.Domain` and the method `AuthenticateUser` in `UserRepository.cs`.

Tip 2: We don't want to use the custom class Guard.cs, instead we want to use built-in ArgumentExceptions. 

Tip 3: Remember to include the `IUserCredentialQuery` in the constructor to be able to use the `GetUserCredential` method we need 

<details>
  <summary> Code snippet (spoiler!) </summary>

**UserRepository Class**
```csharp
public class UserRepository(IUserCredentialQuery userCredentialQuery,
                            IPersonQuery personQuery, 
                            IPersonCommand personCommand, 
                            IHashUtils hashUtils,
                            IUserCredentialCommand credentialCommand) : IUserRepository
{
    private readonly IPersonQuery _personQuery = personQuery;
    private readonly IPersonCommand _personCommand = personCommand;
    private readonly IUserCredentialCommand _credentialCommand = credentialCommand;
    private readonly IHashUtils _hashUtils = hashUtils;
    private readonly IUserCredentialQuery _userCredentialQuery = userCredentialQuery;
```


**AuthenticateUser method**
```csharp
public async Task<bool> AuthenticateUser(string username, string password)
{
  ArgumentException.ThrowIfNullOrEmpty(username);
  ArgumentException.ThrowIfNullOrEmpty(password);
        
  var userCredential = await _userCredentialQueGetUserCredential(username);

  // To avoid a timing attack, we should always compute the hash regardless of whether the user exists.
  var passwordHash = _hashUtils.GetHashString(password);

  if (userCredential is null)
      return false;

  var isAuthenticated = userCredential.PasswordHash == passwordHash;

  return isAuthenticated;
}
```

</details>

## TODO: 5. Implement GetUserCredential
Double click on TODO 5 in the task list.

Tip 1: Get inspiration from the old .NET Framework 4.8.1 project `WhoOwesWhat.DataProvider` and the method `GetUserCredential` in `UserCredentialEntity` folder and refactor.

Tip 2: Get the entity directly from your new `WhoOwesWhatContext.cs`. Other methods in the same class may have some valuable code snippets that can be used with slight modifications.

Tip 3: In order to get the complete entity with references to other entities, use `.Include` (Entity Framework Core syntax for eager loading).

Tip 4: Remember to map the entity to a UserCredential DTO before returning it.


<details>
  <summary> Code snippet (spoiler!) </summary>

```csharp
public async Task<UserCredential?> GetUserCredential(string username)
{
    var userCredential = await _DbContext.UserCredentials
        .Include(u => u.Person)
        .SingleOrDefaultAsync(a => a.Username == username);

    if (userCredential is null)
        return null;

    var person = new Person 
    {
        PersonGuid = userCredential.Person.PersonGuid,
        Displayname = userCredential.Person.Displayname,
        Mobile = userCredential.Person.Mobile,
        IsDeleted = userCredential.Person.IsDeleted
    };

    return new UserCredential
    {
        Email = userCredential.Email,
        Username = userCredential.Username,
        PasswordHash = userCredential.PasswordHash,
        Person = person
    };
}
```

</details>

## TODO: 6. Implement GetPersonByUsername
Double click on TODO 6 in the task list.

Tip 1: Get inspiration from the old .NET Framework 4.8.1 project `WhoOwesWhat.DataProvider` and the method `GetPersonByUsername` in `PersonQuery.cs` and refactor.

Tip 2: Get the entity directly from your new `WhoOwesWhatContext.cs`. Other methods in the same class may have some valuable code snippets that can be used with slight modifications.

Tip 3: In order to get the complete entity with references to other entities, use `.Include` (Entity Framework Core syntax).

Tip 4: Remember to map the entity to a UserCredential DTO before returning it.

<details>
  <summary> Code snippet (spoiler!) </summary>
	
```csharp
public async Task<Domain.DTO.Person?> GetPersonByUsername(string username)
{
    var userCredential = await _DbContext.UserCredentials
        .Include(uc => uc.Person)
        .SingleOrDefaultAsync(uc => uc.Username == username);

    if (userCredential is null || userCredential.Person is null)
        return null;

    return new Domain.DTO.Person 
    {
        PersonGuid = userCredential.Person.PersonGuid,
        Displayname = userCredential.Person.Displayname,
        Mobile = userCredential.Person.Mobile,
        IsDeleted = userCredential.Person.IsDeleted
    };
}
```

</details>

