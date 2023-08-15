---
title: Features
weight: -20
---

<!--more-->

{{< toc >}}

# Herts stack

<img src="/img16.png" width="600">

## Types of APIs that can be implemented

* gRPC Unary communication
* gRPC Client Steaming communication
* gRPC Server Streaming communication
* gRPC Bidirectional Streaming communication
* gRPC Reactive Streaming communication (Same as Bidirectional Streaming. This is original of Herts)
* HTTP(REST API) communication

## Interceptor on Server

Intercept data before call method.

* Add authorized header on interceptor
* Encrypt and Decrypt data on interceptor
* Out put log on interceptor
* Error handling
* Retry processing

<img src="/img02.png" width="600">

## Original gRPC Load balancing
**Herts supports Herts Load balancing.**  
Load balance bi-directional communication via external queues. So you need to use ConnectionId which can be obtained from Herts.

<img src="/img03.png" width="600">

**※ See About gRPC load balancing**

Please refer to the following document for the load balancing method of grpc.  
See: https://grpc.io/blog/grpc-load-balancing/

## Metrics with Java Micrometer

By turning on the metrics function, you can collect metrics exclusively for **Prometheus**.

* Call Request per seconds
  * per definition Method
* Call RPC Command per second
    * per definition Method
* Latency for HTTP and gRPC method
    * per definition Method
* JVM metrics
* Machine metrics

## Gateway between gRPC and HTTP

It supports Gateway for easy connection to grpc.

## HTTP Client Code generation

Http Client code generation for Typescript from `HertsService interface`.

Supports codegen  
* Typescript
