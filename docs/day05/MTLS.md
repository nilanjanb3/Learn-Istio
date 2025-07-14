### 📌 Istio: mTLS Between Services

---

### 🔐 **What is mTLS Between Services?**

**mTLS (Mutual TLS)** ensures that **all communication between services is encrypted and both ends are authenticated**.

* **Server verifies client**
* **Client verifies server**
* **Traffic is encrypted end-to-end**

This prevents **man-in-the-middle attacks**, **service spoofing**, and **data leakage** inside your cluster.

---

### 🛠️ **How Istio Implements mTLS**

* Istio automatically **injects Envoy sidecars** to all pods in the mesh.
* Envoy proxies handle mTLS handshake—**app code does not change**.
* Istio **issues certificates** and manages their lifecycle (rotation, revocation).

---

### ⚙️ **How to Enable mTLS in Istio**

#### 1. **Mesh-wide mTLS (All Services)**

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT
```

* Applies to all namespaces by default.
* All communication within the mesh is **encrypted and mutually authenticated**.

---

#### 2. **Namespace-Scoped mTLS**

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: my-namespace
spec:
  mtls:
    mode: STRICT
```

* Only affects services in `my-namespace`.

---

#### 3. **Per-Service mTLS**

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: my-service-mtls
  namespace: my-namespace
spec:
  selector:
    matchLabels:
      app: my-service
  mtls:
    mode: STRICT
```

* Enforces mTLS only for `my-service` pods.

---

### 🧩 **How It Works (Traffic Flow Example)**

1. **Service A** wants to talk to **Service B**.
2. The request from A hits its **Envoy sidecar**.
3. Envoy A initiates a **mTLS handshake** with Envoy B.
4. Both Envoys exchange certificates, verify identities, and establish an encrypted channel.
5. The request is forwarded securely from Envoy A → Envoy B → Service B.

---

### 🌍 **Real-World Use Cases**

| Scenario                     | Benefit of mTLS                                     |
| ---------------------------- | --------------------------------------------------- |
| **Compliance (GDPR, HIPAA)** | Encrypts sensitive data in transit                  |
| **Multi-tenant clusters**    | Prevents tenants from sniffing each other's traffic |
| **Zero Trust Security**      | Verifies service identity for every request         |

---

### ✅ **Summary Table**

| mTLS Scope      | Resource Used                     | Example Use             |
| --------------- | --------------------------------- | ----------------------- |
| **Mesh-wide**   | PeerAuthentication (istio-system) | Secure entire cluster   |
| **Namespace**   | PeerAuthentication (namespace)    | Isolate team/apps       |
| **Per-service** | PeerAuthentication (selector)     | Secure specific service |

> **With Istio, you get automatic, scalable, and manageable mTLS between every service—no app code changes needed.**

---

Let me know the next topic!
