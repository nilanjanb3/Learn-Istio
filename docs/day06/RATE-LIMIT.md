### 📌 Istio Rate Limiting (Envoy Local & External Adapter)

---

### 🚦 **What Is Rate Limiting?**

Rate limiting controls the **number of requests** a service will accept over a period of time, protecting downstream services from **traffic spikes**, **abuse**, or **accidental overload**.

---

### ⚙️ **Types of Rate Limiting**

1. **Envoy Local Rate Limit**

   * Enforced **per-proxy** (sidecar) before requests hit your application.
   * Good for **simple quotas** (e.g., 20 requests/sec per instance).

2. **Global Rate Limit via External Adapter**

   * Delegates decision to a **centralized rate-limit service** (e.g., Redis-backed).
   * Supports **global quotas**, **user-based limits**, and **dynamic rules**.

---

### 🛠️ **Envoy Local Rate Limit Example**

Use an **EnvoyFilter** to insert the `local_ratelimit` HTTP filter for a given workload:

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: reviews-local-rate-limit
  namespace: istio-system
spec:
  workloadSelector:
    labels:
      app: reviews
  configPatches:
    - applyTo: HTTP_FILTER
      match:
        context: SIDECAR_INBOUND
        listener:
          filterChain:
            filter:
              name: "envoy.filters.network.http_connection_manager"
      patch:
        operation: INSERT_BEFORE
        value:
          name: envoy.filters.http.local_ratelimit
          typed_config:
            "@type": type.googleapis.com/envoy.extensions.filters.http.local_ratelimit.v3.LocalRateLimit
            stat_prefix: http_local_rate_limiter
            token_bucket:
              max_tokens: 100     # burst capacity
              tokens_per_fill: 50 # refill rate
              fill_interval: 1s   # refill interval
            filter_enabled:
              runtime_key: local_rate_limit_enabled
              default_value:
                numerator: 100
                denominator: HUNDRED
            filter_enforced:
              runtime_key: local_rate_limit_enforced
              default_value:
                numerator: 100
                denominator: HUNDRED
```

* **max\_tokens** & **tokens\_per\_fill** define a 50 req/sec steady rate with a 100-request burst capacity.
* Applied on **all inbound** requests to the `reviews` service.

---

### 🔗 **External Rate Limit Adapter Example**

Use the [Envoy Rate Limit Service](https://github.com/envoyproxy/ratelimit) as an external adapter:

```yaml
# 1. Define a ServiceEntry for the rate-limit service
apiVersion: networking.istio.io/v1beta1
kind: ServiceEntry
metadata:
  name: ratelimit
spec:
  hosts:
  - ratelimit.default.svc.cluster.local
  ports:
  - number: 8081
    name: http
    protocol: HTTP
  resolution: DNS

# 2. Attach the gRPC “ext_authz” filter via EnvoyFilter
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: reviews-global-rate-limit
  namespace: istio-system
spec:
  workloadSelector:
    labels:
      app: reviews
  configPatches:
    - applyTo: HTTP_FILTER
      match:
        context: SIDECAR_INBOUND
        listener:
          filterChain:
            filter:
              name: "envoy.filters.network.http_connection_manager"
      patch:
        operation: INSERT_AFTER
        value:
          name: envoy.filters.http.ext_authz
          typed_config:
            "@type": type.googleapis.com/envoy.extensions.filters.http.ext_authz.v3.ExtAuthz
            grpc_service:
              google_grpc:
                target_uri: "ratelimit.default.svc.cluster.local:8081"
                stat_prefix: "rate_limit"
            failure_mode_allow: false
```

* The external adapter evaluates **global policies** (e.g., per-user or per-API key limits).
* You configure the adapter itself (in its own ConfigMap) to enforce rules like “max 1,000 calls per minute per API key.”

---

### 🌍 **Real-World Use Cases**

| Use Case                          | Technique                 | Benefit                                 |
| --------------------------------- | ------------------------- | --------------------------------------- |
| Protect downstream databases      | Local Rate Limit          | Stops abusive bursts at the proxy       |
| API tiering (free vs. paid users) | External Adapter          | Enforce user-specific quotas centrally  |
| Global quota across replicas      | External Adapter          | One shared counter across all pods      |
| Gradual traffic ramp-up           | Local + External combined | Burst handling locally + global control |

---

### ✅ **Key Takeaways**

* **Local Rate Limit** (EnvoyFilter):

  * Simple, per-instance quotas
  * Low latency, no external calls

* **External Rate Limit** (ext\_authz + Adapter):

  * Centralized, dynamic policies
  * Supports user-level, global limits

Both methods let you **add or adjust quotas** without changing your application code—just update Istio/Envoy configs.
