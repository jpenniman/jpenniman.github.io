---
layout: post
title: The Repository Pattern and Entity Framework Core
date: 2025-12-01
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: Let's dispel some myths and misconceptions around the Repository Pattern.
tags:
- dotnet
- C#
- entity framework
- repository pattern
- unit of work pattern
- data access objects
---

## Let's dispel some myths and misconceptions

Let's review the Repository Pattern. Many developers, and therefor blogs and other material, describe the Repository
Pattern like this:

> Abstracts the business logic from the database implementation.

And implement it like this:

``` cs
public class CustomerRepository
{
    public Customer Get(int id) {...}
    public Customer[] GetAll() {...}
    public void Insert(Customer c) {...}
    public void Update(Customer c) {...}
    public void Delete(int id) {...}
}
```

However, this is <ins>*not*</ins> the Repository Pattern. Not even close. It is the Data Mapper Pattern (also called the
Data Access Object Pattern). The **Data Mapper Pattern** is described by Martin Fowler in "Patterns of Enterprise
Application Architecture" as

> A layer of mappers that moves data between objects and a database while keeping them independent of each other and
> the mapper itself.

and implemented as:

``` cs
public class CustomerDataMapper
{
    public void Insert(Customer c) {...}
    public void Update(Customer c) {...}
    public void Delete(int id) {...}
}
```

Fowler goes on to explain that querying is best suited with the Lazy Load Pattern and/or Query Object Pattern, but
admits that this may be over kill for some apps and a simple interface for find methods may be sufficient. So we have:

``` cs
public class CustomerQuery
{
    public Customer Get(int id) {...}
    public Customer[] GetAll() {...}
    public Customer[] FindByName() {...}
}
```

for the simple interface, and a more robust query object has criterion for building up a query:

``` cs
public class CustomerQuery
{
    public Criteria Criteria { get; set; }
    public Customer[] Execute() {...}
}
```

The Data Access Object Pattern is similar and states:

> The Data Access Object (DAO) design pattern aims to separate the application's business logic from the persistence
> layer, typically a database or any other storage mechanism. By using DAOs, the application can access and manipulate
> data without being dependent on the specific database implementation details.

With a defined interface of:

``` cs
public class CustomerDao
{
    public Customer Get(int id) {...}
    public Customer[] GetAll() {...}
    public Customer[] FindByName() {...}
    public void Insert(Customer c) {...}
    public void Update(Customer c) {...}
    public void Delete(int id) {...}
}
```

Look familiar?

So, what then is the Repository Pattern? The Repository Pattern is described by Martin Fowler in "Patterns of Enterprise
Application Architecture" as:

> Mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.
> A Repository mediates between the domain and data mapping layers, acting like an in-memory domain object collection. 
> Client objects construct query specifications declaratively and submit them to Repository for satisfaction.

The key there is the repository is an in-memory collection of domain objects. Under the hood it will use DAOs and
Query Objects to fetch data into the in-memory collection and save changes back out.

## Unit of Work Pattern

What if we need to perform work across multiple repositories or DAOs in a single atomic unit or work--transaction?
This is where he Unit of Work Pattern comes in. The Unit of Work Pattern is described by Martin Fowler in
"Patterns of Enterprise Application Architecture" as:

> Maintains a list of objects affected by a business transaction and coordinates the writing out of changes and the
> resolution of concurrency problems.

Implemented as:

``` cs
public class UnitOfWork
{
    public void RegisterNew(object obj) {...}
    public void RegisterDirty(object obj) {...}
    public void RegisterClean(object obj) {...}
    public void RegisterDeleted(object obj) {...}
    public void Commit() {...}
    public void Rollback() {...}
}
```

By tracking what is new vs clean vs dirty (modified) vs deleted, the `Commit()` method knows what methods on the Data
Mapper/DAO to call to persist the changes.

## Why do we need these patterns?

Data Mapper/DAO Pattern, Query Object Pattern, Repository Pattern, and Unit of Work Patterns are designed to abstract
the database and operations against the database from the application's business logic.

A note on interfaces: The use of an interface to decouple implementation has nothing to do with the patterns themselves.
In fact, all the code samples in "Patterns of Enterprise Application Architecture" are just classes, no interfaces. The
use of interfaces to decouple interface from implementation is a Ports and Adapters/Dependency Inversion Principle
concept.

Why would we want to use these patterns?

There are two common reasons we would want to do this:

1. Replacement of the underlying database technology some day.
2. Testing. When coupled with the Dependency Inversion Principle, the use of abstract interfaces with each of these
   patterns allows them to be mocked in unit tests.

Some will add a 3rd reason: reusability. DO NOT DO THIS. This is the wrong reason and is what leads to a big ball of
mud. If use-case A and use-case B may seem to use the same query *now*, but likely won't in the future. In 10 years a
requirement change (or bug fix) to use-case A requires a tweak the the shared code and breaks use-case B. Use-cases 
evolve separately, they should have their own data access code, even if the look identical now.

## So what does this have to do with Entity Framework?

Entity Framework is an ORM--Object Relational Mapper--and implements the many patterns defined in the Object Relational
Patterns sections of "Patterns of Enterprise Application Architecture" including the Repository Pattern and Unit Of Work
Pattern.

> DbContext is a combination of the Unit Of Work and Repository patterns.
> \- [Microsoft Documentation](https://learn.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.dbcontext)

Since it *is* an abstraction over the database and operations against it, we don't strictly *have* to implement those
patterns *again* on top of it.

### What if I want to replace the database someday?

Very true. This *could* happen. Unlikely, but does ensure the architecture is able to adapt to that change. The good
news is Entity Framework already allows us to swap database implementations without changing any data access code.

Such a database swap is only realistic, with or without EF, if the database is strictly a data store and the swap is to
another database of the same type. If the application takes advantage of stored procedures and/or triggers, UDFs, UDTs,
etc., all that needs to be re-written anyway. If you are moving from SQL Server to MongoDB for example, the data model
itself likely needs to be redesigned to be more document friendly--Mongo and denormalization don't play well together.

### What about testing?

Well, what are we really testing? Most applications are basic CRUD. Even apps that have some complex business logic for
some use-cases, the majority of the code is still just CRUD. Mocking the database in most systems offers little-to-no
value.

In the past, databases were very slow and creating one for each component under test even slower so it made sense to be
able to mock for those use-cases. However, these days, this can be done extremely fast. We get better value writing our
tests directly against the database we would use in production. If we do want something lighter for our CI tests, we
could use the Sqlite EF provider with an in-memory Sqlite database.

## But I still want to use the DIP with EF!

Ok, fine. But why? One argument I hear a lot is "I might want to replace EF with another ORM or eliminate an ORM all
together. Well, even if "abstracted away" this is still a lot of work. Since your data access should be per use-case
away to avoid a big ball of mud, "change it in one place and it's changed everywhere" is a myth.

If you insist, let's look at such an implementation:

We will start with a real repository, unit of work, data mapper, and query object. After all, those are what will need
to be reimplemented if you decide to rip out EF without impacting any business code. Alternatively, you *could* just
implement a Data Mapper/DAO for your use-case if that is all you really need. If that is all you need, why are you
using EF? Is it to swap the the database someday? Testing? Because it is the cool thing to do? If you are wrapping EF
up in a simple DAO, you're losing all the power of EF.

Let's assume a DbContext with 2 aggregate roots:

``` cs
public class SalesDbContext
{
    public DbSet<Customer> Customers {get; set;}
    public DbSet<Order> Orders {get; set;}
}
```

We would have 2 repository implementations:

``` cs
public class CustomerRepository
{

} 
```