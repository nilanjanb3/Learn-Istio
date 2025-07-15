### 📌 Istio: AuthorizationPolicy

---

### 🔒 **What is an AuthorizationPolicy?**

An **AuthorizationPolicy** is an Istio resource that **controls who (which identities) can access which services or workloads**, and under what conditions.

* It works at **Layer 7 (application layer)** for HTTP/gRPC traffic.
* You can allow or deny traffic based on **source identity**, **request attributes**, or **other conditions**.

---

### ⚙️ **Key Concepts**

| Concept        | Definition                                                            |
| -------------- | --------------------------------------------------------------------- |
| **Selector**   | Targets workloads (pods) by labels                                    |
| **Rules**      | One or more conditions defining “allow” or “deny”                     |
| **Principals** | SPIFFE identities (e.g., `spiffe://…/sa/default`) of calling services |
| **Conditions** | HTTP paths, methods, headers, ports, namespaces                       |
| **Action**     | `ALLOW` (default) or `DENY`                                           |

---

### 🛠️ **Basic Example: Allow Orders → Payments**

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-orders-to-payments
  namespace: payments
spec:
  # Apply to the 'payments' service
  selector:
    matchLabels:
      app: payments
  # Allow requests only from the 'orders' service account
  rules:
  - from:
    - source:
        principals: ["spiffe://cluster.local/ns/orders/sa/default"]
```

* **What this does**: Only the `orders` service (running under the `default` service account in the `orders` namespace) can call `payments`.

---

### 🧩 **Advanced Conditions**

You can refine policies further:

* **By HTTP Path or Method**

  ```yaml
  rules:
  - to:
    - operation:
        methods: ["POST"]
        paths: ["/charge"]
  ```

  Only allow `POST /charge` requests.

* **By Headers**

  ```yaml
  rules:
  - when:
    - key: request.headers[x-user-role]
      values: ["admin"]
  ```

  Only requests with header `x-user-role: admin`.

* **By Namespace or Source IP**

  ```yaml
  rules:
  - from:
    - source:
        namespaces: ["frontend"]
        ipBlocks: ["10.0.0.0/16"]
  ```

---

### 🌍 **Real-World Use Cases**

| Use Case                   | Policy Example                                               |
| -------------------------- | ------------------------------------------------------------ |
| **Service Isolation**      | Only allow `billing` to talk to `payments`.                  |
| **Role-Based Access**      | Only admins (`x-user-role: admin`) can access `/admin` APIs. |
| **Environment Separation** | Block `dev` namespace from calling `prod` services.          |
| **Deny All by Default**    | Start with a global `DENY` policy, then selectively `ALLOW`. |

---

### ✅ **Summary**

1. **Define Scope**: Use `selector` to choose target workloads.
2. **Write Rules**: Use `from`, `to`, and `when` blocks for source identities and request attributes.
3. **Apply Action**: Default is `ALLOW`; you can also set `action: DENY` for explicit denials.

With **AuthorizationPolicy**, you get **fine-grained, identity-based access control** across your service mesh—no code changes required.
