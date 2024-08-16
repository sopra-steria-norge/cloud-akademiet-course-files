# Migrate database and update Entity Framework - HOW TO

## Restore nuget packages and run project

Make sure that this works before continuing.

## `cd` into the correct folder from your your command line interface (CLI)

--> `cd WhoOwesWhat.DataProvider`

## Make Entity Framework work
1. Make sure that a supported version of Entity Framework is used, and that you can use this via your CLI. In this case we have a .NET Framework project, which means that we cannot use Entity Framework Core until we have migrated the service to .NET Core (i.e. .NET 8). Before we reach that point, we will use the latest supported version of **Entity Framework 6**.

![EF Core not working from CLI for the project](https://github.com/sopra-steria-norge/cload-akademiet-course-files/blob/main/images/db-migration-images/ef-core-does-not-work.png)

2. Install latest supported version of **Entity Framework 6** for the project WhoOwesWhat.DataProvider
3. Use **Entity Framework 6** from Package Manager in Visual Studio 2022 with the relevant commands as stated below

## Convert the XML-style project to SDK-style project
1. We suggest you use Copilot or similar tools to convert the .csproj file

    [WhoOwesWhat.DataProvider_xml_version.csproj](https://github.com/sopra-steria-norge/cload-akademiet-course-files/blob/main/code/db-migration/WhoOwesWhat.DataProvider_xml_version.csproj)

    [WhoOwesWhat.DataProvider_SDK_style.csproj](https://github.com/sopra-steria-norge/cload-akademiet-course-files/blob/main/code/db-migration/WhoOwesWhat.DataProvider_SDK_style.csproj)

    This is done to make it easier for Entity framework to seed the database for your project

- Remember to restore nuget packages and build the solution after converting to SDK style project. Might help to restart Visual Studio also. 

## Add connection string to `web.config`
In order to be able to seed the WhoOwesWhat database to your SQL Server the correct connection string must be added. This should look somthing like this: 

`<connectionStrings>
    <add name="DefaultConnection" connectionString="Data Source=.\SQLEXPRESS;Integrated Security=True;Database=WhoOwesWhat;Connect 
    Timeout=30;Encrypt=False;" providerName="System.Data.SqlClient" />
</connectionStrings>`

## Add a initial migration

In Package Manager run the commands:

- Enable-Migrations
- Add-Migration "InitialMigration"

PS. Might have to move the `MigrationConfigurations.cs` file into the new folder 'Migrations' that might be auto-generated when adding InitialMigration. 

Continue in Package Manager and run the command to seed the database (or update if already created): 
- Database-Update

## Relevant commands

**Package manager**
- Enable-Migrations
- Add-Migration "some_migration"
- Update-Database

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
