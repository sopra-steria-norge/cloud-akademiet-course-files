# Cloud Akademiet Workshop Documentation

## Getting Started

**New Participants**: Start here → [Introduction to the Workshop](./Introduction.md)

## Table of Contents

### Workshop Structure
- [Introduction](./Introduction.md) - What to expect and how to get started
- [Workshop Overview](./Workshop-Overview.md) - Comprehensive overview of the entire workshop structure and process
- [Technical Architecture](./Technical-Architecture.md) - Architecture diagrams and technical transformation details

### Legacy Application (.NET Framework 4.8)
- `/WhoOwesWhat-Net48/` - Starting point for the modernization workshop

### Workshop Steps
1. [Dependency Injection Migration](/cloud-akademiet-course-files/1_Dependency_Injection_Switch_to_NetCore_native.md) - Moving from Ninject to native ASP.NET Core DI
2. [Database Migration](/cloud-akademiet-course-files/2_Database_migration_and_seed_database.md) - Entity Framework updates and database seeding
3. [Entity Framework Core Migration](/cloud-akademiet-course-files/3_Migrate_to_entity_framework_core.md) - Moving from EF6 to EF Core
4. [Authentication Modernization](/cloud-akademiet-course-files/4_Authentication_with_MSAL_in_Swagger.md) - Implementing MSAL in Swagger
5. [Containerization](/cloud-akademiet-course-files/5_Containerize_application_and_run_locally.md) - Adding Docker support and running locally

### Modernized Application (.NET 8)
- `/WhoOwesWhat-net8/` - Reference implementation of the fully modernized application

### DevBox Setup
- `/cloud-akademiet-devbox/` - Resources for setting up consistent development environments

### Azure Deployment
- `/cloud-akademiet-app-deploy/` - Infrastructure as Code and scripts for deploying to Azure

## Getting Started

1. Access your assigned DevBox environment
2. Open the legacy application (`/WhoOwesWhat-Net48/`)
3. Follow the numbered guide documents in `/cloud-akademiet-course-files/`
4. Reference the modernized implementation (`/WhoOwesWhat-net8/`) as needed
5. Deploy your modernized application using `/cloud-akademiet-app-deploy/`

## Quick Links

- [DevBox Setup Instructions](/cloud-akademiet-devbox/README.md)
- [Azure Deployment Instructions](/cloud-akademiet-app-deploy/README.md)