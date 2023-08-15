---
title: Http Service
weight: -15
---

<!--more-->

{{< toc >}}

## HTTP Service
Simple HTTP Server and Client.

<img src="/img04.png" width="500"/>

## Install requirements

build.gradle
```bash
dependencies {
    {{< var.inline >}}{{ $.Site.Params.hertsCore }}{{< / var.inline >}}
    {{< var.inline >}}{{ $.Site.Params.hertsHttp }}{{< / var.inline >}}
    implementation 'org.hertsstack:herts-http-client:1.0.0'
}
```

## Quick start

### Shared implementation

You can define interface of HTTP API endpoint.  
This interface uses by server and client.  
Required `@HertsHttp` and `extends HertsService`

HttpService.class
```java
@HertsHttp
public interface HttpService extends HertsService {
  
    String helloWorld();
    
    Map<String, String> getUser(String id);
    
}
```

Implementation of Interface.  
You should implement `extends HertsServiceHttp<T>`. Generics is your interface.

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

### Server implementation

Start **Herts HTTP Server**

```java
public class Main {
  
    public static void main(String[] args) {

        // Inject other class if need it
        var httpService = new HttpServiceImpl();
        
        HertsHttpEngine engine = HertsHttpServer.builder()
            .registerHertsHttpService(httpService)
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
        var httpService = client.createHertsService(HttpService.class);
        
        var res01 = httpService.helloWorld();
        System.out.println(res01);

        var res02 = httpService.getUser("ID");
        System.out.println(res02);
    }
}
```
