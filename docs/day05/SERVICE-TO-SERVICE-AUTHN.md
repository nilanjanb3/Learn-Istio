### 📌 Istio: Service-to-Service Authentication

---

### 🔒 **What is Service-to-Service Authentication?**

**Service-to-service authentication** ensures that when one service calls another, **the caller’s identity is verified**.
This guarantees that only **authorized, trusted services** can communicate with each other—no service can “spoof” its identity.

---

### 🛠️ **How Istio Provides Authentication**

Istio supports two major types of service authentication:

1. **Transport Authentication (mTLS)**

   * **Who**: Performed by Envoy sidecars
   * **What**: Uses **mutual TLS** to authenticate both client and server with X.509 certificates.
   * **Result**: Only pods with valid Istio-issued certs can talk to each other.

2. **Origin Authentication (JWT/OAuth)**

   * **Who**: Performed at the HTTP layer (optional)
   * **What**: Validates tokens (JWT, OAuth2) sent by external clients (users, apps).
   * **Result**: Only requests with valid tokens reach protected services.

---

### 🔑 **Service Identity in Istio**

* Each workload (service account) in the mesh gets a unique identity, e.g.:
  `spiffe://cluster.local/ns/default/sa/my-service-account`
* Istio issues and rotates X.509 certificates mapped to these identities.

---

### ⚙️ **Enforcing Authentication**

#### 1. **Enable mTLS for Service-to-Service Calls**

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

* **All services must present valid certs** to communicate—no cert, no access.

#### 2. **Restrict Calls Using Authorization Policy**

You can restrict which services can call another using their authenticated identity.

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-orders-to-payments
  namespace: payments
spec:
  selector:
    matchLabels:
      app: payments
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/orders/sa/default"]
```

* Only the `orders` service (with this service account) can call the `payments` service.

#### 3. **(Optional) Require JWT for End-User Auth**

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
  namespace: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  jwtRules:
  - issuer: "https://issuer.example.com"
    jwksUri: "https://issuer.example.com/.well-known/jwks.json"
```

* Only requests with a valid JWT token from this issuer are accepted.

---

### 🌍 **Real-World Example**

| Scenario                        | Authentication Used        | Result                                                      |
| ------------------------------- | -------------------------- | ----------------------------------------------------------- |
| `frontend` → `backend`          | mTLS                       | `backend` only accepts requests with valid service identity |
| External API Gateway → `orders` | JWT                        | Only requests with valid user tokens are allowed            |
| `orders` → `payments`           | mTLS + AuthorizationPolicy | Only authorized services can access sensitive endpoints     |

---

### ✅ **Summary Table**

| Type                    | What It Secures                | How to Enable         |
| ----------------------- | ------------------------------ | --------------------- |
| **mTLS**                | Service-to-service calls       | PeerAuthentication    |
| **AuthorizationPolicy** | Which services can access what | AuthorizationPolicy   |
| **JWT (optional)**      | End-user API calls             | RequestAuthentication |

> **With Istio, all service-to-service communication can be authenticated, authorized, and encrypted—by default, with no code changes.**

---

Let me know your next topic!
