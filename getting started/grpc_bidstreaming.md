---
title: gRPC Bid Streaming 
weight: -20
---

<!--more-->

{{< toc >}}

## gRPC Bidirectional Streaming

gRPC Bidirectional Streaming Server and Client.

<img src="/img08.png" width="500"/>

## Install requirements

build.gradle
```bash
def grpcVersion = 'x.x.x'
dependencies {
    {{< var.inline >}}{{ $.Site.Params.hertsCore }}{{< / var.inline >}}
    {{< var.inline >}}{{ $.Site.Params.hertsRpc}}{{< / var.inline >}}
    {{< var.inline >}}{{ $.Site.Params.hertsRpcClient }}{{< / var.inline >}}

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

※ Server Steaming Interface supports `StreamObserver` method only. Also, Required `StreamObserver` on method parameter

ServerStreamingService.java
```java
import io.grpc.stub.StreamObserver;

@HertsRpcService(value = HertsType.BidirectionalStreaming)
public interface BidirectionalStreamingService extends HertsService {

    StreamObserver<Hoo> hoo(final StreamObserver<Hoo> responseObserver);

}
```

Implementation of Interface.  
You should implement `extends HertsServiceServerStreaming<T>`. Generics is your interface.

UnaryServiceImpl.java
```java
public class BidirectionalStreamingServiceImpl extends HertsServiceBidirectionalStreaming<BidirectionalStreamingService> implements BidirectionalStreamingService {
    
    @Override
    public StreamObserver<Hoo> hoo(final StreamObserver<Foo> responseObserver) {
        return new StreamObserver<Hoo>() {
            @Override
            public void onNext(Hoo hoo) {
                Foo foo = new Foo();
                foo.setName("I am foo");
                responseObserver.onNext(foo);
            }

            @Override
            public void onError(Throwable t) {
            }

            @Override
            public void onCompleted() {
                responseObserver.onCompleted();
            }
        };
    }
    
}
```

### Server implementation

Start **gRPC BidirectionalStreaming Server**

```java
public class Main {
  
    public static void main(String[] args) {

        // Inject other class if need it
        var service = new BidirectionalStreamingServiceImpl();

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
2023-06-15 13:18:35.747 INFO org.hertstack.rpc.ReflectMethod printMethodName BidirectionalStreamingService stats
2023-06-15 13:18:35.757 INFO org.hertstack.rpc.ReflectMethod printMethodName BidirectionalStreamingService/hoo
2023-06-15 13:18:35.946 INFO org.hertstack.rpc.HertsRpcServerEngineBuilder start Started Herts RPC server. gRPC type BidirectionalStreaming Port 9999
```

### Client implementation

Start **gRPC BidirectionalStreaming Client**

```java
public class Main {
  
    public static void main(String[] args) {

        HertsRpcClient client = HertsRpcClientBuilder
                .builder("localhost")
                .secure(false)
                .registerHertsRpcServiceInterface(BidirectionalStreamingService.class)
                .connect();

        BidirectionalStreamingService service = client.createHertsRpcService(BidirectionalStreamingService.class);
        
        var res = service.foo(new StreamObserver<Foo>() {
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
        
        for (var i = 0; i < 10; i++) {
            Hoo hoo = new Hoo();
            hoo.setName("I am Hoo_" + i);
            res.onNext(hoo);
        }
        res.onCompleted();
    }
}
```
