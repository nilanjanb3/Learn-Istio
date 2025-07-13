Here’s a practical lab to **configure timeouts and retries** for your microservices using Istio! This helps make your applications more resilient to temporary failures and slow responses.

We’ll use the `reviews` service from the Bookinfo app for demonstration.

---

## 🧪 Step 1: Prerequisites

* Istio and Bookinfo app are deployed and running
* Namespace (e.g. `default`) has sidecar injection enabled

---

## 🧪 Step 2: Configure a Timeout

By default, Istio applies a 15-second timeout to HTTP requests. Let’s set a **shorter custom timeout** (e.g., 2 seconds) for calls to the `reviews` service.

```yaml
# virtual-service-reviews-timeout.yaml
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
      timeout: 2s
```

**Apply it:**

```bash
kubectl apply -f virtual-service-reviews-timeout.yaml
```

---

## 🧪 Step 3: Configure Retries

Let’s add a policy to **retry failed requests** up to 3 times, with a 1-second gap between retries.

```yaml
# virtual-service-reviews-retries.yaml
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
      retries:
        attempts: 3
        perTryTimeout: 1s
        retryOn: gateway-error,connect-failure,refused-stream
```

**Apply it:**

```bash
kubectl apply -f virtual-service-reviews-retries.yaml
```

---

## 🧪 Step 4: Combine Timeouts and Retries

You can set **both timeout and retries** together in the same VirtualService.

```yaml
# virtual-service-reviews-timeout-retries.yaml
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
      timeout: 2s
      retries:
        attempts: 3
        perTryTimeout: 1s
        retryOn: gateway-error,connect-failure,refused-stream
```

**Apply it:**

```bash
kubectl apply -f virtual-service-reviews-timeout-retries.yaml
```

---

## 🧪 Step 5: Test the Behavior

* To **see the effect of timeouts**: Try injecting a delay into the `reviews` service (see previous fault injection lab). If the response takes longer than `timeout: 2s`, users will get a timeout error.
* To **see retries**: Simulate temporary failures (e.g., by scaling down pods or injecting errors). You should see multiple retry attempts in the logs.

---

## 🧹 Cleanup

To remove custom timeout/retry settings and revert to default:

```bash
kubectl delete virtualservice reviews
```

Or, reapply your previous routing config.

---

## ✅ Summary

| Feature | Example Config Key     | Effect                                     |
| ------- | ---------------------- | ------------------------------------------ |
| Timeout | `timeout: 2s`          | Fails requests that take longer than 2 sec |
| Retries | `retries: attempts: 3` | Retries failed requests up to 3 times      |

This is essential for real-world production apps—making them more robust and user-friendly!

Want to try **circuit breaking**, **rate limiting**, or another advanced Istio feature? Just say the word!
