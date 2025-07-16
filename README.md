## 🛣️ **Istio Learning Roadmap (10 Days)**

---

### **Day 1: Introduction to Service Mesh & Istio Basics**
**Concepts:**
- [What is a Service Mesh?](/docs/day01/01-WHAT-IS-A-SERVICE-MESH.MD)
- [Why Istio? Benefits over traditional networking](./docs/day01/02-WHY-ISTIO.MD)
- [Istio Architecture: Data Plane vs Control Plane](./docs/day01/03-ISTIO-ARCHITECTURE.MD)
- [Envoy Proxy Basics](./docs/day01/04-ENVOY-PROXY-BASICS.MD)
- [What is mTLS?](./docs/day01/05-WHAT-IS-mTLS.MD)
- [Istio Networking Basics](./docs/day01/06-ISTIO-NETWORKING-BASICS.md)

**Hands-On:**
- [Set up a Kubernetes cluster (minikube/kind/EKS/GKE)](./labs/day01/01-SETUP-LOCAL-CLUSTER.MD)
- [Install Istio using `istioctl`](./labs/day01/02-INSTALL-ISTIO.md)
- [Verify control plane components](./labs/day01/03-VERIFY-COMPONENTS.md)

**Use Cases:**
- Decoupling traffic management from application code
- Simplifying observability in microservices

---

### **Day 2: Istio Traffic Management - Basics**
**Concepts:**
- [VirtualServices, DestinationRules](./docs/day02/VIRTUAL-SERVICE-DESTINATION-RULE.MD)
- [Traffic Routing: Canary, A/B Testing, Header-based routing](./docs/day02/CANARY-AB-HEADER-BASED-ROUTING.MD)

**Hands-On:**
- [Deploy a microservices app (Bookinfo)](./labs/day02/01-DEPLOY-MS-BOOKINFO.MD)
- [Create traffic rules to route traffic between versions](./labs/day02/02-TRAFFIC-RULE-BETWEEN-VERSIONS.MD)
- [Demo canary deployments](./labs/day02/03-DEMO-CANARY-DEPLOYMENT.MD)

**Use Cases:**
- Progressive delivery strategies in production

---

### **Day 3: Traffic Management - Advanced**
**Concepts:**
- [Fault Injection](./docs/day03/FAULT-INJECTION.md)
- [Timeouts and Retries](./docs/day03/TIMEOUT-AND-RETRY.md)
- [Circuit Breakers](./docs/day03/CIRCUIT-BREAKER.md)
- [Mirroring Traffic](./docs/day03/MIRROR-TRAFFIC.md)

**Hands-On:**
- [Inject faults into service traffic](./labs/day03/01-FAULT-INJECTION.md)
- [Configure retry/timeouts with failure simulation](./labs/day03/02-TIMEOUT-AND-RETRIES.md)
- Use `curl` to observe failover behavior

**Use Cases:**
- Build resilient microservices

---

### **Day 4: Observability & Telemetry**
**Concepts:**
- [Metrics, Logs, and Traces](./docs/day04/METRIC-LOGS-TRACES.md)
- Istio integration with:
  - Prometheus
  - Grafana
  - Jaeger
  - [Kiali](./docs/day04/ISTIO-INTEGRATION-WITH-KIALI.md)

**Hands-On:**
- Deploy observability stack with Istio addons
- Visualize metrics and traces
- [Use Kiali dashboard to explore the service graph](./labs/day04/01-KIALI.md)

**Use Cases:**
- SLA/SLO enforcement
- Troubleshooting and performance optimization

---

### **Day 5: Istio Security - mTLS and Identity**
**Concepts:**
- [Zero-trust networking](./docs/day05/ZERO-TRSUT-NETWORKING.md)
- [mTLS between services](./docs/day05/MTLS.md)
- [Service-to-service authentication](./docs/day05/SERVICE-TO-SERVICE-AUTHN.md)
- [Identity-based authorization (SPIFFE/SPIRE)](./docs/day05/IDENTITY-BASED-AUTHN.md)

