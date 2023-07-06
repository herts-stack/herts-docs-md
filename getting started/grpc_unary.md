---
title: gRPC Unary
weight: -20
---

<!--more-->

{{< toc >}}

## gRPC Unary

gRPC Unary Server and Client.

<img src="/img05.png" width="500"/>

## Install requirements

build.gradle
```bash
dependencies {
    implementation 'org.hertsstack:herts-core:1.0.0'
    implementation 'org.hertsstack:herts-rpc:1.0.0'
    implementation 'org.hertsstack:herts-rpc-client:1.0.0'
}
```

## Quick start

### Shared implementation

You can define interface of gRPC endpoint.  
This interface uses by server and client.  
Required `@HertsRpcService` and `extends HertsService`

UnaryService.java
```java
@HertsRpcService(value = HertsType.Unary)
public interface UnaryService extends HertsService {

    String helloWorld();

    Map<String, String> getUser(String id);

}
```

Implementation of Interface.  
You should implement `extends HertsServiceUnary<T>`. Generics is your interface.

UnaryServiceImpl.java
```java
public class UnaryServiceImpl extends HertsServiceUnary<UnaryService> implements UnaryService {
    
    @Override
    public String helloWorld() {
        return "hello world";
    }
    
    @Overide
    public Map<String, String> getUser(String id) {
        return Collections.singletonMap("name", "foo");
    }
}
```

### Server implementation

Start **gRPC Unary Server**

```java
public class Main {
  
    public static void main(String[] args) {

        // Inject other class if need it
        var service = new UnaryServiceImpl();

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
2023-06-15 13:18:35.747 INFO org.hertstack.rpc.ReflectMethod printMethodName UnaryService stats
2023-06-15 13:18:35.757 INFO org.hertstack.rpc.ReflectMethod printMethodName UnaryService/helloWorld
2023-06-15 13:18:35.757 INFO org.hertstack.rpc.ReflectMethod printMethodName UnaryService/getUser
2023-06-15 13:18:35.946 INFO org.hertstack.rpc.HertsRpcServerEngineBuilder start Started Herts RPC server. gRPC type Unary Port 9999
```

### Client implementation

Start **gRPC Unary Client**

```java
public class Main {
  
    public static void main(String[] args) {

        HertsRpcClient client = HertsRpcClientBuilder
                .builder("localhost")
                .secure(false)
                .registerHertsRpcServiceInterface(UnaryService.class)
                .connect();

        UnaryService service = client.createHertsRpcService(UnaryService.class);
        
        var res01 = service.helloWorld();
        System.out.println(res01);

        var res02 = service.getUser("ID");
        System.out.println(res02);
    }
}
```
