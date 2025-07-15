Absolutely! Here’s a **hands-on lab** to set up and use the **Kiali dashboard** to visualize your Istio service mesh, including the service graph of your microservices.

Kiali provides deep visibility into the service mesh, showing how services connect, traffic flows, and where errors or bottlenecks might be.

---

## 🧪 Step 1: Prerequisites

* Istio is installed and running in your cluster.
* Bookinfo app is deployed.
* (Recommended) Prometheus is installed (usually part of Istio’s demo profile).
* You have `kubectl` access.

---

## 🧪 Step 2: Install Kiali (if not already installed)

You can install Kiali using Istio’s built-in resources.

### **Option 1: Using Istioctl**

```bash
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/kiali.yaml
kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/prometheus.yaml
```

> **Tip:** Wait for the Kiali pod to be `Running`:
>
> ```bash
> kubectl get pods -n istio-system | grep kiali
> ```

---

## 🧪 Step 3: Port-forward the Kiali Service

Run this command to access Kiali locally:

```bash
kubectl port-forward svc/kiali -n istio-system 20001:20001
```

Now, open your browser and go to:

```
http://localhost:20001
```

Login credentials are usually **admin/admin** by default (or no login prompt for demo profile).

---

## 🧪 Step 4: Explore the Service Graph

1. In Kiali, select your namespace (e.g., `default`) from the **namespace drop-down**.
2. Go to the **Graph** tab.
3. You will see a **visual graph** of your services (Bookinfo: productpage, reviews, ratings, details).
4. You can:

   * Hover over services to see **metrics** (RPS, error rate, response time).
   * Click a service to see **details**, health, and recent traces.
   * Watch real-time traffic as you generate load (refresh productpage in your browser to see graph updates).

---

## 🧪 Step 5: Advanced Visualization

* Use Kiali to filter by **workloads, apps, services**.
* Check the **Traffic Animation** option to see real-time request flow.
* Inspect **Istio configuration** (VirtualServices, DestinationRules, Gateways) from the left menu.
* View **metrics** and **traces** for detailed diagnostics.

---

## 🧹 Step 6: Cleanup (Optional)

If you want to remove Kiali:

```bash
kubectl delete -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/kiali.yaml
kubectl delete -f https://raw.githubusercontent.com/istio/istio/release-1.20/samples/addons/prometheus.yaml
```

---

## ✅ What You Achieved

* Installed and accessed the Kiali dashboard.
* Explored the real-time **service graph** and mesh health.
* Investigated Istio’s configuration and metrics.
* Built intuition for troubleshooting and mesh visualization.

---

**Want to try other Istio observability tools like Grafana, Jaeger, or experiment with traffic shifting and seeing it live in Kiali?** Just ask!
