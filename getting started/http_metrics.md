---
title: Metrics for Http Service 
weight: -20
---

<!--more-->

{{< toc >}}

## Metrics for Http Server

{{< hint type=note >}}
**Precondition**  
This document uses Http Service.  
See: <a href="/getting-started/http/">Http Service</a>
{{< /hint >}}

## Install requirements

build.gradle
```bash
def grpcVersion = 'x.x.x'
dependencies {
    implementation 'org.hertsstack:herts-core:1.0.0'
    implementation 'org.hertsstack:herts-metrics:1.0.0'
    implementation 'org.hertsstack:herts-http:1.0.0'
    implementation 'org.hertsstack:herts-http-client:1.0.0'

    implementation "io.grpc:grpc-protobuf:${grpcVersion}"
    implementation "io.grpc:grpc-services:${grpcVersion}"
    implementation "io.grpc:grpc-stub:${grpcVersion}"
}
```

## Quick start

### Server implementation

You need to create **HertsMetricsSetting**.

Main.java
```java
public class Main {

    public static void main(String[] args) {
        HertsMetricsSetting metrics = HertsMetricsSetting.builder()
                .isRpsEnabled(true)
                .isLatencyEnabled(true)
                .isErrRateEnabled(true)
                .isServerResourceEnabled(true)
                .isJvmEnabled(true)
                .build();

        // Inject other class if need it
        var httpService = new HttpServiceImpl();

        HertsHttpEngine engine = HertsHttpServer.builder()
                .registerHertsHttpService(httpService)
                .setMetricsSetting(metrics) // HERE
                .build();

        engine.start();
    }
}
```

Run server. Default Server port is `9999`.  
You can call **/metricsz** for Prometheus

```bash
java -jar {Your Jar path}
2023-06-21 11:49:34.140 INFO org.hertstack.http.HertsHttpServer start [POST]    /api/HttpService/helloWorld
2023-06-21 11:49:34.143 INFO org.hertstack.http.HertsHttpServer start [OPTIONS] /api/HttpService/helloWorld
2023-06-21 11:49:34.144 INFO org.hertstack.http.HertsHttpServer start [GET]     /metricsz
2023-06-21 11:49:34.198 INFO org.hertstack.http.HertsHttpServer start Started Herts HTTP server. Port 9999
```

Then you can get metrics.

