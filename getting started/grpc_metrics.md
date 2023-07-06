---
title: Metrics for gRPC 
weight: -20
---

<!--more-->

{{< toc >}}

## Metrics for gRPC Server

{{< hint type=note >}}
**Precondition**  
This document uses Unary gRPC.  
See: <a href="/getting-started/grpc_unary/">gRPC Unary</a>
{{< /hint >}}

## Install requirements

build.gradle
```bash
def grpcVersion = 'x.x.x'
dependencies {
    implementation 'org.hertsstack:herts-core:1.0.0'
    implementation 'org.hertsstack:herts-metrics:1.0.0'
    implementation 'org.hertsstack:herts-rpc:1.0.0'
    implementation 'org.hertsstack:herts-rpc-client:1.0.0'

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
        
        // Metrics build
        HertsMetricsSetting metrics = HertsMetricsSetting.builder()
                .isRpsEnabled(true)     // Measure Rps
                .isLatencyEnabled(true) // Measure Latency
                .isErrRateEnabled(true) // Measure Error rate
                .isJvmEnabled(true)     // Measure JVM metric
                .build();

        // Inject other class if need it
        var service = new UnaryServiceImpl();
        
        HertsRpcServerEngine engine = HertsRpcServerEngineBuilder.builder()
                .registerHertsRpcService(service)
                .enableMetrics(metrics) // HERE
                .build();

        engine.start();
    }
}
```

Run server. Default Server port is `9999`, Metrics server port is `8888`.  
You can call **/metricsz** for Prometheus
```bash
java -jar {Your Jar path}
2023-06-15 13:18:35.747 INFO org.hertstack.rpc.ReflectMethod printMethodName UnaryService stats
2023-06-15 13:18:35.757 INFO org.hertstack.rpc.ReflectMethod printMethodName UnaryService/helloWorld
2023-06-15 13:18:35.757 INFO org.hertstack.rpc.ReflectMethod printMethodName UnaryService/getUser
2023-06-15 13:18:35.946 INFO org.hertstack.rpc.HertsRpcServerEngineBuilder start Started Herts RPC server. gRPC type Unary Port 9999
2023-06-20 17:18:00.138 INFO org.hertstack.metrics.HertsMetricsServer start Started Herts metrics server. Port 8888
```

Then you can get metrics.

```bash
$ curl -i localhost:8888/metricsz

HTTP/1.1 200 OK
Date: Tue, 20 Jun 2023 08:18:59 GMT
Content-Type: application/openmetrics-text; version=1.0.0; charset=utf-8
Transfer-Encoding: chunked
Server: Jetty(10.0.15)

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
# HELP jvm_memory_used_bytes The amount of used memory
# TYPE jvm_memory_used_bytes gauge
jvm_memory_used_bytes{area="nonheap",id="CodeHeap 'profiled nmethods'",} 3463680.0
jvm_memory_used_bytes{area="heap",id="G1 Survivor Space",} 6291456.0
jvm_memory_used_bytes{area="heap",id="G1 Old Gen",} 0.0
jvm_memory_used_bytes{area="nonheap",id="Metaspace",} 3.0093288E7
jvm_memory_used_bytes{area="nonheap",id="CodeHeap 'non-nmethods'",} 1266560.0
jvm_memory_used_bytes{area="heap",id="G1 Eden Space",} 4.4040192E7
jvm_memory_used_bytes{area="nonheap",id="Compressed Class Space",} 3505856.0
jvm_memory_used_bytes{area="nonheap",id="CodeHeap 'non-profiled nmethods'",} 759424.0
# HELP jvm_buffer_memory_used_bytes An estimate of the memory that the Java virtual machine is using for this buffer pool
# TYPE jvm_buffer_memory_used_bytes gauge
jvm_buffer_memory_used_bytes{id="mapped",} 0.0
jvm_buffer_memory_used_bytes{id="direct",} 6299928.0
# HELP rpc_cmd_error_rate
# TYPE rpc_cmd_error_rate gauge
rpc_cmd_error_rate{MethodName="helloWorld",} 0.0
# HELP rpc_cmd_count_total
# TYPE rpc_cmd_count_total counter
rpc_cmd_count_total{MethodName="helloWorld",} 1.0
# HELP jvm_memory_committed_bytes The amount of memory in bytes that is committed for the Java virtual machine to use
# TYPE jvm_memory_committed_bytes gauge
jvm_memory_committed_bytes{area="nonheap",id="CodeHeap 'profiled nmethods'",} 3473408.0
jvm_memory_committed_bytes{area="heap",id="G1 Survivor Space",} 6291456.0
jvm_memory_committed_bytes{area="heap",id="G1 Old Gen",} 4.80247808E8
jvm_memory_committed_bytes{area="nonheap",id="Metaspace",} 3.0982144E7
jvm_memory_committed_bytes{area="nonheap",id="CodeHeap 'non-nmethods'",} 2555904.0
jvm_memory_committed_bytes{area="heap",id="G1 Eden Space",} 5.0331648E7
jvm_memory_committed_bytes{area="nonheap",id="Compressed Class Space",} 3801088.0
jvm_memory_committed_bytes{area="nonheap",id="CodeHeap 'non-profiled nmethods'",} 2555904.0
# HELP rpc_cmd_latency_seconds
# TYPE rpc_cmd_latency_seconds histogram
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.001",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.001048576",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.001398101",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.001747626",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.002097151",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.002446676",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.002796201",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.003145726",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.003495251",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.003844776",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.004194304",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.005592405",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.006990506",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.008388607",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.009786708",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.011184809",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.01258291",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.013981011",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.015379112",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.016777216",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.022369621",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.027962026",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.033554431",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.039146836",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.044739241",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.050331646",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.055924051",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.061516456",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.067108864",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.089478485",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.111848106",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.134217727",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.156587348",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.178956969",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.20132659",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.223696211",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.246065832",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.268435456",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.357913941",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.447392426",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.536870911",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.626349396",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.715827881",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.805306366",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.894784851",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="0.984263336",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="1.073741824",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="1.431655765",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="1.789569706",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="2.147483647",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="2.505397588",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="2.863311529",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="3.22122547",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="3.579139411",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="3.937053352",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="4.294967296",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="5.726623061",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="7.158278826",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="8.589934591",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="10.021590356",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="11.453246121",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="12.884901886",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="14.316557651",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="15.748213416",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="17.179869184",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="22.906492245",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="28.633115306",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="30.0",} 1.0
rpc_cmd_latency_seconds_bucket{MethodName="helloWorld",le="+Inf",} 1.0
rpc_cmd_latency_seconds_count{MethodName="helloWorld",} 1.0
rpc_cmd_latency_seconds_sum{MethodName="helloWorld",} 1.43459E-4
# HELP rpc_cmd_latency_seconds_max
# TYPE rpc_cmd_latency_seconds_max gauge
rpc_cmd_latency_seconds_max{MethodName="helloWorld",} 1.43459E-4
# HELP jvm_buffer_total_capacity_bytes An estimate of the total capacity of the buffers in this pool
# TYPE jvm_buffer_total_capacity_bytes gauge
jvm_buffer_total_capacity_bytes{id="mapped",} 0.0
jvm_buffer_total_capacity_bytes{id="direct",} 6299927.0
```