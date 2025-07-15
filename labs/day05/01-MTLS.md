Here's a **step-by-step lab** to enable **strict mTLS (Mutual TLS)** in your Istio service mesh. This will **enforce encrypted communication** between your services, ensuring all service-to-service traffic inside the mesh is authenticated and encrypted.

---

## 🧪 Step 1: Prerequisites

* Istio is installed and running
* Bookinfo app is deployed
* Sidecar injection is enabled in your namespace (`default`)

---

## 🧪 Step 2: Verify mTLS Status (Optional)

You can check the current mTLS status using:

```bash
istioctl authn tls-check
```

This will show which namespaces/services are using mTLS and at what mode.

---

## 🧪 Step 3: Enable STRICT mTLS Mesh-wide

You’ll use an **Istio PeerAuthentication** resource in the `istio-system` namespace to enforce strict mTLS for all namespaces:

```yaml
# peer-authentication-strict.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: istio-system
spec:
  mtls:
    mode: STRICT
```

**Apply it:**

```bash
kubectl apply -f peer-authentication-strict.yaml
```

This ensures **all workloads in the mesh require mTLS**. Any non-Istio traffic will be rejected.

---

## 🧪 Step 4: Enable mTLS for a Specific Namespace (Optional)

If you want to enable STRICT mTLS only for a namespace (e.g., `default`), use:

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: default
spec:
  mtls:
    mode: STRICT
```

Apply with:

```bash
kubectl apply -f - <<EOF
<yaml content here>
EOF
```

---

## 🧪 Step 5: Test Service-to-Service Communication

* Make sure all services in the mesh have sidecars (istio-proxy).
* Generate some traffic (refresh Bookinfo productpage, or use `curl` between services).
* If a pod **without** a sidecar tries to talk to another service, the connection should be **rejected**.

---

## 🧪 Step 6: (Optional) Visualize mTLS Status in Kiali

* Open the Kiali dashboard (`http://localhost:20001`)
* In the Graph view, mTLS-enabled edges between services will appear with a **lock icon**.

---

## 🧹 Step 7: Remove Strict mTLS (Cleanup)

To remove mesh-wide strict mTLS:

```bash
kubectl delete -f peer-authentication-strict.yaml
```

---

## ✅ What You Achieved

* Enforced **STRICT mTLS** mesh-wide (or namespace-wide)
* All pod-to-pod traffic is **encrypted and authenticated**
* Visualized and validated mTLS status

---

Ready to try **AuthorizationPolicies**, test mTLS failures, or integrate with SPIFFE/SPIRE identities? Just ask!
