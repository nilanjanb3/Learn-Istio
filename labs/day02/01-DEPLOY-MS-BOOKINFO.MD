Here’s a complete hands-on **lab to deploy the Bookinfo microservices application** on your Istio-enabled Kubernetes cluster. This app is often used to demonstrate Istio’s capabilities such as traffic routing, telemetry, and observability.

---

### ✅ Prerequisites

Before starting:
- A working **Kubernetes cluster** (e.g., Kind, Minikube)
- **Istio is installed** using `istioctl`
- **Sidecar injection is enabled** for the namespace where you’ll deploy the app

---

### 🧪 Step 1: Enable Sidecar Injection

Make sure automatic sidecar injection is enabled for the namespace you’re using (we’ll use `default` for simplicity):

```bash
kubectl label namespace default istio-injection=enabled --overwrite
```

---

### 🧪 Step 2: Deploy the Bookinfo Application

Apply the main manifest that deploys the app’s services and deployments:

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/bookinfo/platform/kube/bookinfo.yaml
```

Wait a few seconds, then check that all pods are running:

```bash
kubectl get pods
```

You should see pods for `productpage`, `details`, `reviews`, and `ratings`.

---

### 🧪 Step 3: Verify the Services

List all services deployed:

```bash
kubectl get svc
```

Look for services like `details`, `reviews`, `ratings`, and `productpage`.

---

### 🧪 Step 4: Expose the Application Using Istio Gateway

Now expose the app externally using Istio’s **Ingress Gateway**.

Apply the Gateway and VirtualService configuration:

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/bookinfo/networking/bookinfo-gateway.yaml
```

Check that the gateway was created:

```bash
kubectl get gateway
```

---

### 🧪 Step 5: Access the Application (Local Cluster Setup)

For Kind or Minikube (which don’t provide external IPs), port-forward the Istio Ingress Gateway:

```bash
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```

Now visit in your browser:

```
http://localhost:8080/productpage
```

You should see the **Bookinfo product page UI** loading with reviews, ratings, etc.

---

### 🧪 Step 6: Observe the Sidecars (Envoy)

Check that sidecars are injected:

```bash
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{range .spec.containers[*]}{.name}{"\t"}{end}{"\n"}'
```

You’ll see two containers per pod (`<app>`, `istio-proxy`).

---

### 🧪 Step 7: Cleanup (Optional)

To delete all Bookinfo resources:

```bash
kubectl delete -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/bookinfo/platform/kube/bookinfo.yaml
kubectl delete -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/bookinfo/networking/bookinfo-gateway.yaml
```

---

### 📌 Summary of What You Achieved

- Deployed a real-world microservices app
- Verified pod and service health
- Enabled Istio sidecar injection
- Exposed the app using Istio Gateway
- Accessed the app on localhost
- Observed sidecar proxies (Envoy)

Let me know when you're ready for traffic management labs like **Canary deployments**, **A/B testing**, or **header-based routing**!