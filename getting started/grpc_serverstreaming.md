---
title: gRPC Server Streaming 
weight: -20
---

<!--more-->

{{< toc >}}

## gRPC Server Streaming

gRPC Server Streaming Server and Client.

<img src="/img06.png" width="500"/>

## Install requirements

build.gradle
```bash
def grpcVersion = 'x.x.x'
dependencies {
    implementation 'org.hertsstack:herts-core:1.0.0'
    implementation 'org.hertsstack:herts-rpc:1.0.0'
    implementation 'org.hertsstack:herts-rpc-client:1.0.0'
    
    implementation "io.grpc:grpc-protobuf:${grpcVersion}"
    implementation "io.grpc:grpc-services:${grpcVersion}"
    implementation "io.grpc:grpc-stub:${grpcVersion}"
}
```

## Quick start

### Shared implementation

You can define interface of gRPC endpoint.  
This interface uses by server and client.  
Required `@HertsRpcService` and `extends HertsService`

※ Server Steaming Interface supports `void` method only. Also, Required `StreamObserver` on method parameter

ServerStreamingService.java
```java
import io.grpc.stub.StreamObserver;

@HertsRpcService(value = HertsType.ServerStreaming)
public interface ServerStreamingService extends HertsService {
    
    void foo(String id, final StreamObserver<Foo> responseObserver);

}
```

Implementation of Interface.  
You should implement `extends HertsServiceServerStreaming<T>`. Generics is your interface.

UnaryServiceImpl.java
```java

public class ServerStreamingServiceImpl extends HertsServiceServerStreaming<ServerStreamingService> implements ServerStreamingService {
    
    @Override
    public void foo(String id, final StreamObserver<Foo> responseObserver) {
        for (int i = 1; i <= 10; i++) {
            Foo foo = new Foo();
            foo.setName("name_" + id + "_" + i);
            responseObserver.onNext(foo);
        }
        responseObserver.onCompleted();
    }
    
}
```

### Server implementation

Start **gRPC ServerStreaming Server**

```java
public class Main {
  
    public static void main(String[] args) {

        // Inject other class if need it
        var service = new ServerStreamingServiceImpl();

        HertsRpcServerEngine engine = HertsRpcServerEngineBuilder.builder()
                .registerHertsRpcService(service)
                .build();

        engine.start();
    }
}
```

Run server. Default port is `9999`
```bash
java -jar {Your Jar path}
2023-06-15 13:18:35.747 INFO org.hertstack.rpc.ReflectMethod printMethodName ServerStreamingService stats
2023-06-15 13:18:35.757 INFO org.hertstack.rpc.ReflectMethod printMethodName ServerStreamingService/foo
2023-06-15 13:18:35.946 INFO org.hertstack.rpc.HertsRpcServerEngineBuilder start Started Herts RPC server. gRPC type ServerStreaming Port 9999
```

### Client implementation

Start **gRPC Unary Client**

```java
public class Main {
  
    public static void main(String[] args) {

        HertsRpcClient client = HertsRpcClientBuilder
                .builder("localhost")
                .secure(false)
                .registerHertsRpcServiceInterface(ServerStreamingService.class)
                .connect();

        ServerStreamingService service = client.createHertsRpcService(ServerStreamingService.class);
        
        service.foo("ABC_id", new StreamObserver<Foo>() {
            @Override
            public void onNext(Foo res) {
                System.out.println(res.getName());
            }

            @Override
            public void onError(Throwable t) {
                t.printStackTrace();
            }

            @Override
            public void onCompleted() {
                System.out.println("Completed");
            }
        });
    }
}
```
