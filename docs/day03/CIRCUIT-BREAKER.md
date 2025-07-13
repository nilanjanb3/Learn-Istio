### 📌 Istio: Circuit Breakers

---

### 🚦 **What is a Circuit Breaker?**

A **Circuit Breaker** is a resilience pattern used to **prevent system overload and cascading failures**. It detects when a service is failing or slow, and **stops sending requests** to it until it recovers.

> **Goal:** Protect healthy services from being overwhelmed by unhealthy ones.

In Istio, circuit breaker settings are applied using **DestinationRule** (not in VirtualService).

---

### ⚙️ **How Does It Work?**

* Monitors the health of service endpoints.
* If too many errors/connections/timeouts are detected, it “opens” the circuit: new requests fail immediately, rather than piling up.
* Once the service recovers, the circuit “closes,” and requests flow again.

---

### 🛠️ **Key Settings in Istio Circuit Breaker**

Defined in the `trafficPolicy` section of a **DestinationRule**.

| Setting                      | What It Does                                     |
| ---------------------------- | ------------------------------------------------ |
| **maxConnections**           | Limits concurrent connections to the service     |
| **http1MaxPendingRequests**  | Max number of queued requests (for HTTP/1)       |
| **maxRequestsPerConnection** | Max requests per connection                      |
| **outlierDetection**         | Detects and ejects unhealthy hosts automatically |

---

### 🧪 **Example: Simple Circuit Breaker**

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews-cb
spec:
  host: reviews
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
```

➡️ This limits each client to **1 connection** and **1 pending request** at a time to the `reviews` service.

---

### 🧪 **Example: Outlier Detection (Automatic Ejection of Unhealthy Endpoints)**

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews-outlier
spec:
  host: reviews
  trafficPolicy:
    outlierDetection:
      consecutive5xxErrors: 3
      interval: 10s
      baseEjectionTime: 30s
      maxEjectionPercent: 100
```

* If a service instance returns **3 consecutive 5xx errors** in a **10s window**, it will be **ejected** (removed from load balancing) for **30s**.

---

### 🌍 **Real-World Use Cases**

| Scenario                    | Use                                                                                        |
| --------------------------- | ------------------------------------------------------------------------------------------ |
| **Sudden Service Failure**  | Circuit breaker stops sending requests to failing instances, protecting upstream services. |
| **Slow Downstream Service** | Prevents too many pending requests from piling up, avoiding resource exhaustion.           |
| **Rolling Deployments**     | Keeps new, unstable instances from bringing down the mesh during rollout.                  |

---

### ✅ **Summary Table**

| Setting                     | What it Protects Against           |
| --------------------------- | ---------------------------------- |
| **Connection Pool**         | Too many open/pending connections  |
| **Outlier Detection**       | Repeated errors from bad instances |
| **Max Requests/Connection** | Resource exhaustion, slow leaks    |

* **Configured in:** `DestinationRule` > `trafficPolicy`
* **No code changes needed**—all controlled via Istio YAML!

---

Let me know the next topic whenever you're ready!
