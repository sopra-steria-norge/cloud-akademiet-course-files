# Migrate to EF Core - HOW TO

## Starting branch
[ef-core-quickstart](https://github.com/sopra-steria-norge/WhoOwesWhat-Net48/tree/ef-core-quickstart)

## NB! We've already done the following steps in order to be able to time box this workshop within a reasonable time frame: 
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

## Use the built-in `Task list` in Visual Studio to locate the TODO's in your solution

![Task list](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/ef-core-migration-images/task-list.png)

## TODO: 1. Finish UserCredential Entity
Rewrite the class (entity) `UserCredential.cs` in the Entity folder so that they conform to .NET 8 and EF Core libraries.

Key aspects to keep in mind: 
1. In .NET 8 it is recommended to use the `required` parameter on the object properties, rather than using [DataAnnotations].
2. We'll switch to reference to the primary key instead of using a code defined entity id. See example in the code below where PersonId has been removed and replaced by

```csharp
[ForeignKey("PersonId")]
public required Person Person { get; set; }
```
and the virtual property for `Person` have been removed.

3. Using .NET 8 syntax for namespaces, i.e. removing braces `{}` and replace with `;`.

Tip: look at the other refactored classes and the old version from the .NET Framework 4.8.1 project.

## TODO: 2. Finish WhoOwesWhatContext	
Finish the .NET 8 `WhoOwesWhatContext` class. Start with the `DbSet`'s. We want to keep the relations and mapping attributes in the entities where this is applicable. Add missing relations or more complex relations later when needed. 

In short: 

1. Copy & paste the `DbSets<...> SomeProperty` from the .NET Framework 4.8.1 project.
2. Make sure that the `Payer` and `Consumer` classes are added to the `OnModelCreating` method so that these classes can reference the abstract class (entity) `Transaction`. 

## TODO: 3. Inject DbContext	
Inject the newly created .NET 8 DbContext to the `Program.cs` file in `WhoOwesWhat.Service.Net8`

<details>
  <summary> Code snippet (spoiler!) </summary>
	
```csharp
builder.Services.AddDbContext<WhoOwesWhatContext>(options =>
{
	options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection"));
});
```
</details>

## TODO: 4. Implement AuthenticateUser	
Tip 1: Get inspiration from the old .NET Framework 4.8.1 project and the method `AuthenticateUser`.

Tip 2: We don't want to use the custom class Guard.cs, instead we want to use built-in ArgumentExceptions. 

<details>
  <summary> Code snippet (spoiler!) </summary>
	
Replace custom Guard clauses with built in ArgumentException where applicable:

```csharp
ArgumentException.ThrowIfNullOrEmpty(variableToCheck)
```

</details>

## TODO: 5. Implement GetUserCredential	
Tip 1: Get inspiration from the old .NET Framework 4.8.1 project and the method `GetUserCredential` and refactor.

Tip 2: Get the entities directly from your new `WhoOwesWhatContext.cs`. Other classes may have some valuable code snippets that can be used with slight modifications.

Tip 3: In order to get the complete entity with references to other entities, use `.Include` (Entity Framework Core syntax).


<details>
  <summary> Code snippet (spoiler!) </summary>
	
Use LINQ queries to get data:

```csharp
var userCredential = await _DbContext.UserCredentials.Include(u => u.Person).SingleOrDefaultAsync(a => a.Username == username);
```

</details>

## TODO: 6. Implement GetPersonByUsername 
Tip 1: Get inspiration from the old .NET Framework 4.8.1 project and the method `GetPersonByUsername` and refactor.

Tip 2: Get the entities directly from your new `WhoOwesWhatContext.cs`. Other classes may have some valuable code snippets that can be used with slight modifications.

Tip 3: In order to get the complete entity with references to other entities, use `.Include` (Entity Framework Core syntax).


<details>
  <summary> Code snippet (spoiler!) </summary>
	
Use LINQ queries to get data:

```csharp
var credential = await _DbContext.UserCredentials
                                        .Include(u => u.Person)
                                        .SingleOrDefaultAsync(a => a.Username == username);
```

</details>

