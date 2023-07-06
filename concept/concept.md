---
title: Concept
weight: -20
---

<!--more-->

{{< toc >}}

## What is Herts real time framework
Herts is a framework for network communication.

It supports two communication protocols, HTTP2 and HTTP and internally uses the `grpc` and `servlet` package.

## Why use Herts framework
1. The defined Interface will be the endpoint as it is.
2. Everything can be defined as a Java Object, so it is complete in Java and easy to share
3. It makes load balancing easy by offering Herts dedicated load balancing.

Sample shared interface definition.  
The endpoint of the server is started based on the below Interface.  
Also, The client simply calls the below Interface.

<img src="/img01.png" >

## Point
* You can call server functions without being aware of the communication layer
* You only need to define the communication data structure in Java
* StreamObserver from the existing grpc package can be used as is
* Support HTTP Server and gRPC server