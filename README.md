# Clean Architecture Microservice Template

Welcome to the Clean Architecture Microservice Template built with .NET! This template provides a solid foundation to start building scalable and maintainable microservices using the principles of Clean Architecture. The template is structured into four main projects: Core, Use Cases, Infrastructure, and API.

## Table of Contents

1. [Project Structure](#project-structure)
2. [Core Project](#core-project)
3. [Use Cases Project](#use-cases-project)
4. [Infrastructure Project](#infrastructure-project)
5. [API Project](#api-project)
6. [Example Project](#example-folder)
7. [Design Decisions](#design-decisions)
8. [Getting Started](#getting-started)
9. [Contributing](#contributing)
10. [License](#license)

## Project Structure

The solution is divided into the following projects:

- **Core**: Contains the domain layer.
- **Use Cases**: Contains the application layer.
- **Infrastructure**: Contains the infrastructure layer.
- **API**: Contains the presentation layer.

## Core Project

The Core project represents the domain layer of the application and contains:

- **Entities**: The core business objects.
- **Aggregates**: Aggregate roots which are clusters of domain objects.
- **Value Objects**: Objects that are immutable and have no identity.
- **Domain Events & Handlers**: Events that represent something which has occurred in the domain.
- **Domain Services**: Services that encapsulate domain logic.
- **Abstractions**: Interfaces and abstract classes to decouple implementations.

## Use Cases Project

The Use Cases project represents the application layer and contains:

- **Commands**: Commands that represent actions to change state.
- **Queries**: Queries to retrieve data.
- **Validation**: Logic for validating commands and queries.
- **Authentication**: Services and logic for authenticating users.
- **Authorization**: Policies and handlers for authorizing user actions.
- **Integration Events**: Events for integrating with other systems or services.

## Infrastructure Project

The Infrastructure project represents the infrastructure layer and contains:

- **Abstraction Implementations**: Implementations of abstractions defined in the Core project.
- **Database Configurations**: Configurations and setup for database connections and migrations.
- **Repositories**: Data access logic to interact with the database.

## API Project

The API project represents the presentation layer and contains:

- **APIs**: Controllers and endpoints to expose application functionality.
- **Service Registration**: Configuration for dependency injection and service setup.

## Example Folder

The example folder is a clone of template projects with actual aggregates and functionalities. 
It is based on the [eShop reference application](https://github.com/dotnet/eShop) by Microsoft. 
Although the example does not include all functionalities of the original repository, it is meant to keep a local 
copy of the implementation of DDD fundamental concepts,  such as aggregates, domain events, etc.

## Design Decisions

### Use Cases Project and Repositories

The Use Cases project relies on repositories to write and retrieve data. The Core project exposes two kinds of repositories:

- **IRepository<T>**: This repository interface is primarily dedicated to write operations but can also include retrieve methods.
- **IReadRepository<T>**: This repository interface is dedicated to pure read operations. It leverages specifications to encapsulate query logic, ensuring that read operations are efficient and maintainable.

### Repository Interfaces

- **IRepository<T>**:
    - Primarily handles create, update, and delete operations.
    - Can include read methods if needed, but its main focus is on write operations.
    - Has a reference to IUnitOfWork. The UoW pattern ensures that all changes to an aggregate are committed as a single transaction. This means that either all changes are saved to the database, or none are, preserving the consistency of the aggregate.

- **IReadRepository<T>**:
    - Dedicated to handling read operations.
    - Utilizes specifications to define queries, which promotes reusability and separation of concerns in data retrieval logic.

### HILO

The Hi/Lo algorithm is a technique for generating unique identifiers for database entities. It's particularly useful in scenarios where an application needs to assign IDs to entities even before they are persisted in the database. This aligns well with the principles of Domain-Driven Design (DDD).

This algorithm utilizes two values:
- Hi: A high-value number obtained from the database. This value is typically retrieved through a dedicated table or sequence. 
- Lo: A low-value counter maintained by the application. It has a predefined range (often called incrementSize).

The Hi/Lo algorithm combines these values to generate a unique identifier. The formula usually involves multiplying hi by the incrementSize and adding the current lo value.
Once an identifier is generated using the current lo, the counter is incremented.

In DDD, aggregates are groups of related entities that are treated as a single unit of consistency. They encapsulate complex domain logic and ensure the data within the aggregate remains consistent.

To establish relationships between entities within an aggregate, and to perform operations that involve these relationships, unique identifiers are crucial. Hi/Lo allows assigning these IDs even before persistence, enabling the application to manage the aggregate's internal structure and behavior effectively.

### Transactions

The `TransactionBehavior` mediator behavior is designed to wrap each request in a database transaction, ensuring that operations are executed within a transactional context. This guarantees that all changes are either committed or rolled back as a single unit, maintaining data integrity. If there's already an active transaction, the request proceeds without creating a new transaction.

Creating transaction and executing request handlers are wrapper in a  execution strategy’s execute block.

An execution strategy in Entity Framework Core handles transient failures such as network issues or database timeouts. These strategies are essential for ensuring the robustness and resilience of  application's database operations.

```csharp
var strategy = _dbDbContext.Database.CreateExecutionStrategy();
```

The `CreateExecutionStrategy` method returns an instance of `IExecutionStrategy`. The default implementation for SQL Server is `SqlServerRetryingExecutionStrategy`, which retries the operation if it encounters a transient failure.

The execution strategy's `ExecuteAsync` method wraps the transactional code:

```csharp
await strategy.ExecuteAsync(async () =>
{
    await using var transaction = await _dbDbContext.BeginTransactionAsync();
    using (_logger.BeginScope(new List<KeyValuePair<string, object>> { new("TransactionContext", transaction.TransactionId) }))
    {
        _logger.LogInformation("Begin transaction {TransactionId} for {CommandName} ({@Command})", transaction.TransactionId, typeName, request);

        response = await next();

        _logger.LogInformation("Commit transaction {TransactionId} for {CommandName}", transaction.TransactionId, typeName);

        await _dbDbContext.CommitTransactionAsync(transaction);
    }
});
```

> **PS:** The BeginTransactionAsync() method will create a new transaction with database isolation level set to **ReadCommitted**
>

## Getting Started

## Contributing

Contributions are welcome! If you find any issues or have suggestions for improvements, please open an issue or submit a pull request.

1. **Fork the repository**.
2. **Create a new branch**: `git checkout -b feature/your-feature-name`.
3. **Commit your changes**: `git commit -m 'Add some feature'`.
4. **Push to the branch**: `git push origin feature/your-feature-name`.
5. **Open a pull request**.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for more details.

---

Happy coding! If you have any questions or need further assistance, feel free to reach out.