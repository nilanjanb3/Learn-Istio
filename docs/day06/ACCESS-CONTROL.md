### 📌 Istio: Access Control Based on JWT, IP, Namespace, etc.

---

Istio’s **AuthorizationPolicy** can match on a variety of request attributes to allow or deny traffic. You combine `from`, `to`, and `when` clauses to enforce rules based on:

* **JWT claims** (origin authentication)
* **Source IP blocks**
* **Source or target namespaces**
* **HTTP headers, paths, methods**

---

### ⚙️ Key Matching Fields

| Field                   | Description                                         | Example Value                                            |
| ----------------------- | --------------------------------------------------- | -------------------------------------------------------- |
| **principals**          | SPIFFE IDs or JWT issuer+subject identifying caller | `spiffe://cluster.local/ns/front/sa/default`             |
| **namespaces**          | Kubernetes namespaces of calling pods               | `["prod","frontend"]`                                    |
| **ipBlocks**            | CIDR ranges of source IPs                           | `["10.0.0.0/16","192.168.1.0/24"]`                       |
| **request.auth.claims** | JWT claims from an authorization token              | `{ "iss": "https://auth.example.com", "role": "admin" }` |
| **request.headers**     | HTTP header values                                  | `x-user-role: ["admin","poweruser"]`                     |
| **operation.paths**     | HTTP URL paths                                      | `["/admin/*","/charge"]`                                 |
| **operation.methods**   | HTTP methods                                        | `["GET","POST"]`                                         |

---

### 🧪 Examples

1. **Allow Only Admins via JWT Claim**
   Require incoming JWT with `role=admin`.

   ```yaml
   apiVersion: security.istio.io/v1beta1
   kind: AuthorizationPolicy
   metadata:
     name: require-admin
     namespace: myapp
   spec:
     selector:
       matchLabels:
         app: myapp
     rules:
     - when:
       - key: request.auth.claims[role]
         values: ["admin"]
   ```
2. **Deny Traffic from Outside CIDR**
   Block any source not in the 10.0.0.0/16 network.

   ```yaml
   apiVersion: security.istio.io/v1beta1
   kind: AuthorizationPolicy
   metadata:
     name: allow-internal-only
     namespace: myapp
   spec:
     selector:
       matchLabels:
         app: myapp
     rules:
     - from:
       - source:
           ipBlocks: ["10.0.0.0/16"]
   ```
3. **Allow Calls Only from Same Namespace**
   Enforce that only pods in `payments` namespace may call.

   ```yaml
   apiVersion: security.istio.io/v1beta1
   kind: AuthorizationPolicy
   metadata:
     name: same-namespace-only
     namespace: payments
   spec:
     selector:
       matchLabels:
         app: payments
     rules:
     - from:
       - source:
           namespaces: ["payments"]
   ```
4. **Restrict by HTTP Path and Method**
   Only allow `POST` to `/admin/*` for service owners.

   ```yaml
   apiVersion: security.istio.io/v1beta1
   kind: AuthorizationPolicy
   metadata:
     name: admin-post-only
     namespace: myapp
   spec:
     selector:
       matchLabels:
         app: myapp
     rules:
     - to:
       - operation:
           methods: ["POST"]
           paths: ["/admin/*"]
       when:
       - key: request.auth.claims[role]
         values: ["owner","admin"]
   ```

---

### 🌍 Real-World Use Cases

| Scenario                       | Control Criteria        | Benefit                                 |
| ------------------------------ | ----------------------- | --------------------------------------- |
| Public API vs Internal API     | JWT issuer + audience   | Only valid tokens reach internal routes |
| On-prem vs Cloud Traffic       | IP blocks               | Ensure only corporate networks connect  |
| Multi-tenant Cluster Isolation | Namespace               | Tenant A can’t call Tenant B services   |
| Feature Flags for Beta Users   | HTTP header / JWT claim | Route and enforce policies for betas    |

---

### ✅ Summary

* **JWT Claims**: Use `request.auth.claims[...]` to require specific roles or audiences.
* **IPBlocks**: Use CIDR lists to restrict by source network.
* **Namespaces**: Restrict service calls to pods within certain namespaces.
* **Headers & Paths**: Combine `when` and `to.operation` for HTTP-level control.

These controls let you implement **fine-grained, context-aware security** without touching application code.
