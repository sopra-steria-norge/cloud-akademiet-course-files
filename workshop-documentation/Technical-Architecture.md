# Technical Architecture Overview

This document outlines the technical architecture of both the legacy application and the modernized application.

## Legacy Application Architecture (.NET Framework 4.8)

```
┌───────────────────────────────────────────────────┐
│                                                   │
│  WhoOwesWhat.NancyServer                          │
│  ├── Hosting & API endpoints                      │
│  └── Web.config configuration                     │
│                                                   │
├───────────────────────────────────────────────────┤
│                                                   │
│  WhoOwesWhat.Service.Controller                   │
│  └── API Controllers                              │
│                                                   │
├───────────────────────────────────────────────────┤
│                                                   │
│  WhoOwesWhat.Ninject                              │
│  └── Dependency Injection Container               │
│                                                   │
├───────────────────────────────────────────────────┤
│                                                   │
│  WhoOwesWhat.Domain                               │
│  └── Business Logic                               │
│                                                   │
├───────────────────────────────────────────────────┤
│                                                   │
│  WhoOwesWhat.DataProvider                         │
│  ├── Entity Framework 6.x                         │
│  ├── SQL Repository                               │
│  └── Database Access                              │
│                                                   │
└───────────────────────────────────────────────────┘
```

### Legacy Application Characteristics:

- **Framework**: .NET Framework 4.8
- **Data Access**: Entity Framework 6.x
- **Dependency Injection**: Ninject
- **API Framework**: Nancy
- **Authentication**: Legacy/custom authentication
- **Configuration**: Web.config XML-based
- **Project Structure**: Traditional XML-based .csproj files
- **Deployment**: Traditional IIS deployment

## Modernized Application Architecture (.NET 8)

```
┌───────────────────────────────────────────────────┐
│                                                   │
│  WhoOwesWhat.Service.Net8                         │
│  ├── ASP.NET Core Hosting                         │
│  ├── API Controllers                              │
│  ├── Native Dependency Injection                  │
│  ├── Swagger + OAuth Integration                  │
│  └── appsettings.json configuration               │
│                                                   │
├───────────────────────────────────────────────────┤
│                                                   │
│  WhoOwesWhat.Domain.Net8                          │
│  └── Business Logic                               │
│                                                   │
├───────────────────────────────────────────────────┤
│                                                   │
│  WhoOwesWhat.DataProvider.Net8                    │
│  ├── Entity Framework Core                        │
│  └── Database Access                              │
│                                                   │
└───────────────────────────────────────────────────┘
```

### Modernized Application Characteristics:

- **Framework**: .NET 8
- **Data Access**: Entity Framework Core
- **Dependency Injection**: Native ASP.NET Core DI
- **API Framework**: ASP.NET Core Web API
- **Authentication**: Microsoft Identity Platform (MSAL)
- **Configuration**: appsettings.json
- **Project Structure**: SDK-style .csproj files
- **Deployment**: Docker container in Azure App Service

## Azure Deployment Architecture

After modernization, the application is deployed to Azure with the following resources:

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Azure Resource Group (Per Participant)                         │
│                                                                 │
│  ┌─────────────────────┐    ┌────────────────────────────────┐  │
│  │                     │    │                                │  │
│  │  Azure App Service  │    │  Azure SQL Database            │  │
│  │  ├── Containerized  │────┤  ├── WhoOwesWhat DB           │  │
│  │  │   Application    │    │  └── Entity Framework         │  │
│  │  └── HTTPS Endpoint │    │      Migrations               │  │
│  │                     │    │                                │  │
│  └─────────────────────┘    └────────────────────────────────┘  │
│          │                                                      │
│          │                  ┌────────────────────────────────┐  │
│          │                  │                                │  │
│          └──────────────────┤  Azure Application Insights    │  │
│                             │  └── Monitoring & Telemetry    │  │
│                             │                                │  │
│                             └────────────────────────────────┘  │
│                                                                 │
│  ┌─────────────────────┐    ┌────────────────────────────────┐  │
│  │                     │    │                                │  │
│  │  Azure Container    │    │  User-Assigned                 │  │
│  │  Registry           │    │  Managed Identity              │  │
│  │  └── Application    │    │  └── Service Authentication    │  │
│  │      Image          │    │                                │  │
│  │                     │    │                                │  │
│  └─────────────────────┘    └────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Key Technical Transformations

### 1. Dependency Injection
- **Before**: Ninject container with manual registration
- **After**: Native ASP.NET Core DI with service collection

### 2. Data Access
- **Before**: Entity Framework 6.x with custom contexts
- **After**: Entity Framework Core with modern data access patterns

### 3. Authentication
- **Before**: Custom authentication
- **After**: Microsoft Identity Platform (MSAL) with OAuth 2.0

### 4. API Framework
- **Before**: Nancy framework
- **After**: ASP.NET Core Web API with Swagger

### 5. Containerization
- **Before**: No containerization
- **After**: Docker containerization with multi-stage builds

### 6. Configuration
- **Before**: Web.config XML-based configuration
- **After**: appsettings.json with environment-specific overrides

### 7. Deployment
- **Before**: Manual/traditional deployment
- **After**: IaC-based Azure deployment with Bicep