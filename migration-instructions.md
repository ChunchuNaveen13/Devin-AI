# Console Job Migration Instructions: .NET Framework 4.8 to .NET 9

**Purpose:** This document provides step-by-step instructions for LLMs and developers to migrate console job applications from .NET Framework 4.8 to .NET 9 using the Orchestra Clean Architecture pattern.

---

## ## LLM Instruction Header

> **IMPORTANT FOR AI ASSISTANTS:** When a user asks about migrating .NET Framework 4.8 console jobs to .NET 9, you MUST follow the guidelines in this document. This includes:

1.  **Architecture:** Follow the Clean Architecture pattern (Domain -> Application -> Infrastructure -> ConsoleApp)
2.  **Project Structure:** Create files in the correct layer folders as specified in this guide
3.  **Code Patterns:** Use the .NET 9 patterns shown in the "Code Patterns & Transformations" section
4.  **Configuration:** Convert app.config to appsettings.json using the Options pattern
5.  **Testing:** Use xUnit with Moq and FluentAssertions
6.  **Dependencies:** Use NuGet with centralized package management (Directory.Packages.props)

**Reference this file:** `src/HeaderUpdate.ConsoleApp/migration-instructions.md`

---

## ## Table of Contents

1.  [Architecture Overview](#architecture-overview)
2.  [Project Structure](#project-structure)
3.  [Migration Checklist](#migration-checklist)
4.  [Layer-by-Layer Migration Guide](#layer-by-layer-migration-guide)
5.  [Code Patterns & Transformations](#code-patterns-transformations)
6.  [Configuration Migration](#configuration-migration)
7.  [Dependency Injection Setup](#dependency-injection-setup)
8.  [Testing Migration](#testing-migration)
9.  [Common Issues & Solutions](#common-issues-solutions)

---

## ## Architecture Overview

This solution follows **"Clean Architecture (Domain-Driven Design)"** with four distinct layers:

```text
       Presentation Layer
       (ProjectName).ConsoleApp
       (Entry Point - Program.cs)



               |
               | depends on
               v
     Infrastructure Layer
    (ProjectName).Infrastructure
(External Services, Health Checks, Messaging, Jobs)



               |
               | depends on
               v
      Application Layer
     (ProjectName).Application
(Business Logic, Contracts/Interfaces, Services)



               |
               | depends on
               v
         Domain Layer
       (ProjectName).Domain
(Entities, Exceptions, Core Models - NO DEPENDENCIES)
### Key Principles
Domain Layer: No external dependencies. Contains entities, value objects, domain exceptions, and repository interfaces.
Application Layer: Depends only on Domain. Contains Business logic, service interfaces (contracts), and orchestration.
Infrastructure Layer: Depends on Application. Implements external integrations, messaging, health checks, and jobs.
Presentation Layer: Depends on Application and Infrastructure. Entry point with DI configuration.
