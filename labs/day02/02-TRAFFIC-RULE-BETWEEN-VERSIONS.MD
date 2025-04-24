Great! This lab will help you **create traffic routing rules in Istio** to split traffic between different versions of a microservice (e.g., `v1`, `v2`, `v3`). You’ll do this using **VirtualService** and **DestinationRule**, which are core Istio concepts for traffic control.

We’ll use the **`reviews` service** from the Bookinfo application, which has 3 versions:
- `v1`: No star ratings
- `v2`: Black star ratings
- `v3`: Red star ratings

---

### ✅ Prerequisites

Make sure you’ve already:
- Installed Istio
- Deployed the Bookinfo app
- Applied the `bookinfo-gateway.yaml`

---

### 🧪 Step 1: Define Destination Rules

Destination rules define the available versions (subsets) of a service:

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
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3
```

Apply it:

```bash
kubectl apply -f destination-rule-reviews.yaml
```

---

### 🧪 Step 2: Route 100% Traffic to v1

Let’s start by sending all traffic to `reviews` version `v1`.

```yaml
# virtual-service-reviews-v1.yaml
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
kubectl apply -f virtual-service-reviews-v1.yaml
```

Now visit `http://localhost:8080/productpage` (assuming port-forwarding) and **you will not see any star ratings** (v1 behavior).

---

### 🧪 Step 3: Split Traffic Between v1 and v2 (e.g., 50/50)

Update the `VirtualService` to split traffic:

```yaml
# virtual-service-reviews-split.yaml
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
      weight: 50
    - destination:
        host: reviews
        subset: v2
      weight: 50
```

Apply it:

```bash
kubectl apply -f virtual-service-reviews-split.yaml
```

Now, reload `http://localhost:8080/productpage` multiple times — you’ll randomly see **no stars (v1)** or **black stars (v2)** depending on routing.

---

### 🧪 Step 4: Route Based on Headers (A/B Testing)

Route traffic based on user identity or header (e.g., `end-user: jason` gets v2, everyone else gets v1):

```yaml
# virtual-service-reviews-ab.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```

Apply it:

```bash
kubectl apply -f virtual-service-reviews-ab.yaml
```

To test, use `curl`:

```bash
curl -H "end-user: jason" http://localhost:8080/productpage
```

Jason sees **black stars** (v2), while others see **no stars** (v1).

---

### 🧪 Step 5: Clean Up (Optional)

```bash
kubectl delete virtualservice reviews
kubectl delete destinationrule reviews
```

---

### ✅ Summary

| Scenario | Config Used |
|----------|-------------|
| Route 100% to v1 | `virtual-service-reviews-v1.yaml` |
| 50/50 traffic split | `virtual-service-reviews-split.yaml` |
| Header-based A/B testing | `virtual-service-reviews-ab.yaml` |

These techniques are used for **Canary deployments, A/B testing, gradual rollouts, and targeted experiments** in real-world service mesh environments.

Let me know when you want to try **mTLS**, **rate limiting**, or **failover rules** next!