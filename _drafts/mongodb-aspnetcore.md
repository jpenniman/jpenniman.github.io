---
layout: post
title: Using MongoDB in AspNetCore
date: 2020-03-20
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: Let's take a look at using MongoDB in AspNet Core applications and microservices.
tags:
- dotnet
- C#
- MongoDB
---

The MongoDB client in the C# driver is designed to be reused throughout the lifetime of the application, as it manages
connection pooling.

A couple ways we can handle this:

* Dependency Injection
* Wrapping it in an Singleton "ClientFactory"

There are pros and cons to each.

## Dependency Injection

The simplest approach is to simply register the client as a singleton in our IoC container and let the container handle
the lifetime.

```cs
builder.Services.AddSingleton<IMongoClient>(_ => 
    new MongoClient(builder.Configuration.GetConnectionString("DefaultConnection")));
```

Then just inject it:

```cs
class CustomerDao
{
    readonly IMongoClient _mongoClient;
    public CustomerDao(IMongoClient mongoClient)
    {
        _mongoClient = mongoClient;
    }

    ...
}
```

### Pros

* Very simple. 
* It does not require extra ceremony and uses a basic factory method as part of the container registration.

### Cons

* It ties the solution to DI.
* Risks transient usage.

## Singleton Factory

```cs
public static class MongoClientFactory
{
    static MongoClientSettings? _settings;
    static Lazy<IMongoClient> _instance = new(() => new MongoClient(_settings));

    public static IMongoClient GetClient()
    {
        return _instance.Value;
    }

    public static void Configure(string connectionString)
    {
        if (_instance.IsValueCreated)
            throw new InvalidOperationException("MongoClientFactory has already been configured.");

        _settings = MongoClientSettings.FromConnectionString(connectionString);
    }
}
```

You can still add DI if you want, but now, creation of the client with appropriate settings is not dependent on the use
of DI--e.g. when using AWS Lambda. Instead of using a factory method delegate in the registration, we use the
Abstract Factory we created.

```cs
builder.Services.AddSingleton<IMongoClient>(_ => 
{
    MongoClientFactory.Configure(builder.Configuration.GetConnectionString("DefaultConnection"));
    return MongoClientFactory.GetClient();
});
```

Or without the hoisting...

```cs
builder.Services.AddSingleton<IMongoClient>(p => 
{
    var cfg = p.GetRequiredService<IConfiguration>();
    MongoClientFactory.Configure(cfg.GetConnectionString("DefaultConnection"));
    return MongoClientFactory.GetClient();
});
```

Alternatively, we let the DAO deal with the creation.

```cs
class CustomerDao
{
    readonly IMongoClient _mongoClient;
    public CustomerDao()
    {
        _mongoClient = MongoClientFactory.GetClient();
    }

    ...
}
```

### Pros

* Ensures that no matter how it is used, there is only one instance of `MongoClient`
* Removes the dependency on an IoC container
* Clearly expresses the intent regardless of the presents of an IoC container.

### Cons

* More code to maintain for something that might never happen.

## Conclusion

The key take-away here is that there should only be a single instance of `MongoClient`. How we choose to handle this
depends on the needs of the project. My personal preference--keep it simple and just use leverage the IoC container if
building an AspNet Core app/service. 