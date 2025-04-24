Let’s walk through a **hands-on lab to demo a Canary Deployment** using Istio.

A **canary deployment** gradually rolls out a new version of a service to a small subset of users before a full rollout. We’ll use Istio's `VirtualService` and `DestinationRule` to implement this.

---

### 🧪 Scenario

We'll use the `reviews` service from the Bookinfo app (which has versions `v1`, `v2`, `v3`).

We’ll simulate:
- Initial state: all traffic → `v1`
- Canary rollout: small % → `v3`, rest → `v1`
- Final state: 100% → `v3`

---

### ✅ Prerequisites

Ensure:
- Istio is installed and running
- Bookinfo app is deployed
- Namespace (e.g. `default`) has sidecar injection enabled
- You’ve exposed the app using `bookinfo-gateway.yaml`

---

### 🧪 Step 1: Define `DestinationRule`

We need to define subsets for different versions.

```yaml
# destination-rule-reviews.yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v3
    labels:
      version: v3
```

Apply it:

```bash
kubectl apply -f destination-rule-reviews.yaml
```

---

### 🧪 Step 2: Send 100% Traffic to `v1`

```yaml
# virtual-service-reviews-100-v1.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 100
```

Apply it:

```bash
kubectl apply -f virtual-service-reviews-100-v1.yaml
```

✅ **Verify:** Open `http://localhost:8080/productpage` → **no stars (v1)**

---

### 🧪 Step 3: Canary Deployment — 10% to `v3`

```yaml
# virtual-service-reviews-canary.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v3
      weight: 10
    - destination:
        host: reviews
        subset: v1
      weight: 90
```

Apply it:

```bash
kubectl apply -f virtual-service-reviews-canary.yaml
```

🔁 **Test multiple times** by refreshing `http://localhost:8080/productpage`. Most requests will go to `v1`, some to `v3` (you’ll see red stars on `v3`).

---

### 🧪 Step 4: Roll Forward — 100% to `v3`

```yaml
# virtual-service-reviews-100-v3.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - route:
    - destination:
        host: reviews
        subset: v3
      weight: 100
```

Apply it:

```bash
kubectl apply -f virtual-service-reviews-100-v3.yaml
```

✅ **Result:** All users see **red stars (v3)**.

---

### 🧪 Step 5: Optional Cleanup

```bash
kubectl delete virtualservice reviews
kubectl delete destinationrule reviews
```

---

### 🧠 Summary

| Phase               | Version Routing               |
|--------------------|-------------------------------|
| Initial            | 100% → `v1`                   |
| Canary             | 90% → `v1`, 10% → `v3`        |
| Full Rollout       | 100% → `v3`                   |

---

Let me know if you want to extend this to:
- Canary with Prometheus/Grafana observability
- Automated rollback on errors
- Header-based or geo-based routing in canary

Ready for the next lab?