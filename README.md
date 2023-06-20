# The Clean API Template 

## Introduction to Project
This project is a sample application that follows the principles of Clean Architecture in .NET Core. It provides a modular structure with separate layers for the API, application logic, domain entities, and infrastructure. The project incorporates popular frameworks and libraries such as ASP.NET Core, Entity Framework Core, MediatR, AutoMapper, FluentValidation, Serilog, and Swagger. With Docker support, the application can be easily containerized and deployed in a Docker environment.

## Introduction to Clean Architecture
Clean Architecture is an architectural pattern that emphasizes the separation of concerns and the independence of an application's business rules from the external frameworks and technologies it utilizes. It provides a structured approach to software development, promoting code maintainability, testability, and scalability.

Developed by Robert C. Martin (also known as Uncle Bob), Clean Architecture focuses on designing systems that are flexible, adaptable, and resistant to changes in requirements or technology choices. It encourages developers to create software that can evolve independently, with minimal impact on other parts of the system.

Each layer in Clean Architecture has a specific responsibility and encapsulates a different aspect of the application. The layers are organized in a way that allows for modularity, maintainability, and flexibility. The separation of concerns ensures that changes made to one layer have minimal impact on other layers.

<img width="400" alt="clean-removebg-preview" src="https://github.com/edinSahbaz/clean-api-template/assets/47791892/5d3afb3b-042e-417f-a7fd-75f6647d5118">

#### Presentation Layer:
The Presentation Layer is responsible for handling the user interface and interaction. It includes components like web APIs, user interfaces, or command-line interfaces. This layer is primarily concerned with receiving user input, displaying information, and coordinating the flow of data between the user and the application.

#### Application Layer:
The Application Layer contains the application-specific business logic and use cases. It acts as an intermediary between the Presentation Layer and the Domain Layer. This layer orchestrates the execution of use cases by coordinating the interactions between different components and applying business rules. It does not contain any infrastructure-related or implementation-specific details.

#### Domain Layer:
The Domain Layer encapsulates the core business logic and entities of the application. It represents the heart of the system and contains business rules, entities, value objects, and domain-specific logic. The Domain Layer is independent of any external frameworks or technologies and should be the most stable and reusable part of the architecture.

#### Infrastructure Layer:
The Infrastructure Layer handles external concerns and provides implementations for data access, external services, frameworks, and other infrastructure-related code. It includes components like databases, file systems, third-party APIs, and external integrations. The Infrastructure Layer interacts with the external world, enabling the application to store and retrieve data, communicate with external systems, and handle cross-cutting concerns like logging or caching.

#### Persistence Layer:
The Persistence Layer is a specific type of infrastructure layer that deals with data persistence and storage. It includes implementations of repositories, data mappers, or Object-Relational Mapping (ORM) frameworks. This layer provides the necessary mechanisms to persist and retrieve domain entities and data from a database or other storage systems.

## Benefits Using of Clean Architecture
#### Maintainability: 
Clean Architecture separates concerns and reduces code dependencies, making it easier to understand and modify individual components without impacting the entire system.

#### Testability: 
Clean Architecture promotes thorough unit testing by isolating business logic from external dependencies, allowing for focused and reliable testing of core application functionality.

#### Scalability: 
The modular structure of Clean Architecture enables independent scaling of specific components or layers, ensuring optimal performance and resource allocation.

#### Technology Independence: 
Clean Architecture decouples the core business logic from implementation details, enabling flexibility in adopting new technologies or frameworks without extensive code changes.

#### Flexibility and Adaptability:
Clean Architecture allows for easier modification and evolution of the system as requirements change, thanks to the separation of concerns and minimal ripple effects across the codebase.

#### Code Reusability: 
Clean Architecture promotes the creation of reusable code by keeping core business logic separate, reducing duplication, and improving development efficiency.

## Project Structure
### Domain Layer
The Domain layer sits at the core of the Clean Architecture. Here we define things like entities, value objects, aggregates, domain events, exceptions, repository interfaces, etc.

Here is the folder structure used in this template:
![domain](https://github.com/edinSahbaz/clean-api-template/assets/47791892/974e150d-d9c4-4ed1-967a-ab557092e019)

You can introduce more things here if you think it's required.

One thing to note is that the Domain layer is not allowed to reference other projects in your solution.

### Application Layer
The Application layer sits right above the Domain layer. It acts as an orchestrator for the Domain layer, containing the most important use cases in your application.

You can structure your use cases using services or using commands and queries.

I'm a big fan of the CQRS pattern, so I like to use the command and query approach.

Here is the folder structure used in this template:
![application](https://github.com/edinSahbaz/clean-api-template/assets/47791892/7c4d102f-5b00-442b-b710-a8af6f2a4402)

In the Abstractions folder, we define the interfaces required for the Application layer. The implementations for these interfaces are in one of the upper layers.

For every entity in the Domain layer, we create one folder with the commands, queries, and events definitions.

### Infrastructure Layer
The Infrastructure layer contains implementations for external-facing services.

What would fall into this category?

* Databases - PostgreSQL, MongoDB
* Identity providers - Auth0, Keycloak
* Emails providers
* Storage services - AWS S3, Azure Blob Storage
* Message queues - Rabbit MQ
  
Here is the folder structure used in this template:
![infra](https://github.com/edinSahbaz/clean-api-template/assets/47791892/db57066b-91ea-4560-8295-aa945d59fea6)

This project contains an implementation of DbContext if you use EF Core.

It's not uncommon to make the Persistence folder its project. I do this to have all database facing-code inside of one project.

### Presentation Layer
The Presentation layer is the entry point to our system. Typically, you would implement this as a Web API project.

The most important part of the Presentation layer is the Controllers(Endpoints), which define the API endpoints in our system.

Here is the folder structure used in this template:
![pres](https://github.com/edinSahbaz/clean-api-template/assets/47791892/796222c9-e092-48ea-b263-a2cb80edfde2)

In this case, we moved the Presentation layer away from the actual Web API project. We do this to isolate the Controllers and enforce stricter constraints.

## Logging

### About Serilog
Serilog is a popular logging library for .NET applications. It provides a flexible and efficient logging framework that allows developers to capture and store log messages for debugging, monitoring, and troubleshooting purposes. With Serilog, you can easily configure log sinks, define log levels, and customize log message formatting to meet the specific requirements of your application. It offers support for structured logging, enabling the capture of rich, contextual information in log events. 

Serilog integrates well with various logging targets, such as console output, files, databases, and third-party logging services, making it a versatile choice for implementing logging capabilities in your .NET projects.

### Serilog configuration
``` csharp
builder.Host.UseSerilog((context, configuration) => 
    configuration.ReadFrom.Configuration(context.Configuration));
```

``` csharp
app.UseSerilogRequestLogging();
```

``` json
{
  "Serilog": {
    "Using": [ "Serilog.Sinks.Console", "Serilog.Sinks.File" ],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      { "Name": "Console" },
      {
        "Name": "File",
        "Args": {
          "path": "/logs/log-.txt",
          "rollingInterval": "Day",
          "rollOnFileSizeLimit": true,
          "formatter": "Serilog.Formatting.Compact.CompactJsonFormatter, Serilog.Formatting.Compact"
        }
      }
    ],
    "Enrich": [ "FromLogContext", "WithMachineName", "WithThreadId" ]
  },
}
```

## Architecture Tests
