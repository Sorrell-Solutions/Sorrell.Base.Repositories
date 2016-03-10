#Sorrell.Base.Repositories
Implementation of Repository and Unit of Work patterns for Entity Framework 6.1. NuGet package available.

It's highly recommended to view Mosh Hamedani's presentation on the implementation of the Repository and Unit of Work patterns: https://www.youtube.com/watch?v=rtXpYpZdOzM

##Getting Started

I recommend looking at the included sample project `Repositories.Sample` for a basic example of how to create the repository and unit of work classes and use them.

The quickest way to get the required libraries in your data layer is to use the NuGet package.

###Getting Started with the Repository Pattern

After adding the NuGet package to your project, you will need to:

1. Create a repository interface for each of your strong entities. These interfaces will define the repository methods specific to that entity. Generic methods, like `Get` and `GetAll` are defined in the base interface. This repository interface inherits from the generic `IRepository<TEntity, TId>` interface.
2. Create a repository class for each of your strong entities. This repository class should inherit from the generic `EFRepository<TEntity, TId>` class and implement the repository interface you created in step 1.

Details below.

####Creating Repository Interfaces

```
public interface IOrderRepository : IRepository<Order, int>
```

`TEntity` is your domain object type. `TId` is the type of the primary key field. Special use case is a type with a compound primary key (and not a weak entity).

You define a method in the interface for each operation you want the repository to perform that's not already included in the base interface, for example, you could define a method `GetTopOrders(int top)` like this:

```
IEnumerable<Order> GetTopOrders(int top);
```

####Creating Repository Classes

```
public class OrderRepository : EFRepository<Order, int>, IOrderRepository
```

For ease of use, you'll want to create a private property that will cast the base class' `Context` property back to the specific type of your EF DbContext:

```
private YourDbContext MyContext { get { return Context as YourDbContext; } }
```

You will need to provide implementations for each of the methods you defined in the interface:

```
public IEnumerable<Order> GetTopOrders(int top)
{
    // code here
    // use the MyContext property to get access to the Entity Framework's DbContext
}
```

###Getting Started with the Unit of Work Pattern

1. Create an `IUnitOfWork` interface that inherits from `IDisposable`.
2. Create a `UnitOfWork` class that inherits from the generic `EFUnitOfWork<TDbContext>` class and implements the `IUnitOfWork` interface from step 1.

####Creating the IUnitOfWork Interface

```
public interface IUnitOfWork : IDisposable
```

Add a read-only property for each repository that is of the repository's interface type:

```
    IOrderRepository Orders { get; }
```

####Creating the UnitOfWork Class

```
public class UnitOfWork : EFUnitOfWork<YourDbContext>, IUnitOfWork
```

Create a property for each repository with a public getter and private setter:

````
    public IOrderRepository Orders { get; private set; }
````

Create a constructor that takes your DbContext class as a parameter. The constructor also initializes each of the repositories.

```
public UnitOfWork(YourDbContext context)
    : base(context)
{
    Orders = new OrderRepository(context);
}
```

Correctly implement IDisposable. Note: the DbContext instance is disposed in the generic `EFUnitOfWork<TDbContext>` base class.

```
#region IDisposable Support
    private bool disposedValue = false; // To detect redundant calls

    protected override void Dispose(bool disposing)
    {
        if (!disposedValue)
        {
            // TODO: set large fields to null - this would include your repositories
            Orders = null;

            disposedValue = true;
        }
        
        base.Dispose(disposing);
    }
#endregion
```

###Using the Repositories and Unit of Work in Your Code

Essentially, you will use the interfaces and classes to abstract away Entity Framework. Where you would ordinarily use a DbContext object, you will now use a UnitOfWork object:

```
// Or use dependency injection to remove the reference to the EF DbContext object
using (IUnitOfWork u = new UnitOfWork(new YourDbContext())
{
    // query, update, etc.
    
    u.Complete(); // Persist updates
}
```

##Future

I would like to improve the project by
- Adding T4 templates to generate the Repository interfaces and classes based on an Entity Framework DbContext class.
- Adding Unity Dependency Injection (DI) to the sample project.