```bash
$ curl -X GET -i localhost:9999/metricsz
HTTP/1.1 200 OK
Date: Wed, 21 Jun 2023 02:40:37 GMT
Content-Type: application/openmetrics-text; version=1.0.0; charset=utf-8
Transfer-Encoding: chunked
Server: Jetty(10.0.15)

# HELP http_requests_error_rate
# TYPE http_requests_error_rate gauge
http_requests_error_rate{MethodName="helloWorld",} 0.0
# HELP system_cpu_count The number of processors available to the Java virtual machine
# TYPE system_cpu_count gauge
system_cpu_count 10.0
# HELP jvm_buffer_memory_used_bytes An estimate of the memory that the Java virtual machine is using for this buffer pool
# TYPE jvm_buffer_memory_used_bytes gauge
jvm_buffer_memory_used_bytes{id="mapped",} 0.0
jvm_buffer_memory_used_bytes{id="direct",} 74010.0
# HELP jvm_memory_max_bytes The maximum amount of memory in bytes that can be used for memory management
# TYPE jvm_memory_max_bytes gauge
jvm_memory_max_bytes{area="nonheap",id="CodeHeap 'profiled nmethods'",} 1.22896384E8
jvm_memory_max_bytes{area="heap",id="G1 Survivor Space",} -1.0
jvm_memory_max_bytes{area="heap",id="G1 Old Gen",} 8.589934592E9
jvm_memory_max_bytes{area="nonheap",id="Metaspace",} -1.0
jvm_memory_max_bytes{area="nonheap",id="CodeHeap 'non-nmethods'",} 5849088.0
jvm_memory_max_bytes{area="heap",id="G1 Eden Space",} -1.0
jvm_memory_max_bytes{area="nonheap",id="Compressed Class Space",} 1.073741824E9
jvm_memory_max_bytes{area="nonheap",id="CodeHeap 'non-profiled nmethods'",} 1.22912768E8
# HELP jvm_buffer_count_buffers An estimate of the number of buffers in the pool
# TYPE jvm_buffer_count_buffers gauge
jvm_buffer_count_buffers{id="mapped",} 0.0
jvm_buffer_count_buffers{id="direct",} 7.0
# HELP system_load_average_1m The sum of the number of runnable entities queued to available processors and the number of runnable entities running on the available processors averaged over a period of time
# TYPE system_load_average_1m gauge
system_load_average_1m 2.24072265625
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="nonheap",id="CodeHeap 'profiled nmethods'",} 4182272.0
jvm_memory_used_bytes{area="heap",id="G1 Survivor Space",} 8388608.0
jvm_memory_used_bytes{area="heap",id="G1 Old Gen",} 0.0
jvm_memory_used_bytes{area="nonheap",id="Metaspace",} 2.709472E7
jvm_memory_used_bytes{area="nonheap",id="CodeHeap 'non-nmethods'",} 1247488.0
jvm_memory_used_bytes{area="heap",id="G1 Eden Space",} 2.3068672E7
jvm_memory_used_bytes{area="nonheap",id="Compressed Class Space",} 3024840.0
jvm_memory_used_bytes{area="nonheap",id="CodeHeap 'non-profiled nmethods'",} 952192.0
# HELP system_cpu_usage The "recent cpu usage" of the system the application is running in
# TYPE system_cpu_usage gauge
system_cpu_usage 0.0
# HELP http_requests_latency_seconds
# TYPE http_requests_latency_seconds histogram
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.001",} 0.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.001048576",} 0.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.001398101",} 0.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.001747626",} 0.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.002097151",} 0.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.002446676",} 0.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.002796201",} 0.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.003145726",} 0.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.003495251",} 0.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.003844776",} 0.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.004194304",} 0.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.005592405",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.006990506",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.008388607",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.009786708",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.011184809",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.01258291",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.013981011",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.015379112",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.016777216",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.022369621",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.027962026",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.033554431",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.039146836",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.044739241",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.050331646",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.055924051",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.061516456",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.067108864",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.089478485",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.111848106",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.134217727",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.156587348",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.178956969",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.20132659",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.223696211",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.246065832",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.268435456",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.357913941",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.447392426",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.536870911",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.626349396",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.715827881",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.805306366",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.894784851",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="0.984263336",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="1.073741824",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="1.431655765",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="1.789569706",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="2.147483647",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="2.505397588",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="2.863311529",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="3.22122547",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="3.579139411",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="3.937053352",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="4.294967296",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="5.726623061",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="7.158278826",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="8.589934591",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="10.021590356",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="11.453246121",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="12.884901886",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="14.316557651",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="15.748213416",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="17.179869184",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="22.906492245",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="28.633115306",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="30.0",} 1.0
http_requests_latency_seconds_bucket{MethodName="helloWorld",le="+Inf",} 1.0
http_requests_latency_seconds_count{MethodName="helloWorld",} 1.0
http_requests_latency_seconds_sum{MethodName="helloWorld",} 0.00500925
# HELP http_requests_latency_seconds_max
# TYPE http_requests_latency_seconds_max gauge
http_requests_latency_seconds_max{MethodName="helloWorld",} 0.00500925
# HELP jvm_memory_committed_bytes The amount of memory in bytes that is committed for the Java virtual machine to use
# TYPE jvm_memory_committed_bytes gauge
jvm_memory_committed_bytes{area="nonheap",id="CodeHeap 'profiled nmethods'",} 4259840.0
jvm_memory_committed_bytes{area="heap",id="G1 Survivor Space",} 8388608.0
jvm_memory_committed_bytes{area="heap",id="G1 Old Gen",} 4.8234496E8
jvm_memory_committed_bytes{area="nonheap",id="Metaspace",} 2.8360704E7
jvm_memory_committed_bytes{area="nonheap",id="CodeHeap 'non-nmethods'",} 2555904.0
jvm_memory_committed_bytes{area="heap",id="G1 Eden Space",} 4.6137344E7
jvm_memory_committed_bytes{area="nonheap",id="Compressed Class Space",} 3538944.0
jvm_memory_committed_bytes{area="nonheap",id="CodeHeap 'non-profiled nmethods'",} 2555904.0
# HELP process_cpu_usage The "recent cpu usage" for the Java Virtual Machine process
# TYPE process_cpu_usage gauge
process_cpu_usage 0.0
# HELP http_requests_count_total
# TYPE http_requests_count_total counter
http_requests_count_total{MethodName="helloWorld",} 1.0
# HELP jvm_buffer_total_capacity_bytes An estimate of the total capacity of the buffers in this pool
# TYPE jvm_buffer_total_capacity_bytes gauge
jvm_buffer_total_capacity_bytes{id="mapped",} 0.0
jvm_buffer_total_capacity_bytes{id="direct",} 74010.0
```