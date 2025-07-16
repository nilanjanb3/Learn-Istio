### 📌 Istio Gateway vs Istio Ingress Gateway

---

| Aspect         | **Istio Gateway (CRD)**                                                                                                | **Istio Ingress Gateway (Deployment)**                                                                             |
| -------------- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| **Definition** | A **configuration resource** (`Gateway` CRD) that specifies ports, protocols, TLS settings and hosts for edge traffic. | A **running Envoy proxy** (typically the `istio-ingressgateway` service) that applies the `Gateway` configuration. |
| **Role**       | Declares *what* traffic should be exposed and how to terminate it.                                                     | Implements *how* that traffic is handled at the network edge.                                                      |
| **Scope**      | Abstract, namespaced resource; you can define multiple `Gateway` CRs (e.g., one per environment or domain).            | Concrete pods/services in the `istio-system` (or other) namespace. Usually deployed once per mesh or per cluster.  |
| **Binding**    | Referred to by name in `VirtualService`’s `gateways:` field to connect host/port definitions to routing rules.         | Selected by the `Gateway` via its `spec.selector` labels (e.g., `istio: ingressgateway`).                          |
| **Lifecycle**  | Managed via `kubectl apply/delete` like any CRD; purely declarative.                                                   | Managed like any Kubernetes Deployment/Service; can be scaled, patched, or secured independently.                  |

---

### ⚙️ How They Work Together

1. **Define a Gateway**
   A `Gateway` CRD declares that traffic arriving on port 80/443 for `example.com` should be handled:

   ```yaml
   apiVersion: networking.istio.io/v1alpha3
   kind: Gateway
   metadata:
     name: web-gateway
     namespace: istio-system
   spec:
     selector:
       istio: ingressgateway        # <-- matches the ingress gateway deployment
     servers:
     - port:
         number: 443
         name: https
         protocol: HTTPS
       tls:
         mode: SIMPLE
         credentialName: tls-secret
       hosts:
       - "example.com"
   ```

2. **Deploy the Ingress Gateway**
   The `istio-ingressgateway` Deployment (provided by Istio’s installation) runs Envoy pods labeled `istio=ingressgateway`. These pods read the `Gateway` CR and listen on the specified ports.

3. **Route Traffic with VirtualService**
   A `VirtualService` ties the `Gateway` to backend services:

   ```yaml
   apiVersion: networking.istio.io/v1alpha3
   kind: VirtualService
   metadata:
     name: web-routing
     namespace: istio-system
   spec:
     hosts:
     - "example.com"
     gateways:
     - web-gateway
     http:
     - match:
       - uri:
           prefix: /app
       route:
       - destination:
           host: app-service
           port:
             number: 80
   ```

---

### 🌍 Real-World Use Case

* **Multi-Domain**:

  * Define multiple `Gateway` CRs (e.g., `api-gateway`, `web-gateway`) pointing to the same `istio-ingressgateway` deployment but terminating different TLS certificates and handling different hostnames.
* **Canary at the Edge**:

  * Update a second ingress gateway deployment (`istio-canary-ingressgateway`) with experimental EnvoyFilter patches, and bind a “canary-gateway” CRD to it to test edge behavior without affecting the stable ingress.

---

### ✅ Key Takeaways

* **Gateway (CRD)** = *Declarative spec* of exposed ports, protocols, and TLS for edge traffic.
* **Ingress Gateway (Deployment)** = *Concrete Envoy pods* that implement those specs and actually terminate and forward traffic.

Use the CRD to define *what* you want, and the ingress gateway pods to control *how* it’s done.
