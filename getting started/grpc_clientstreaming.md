---
title: gRPC Client Streaming 
weight: -20
---

<!--more-->

{{< toc >}}

## gRPC Client Streaming

gRPC Client Streaming Server and Client.

<img src="/img07.png" width="500"/>

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

ClientStreamingService.java
```java
import io.grpc.stub.StreamObserver;

@HertsRpcService(value = HertsType.ClientStreaming)
public interface ClientStreamingService extends HertsService {

    StreamObserver<Foo> foo(String id);

}
```

Implementation of Interface.  
You should implement `extends HertsServiceClientStreaming<T>`. Generics is your interface.

ClientStreamingServiceImpl.java
```java
public class ClientStreamingServiceImpl extends HertsServiceClientStreaming<ClientStreamingService> implements ClientStreamingService {
    
    @Override
    public StreamObserver<Foo> foo(String id) {
        System.out.println(id);
        return new StreamObserver<Foo>() {
            @Override
            public void onNext(Foo foo) {
                System.out.println(foo.getName());
            }

            @Override
            public void onError(Throwable t) {
            }

            @Override
            public void onCompleted() {
                System.out.println("Completed");
            }
        };
    }
    
}
```

### Server implementation

Start **gRPC ClientStreaming Server**

```java
public class Main {
  
    public static void main(String[] args) {

        // Inject other class if need it
        var service = new ClientStreamingServiceImpl();

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
2023-06-15 13:18:35.747 INFO org.hertstack.rpc.ReflectMethod printMethodName ClientStreamingService stats
2023-06-15 13:18:35.757 INFO org.hertstack.rpc.ReflectMethod printMethodName ClientStreamingService/foo
2023-06-15 13:18:35.946 INFO org.hertstack.rpc.HertsRpcServerEngineBuilder start Started Herts RPC server. gRPC type ClientStreaming Port 9999
```

### Client implementation

Start **gRPC Client Streaming**

```java
public class Main {
  
    public static void main(String[] args) {

        HertsRpcClient client = HertsRpcClientBuilder
                .builder("localhost")
                .secure(false)
                .registerHertsRpcServiceInterface(ClientStreamingService.class)
                .connect();

        ClientStreamingService service = client.createHertsRpcService(ClientStreamingService.class);
        
        var res = service.foo("ID");
        for (int i = 0; i < 10; i++) {
            Foo foo = new Foo();
            foo.setName("Name_" + i);
            res.onNext(foo);
        }
        res.onCompleted();
    }
}
```
