# Add migration, update to Entity Framework 6 and seed a new WhoOwesWhat database - HOW TO

## Starting branch
[dependency-injection](https://github.com/sopra-steria-norge/WhoOwesWhat-Net48/tree/dependency-injection)

## Restore nuget packages and run project

- Run Restore & Build solution in Visual Studio.

Make sure that `WhoOwesWhat.Service.Net8` is `Set as start up project` and run the project with the `https` profile (F5).

If you get this error, you need to setup folder permissions:

<img width="481" height="233" alt="image" src="https://github.com/user-attachments/assets/fafd1212-39bf-4c20-bd66-f558fc6219e8" />

Click this >> https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/Folder_permissions_to_run_projects.md

Else; move on...

- Click on any endpoint in Swagger to get the next error:

On the first run you should get a `System.IO.FileNotFoundException` that refers to the `SqlClient`.

![EF Core not working from CLI for the project](https://github.com/sopra-steria-norge/cloud-akademiet-course-files/blob/main/images/db-migration-images/System.IO.FileNotFoundException_on_first_run.png)

Make sure that this is happening before continuing

## Updating EntityFramework

1. Install latest supported version of **Entity Framework 6.5.1** for the solution:

<img width="534" height="297" alt="image" src="https://github.com/user-attachments/assets/97655a57-6800-4017-957e-fe4a787363cd" />

&nbsp; 

- Update all 4 projects to use EntityFramework to 6.5.1:

</br>
<img width="664" height="381" alt="image" src="https://github.com/user-attachments/assets/4a46467a-27fe-444d-9c5f-ba972c249d00" />

&nbsp;

2. Go to "packages" folder on root folder and delete all EntityFramework.* packages (This is to avoid another error later when trying to "Enable-Migrations" and "Add-Migration")

&nbsp;

3. Verify update by using VSCode to search any leftover: Ctrl+Shift+F and search for 6.1.2
&nbsp;

<img width="468" height="233" alt="image" src="https://github.com/user-attachments/assets/facd11a2-17ec-4ff2-8097-16719f935cf0" />



# Install dotnet ef as global tool

- `cd` into the correct folder from your your command line interface (CLI) in Visual Studio

- Open console in Visual Studio:

<img width="653" height="627" alt="image" src="https://github.com/user-attachments/assets/eb56b966-6940-4f3a-ac4f-f8739e043fa4" />

- `cd WhoOwesWhat.DataProvider`
- Set "Default project to "WhoOwesWhat.DataProvider"
- Run "dotnet tool install --global dotnet-ef"

<img width="718" height="359" alt="image" src="https://github.com/user-attachments/assets/90b863d7-2811-4f74-a69b-a29278a24a2f" />


## Add connection string directly to the `WhoOwesWhatContext` contructor
In order to be able to seed the WhoOwesWhat database to your SQL Server the correct connection string must be added. This should look something like this: 

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

- Enable-Migrations -Verbose

Verbose will show which tooling this command is actually running. The correct one would be "...\WhoOwesWhat-Net48\packages\EntityFramework.6.5.1\tools\...", otherwise, if not try to get help :)

> If you get an error running this command; Run the script "RunEFMigrations.ps1" instead

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

Update the contructor in the class `WhoOwesWhatContext`, i.e.refactor the contructor and add a new default / empty contructor: 
       
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
