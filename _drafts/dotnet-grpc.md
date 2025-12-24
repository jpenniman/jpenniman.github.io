---
layout: post
title: gRPC
date: 2020-03-20
image: /blog/images/blank-tile.png
author: Jason M Penniman
excerpt: gRPC
tags:
- dotnet
- C#
---


```cs
[ServiceContract]
public interface ICustomerService
{
    [OperationContract]
    string GetCustomer();
}
```

```cs
public class CustomerService : ICustomerService
{
    static int InstanceCount = 0;
    public CustomerService()
    {
        InstanceCount++;
    }
    
    public string GetCustomer()
    {
        return $"Customer {InstanceCount}";
    }
}
```



```cs
// Takes care of registering gRPC server and the protobuf-net.grpc bits
builder.Services.AddCodeFirstGrpc();

// By default, the service provider will resolve the instances as transient. If we want singleton
// or scoped, we need to explicitly register the class, not the interface, for code-first to pick it up
builder.Services.AddSingleton<OrderService>();

// If we want to resolve the interface to the same instance as the class elsewhere in the app,
// we need to register the interface separately and resolve the implementation explicitly.
builder.Services.AddSingleton<IOrderService>(p => p.GetRequiredService<OrderService>());

...

app.MapGrpcService<CustomerService>();
app.MapGrpcService<OrderService>();
```

```cs
[ServiceContract]
public interface IOrderService
{
    [OperationContract]
    string GetOrder();
}

public class OrderService : IOrderService
{
    static int InstanceCount = 0;
    public OrderService()
    {
        InstanceCount++;
    }
    
    public string GetOrder()
    {
        return $"Order {InstanceCount}";
    }
}
```

