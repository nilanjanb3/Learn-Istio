Here’s a **step-by-step lab** to **inject faults** (delays or HTTP errors) into service traffic using Istio. Fault injection helps you test the resilience of your microservices by simulating real-world failures such as high latency or service outages.

We’ll use the `reviews` service from the Bookinfo app for this demo.

---

## 🧪 Step 1: Prerequisites

* Istio is installed and working
* Bookinfo app is running
* Namespace (`default`) has sidecar injection enabled
* You can access `http://localhost:8080/productpage`

---

## 🧪 Step 2: Inject an HTTP Delay

Let’s add a **5-second delay** for 100% of traffic to the `reviews` service.

```yaml
# virtual-service-reviews-delay.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
    - fault:
        delay:
          percentage:
            value: 100.0
          fixedDelay: 5s
      route:
        - destination:
            host: reviews
            subset: v1
```

**Apply it:**

```bash
kubectl apply -f virtual-service-reviews-delay.yaml
```

**Test:**
Open or refresh `http://localhost:8080/productpage` — the page will hang for about 5 seconds before loading, because calls to `reviews` are delayed.

---

## 🧪 Step 3: Inject HTTP Errors

Let’s simulate **HTTP 500 errors** for 50% of the traffic to the `reviews` service.

```yaml
# virtual-service-reviews-abort.yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
    - fault:
        abort:
          percentage:
            value: 50.0
          httpStatus: 500
      route:
        - destination:
            host: reviews
            subset: v1
```

**Apply it:**

```bash
kubectl apply -f virtual-service-reviews-abort.yaml
```

**Test:**
Reload `http://localhost:8080/productpage` multiple times. About half the time, the reviews section will show errors (or the page might break if it doesn’t handle errors gracefully).

---

## 🧪 Step 4: Remove Fault Injection (Cleanup)

To remove all fault injections and restore normal traffic:

```bash
kubectl delete virtualservice reviews
```

Or, **apply your previous VirtualService** YAML to restore your old routing.

---

## 🧠 What You Learned

* How to **add delays** and **inject errors** into service traffic
* How to observe app behavior and resilience under failure conditions
* How to revert to normal traffic

---

Want to try **timeouts, retries, circuit breakers, or advanced chaos testing** next? Just ask!
