### 📌 What is a Service Mesh?

A **Service Mesh** is a **dedicated infrastructure layer** that helps manage communication **between microservices** in a **reliable, secure, and observable** way.

---

### 💡 Why is it needed?

In a microservices architecture:
- Services need to **communicate** with each other frequently.
- You need **security (mTLS)**, **retries**, **failovers**, **monitoring**, and **traffic control**.
- Adding this logic to every service increases complexity.

A **Service Mesh** removes that burden by moving these concerns out of your app code into a separate layer.

---

### ⚙️ How it works?

- Deploys a **sidecar proxy** (like Envoy) next to each service instance.
- All traffic goes **through the sidecar** → it handles routing, retries, security, etc.

```
[Service A] <--> [Sidecar Proxy A] <====> [Sidecar Proxy B] <--> [Service B]
```

---

### 🛠 Example (Istio):

In Istio:
- The sidecar proxy is **Envoy**.
- You can define traffic rules (like 50% to v1, 50% to v2).
- It supports **automatic mTLS**, observability via **Prometheus/Grafana**, and **traffic shaping**.

---

### 🌍 Real-world use case:

1. **Canary Deployment**:
   - Deploy v2 of service, send 10% of traffic → observe → scale up.
2. **Zero Trust Security**:
   - Enforce mutual TLS and fine-grained access control between services.
3. **Outage Recovery**:
   - Automatic retries/failover without changing app code.

---

### ✅ Key Benefits:
- **Security**: mTLS, policies, encryption
- **Traffic Control**: routing, retries, failovers
- **Observability**: metrics, logs, tracing
- **Decouples infra logic** from app logic

---