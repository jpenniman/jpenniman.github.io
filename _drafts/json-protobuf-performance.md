---
layout: post
title: JSON vs Protobuf Performance and Cost Comparison 
date: 2026-03-28
image: /blog/images/json-vs-protobuf.png
author: Jason M Penniman
excerpt: Communication between services can be costly in more ways than one. Let's compare Rest/JSON and gRPC/Protobuf. 
tags:
- dotnet
- C#
- gRPC
- Protobuf-Net
- protobuf
- json
---
# Introduction

Any time we're working with a distributed system, some of those distributed components often need to talk to each other.
That communication is costly in more ways than one. There is the compute and memory cost of serialization overhead
impacting speed of the application at scale and the monetary cost of the network traffic when running in a public cloud such as
AWS or Azure.

The two most common serialization formats for out-of-process inter-service communication are JSON and Protobuf. The
former most commonly used with Rest, GraphQL, or JSON-API, and the later gRPC.

**JSON** is a string serialization format that is extremely portable--easily readable and writable by any language.

**Protobuf** is a portable binary format. Unlike binary serialization of the past where we needed exact binary compatible
type libraries on both ends, and usually the same language, protobuf does not have these constraints. We can have a Go
service call a dotnet service call a Java service and all works.

Let's see how their costs stack up against each other.

# The Comparison