**Hands-On:**
- [Enable strict mTLS](./labs/day05/01-MTLS.md)
- [Verify certificates and secure communication](./labs/day05/02-VERIFY-CERTIFICATE.md)
- [Apply peer & request authentication policies](./labs/day05/03-PEER-AUTHN.md)

**Use Cases:**
- Enforcing secure comms in regulated environments

---

### **Day 6: Authorization & Policy Enforcement**
**Concepts:**
- [AuthorizationPolicy](./docs/day06/AUTHZ.md)
- [Access Control based on JWT, IP, Namespace, etc.](./docs/day06/ACCESS-CONTROL.md)
- [Rate limiting (with Envoy or external adapter)](./docs/day06/RATE-LIMIT.md)

**Hands-On:**
- Define fine-grained RBAC for services
- Block unauthorized access with policies
- Apply rate-limiting policies

**Use Cases:**
- API protection, secure multi-tenant architectures

---

### **Day 7: Ingress Gateway, Egress, and TLS Termination**
**Concepts:**
- [Istio Gateway vs Kubernetes Ingress](./docs/day07/ISTIO-GATEWAY-VS-K8S-INGRESS.md)
- [Istio Gateway vs Istio Ingress Gateway](./docs/day07/ISTIO-GATEWAY-VS-ISTIO-INGRESS-GATEWAY.md) 
- [TLS termination](./docs/day07/TLS.TERMINATION.md)
- [Egress traffic control](./docs/day07/EGRESS-TRAFFIC.md)
- [External service access](./docs/day07/EXTERNAL-SERVICE.md)

**Hands-On:**
- Configure Istio Ingress Gateway
- Secure app with HTTPS
- Monitor and restrict egress traffic

**Use Cases:**
- Secure APIs with TLS, control data exfiltration

---

### **Day 8: Istio in Production**
**Concepts:**
- [Best practices for production deployments](./docs/day08/ISTIO-PRODUCTION.md)
- [Istio upgrades and versioning](./docs/day08/ISTIO-UPGRADE-AND-VERSIONING.md)
- [Configuration profiles](./docs/day08/ISTIO-PROFILES.md)
- [Multi-tenancy and multi-cluster deployment overview](./docs/day08/MULTI-TENANCY.md)

**Hands-On:**
- Use revision-based upgrades
- Create custom IstioOperator configs
- Enable access logs and audit trails

---

### **Day 9: Real-world Integrations**
**Concepts:**
- Istio with CI/CD tools (Jenkins, ArgoCD, GitOps)
- Istio with Service Discovery and DNS
- Using Istio with other observability stacks (e.g., Loki)

**Hands-On:**
- Automate traffic shifting with CI/CD
- Trigger rollout with GitOps changes
- Observe trace correlation via logs

---

### **Day 10: Troubleshooting & Debugging Istio**
**Concepts:**
- Common failure scenarios
- Debugging tools: `istioctl`, Envoy admin, `kubectl`
- Logging and performance tuning

**Hands-On:**
- Debug Envoy config
- Use `istioctl analyze`, `proxy-status`
- Fix broken routes and policies

---

## ✅ Real-World Project Ideas (Post-Learning)

You can start these after Day 10, based on your focus area:

---

### 🔧 **Project 1: Secure Multi-Tenant Microservices with Istio**
- Deploy multiple apps in separate namespaces
- Enforce strict mTLS, RBAC policies per tenant
- Visualize network flows in Kiali

---

### 🚀 **Project 2: GitOps-based Canary Deployments using Istio and ArgoCD**
- Setup ArgoCD
- Automate VirtualService changes via Git
- Use Kiali to validate rollouts

---

### 📊 **Project 3: Full Observability Stack with Alerting**
- Integrate Prometheus, Grafana, Loki, Jaeger
- Create dashboards for SLOs
- Set alerts on latency, errors

---

### 🌐 **Project 4: Ingress Gateway with TLS and Rate Limiting**
- Serve app via HTTPS
- Add JWT-based Authorization
- Apply rate limiting to external traffic

---

### 🧪 **Project 5: Chaos Engineering with Istio and Litmus**
- Simulate service crashes or latency
- Validate recovery using retries, circuit breakers

---