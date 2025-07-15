Here’s a hands-on lab to **apply PeerAuthentication** (mTLS enforcement) **and RequestAuthentication** (JWT validation) policies in Istio. You’ll enforce encrypted, authenticated service-to-service communication **and** require inbound requests to carry a valid JWT.

---

## ✅ Prerequisites

* Istio **1.10+** installed on your cluster
* Bookinfo app (or any sample app) deployed in namespace `default`
* Sidecar injection enabled in `default`
* A sample JWT issuer & audience (we’ll use [Auth0’s demo token](https://auth0.com/docs/tokens))

---

## 🧪 Step 1: Enforce mTLS with PeerAuthentication

1. **Create a mesh-wide strict mTLS policy** so that *all* pods must speak TLS to each other:

   ```yaml
   # peer-authentication-mesh-strict.yaml
   apiVersion: security.istio.io/v1beta1
   kind: PeerAuthentication
   metadata:
     name: mesh-default
     namespace: istio-system
   spec:
     mtls:
       mode: STRICT
   ```

2. **Apply it:**

   ```bash
   kubectl apply -f peer-authentication-mesh-strict.yaml
   ```

3. **Verify** by port-forwarding Envoy admin port and scraping certs:

   ```bash
   kubectl port-forward deploy/productpage-v1 15000:15000
   curl http://localhost:15000/certs | jq .
   # you should see certificate chains with SPIFFE SANs
   ```

---

## 🧪 Step 2: Allow Plain-Text for Ingress (Optional)

If you need to accept external (plain-text) traffic at the ingress gateway before mTLS takes over inside the mesh, create a namespace-scoped PeerAuthentication:

```yaml
# peer-authentication-ingress-permissive.yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: ingress-permissive
  namespace: istio-system
spec:
  selector:
    matchLabels:
      istio: ingressgateway
  mtls:
    mode: PERMISSIVE
```

```bash
kubectl apply -f peer-authentication-ingress-permissive.yaml
```

---

## 🧪 Step 3: Require JWTs with RequestAuthentication

1. **Define a RequestAuthentication** policy for the `productpage` service in `default`. This tells Envoy to validate incoming JWTs:

   ```yaml
   # request-authn-jwt.yaml
   apiVersion: security.istio.io/v1beta1
   kind: RequestAuthentication
   metadata:
     name: jwt-validate
     namespace: default
   spec:
     selector:
       matchLabels:
         app: productpage
     jwtRules:
     - issuer: "https://YOUR_DOMAIN/"
       jwksUri: "https://YOUR_DOMAIN/.well-known/jwks.json"
       forwardOriginalToken: true
   ```

   > **Tip:** Replace `YOUR_DOMAIN` with your JWT issuer domain (e.g., Auth0).

2. **Apply it:**

   ```bash
   kubectl apply -f request-authn-jwt.yaml
   ```

---

## 🧪 Step 4: Enforce Access with AuthorizationPolicy

Now block any request *without* a valid JWT:

```yaml
# authz-policy-require-jwt.yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: require-jwt
  namespace: default
spec:
  selector:
    matchLabels:
      app: productpage
  action: DENY
  rules:
  - from:
    - notRequestPrincipals: ["*"]
```

```bash
kubectl apply -f authz-policy-require-jwt.yaml
```

This policy **denies** any request to `productpage` that hasn’t presented a validated JWT principal.

---

## 🧪 Step 5: Test Your Policies

1. **Without JWT**:

   ```bash
   curl -i http://<INGRESS_HOST>/productpage
   ```

   You should get a **403 Forbidden**.

2. **With JWT**:

   ```bash
   TOKEN=<your_jwt_here>
   curl -i -H "Authorization: Bearer $TOKEN" http://<INGRESS_HOST>/productpage
   ```

   You should see the **200 OK** HTML of the product page.

3. **mTLS Verification** (inside the mesh):

   ```bash
   kubectl exec -it deploy/productpage-v1 -c istio-proxy -- \
     curl -v --resolve "ratings.default.svc.cluster.local:15002:127.0.0.1" \
     https://ratings.default.svc.cluster.local:15002/ratings/1
   ```

   The `https` call between sidecars confirms mTLS is enforced.

---

## 🧹 Cleanup

```bash
kubectl delete -f authz-policy-require-jwt.yaml
kubectl delete -f request-authn-jwt.yaml
kubectl delete -f peer-authentication-ingress-permissive.yaml
kubectl delete -f peer-authentication-mesh-strict.yaml
```

---

### ✅ What You Achieved

* **PeerAuthentication** enforced STRICT mTLS mesh-wide
* (Optional) PERMISSIVE mode on ingress gateway
* **RequestAuthentication** validated incoming JWTs
* **AuthorizationPolicy** blocked unauthenticated requests

Feel free to tweak selectors, namespaces, or JWT rules for your own services!
