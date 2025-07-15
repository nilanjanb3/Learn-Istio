Absolutely! Here’s a practical lab to **verify mTLS certificates and secure communication** between services in your Istio mesh.

This will help you understand how Istio issues, rotates, and uses certificates for encrypting and authenticating pod-to-pod traffic.

---

## 🧪 Step 1: Prerequisites

* Istio installed and STRICT mTLS enabled (previous lab)
* Bookinfo app deployed and working
* Sidecar injection enabled

---

## 🧪 Step 2: Test Secure Communication

**A. Simulate a call between services:**

Exec into a pod (e.g., `productpage`) and try to `curl` another service (e.g., `details`):

```bash
kubectl exec -it productpage-v1-xxxxxxx -c istio-proxy -- \
curl -s http://details:9080/details/1
```

If successful, the connection is secured with mTLS (all traffic from sidecar-to-sidecar is automatically encrypted and authenticated).

**B. Test communication from a pod without a sidecar**
(Try deploying a busybox pod without Istio injection and attempt to call a service.) It should **fail** with mTLS STRICT.

---

## 🧪 Step 3: (Optional) View Envoy Proxy Config

For advanced debugging, you can view the Envoy config dump:

```bash
kubectl exec -it productpage-v1-xxxxxxx -c istio-proxy -- curl http://localhost:15000/config_dump
```

Look for `common_tls_context` to see certificate details.

---

## 🧹 Cleanup

No cleanup needed, unless you want to disable STRICT mTLS.

---

## ✅ What You Verified

* Istio issues and mounts unique certificates per workload (service account identity)
* Pod-to-pod traffic is encrypted and authenticated (mTLS)
* mTLS status via Istio CLI and Kiali dashboard
* Communication fails if traffic isn’t using mTLS

---

Let me know if you want to explore **certificate rotation**, **custom CA**, or dive deeper into Istio security features!
