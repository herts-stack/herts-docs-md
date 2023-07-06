---
title: gRPC Unary 
weight: -20
---

<!--more-->

{{< toc >}}

## Result
**Command per seconds**

<img src="/img15.png" width="500"/>

**Latency**

<img src="/img13.png" width="500"/>

## Environment

**Server and Client are working on same machine**

* Server Machine
    * OS: `Linux version 5.4.0-148-generic (buildd@lcy02-amd64-112) (gcc version 9.4.0 (Ubuntu 9.4.0-1ubuntu1~20.04.1)) #165-Ubuntu SMP Tue Apr 18 08:53:12 UTC 2023`
    * CPU arc: x86_64
    * CPU: 4
    * Mem: 4GiB
    * Disk: 100GiB
* Client Machine
    * OS: `Linux version 5.4.0-148-generic (buildd@lcy02-amd64-112) (gcc version 9.4.0 (Ubuntu 9.4.0-1ubuntu1~20.04.1)) #165-Ubuntu SMP Tue Apr 18 08:53:12 UTC 2023`
    * CPU arc: x86_64
    * CPU: 4
    * Mem: 4GiB
    * Disk: 100GiB
* Java11
* 1Server - 1Client(1Thread)

## Precondition

**Request body**

```java
public class Payload extends HertsMessage {
    private String reqId;
    private String methodName;

    public String getReqId() {
        return reqId;
    }

    public void setReqId(String reqId) {
        this.reqId = reqId;
    }

    public String getMethodName() {
        return methodName;
    }

    public void setMethodName(String methodName) {
        this.methodName = methodName;
    }
}
```

**Measure range**
* Call Rpc on Client -> Response from Server 

**Time**
* 5min
