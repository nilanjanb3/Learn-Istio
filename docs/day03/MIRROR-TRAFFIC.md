### 📌 Istio: Mirroring Traffic

---

### 🪞 **What is Traffic Mirroring?**

**Traffic Mirroring** (also called **Shadowing**) in Istio lets you **send a copy of live production traffic** to a different service version, **without affecting real users**.

> **Purpose:** Safely test new service versions using real production traffic before a full rollout.

---

### 🛠️ **How Does It Work?**

* All incoming requests are **routed to the main service** as usual.
* **A copy** of each request is sent to another version (the "mirror" service).
* Responses from the mirror service are **ignored** (not sent to the user).

---

### 🧪 **Example: Mirror 100% of Traffic to `v2`**

```yaml
apiVersion: networking.istio.io/v1alpha3
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
  - mirror:
      host: reviews
      subset: v2
    mirrorPercent: 100
```

* **Main traffic:** 100% goes to `reviews v1` (users see this).
* **Mirror traffic:** A **copy** is sent to `reviews v2` for every request.

---

### 🧪 **Example: Partial Mirroring (e.g., 20%)**

```yaml
http:
- route:
  - destination:
      host: reviews
      subset: v1
  mirror:
    host: reviews
    subset: v2
  mirrorPercent: 20
```

* Only **20%** of requests are mirrored to `v2`.

---

### 🌍 **Real-World Use Cases**

| Use Case                     | Why Use Mirroring?                                                  |
| ---------------------------- | ------------------------------------------------------------------- |
| **Test new versions**        | Check if `v2` handles real traffic properly (errors, latency, logs) |
| **Performance benchmarking** | Measure how new code performs at production scale                   |
| **Data migration**           | Ensure new DBs/services work before cutover                         |
| **Debugging**                | Observe issues that may not show up in staging/testing              |

---

### ⚠️ **Considerations**

* **Mirror traffic can affect resource usage** on the mirrored service—be sure it can handle the extra load.
* **Responses from mirrored service are ignored**—it's safe to test breaking changes.
* **No user impact**—users always see responses from the main route.

---

### ✅ **Summary Table**

| Feature       | Description                                                         |
| ------------- | ------------------------------------------------------------------- |
| **Mirroring** | Sends copies of real requests to a test version (not seen by users) |
| **Config**    | `VirtualService > http > mirror` and `mirrorPercent`                |
| **Best For**  | Testing, debugging, migration, benchmarks with real prod traffic    |

---

Let me know the next topic when you’re ready!
