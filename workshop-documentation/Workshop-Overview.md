# Cloud Akademiet Modernization Workshop

This document provides a comprehensive overview of the Cloud Akademiet modernization workshop structure, components, and workflow.

## Repository Structure

This repository contains a complete workshop designed to teach developers how to modernize legacy .NET applications, with the following key components:

1. **`/WhoOwesWhat-Net48/`** - The legacy application (.NET Framework 4.8)
2. **`/cloud-akademiet-course-files/`** - The workshop materials and guides
3. **`/WhoOwesWhat-net8/`** - The fully modernized application (.NET 8)
4. **`/cloud-akademiet-app-deploy/`** - Infrastructure as Code for Azure deployment
5. **`/cloud-akademiet-devbox/`** - DevBox setup for workshop participants

## Workshop Components in Detail

### 1. DevBox Setup (`/cloud-akademiet-devbox/`)

Before participants begin the workshop, administrators set up Microsoft Dev Boxes using the resources in this folder:

- **Infrastructure as Code**: Bicep templates in `/cloud-akademiet-devbox/bicep/` define the Dev Box configuration
- **Image Definition**: `/cloud-akademiet-devbox/imagedefinition/imagedefinition.yaml` configures:
  - Windows 11 with Visual Studio 2022
  - SQL Server Management Studio
  - .NET Framework Developer Pack
  - SQL Server 2022 Express
- **Administrative Scripts**:
  - `delete-devboxes.ps1` and `hibernate-devboxes.ps1` for managing resources
  - `winget.ps1` for installing additional software

The Dev Boxes provide consistent, pre-configured development environments with:
- 8 vCPU, 32 GB RAM, 256 GB storage
- All necessary development tools already installed
- Single sign-on enabled
- Access to necessary Azure resources

### 2. Legacy Application (`/WhoOwesWhat-Net48/`)

The starting point of the workshop - a legacy .NET Framework 4.8 application with:

- Older frameworks and technologies:
  - .NET Framework 4.8
  - Entity Framework 6.x (non-Core version)
  - XML-style project files
  - Traditional dependency injection (Ninject)
  - Classic web.config configuration
  - Non-containerized deployment model

### 3. Workshop Process (`/cloud-akademiet-course-files/`)

Participants work through a structured modernization process, with each step building on the previous one:

1. **Switch to Native DI** - Replace Ninject with ASP.NET Core DI system (`1_Dependency_Injection_Switch_to_NetCore_native.md`)
2. **Database Migration** - Update Entity Framework and migrate the database (`2_Database_migration_and_seed_database.md`)
3. **Entity Framework Core** - Upgrade to EF Core from EF6 (`3_Migrate_to_entity_framework_core.md`)
4. **Modern Authentication** - Implement MSAL with Swagger UI integration (`4_Authentication_with_MSAL_in_Swagger.md`)
5. **Containerization** - Add Docker support and run locally (`5_Containerize_application_and_run_locally.md`)

Each step includes:
- Clear starting points (Git branches)
- Step-by-step instructions
- Screenshots and examples
- Explanations of concepts

### 4. Modernized Application (`/WhoOwesWhat-net8/`)

The "finished state" reference implementation showing the target architecture:

- Modern technologies and practices:
  - .NET 8
  - Entity Framework Core
  - SDK-style projects
  - Native ASP.NET Core dependency injection
  - Modern configuration (appsettings.json)
  - Containerized with Docker
  - Azure authentication with MSAL
  - Swagger UI with OAuth integration

Participants can refer to this implementation at any time to check their progress or understand the target architecture.

### 5. Application Deployment (`/cloud-akademiet-app-deploy/`)

Once participants complete the modernization steps, they use the resources in this folder to deploy their application to Azure:

- **Bicep Templates**: Define all required Azure resources in `/cloud-akademiet-app-deploy/bicep/`
- **Deployment Scripts**: PowerShell scripts in `/cloud-akademiet-app-deploy/scripts/` automate the deployment
- **GitHub Actions**: CI/CD workflows for deployment automation

The deployment creates a complete environment for each participant:
- Resource Group (named with participant identifier)
- App Service (running the containerized application)
- SQL Server and database
- Managed Identity for secure access
- Application Insights for monitoring
- Role assignments (Owner for the participant)

## Complete Workshop Flow

1. **Before the Workshop**:
   - Administrators deploy DevBoxes using `/cloud-akademiet-devbox/`
   - Participants are added to required security groups

2. **Workshop Start**:
   - Participants access their pre-configured DevBox
   - They clone the repository and open the legacy application (`/WhoOwesWhat-Net48/`)
   - Course guides in `/cloud-akademiet-course-files/` walk them through each step

3. **Modernization Process**:
   - Follow numbered guides (1-5) in `/cloud-akademiet-course-files/`
   - Start with legacy .NET Framework 4.8 code
   - End with modern .NET 8, containerized application
   - Reference complete implementation in `/WhoOwesWhat-net8/` as needed

4. **Deployment to Azure**:
   - After completing all modernization steps
   - Use `/cloud-akademiet-app-deploy/scripts/deploy.ps1` or GitHub Actions
   - Deploy their modernized app to Azure
   - Test the deployed application via provided URLs

5. **Post-Workshop**:
   - Resources can be cleaned up using scripts in the repository
   - DevBoxes can be hibernated or deleted

## Technical Modernization Journey

The workshop guides participants through these key modernization steps:

1. **Dependency Injection Modernization**:
   - From: Ninject-based DI in .NET Framework
   - To: Native Microsoft DI in ASP.NET Core

2. **Database Access Modernization**:
   - From: Entity Framework 6.x with older patterns
   - To: Entity Framework Core with modern data access patterns

3. **Authentication Modernization**:
   - From: Legacy authentication methods
   - To: Modern MSAL (Microsoft Authentication Library) with OAuth 2.0

4. **Project Structure Modernization**:
   - From: XML-based project files with complex configurations
   - To: SDK-style projects with simplified structure

5. **Deployment Modernization**:
   - From: Traditional deployment methods
   - To: Containerized applications deployed to Azure cloud services

## Benefits of This Approach

- **Consistent Environment**: DevBoxes ensure everyone has identical setup
- **Step-by-Step Guidance**: Clear markdown guides with screenshots
- **Reference Implementation**: Complete example to compare against
- **Real-World Deployment**: Practical experience with Azure resources
- **Isolated Resources**: Each participant gets their own Azure environment

This comprehensive workshop provides a complete end-to-end experience of modernizing and deploying a .NET application, covering the entire application lifecycle from legacy code to cloud deployment.