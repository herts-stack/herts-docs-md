---
title: gRPC Client automatic reconnection
weight: -20
---

<!--more-->

{{< toc >}}

## gRPC Client automatics reconnection

gRPC Client Streaming Server and Client.  
If you need to reconnect to server, You can enable this parameter.

* Server updated
* Server down
* etc

## Install requirements

build.gradle
```bash
def grpcVersion = 'x.x.x'
dependencies {
    {{< var.inline >}}{{ $.Site.Params.hertsCore }}{{< / var.inline >}}
    {{< var.inline >}}{{ $.Site.Params.hertsRpcClient }}{{< / var.inline >}}

    implementation "io.grpc:grpc-protobuf:${grpcVersion}"
    implementation "io.grpc:grpc-services:${grpcVersion}"
    implementation "io.grpc:grpc-stub:${grpcVersion}"
}
```

## Quick start

### Client Code 

You can set `.autoReconnection(true)` on builder.

```java
public class Main {
  
    public static void main(String[] args) {

        HertsRpcClient client = HertsRpcClientBuilder
                .builder("localhost")
                .secure(false)
                .registerHertsRpcServiceInterface(ClientStreamingService.class)
                .autoReconnection(true) // HERE
                .connect();

        ClientStreamingService service = client.createHertsRpcService(ClientStreamingService.class);
    }
}
```