For all these tests, I serialized an array of 100 fully populated Customer objects as defined below. For JSON
serialization I used System.Text.Json and for protobuf I used Protobuf-Net by Marc Gravell 
[https://github.com/protobuf-net/protobuf-net](https://github.com/protobuf-net/protobuf-net).

A real-world application for this simulation would be an application's backend-for-frontend or a Blazor web application
hosted in a DMZ or public subnet calling a backend microservice/monolith in a private network.

```cs
[ProtoContract]
public class Customer
{
    [ProtoMember(1)]
    public Guid Id { get; set; }
    [ProtoMember(2)]
    public string Name { get; set; } = string.Empty;
    [ProtoMember(3)]
    public int Age { get; set; }
    [ProtoMember(4)]
    public string Email { get; set; } = string.Empty;
    [ProtoMember(5)]
    public string Phone { get; set; } = string.Empty;
    [ProtoMember(6)]
    public string Address { get; set; } = string.Empty;
    [ProtoMember(7)]
    public string City { get; set; } = string.Empty;
    [ProtoMember(8)]
    public string State { get; set; } = string.Empty;
    [ProtoMember(9)]
    public string Zip { get; set; } = string.Empty;
}
```

## Raw Serialization Performance

I first looked at serialization/deserialization performance.

| Method                     | Mean     | Error    | StdDev   | Gen0   | Gen1   | Allocated |
|--------------------------- |---------:|---------:|---------:|-------:|-------:|----------:|
| ProtobufNet_Serialize      | 19.48 us | 0.038 us | 0.034 us | 3.4790 | 0.1831 |  43.02 KB |   
| ProtobufNet_Deserialize    | 24.61 us | 0.094 us | 0.088 us | 3.3875 | 0.4578 |  41.51 KB |
| SystemTextJson_Serialize   | 22.28 us | 0.046 us | 0.039 us | 3.1738 |      - |  39.34 KB |
| SystemTextJson_Deserialize | 42.41 us | 0.066 us | 0.055 us | 3.4790 | 0.4272 |  43.16 KB |

We can see that in .Net 10, System.Text.Json is nearly as fast as Protobuf.Net for serializing.
Deserialization, however, Protobuf.Net is nearly 2X faster.

## What about size?

The size of the resulting payload is where the real money is. Amazon and Azure, for example, charge for traffic between
availability zones.

I used raw JSON as the baseline since that is the default for AspNet Core and the most common devs use.

| Test                       | Size (bytes) | Improvement
|----------------------------|-------------:|------------:
| JSON                       | 19,977        | baseline
| Protobuf                   | 10,776        | 46.1%
| JSON Gzipped (Fastest)     | 5,433         | 72.8%
| JSON Gzipped (Optimal)     | 3,424         | 82.9%
| Protobuf Gzipped (Fastest) | 3,321         | 83.4%
| Protobuf Gzipped (Optimal) | 2,992         | 85.0%

We can see that protobuf is nearly half the size as the JSON payload. Twice and fast and half the size--not a bad gain.

Where it gets interesting is when we throw compression into the mix. If we gzip the payload, the difference becomes
marginal at optimal compression, but is still statistically significant (10%) using fastest compression when compared
to raw JSON.

Comparing just the compression on its own, we can see gzipped protobuf payloads are still quite a bit smaller.

| Test                       | Size (bytes) | Improvement
|----------------------------|-------------:|------------:
| JSON Gzipped (Fastest)     | 5,433         | baseline
| JSON Gzipped (Optimal)     | 3,424         | 37.0%
| Protobuf Gzipped (Fastest) | 3,321         | 38.9%
| Protobuf Gzipped (Optimal) | 2,992         | 44.9%

## Why Does Size Matter?

Well, let's put some dollar amounts to it. AWS and Azure charge $0.02 per GB of transfer between
availability zones ($0.01 to leave AZ 1 and $0.01 to enter AZ 2).

Let's assume 1,000,000 requests per day for this payload. That makes the baseline, 19.977 GB per day or 599 GB per
month. That may sound like a lot of requests, but for an 8-hour peak period, that is only ~35 requests per second.

| Test                       | Size (bytes)  | Improvement | Cost/Month<br/>(USD) | Monthly<br/>Savings | Annual<br/>Savings
|----------------------------|--------------:|------------:|------------:|-------------:|------------:
| JSON                       | 19,977        | baseline    |     $11.99 |           -- | --
| Protobuf                   | 10,776        | 46.1%       |      $8.21 |       $3.78 | $45.32
| JSON Gzipped (Fastest)     | 5,433         | 72.8%       |      $6.93 |       $5.06 | $60.70
| JSON Gzipped (Optimal)     | 3,424         | 82.9%       |      $6.55 |       $5.44 | $65.23
| Protobuf Gzipped (Fastest) | 3,321         | 83.4%       |      $6.54 |       $5.46 | $65.41
| Protobuf Gzipped (Optimal) | 2,992         | 85.0%       |      $6.48 |       $5.51 | $66.08

So, an optimal Gzipped protobuf payload saves ~$5.50 a month or ~$66 a year on data transfer. But this is just this
one API. Imagine similar loads across 100s of APIs in a larger enterprise system and 100s of requests per second.

Something to highlight is the gzipped JSON savings. AWS egress--data transferred from your app running on AWS to your
user's web browser--is charged at $0.09 per GB. So these 1M requests per day for a paged result of customers uncompressed
is $54 per month. We can reduce that by 70%-80% just enabling Gzip response compression. Yes it sounds small, but again,
this is one API and only 35rps. These costs quickly become 10s of thousands of dollars a month for even a modest SaaS
application. 

# Not Just Service-To-Service

Messaging and Caching platforms such as RabbitMQ, Kafka, and Redis work with byte arrays and fall subject to the same
network transfer costs as well as storage costs. If your Kafka message is 80% smaller, you're saving 80% on your Kakfa
storage.

# Conclusion

Even the simplest Architecture decisions such as do we use gRPC or JSON between services or do we enable gzip
compression on our endpoints can have a significant impact on your cloud spend. We cut our data transfer costs nearly in half by choosing gRPC + Protobuf over Rest + JSON and a 70%-80% reduction in costs simply enabling compression with
either format.

The difference between JSON Gzip Optimal and Protobuf Gzip Fast is negligible and may not offset the additional
compute required. 

Weigh your options carefully and consider the best options for your project and wallet. My general rule of thumb for any
sizable SaaS:
- gRPC + Protobuf between services
- Protobuf for message serialization for RabbitMQ, Kafka, Redis/Valkey.
- Enable response compression at your egress points

Cheers!
