# Cloud Akademiet .NET Modernization Workshop - Introduction

Welcome to the Cloud Akademiet .NET Modernization Workshop! This hands-on workshop guides you through the complete process of modernizing a legacy .NET Framework 4.8 application to a contemporary .NET 8 application with cloud-native capabilities.

## What to Expect

During this workshop, you will:

1. **Work with Real-world Code** - You'll start with a fully functional .NET Framework 4.8 application called "WhoOwesWhat" and transform it step-by-step into a modern .NET 8 application.

2. **Learn Modern Development Practices** - You'll replace legacy patterns and frameworks with their modern equivalents:
   - Upgrade from Ninject to ASP.NET Core native Dependency Injection
   - Migrate from Entity Framework 6 to Entity Framework Core
   - Implement modern authentication with Microsoft Authentication Library (MSAL)
   - Containerize the application with Docker
   - Deploy to Azure using Infrastructure as Code (Bicep)

3. **Gain Practical Experience** - Each modernization step is structured as a practical exercise with clear instructions, screenshots, and code examples.

4. **Deploy to Azure** - Upon completion, you'll deploy your modernized application to Azure using Bicep templates that create all necessary resources.

## Workshop Environment

You'll be working in a pre-configured DevBox environment that includes:
- Visual Studio 2022
- SQL Server Management Studio
- .NET SDK and .NET Framework
- Docker and containerization tools
- All necessary extensions and dependencies

## Workshop Structure

The workshop follows this progression:

1. **Dependency Injection Migration** - Replace Ninject with native ASP.NET Core DI
2. **Database Migration** - Update Entity Framework and migrate the database
3. **Entity Framework Core Migration** - Upgrade from EF6 to EF Core
4. **Authentication Modernization** - Implement MSAL in Swagger
5. **Containerization** - Add Docker support and run locally
6. **Azure Deployment** - Deploy the modernized application to Azure

## Where to Find More Information

For detailed information about the workshop:

1. **Workshop Overview**: [Workshop-Overview.md](./Workshop-Overview.md) - Comprehensive information about the workshop structure and components

2. **Technical Architecture**: [Technical-Architecture.md](./Technical-Architecture.md) - Diagrams and explanations of the before and after architecture

3. **Workshop Guides**:
   - [Dependency Injection Migration](/cloud-akademiet-course-files/1_Dependency_Injection_Switch_to_NetCore_native.md)
   - [Database Migration](/cloud-akademiet-course-files/2_Database_migration_and_seed_database.md)
   - [Entity Framework Core Migration](/cloud-akademiet-course-files/3_Migrate_to_entity_framework_core.md)
   - [Authentication with MSAL](/cloud-akademiet-course-files/4_Authentication_with_MSAL_in_Swagger.md)
   - [Containerization](/cloud-akademiet-course-files/5_Containerize_application_and_run_locally.md)

4. **Reference Implementation**: Explore the `/WhoOwesWhat-net8/` directory to see the complete modernized application

## Getting Started

1. Open the solution file for the legacy application: `/WhoOwesWhat-Net48/WhoOwesWhat.sln`
2. Read through the first guide: [Dependency Injection Migration](/cloud-akademiet-course-files/1_Dependency_Injection_Switch_to_NetCore_native.md)
3. Follow the step-by-step instructions in each guide
4. Refer to the reference implementation in `/WhoOwesWhat-net8/` when needed

## Need Help?

If you encounter issues during the workshop:
- Check the reference implementation for guidance
- Review the detailed documentation in this folder
- Ask your workshop facilitator for assistance

Enjoy the workshop and happy modernizing!