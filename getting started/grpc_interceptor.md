---
title: JWT Interceptor for gRPC
weight: -20
---

<!--more-->

{{< toc >}}

## JWT Interceptor for gRPC Server 
{{< hint type=note >}}
**Precondition**  
This document uses Unary gRPC.  
See: <a href="/getting-started/grpc_unary/">gRPC Unary</a>
{{< /hint >}}

gRPC JWT Interceptor Server and Client.  
The interceptor is same as all Herts streaming type.

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

UnaryService.java
```java
@HertsRpcService(value = HertsType.Unary)
public interface UnaryService extends HertsService {

    String helloWorld();

    Map<String, String> getUser(String id);

}
```

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

### Server Interceptor implementation

Herts support interceptor wrapper.  
So You can check metadata of gRPC.

JwtServerInterceptor.java
```java
public class JwtServerInterceptor implements HertsRpcInterceptor {
    private final JwtVerifier jwtProcessor;

    public JwtServerInterceptor() {
        this.jwtProcessor = new JwtVerifier();
    }

    @Override
    public void setResponseMetadata(Metadata metadata) {

    }

    /**
     * Before call Server method.
     * You can verify token this workflow
     * 
     * @param call ServerCall
     * @param requestHeaders Metadata
     */
    @Override
    public <ReqT, RespT> void beforeCallMethod(ServerCall<ReqT, RespT> call, Metadata requestHeaders) {
        Metadata.Key<String> authorization = Metadata.Key.of("Authorization", Metadata.ASCII_STRING_MARSHALLER);
        String token = requestHeaders.get(authorization);
        
        if (token == null || token.isEmpty()) {
            Status status = RpcErrorException.StatusCode.Status16
                    .convertToGrpc(RpcErrorException.StatusCode.Status16)
                    .withDescription("Unauthorized");
            
            call.close(status, requestHeaders);
            return;
        }
        if (this.jwtProcessor.verifyToken(token)) {
            Status status = RpcErrorException.StatusCode.Status16
                    .convertToGrpc(RpcErrorException.StatusCode.Status16)
                    .withDescription("Unauthorized");
            
            call.close(status, requestHeaders);
            return;
        }
    }
}
```

Start **gRPC Unary Server** with Interceptor

Main.java
```java
public class Main {
  
    public static void main(String[] args) {

        // Inject other class if need it
        var service = new UnaryServiceImpl();
        ServerInterceptor interceptor = HertsRpcInterceptBuilder.builder(new JwtServerInterceptor()).build();
        
        HertsRpcServerEngine engine = HertsRpcServerEngineBuilder.builder()
                .registerHertsRpcService(service, interceptor)
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

You need to define **io.grpc.CallCredentials** when call `createHertsRpcService`.  
Also, If you want recreate token by expires, you should  `createHertsRpcService` again.

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

        var token = "";
        service = client.createHertsRpcService(UnaryService.class, generateCredential(token));
        var res01 = service.helloWorld();
        System.out.println(res01);
    }

    private static CallCredentials generateCredential(String token) {
        
        return new CallCredentials() {
            @Override
            public void applyRequestMetadata(RequestInfo requestInfo, Executor appExecutor, MetadataApplier applier) {
                Metadata metadata = new Metadata();
                metadata.put(Metadata.Key.of("Authorization", Metadata.ASCII_STRING_MARSHALLER), "Bearer " + token);
                applier.apply(metadata);
            }

            @Override
            public void thisUsesUnstableApi() {
            }
        };
        
    }

}
```