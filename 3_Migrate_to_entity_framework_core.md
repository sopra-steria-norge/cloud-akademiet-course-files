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
