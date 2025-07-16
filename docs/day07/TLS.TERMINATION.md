### 📌 TLS Termination

---

### 🔒 **What Is TLS Termination?**

**TLS termination** is the process of **decrypting incoming HTTPS/TLS traffic** at the edge of your network (e.g., at a proxy or load balancer) so that internal services can speak plain HTTP.

* **Before termination:** Client ↔️ TLS ↔️ Edge Proxy
* **After termination:** Client ↔️ TLS ↔️ Edge Proxy → HTTP → Backend Service

---

### ⚙️ **Why TLS Termination?**

1. **Offload Encryption**

   * Reduces CPU load on application pods by handling decryption once at the edge.

2. **Centralized Certificate Management**

   * Manage TLS certs in one place (Ingress Gateway) rather than in every service.

3. **Consistent Security Policies**

   * Enforce TLS settings (protocol versions, ciphers) uniformly.

4. **Simplified Backend**

   * Backend services can remain HTTP-only, avoiding TLS config complexity.

---

### 🛠️ **How Istio Handles TLS Termination**

* **In the Istio Gateway** (`Gateway` CRD):
  You specify TLS settings under a `servers` block. The Envoy proxies in the `istio-ingressgateway` deployment terminate TLS, then forward plain HTTP to pods.

#### Example: HTTPS Termination at Ingress Gateway

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: tls-gateway
  namespace: istio-system
spec:
  selector:
    istio: ingressgateway         # targets the ingress gateway pods
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE                # termination mode
      credentialName: tls-secret  # K8s Secret holding cert+key
    hosts:
    - "example.com"
```

* **Mode `SIMPLE`**: Envoy terminates TLS using the provided cert, then speaks HTTP to backends.

---

### 🌍 **Real-World Use Cases**

| Scenario                  | Why TLS Termination at Istio Gateway                                            |
| ------------------------- | ------------------------------------------------------------------------------- |
| **Public Web App**        | Offload cert management and decryption from dozens of pods                      |
| **Multi-domain Hosting**  | Terminate multiple host TLS certs (e.g., `app1.com`, `app2.com`) in one gateway |
| **Internal Service Mesh** | Terminate external TLS at edge, then use mTLS internally for zero-trust         |

---

### ✅ **Key Points**

* **Edge Proxy** (Istio Ingress Gateway) handles HTTPS → HTTP
* **Centralizes** certs and TLS policy in one place
* **Offloads** CPU cost from application pods
* **Simplifies** backend services (no TLS config needed)

With Istio, you declare TLS termination in a `Gateway` CRD—no code changes to your services.
