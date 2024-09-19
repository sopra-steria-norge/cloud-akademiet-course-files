# Add migration, update to Entity Framework 6 and seed a new WhoOwesWhat database - HOW TO

## Restore nuget packages and run project

Run Restore & Build solution in Visual Studio.

Make sure that `WhoOwesWhat.Service.Net8` is `Set as start up project` and run the project with the `https` profile (F5).

On the first run you should get a `System.IO.FileNotFoundException` that refers to the `SqlClient`.

![EF Core not working from CLI for the project](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-migration-images/System.IO.FileNotFoundException_on_first_run.png)

Make sure that this works before continuing.

## `cd` into the correct folder from your your command line interface (CLI) in Visual Studio

--> `cd WhoOwesWhat.DataProvider`

## Make Entity Framework work
1. Install dotnet ef as global tool with the following command: `dotnet tool install --global dotnet-ef` in your CLI.
2. Make sure that a supported version of Entity Framework is used, and that you can use this via your CLI. In this case we have a .NET Framework project, which means that we cannot use Entity Framework Core until we have migrated the service to .NET Core (i.e. .NET 8). Before we reach that point, we will use the latest supported version of **Entity Framework 6**.

![EF Core not working from CLI for the project](https://github.com/sopra-steria-norge/cload-akademiet-course-files/blob/main/images/db-migration-images/ef-core-does-not-work.png)

3. Install latest supported version of **Entity Framework 6.5.1** for the project WhoOwesWhat.DataProvider
![Manage nuget packages](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-migration-images/Update_EntityFrameworkNugetPackage_EF6.5.1.png)
3. Use **Entity Framework 6** from Package Manager in Visual Studio 2022 with the relevant commands as stated below (using `Package manager` instead of `Powershell` since our project file (.csproj) is not an SDK-style project) 

## Add connection string directly to the `WhoOwesWhatContext` contructor
In order to be able to seed the WhoOwesWhat database to your SQL Server the correct connection string must be added. This should look somthing like this: 

        public WhoOwesWhatContext()
            : base("Data Source=.\\SQLEXPRESS;Integrated Security=True;Database=WhoOwesWhat;Connect Timeout=30;Encrypt=False;")
        {
            this.Configuration.LazyLoadingEnabled = false;
            this.Configuration.ProxyCreationEnabled = false;
            this.Database.Log = s => System.Diagnostics.Debug.WriteLine(s);
            Database.SetInitializer(new MigrateDatabaseToLatestVersion<WhoOwesWhatContext, MigrationConfigurations>());

        }

## Make sure to enable migrations
Make sure that you have selected the correct project as `Default project` in `Package Manager`. 

Set `Default project` to `WhoOwesWhat.DataProvider`. This must be done in order to make the Entity Framework commands find the DbContext `WhoOwesWhatContext`.
![Set correct default project](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-migration-images/Set_default_project_Package_Manager.png)

Continue in `Package Manager` and run the command:
- Enable-Migrations

This should result in the creation of the folder `Migrations` in the `WhoOwesWhat.DataProvider` project.

![Created new folder Migrations and configuration class](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-migration-images/Enable_migrations_and_create_Migration_folder.png)

Since we already have an existing configuration file `MigrationConfigurations.cs` we want to use this class instead of the auto-generated class. 

Delete the file you generated `Configuration.cs` and move (cut & paste) the `MigrationConfigurations.cs` file into the new folder 'Migrations'.

## Add a initial migration

In `Package Manager` run the command:

- Add-Migration "InitialMigration"

This should result in the creation of a new class `..._InitialMigration` in the folder `Migrations` in the `WhoOwesWhat.DataProvider` project.

## Seed the database
The reason we are doing this is that we want to seed the database for the `WhoOwesWhat` service before migrating to .NET 8, so that we can get an overview of the existing relations and data structure.

Continue in `Package Manager` and run the command to seed the database (or update if already created): 
- Update-Database -Verbose

## Verify that the database was seeded into your local db (SqlExpress is already set up)
Enter Microsoft SQL Server Management Studio (SSMS) and log into SQLEXPRESS. Remeber to tick the `Trust server certificate` box.

![SSMS log-in](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-migration-images/SSMS_login.png)

You should now have the seeded database `WhoOwesWhat` visible in the dropdown menu. 

![Successfully seeded database](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-migration-images/SSMS_successfully_seeded_WhoOwesWhat_database.png)

## Add connection string to `appsettings.json` and WhoOwesWhatContext to the Dependency Injection (DI) container in `WhoOwesWhat.Service.Net8`
Add the connection string section to `appsettings.json`: 

  `"ConnectionStrings": {
    "DefaultConnection": "Data Source=.\\SQLEXPRESS;Integrated Security=True;Database=WhoOwesWhat;Connect Timeout=30;Encrypt=False;"
  }`

Update DI container: 
1. In the class `ServiceCollectionExtensions` refactor the line regarding `WhoOwesWhatContext` with:
   
`.AddScoped<IWhoOwesWhatContext, WhoOwesWhatContext>(_ =>
        new WhoOwesWhatContext(connectionString))`

3. Add new input parameter:
   
    `public static IServiceCollection AddServices(this IServiceCollection services, string connectionString)`

4. In `Program.cs` refactor to fit the added input parameter:
   
   `builder.Services.AddServices(builder.Configuration.GetConnectionString("DefaultConnection"));`

Update the contructor in the class `WhoOwesWhatContext`:
Refactor the contructor and add a new default / empty contructor: 
       
        public WhoOwesWhatContext()
        {
                
        }              
        
        public WhoOwesWhatContext(string connectionString)
            : base(connectionString)
        {
            this.Configuration.LazyLoadingEnabled = false;
            this.Configuration.ProxyCreationEnabled = false;
            this.Database.Log = s => System.Diagnostics.Debug.WriteLine(s);
            Database.SetInitializer(new MigrateDatabaseToLatestVersion<WhoOwesWhatContext, MigrationConfigurations>());

        }

## Verify that you can send a POST request to the new database `WhoOWesWhat`
Run the service (F5) and try to post a new user via the `POST /User/CreateUser` end point. 

Hopefully you will get a 200 (OK) response!

Go back into SSMS and check that the user you posted was added to the table `Person`. 

`SELECT TOP (1000) [PersonId]
      ,[PersonGuid]
      ,[Displayname]
      ,[Mobile]
      ,[IsDeleted]
  FROM [WhoOwesWhat].[dbo].[Person]`


## Done with this workshop, onto the next! :) 

## Relevant commands

**Package manager**
- Enable-Migrations
- Add-Migration "some_migration"
- Update-Database

Tip: Add the `-Verbose` flag to get more information from the output in `Package Manager`

## Possible errors you may encounter

### XML style project not easy to work with Entity Framework commands

Error:

`C:\src\WhoOwesWhat-Net48\WhoOwesWhat.DataProvider\obj\WhoOwesWhat.DataProvider.csproj.EntityFrameworkCore.targets(4,5): error MSB4006: There is a circular dependency in the target dependency graph involving target "GetEFProjectMetadata". [C:\src\WhoOwesWhat-Net48\WhoOwesWhat.DataProvider\WhoOwesWhat.DataProvider.csproj]
Unable to retrieve project metadata. Ensure it's an SDK-style project. If you're using a custom BaseIntermediateOutputPath or MSBuildProjectExtensionsPath values, Use the --msbuildprojectextensionspath option.`

Convert to SDK style project. Follow the instructions above.

### Duplicate assemblies in the AssemblyInfo.cs file
Error: 

`Severity	Code	Description	Project	File	Line	Suppression State
Error (active)	CS0579	Duplicate 'System.Reflection.AssemblyTitleAttribute' attribute	WhoOwesWhat.DataProvider	C:\src\WhoOwesWhat-Net48\WhoOwesWhat.DataProvider\obj\Debug\net48\WhoOwesWhat.DataProvider.AssemblyInfo.cs	19	`

This can happen when you convert the .csproj file to SDK style. SDK style project does more "under the hood" and the the below lines of explicit assembly config might not be needed anymore (i.e. **delete** these): 


```
// You can specify all the values or you can default the Build and Revision Numbers 
// by using the '*' as shown below:
// [assembly: AssemblyVersion("1.0.*")]
[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyFileVersion("1.0.0.0")]

[assembly: AssemblyTitle("WhoOwesWhat.DataProvider")]
[assembly: AssemblyConfiguration("")]
[assembly: AssemblyCompany("Microsoft")]
[assembly: AssemblyProduct("WhoOwesWhat.DataProvider")]
```

### Make sure `"WhoOwesWhat.DataProvider.Migrations.TestMigration.resources" was correctly embedded or linked into assembly "WhoOwesWhat.DataProvider" at compile time, or that all the satellite assemblies required are loadable and fully signed.`

```
PM> Update-Database
A version of Entity Framework older than 6.3 is also installed. The newer tools are running. Use 'EntityFramework\Update-Database' for the older version.
Specify the '-Verbose' flag to view the SQL statements being applied to the target database.
Applying explicit migrations: [202408151414008_TestMigration].
Applying explicit migration: 202408151414008_TestMigration.
System.Resources.MissingManifestResourceException: Could not find any resources appropriate for the specified culture or the neutral culture.  Make sure "WhoOwesWhat.DataProvider.Migrations.TestMigration.resources" was correctly embedded or linked into assembly "WhoOwesWhat.DataProvider" at compile time, or that all the satellite assemblies required are loadable and fully signed.
   at System.Resources.ManifestBasedResourceGroveler.HandleResourceStreamMissing(String fileName)
   at System.Resources.ManifestBasedResourceGroveler.GrovelForResourceSet(CultureInfo culture, Dictionary2 localResourceSets, Boolean tryParents, Boolean createIfNotExists, StackCrawlMark& stackMark)
   at System.Resources.ResourceManager.InternalGetResourceSet(CultureInfo requestedCulture, Boolean createIfNotExists, Boolean tryParents, StackCrawlMark& stackMark)
   at System.Resources.ResourceManager.InternalGetResourceSet(CultureInfo culture, Boolean createIfNotExists, Boolean tryParents)
   at System.Resources.ResourceManager.GetString(String name, CultureInfo culture)
   at WhoOwesWhat.DataProvider.Migrations.TestMigration.System.Data.Entity.Migrations.Infrastructure.IMigrationMetadata.get_Target()
   at System.Data.Entity.Migrations.DbMigration.GetModel(Func2 modelAccessor)
   at System.Data.Entity.Migrations.DbMigrator.ApplyMigration(DbMigration migration, DbMigration lastMigration)
   at System.Data.Entity.Migrations.Infrastructure.MigratorLoggingDecorator.ApplyMigration(DbMigration migration, DbMigration lastMigration)
   at System.Data.Entity.Migrations.DbMigrator.Upgrade(IEnumerable1 pendingMigrations, String targetMigrationId, String lastMigrationId)
   at System.Data.Entity.Migrations.Infrastructure.MigratorLoggingDecorator.Upgrade(IEnumerable1 pendingMigrations, String targetMigrationId, String lastMigrationId)
   at System.Data.Entity.Migrations.DbMigrator.UpdateInternal(String targetMigration)
   at System.Data.Entity.Migrations.DbMigrator.EnsureDatabaseExists(Action mustSucceedToKeepDatabase)
   at System.Data.Entity.Migrations.DbMigrator.Update(String targetMigration)
   at System.Data.Entity.Infrastructure.Design.Executor.Update.<>c__DisplayClass0_0.<.ctor>b__0()
   at System.Data.Entity.Infrastructure.Design.Executor.OperationBase.Execute(Action action)
Could not find any resources appropriate for the specified culture or the neutral culture.  Make sure "WhoOwesWhat.DataProvider.Migrations.TestMigration.resources" was correctly embedded or linked into assembly "WhoOwesWhat.DataProvider" at compile time, or that all the satellite assemblies required are loadable and fully signed.
```

**FIX**

![Make sure that the build action for .designer.cs and .resx files are set to 'Embedded resource'. Right click file and choose 'Properties'](https://github.com/sopra-steria-norge/cload-akademiet-course-files/blob/main/images/db-migration-images/build-action-to-embed-resource.png)

## Optional step: Convert the XML-style project to SDK-style project
1. We suggest you use Visual Studio Upgrade Assistant to convert the .csproj type. ChatGPT / Copilot might also be a valid option, but be aware of potential errors that may arise.

    [WhoOwesWhat.DataProvider_xml_version.csproj](https://github.com/sopra-steria-norge/cload-akademiet-course-files/blob/main/code/db-migration/WhoOwesWhat.DataProvider_xml_version.csproj)

    [WhoOwesWhat.DataProvider_SDK_style.csproj](https://github.com/sopra-steria-norge/cload-akademiet-course-files/blob/main/code/db-migration/WhoOwesWhat.DataProvider_SDK_style.csproj)

    This is done to make it easier for Entity framework to seed the database for your project

- Remember to restore nuget packages and build the solution after converting to SDK style project. Might help to restart Visual Studio also. 
