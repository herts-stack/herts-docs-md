---
title: JWT Interceptor for Http Service 
weight: -20
---

<!--more-->

{{< toc >}}

## JWT Interceptor for Http Server

{{< hint type=note >}}
**Precondition**  
This document uses Http Service.  
See: <a href="/getting-started/http/">Http Service</a>
{{< /hint >}}

Http JWT Interceptor Server and Client.  

## Install requirements

build.gradle
```bash
def grpcVersion = 'x.x.x'
dependencies {
    {{< var.inline >}}{{ $.Site.Params.hertsCore }}{{< / var.inline >}}
    {{< var.inline >}}{{ $.Site.Params.hertsHttp }}{{< / var.inline >}}
    implementation 'org.hertsstack:herts-http-client:1.0.0'

    implementation "io.grpc:grpc-protobuf:${grpcVersion}"
    implementation "io.grpc:grpc-services:${grpcVersion}"
    implementation "io.grpc:grpc-stub:${grpcVersion}"
}
```

## Quick start

### Shared implementation

HttpService.class
```java
@HertsHttp
public interface HttpService extends HertsService {
  
    String helloWorld();
    
    Map<String, String> getUser(String id);
    
}
```

HttpServiceImpl.class
```java
public class HttpServiceImpl extends HertsServiceHttp<HttpService> implements HttpService {
    
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

Server side interceptor.

JwtServerInterceptor.java
```java
public class JwtServerInterceptor implements HertsHttpInterceptor {
    private final JwtVerifier jwtVerifier;

    public JwtServerInterceptor() {
        this.jwtVerifier = new JwtVerifier();
    }

    @Override
    public void beforeHandle(HertsHttpRequest request) {
        var token = request.getHeader("Authorization");
        if (token == null || token.isEmpty()) {
            throw new HttpErrorException(HttpErrorException.StatusCode.Status401, "Unauthorized");
        }
        if (!this.jwtVerifier.verifyToken(token)) {
            throw new HttpErrorException(HttpErrorException.StatusCode.Status401, "Unauthorized");
        }
    }

    @Override
    public void afterHandle() {
    }
}
```

Start **Herts HTTP Server**

```java
public class Main {
  
    public static void main(String[] args) {

        // Inject other class if need it
        var httpService = new HttpServiceImpl();
        var interceptor = new JwtServerInterceptor();

        HertsHttpEngine engine = HertsHttpServer.builder()
            .registerHertsHttpService(httpService, interceptor)
            .build();

        engine.start();
    }
}
```

Run server. Default port is `9999`
```bash
java -jar {Your Jar path}
2023-06-15 13:16:06.793 INFO org.hertstack.http.HertsHttpServer start HttpServiceImpl endpoint.
2023-06-15 13:16:06.796 INFO org.hertstack.http.HertsHttpServer start [POST]    /api/HttpService/helloWorld
2023-06-15 13:16:06.796 INFO org.hertstack.http.HertsHttpServer start [OPTIONS] /api/HttpService/helloWorld
2023-06-15 13:16:06.796 INFO org.hertstack.http.HertsHttpServer start [POST]    /api/HttpService/getUser
2023-06-15 13:16:06.796 INFO org.hertstack.http.HertsHttpServer start [OPTIONS] /api/HttpService/getUser
2023-06-15 13:16:06.856 INFO org.hertstack.http.HertsHttpServer start Started Herts HTTP server. Port 9999
```

### Client implementation

You need to define **Map(Header key/value)** when call `createHertsService`.  
Also, If you want recreate token by expires, you should  `createHertsService` again.

Start **Herts HTTP Client**

```java
public class Main {

    public static void main(String[] args) {

        HertsHttpClient client = HertsHttpClient
                .builder("localhost")
                .registerHertsService(HttpService.class)
                .secure(false)
                .build();

        // Create interface instance
        var token = ""; 
        var httpService = client.createHertsService(HttpService.class, Collections.singletonMap("Authorization", token));

        var res01 = httpService.helloWorld();
        System.out.println(res01);
    }
}
```