---
title: Herts LoadBalancing 
weight: -20
---

<!--more-->

{{< toc >}}

## Herts Reactive Streaming LoadBalancing 

{{< hint type=note >}}
**Precondition**  
Your server are working on multiple machine.  
You need OSS queue system.  
Currently support **Redis**
{{< /hint >}}

<img src="/img03.png" width="600">

## Install requirements

build.gradle
```bash
def grpcVersion = 'x.x.x'
dependencies {
    implementation 'org.hertsstack:herts-core:1.0.0'
    implementation 'org.hertsstack:herts-rpc:1.0.0'
    implementation 'org.hertsstack:herts-rpc-client:1.0.0'
    implementation 'org.hertsstack:herts-broker:1.0.0'
    implementation 'org.hertsstack:herts-broker-redis:1.0.0'

    // Redis
    implementation 'redis.clients:jedis:4.3.1'

    implementation "io.grpc:grpc-protobuf:${grpcVersion}"
    implementation "io.grpc:grpc-services:${grpcVersion}"
    implementation "io.grpc:grpc-stub:${grpcVersion}"
}
```

## Quick start

### Server Interceptor implementation

You need to define broker for load balancing.

```java
public class Main {

    public static void main(String[] args) {

        // Inject other class if need it
        var service = new ReactiveStreamingServiceImpl();

        var pool = new JedisPool("localhost", 6379);
        var redisBroker = new RedisBroker(pool);
        HertsRpcServerEngine engine = HertsRpcServerEngineBuilder.builder()
                .registerHertsReactiveRpcService(service)
                .loadBalancingBroker(redisBroker) // HERE
                .build();

        engine.start();
    }
}
```

You can check log.
```bash
java -jar {Your Jar path}
2023-06-15 15:07:12.022 INFO org.hertstack.ReflectMethod printMethodName ReactiveStreaming stats
2023-06-15 15:07:12.031 INFO org.hertstack.ReflectMethod printMethodName ReactiveStreaming/registerReceiver
2023-06-15 15:07:12.033 INFO org.hertstack.ReflectMethod printMethodName ReactiveStreamingService stats
2023-06-15 15:07:12.033 INFO org.hertstack.ReflectMethod printMethodName ReactiveStreamingService/publishToRobot
2023-06-15 15:07:12.033 INFO org.hertstack.ReflectMethod printMethodName ReactiveStreamingService/getIds
2023-06-21 13:55:45.721 INFO org.hertstack.service.ReactiveStreamingBase setBroker Setup broker type is Redis 
2023-06-21 13:55:45.862 INFO org.hertstack.HertsRpcServerEngineBuilder start Started Herts RPC server. gRPC type Reactive Port 9999
```

### You need to manage ClientId(ConnectionId) for Herts reactive streaming

You can get connected **clientId**.  
Please check your implementation class.

```java

public class ReactiveStreamingServiceImpl extends HertsServiceReactiveStreaming<ReactiveStreamingService, ReactiveReceiver> implements ReactiveStreamingService {
    
    @Override
    public String publishToRobot(Hoo hoo) {
        var clientId = getClientId(); // HERE
        return uniqId;
    }

    @Override
    public List<String> getIds() {
        return Collections.singletonList("Hello");
    }

}
```

So, Server can get **ClientId(ConnectionId)** from any storage.  
Then you can push to message by **ClientId(ConnectionId)**.
