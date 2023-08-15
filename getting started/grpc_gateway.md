---
title: Gateway
weight: -19
---

<!--more-->

{{< toc >}}

## Herts gRPC Gateway

Herts supports a gateway for gRPC.  
Http Client -> Http Server -> gRPC Server.

## Install requirements

Gateway supports to connect **gRPC with unary server** only.

build.gradle

```bash
def grpcVersion = 'x.x.x'
dependencies {
    {{< var.inline >}}{{ $.Site.Params.hertsGateway }}{{< / var.inline >}}
    {{< var.inline >}}{{ $.Site.Params.hertsHttp }}{{< / var.inline >}}
    {{< var.inline >}}{{ $.Site.Params.hertsHttpClient }}{{< / var.inline >}}
    {{< var.inline >}}{{ $.Site.Params.hertsMetrics }}{{< / var.inline >}}

    implementation "io.grpc:grpc-protobuf:${grpcVersion}"
    implementation "io.grpc:grpc-services:${grpcVersion}"
    implementation "io.grpc:grpc-stub:${grpcVersion}"
}
```

## Quick start

### gRPC Server

gRPC server interface definition.

```java

@HertsRpcService(value = HertsType.Unary)
public interface UserService extends HertsService {

  Map<String, String> getUser(String id);
}
```

Implementation class of interface.

```java
public class UserServiceImpl extends HertsServiceUnary<UserService> implements UserService {

  public UserServiceImpl() {
  }

  public Map<String, String> getUser(String id) {
    return Collections.singletonMap("name", "Foo");
  }
}
```

Start gRPC unary server.

```java
public class Main {

  public static void main(String[] args) {
    HertsMetricsSetting metrics = HertsMetricsSetting.builder()
        .isRpsEnabled(true)
        .isLatencyEnabled(true)
        .build();

    UserService userService = new UserServiceImpl();

    HertsRpcServerEngine engine = HertsRpcServerEngineBuilder.builder()
        .registerHertsRpcService(userService)
        .enableMetrics(metrics)
        .build();

    engine.start();
  }
}
```

### Gateway Server

Start a gateway by UserService interface.   
Gateway is automated a binding interface.

```java
public class Main {

  public static void main(String[] args) {
    HertsGatewayEngine engine = HertsGatewayBuilder.builder()
        .gatewayPort(9876)
        .rpcHost("localhost")
        .rpcPort(9999)
        .registerHertsRpcService(UserService.class, null)
        .build();

    engine.start();
  }
}
```

### Gateway Client

You can call Gateway server by HertsHttpClient package.  
You should call `gatewayApi(true)` on builder

```java
public class Main {
  public static void main(String[] args) {
    
    HertsHttpClient client = HertsHttpClient
        .builder("localhost")
        .registerHertsService(UserService.class)
        .secure(false)
        .gatewayApi(true) 
        .port(9876)
        .build();

    UserService service = client.createHertsService(UserService.class);
    var res = service.getUser("id");
    System.out.println(res);
  }
}
```