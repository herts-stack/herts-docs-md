---
title: gRPC Reactive Streaming
weight: -20
---

<!--more-->

{{< toc >}}

## gRPC Reactive Streaming

gRPC Reactive Streaming Server and Client.

**This streaming option in only Herts function**

<img src="/img09.png" width="700"/>

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

### Shared implementation for Client Receiver

You can define interface of gRPC Receiver.  
This interface uses by client.  
Required `@HertsRpcReceiver` and `extends HertsReceiver`

※ Server Steaming Interface supports `void` method only.

ReactiveReceiver.java
```java
@HertsRpcReceiver
public interface ReactiveReceiver extends HertsReceiver {

    void onReceivedData(String fromClient, String data);

}
```

Implementation of Interface.  

```java
public class ReactiveReceiverImpl implements ReactiveReceiver {

    @Override
    public void onReceivedData(String fromClient, String data) {
        System.out.println("Client: " + fromClient + ", Data: " + data);
    }

}
```

### Shared implementation for Server endpoint

You can define interface of gRPC endpoint.  
This interface uses by server and client.  
Required `@HertsRpcService` and `extends HertsReactiveService`

ReactiveStreamingService.java
```java
@HertsRpcService(value = HertsType.Reactive)
public interface ReactiveStreamingService extends HertsReactiveService {

    String publishToRobot(Hoo hoo);
    
    List<String> getIds();

}
```

Implementation of Interface.  
You should implement `extends HertsServiceReactiveStreaming<TService, TReceiver>`. Generics is your interface.

UnaryServiceImpl.java
```java

public class ReactiveStreamingServiceImpl extends HertsServiceReactiveStreaming<ReactiveStreamingService, ReactiveReceiver> implements ReactiveStreamingService {
    
    @Override
    public String publishToRobot(Hoo hoo) {
        var clientId = getClientId();
        var uniqId = UUID.randomUUID().toString();
        broadcast(clientId).onReceivedData(clientId, "Published_Data");
        return uniqId;
    }

    @Override
    public List<String> getIds() {
        return Collections.singletonList("Hello");
    }

}
```

### Server implementation

Start **gRPC BidirectionalStreaming Server**

Builder methos is `registerHertsReactiveRpcService`.

```java
public class Main {
  
    public static void main(String[] args) {

        // Inject other class if need it
        var service = new ReactiveStreamingServiceImpl();

        HertsRpcServerEngine engine = HertsRpcServerEngineBuilder.builder()
                .registerHertsReactiveRpcService(service)
                .build();

        engine.start();
    }
}
```

Run server. Default port is `9999`
```bash
java -jar {Your Jar path}
2023-06-15 15:07:12.022 INFO org.hertstack.rpc.ReflectMethod printMethodName ReactiveStreaming stats
2023-06-15 15:07:12.031 INFO org.hertstack.rpc.ReflectMethod printMethodName ReactiveStreaming/registerReceiver
2023-06-15 15:07:12.033 INFO org.hertstack.rpc.ReflectMethod printMethodName ReactiveStreamingService stats
2023-06-15 15:07:12.033 INFO org.hertstack.rpc.ReflectMethod printMethodName ReactiveStreamingService/publishToRobot
2023-06-15 15:07:12.033 INFO org.hertstack.rpc.ReflectMethod printMethodName ReactiveStreamingService/getIds
2023-06-15 13:18:35.946 INFO org.hertstack.rpc.HertsRpcServerEngineBuilder start Started Herts RPC server. gRPC type Reactive Port 9999
```

### Client implementation

Start **gRPC ReactiveStreaming Client**

You need to call `registerHertsRpcReceiver`

```java
public class Main {
  
    public static void main(String[] args) {

        HertsRpcClient client = HertsRpcClientBuilder
                .builder("localhost")
                .secure(false)
                .registerHertsRpcServiceInterface(ReactiveStreamingService.class)
                .registerHertsRpcReceiver(new ReactiveReceiver())
                .connect();

        ReactiveStreamingService service = client.createHertsRpcService(ReactiveStreamingService.java);
        
        Hoo hoo = new Hoo();
        var res = service.publishToRobot(hoo);
        System.out.println(res);

        // Wait for catch data on Receiver
        try {
            Thread.sleep(2000);
        } catch (InterruptedException ignore) {
        }
    }
}
```

Client console.
```bash
Client: FD12E285-A4B5-4685-807A-37863EC84F57, Data: Publish_Data
```
