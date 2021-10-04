+++
title = "Istio Series Part 2: Traffic Management"
date = 2021-09-27T06:22:02+07:00
description = ""
draft = false
toc = false
categories = ["technology"]
tags = ["computer", "kubernetes", "istio"]
images = [
  "https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/ASR-33_at_CHM.agr.jpg/800px-ASR-33_at_CHM.agr.jpg"
] # overrides the site-wide open graph image
+++

Back to another Istio post. Previously, we've set up our own Kubernetes
cluster, Istio mesh, toy service and also Kiali dashboard to observe our mesh.
Now we are set to try basic Istio features such as routing, traffic shifting,
fault injection and circuit breaking.

<!--more-->

Before we jump right into the related Istio tasks, I would like to explain a
few of the basic Istio APIs. You can think of Istio service mesh as a bunch of
reverse proxy that work together to routes network traffic intelligently. Much
like traditional reverse proxies, Istio needs to know the services and
endpoints where to direct those traffic that comes into the cluster. For
Kubernetes, Istio can automatically detects services and endpoints to build its
own service registry. Then, Envoy proxies managed by Istio will use this
service registry to route traffic. For more advanced configuration Istio can
also load balance service that have multiple instances, or direct a percentage
of traffic to different versions running at the same time as part of A/B
testing or canary deployment.

> Can we just achieve the same traffic shifting/splitting by scaling the Kubernetes 
> replica?

Yes, but there are a few disadvantages to that approach. By using Istio or
other load balancer the traffic management will be independent of the compute
resources that is Kubernetes Pod. This way the traffic routing doesn't conflict
with pod auto-scaler responsibility to ensure there's enough compute resource.

After that out of the way, let's talk about Istio basic APIs. We can interact
with Istio APIs using Kubernetes Custom Resource Definitions (CRDs).

## Basic APIs

### Gateway

In Istio we have gateway that describes a load balancer at the edge of the
mesh. Here we will list the available servers by their port, hosts and other
available configurations.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: ext-host-gwy
spec:
  selector:
    app: my-gateway-controller
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    hosts:
    - ext-host.example.com
    tls:
      mode: SIMPLE
      credentialName: ext-host-cert
```

For now Istio still doesn't know where to route traffic that comes to the mesh
from this port. We'll take care of that by binding it to virtual service using
the `gateways` field.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: virtual-svc
spec:
  hosts:
  - ext-host.example.com
  gateways:
  - ext-host-gwy
```

### VirtualService

In virtual service lies the bread and butter of Istio routing configurations.
With Virtual service we can route traffic to multiple version of the same
service, different services, or heck even delegate to another virtual service.
This flexibility comes from how Istio decouples where client send their request
from the actual destination workloads.

Let's take a look at another example of virtual service.

```yaml
apiVersion: networking.istio.io/v1alpha3
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
        subset: v3
```

Here we have a virtual service that will route traffic to a destination rule
with host `reviews`. However it also defines two different routes, one for
subset `v2` and another for subset `v3`. We can also see that the route to
version `v2` of our reviews service requires a matching `end-user: jason`
header, and the rest of the request can be routed to version `v3`.

### DestinationRule

Next, we have destination rule where we can define the actual traffic
destination, either using Kubernetes service or service outside the mesh using
service entry that we'll visit later. Besides that we can also set a few
configuration that's attached to the destination such as load balancing option
or connection pooling.

Here's an example of destination rule.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  host: my-svc
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: v3
    labels:
      version: v3
```

Here we can see that `my-svc` service have three different subsets, and use
random load balancer for all but the `v2` subset which is set to round robin.


### ServiceEntry

With ServiceEntry you can add additional entries to Istio service registry.
This way you can leverage Istio to route traffic to service outside your platform
such as web APIs, or internal services outside Kubernetes cluster.

```yaml
---
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: external-svc-mongocluster
spec:
  hosts:
  - mymongodb.somedomain # not used
  addresses:
  - 192.192.192.192/24 # VIPs
  ports:
  - number: 27018
    name: mongodb
    protocol: MONGO
  location: MESH_INTERNAL
  resolution: STATIC
  endpoints:
  - address: 2.2.2.2
  - address: 3.3.3.3
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: mtls-mongocluster
spec:
  host: mymongodb.somedomain
  trafficPolicy:
    tls:
      mode: MUTUAL
      clientCertificate: /etc/certs/myclientcert.pem
      privateKey: /etc/certs/client_private_key.pem
      caCertificates: /etc/certs/rootcacerts.pem
```

Here's an example of service entry configuration for MongoDB instances running
on unmanaged VMs. The destination rule is used to initiate mTLS connections
just like any other services inside the mesh.


## Traffic Management Task

Now that we already have some fundamental concept about Istio basic APIs, let's
go try it out for ourselves. For this practice, besides executing the command
in order I also use `kubectl diff` and the Kiali dashboard to cross check what
resource/configuration has been modified.

### Request Routing

Our Bookinfo application is actually running with multiple versions of services
now. You can check it yourself how the reviews sometimes has no ratings,
ratings or colored ratings.

We will now try to apply virtual services so traffic gonna be routed to the `v1` subsets.

```
istio> kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
virtualservice.networking.istio.io/productpage created
virtualservice.networking.istio.io/reviews created
virtualservice.networking.istio.io/ratings created
virtualservice.networking.istio.io/details created
```

Now try to verify the changes by refreshing the page and also checking the
Kiali dashboard. There should be no ratings in the `v1` subset because `reviews:v1`
doesn't call ratings service.

> insert Kiali dashboard pic of v1

Next, we want to try to change the previous routing by sending some of the
traffic to the `v2` subset. We'll use
`samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml` manifest to
apply this configuration. Before apply this change, let's check the diff by running
`kubectl diff -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml`.

```diff
diff -u -N /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/LIVE-544756195/networking.istio.io.v1alpha3.VirtualService.default.reviews /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/MERGED-442455014/networking.istio.io.v1alpha3.VirtualService.default.reviews
--- /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/LIVE-544756195/networking.istio.io.v1alpha3.VirtualService.default.reviews 2021-10-03 16:34:06.000000000 +0700
+++ /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/MERGED-442455014/networking.istio.io.v1alpha3.VirtualService.default.reviews       2021-10-03 16:34:06.000000000 +0700
@@ -5,7 +5,7 @@
     kubectl.kubernetes.io/last-applied-configuration: |
       {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"reviews","namespace":"default"},"spec":{"hosts":["reviews"],"http":[{"route":[{"destination":{"host":"reviews","subset":"v1"}}]}]}}
   creationTimestamp: "2021-10-03T09:33:17Z"
-  generation: 1
+  generation: 2
   managedFields:
   - apiVersion: networking.istio.io/v1alpha3
     fieldsType: FieldsV1
@@ -29,6 +29,14 @@
   hosts:
   - reviews
   http:
+  - match:
+    - headers:
+        end-user:
+          exact: jason
+    route:
+    - destination:
+        host: reviews
+        subset: v2
   - route:
     - destination:
         host: reviews
```

We can see that it adds another route to `reviews:v2`, and it also requires the
request to have an exactly matching `end-user: jason` header. Perfect! Now
let's apply this manifest.

```
istio> kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml
virtualservice.networking.istio.io/reviews configured
```

Try to login as "jason" and we should see the ratings as uncolored stars.

> Kiali shows some traffic to `reviews:v2` and `ratings:v2`

### Fault Injection

Next, we can try fault injection by introducing 7s delay between `reviews:v2`
and `ratings` service. The `reviews:v2` already has hard-coded timeout of 10s
so we are expecting correct answer only with some delay. Try to check the diff
`kubectl diff -f
samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml` before
applying it.

```diff
diff -u -N /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/LIVE-401048783/networking.istio.io.v1alpha3.VirtualService.default.ratings /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/MERGED-857511906/networking.istio.io.v1alpha3.VirtualService.default.ratings
--- /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/LIVE-401048783/networking.istio.io.v1alpha3.VirtualService.default.ratings 2021-10-04 09:14:58.000000000 +0700
+++ /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/MERGED-857511906/networking.istio.io.v1alpha3.VirtualService.default.ratings       2021-10-04 09:14:58.000000000 +0700
@@ -5,7 +5,7 @@
     kubectl.kubernetes.io/last-applied-configuration: |
       {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"ratings","namespace":"default"},"spec":{"hosts":["ratings"],"http":[{"route":[{"destination":{"host":"ratings","subset":"v1"}}]}]}}
   creationTimestamp: "2021-10-03T09:33:17Z"
-  generation: 1
+  generation: 2
   managedFields:
   - apiVersion: networking.istio.io/v1alpha3
     fieldsType: FieldsV1
@@ -29,6 +29,19 @@
   hosts:
   - ratings
   http:
+  - fault:
+      delay:
+        fixedDelay: 7s
+        percentage:
+          value: 100
+    match:
+    - headers:
+        end-user:
+          exact: jason
+    route:
+    - destination:
+        host: ratings
+        subset: v1
   - route:
     - destination:
         host: ratings
```

We can see that it adds another route to ratings virtual service. Only requests
authenticated as `jason` will experience the additional delay. Let's go apply
this manifest and check the delay for ourselves because no way I can show this
with a static image.

```
istio> kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml
virtualservice.networking.istio.io/ratings configured
```

Turns out our supposedly still working reviews service is unavailable. How this
could happen? It's because the `productpage` service also has hard-coded
timeout of 3s + 1 retry and will give up waiting response from reviews service
after 6s.

To fix this bug, we'll need to either increase the `productpage` to `reviews`
or decrease `reviews` to `ratings` timeout. The fix implemented in `reviews:v3`
is to reduce the internal timeout to 2.5s. To achieve end-to-end flow with
injected delay, we'll still need below 2.5s, for example 2s.

For now we'll just clean up the virtual service by running.

```
istio> kubectl delete -f samples/bookinfo/networking/virtual-service-all-v1.yaml
virtualservice.networking.istio.io "productpage" deleted
virtualservice.networking.istio.io "reviews" deleted
virtualservice.networking.istio.io "ratings" deleted
virtualservice.networking.istio.io "details" deleted
```

### Traffic Shifting

To try traffic shifting we'll need to re-apply the virtual services first.

```
istio> kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml
virtualservice.networking.istio.io/productpage created
virtualservice.networking.istio.io/reviews created
virtualservice.networking.istio.io/ratings created
virtualservice.networking.istio.io/details created
```

With v1 we won't see the ratings at all on the product page because
`reviews:v1` doesn't call ratings service. We'll try to shift half the traffic
of reviews service to `reviews:v3` with
`samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml`

```diff
diff -u -N /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/LIVE-518052740/networking.istio.io.v1alpha3.VirtualService.default.reviews /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/MERGED-577000211/networking.istio.io.v1alpha3.VirtualService.default.reviews
--- /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/LIVE-518052740/networking.istio.io.v1alpha3.VirtualService.default.reviews   2021-10-05 01:23:40.000000000 +0700
+++ /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/MERGED-577000211/networking.istio.io.v1alpha3.VirtualService.default.reviews 2021-10-05 01:23:40.000000000 +0700
@@ -5,7 +5,7 @@
     kubectl.kubernetes.io/last-applied-configuration: |
       {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"reviews","namespace":"default"},"spec":{"hosts":["reviews"],"http":[{"route":[{"destination":{"host":"reviews","subset":"v1"}}]}]}}
   creationTimestamp: "2021-10-04T18:18:25Z"
-  generation: 1
+  generation: 2
   managedFields:
   - apiVersion: networking.istio.io/v1alpha3
     fieldsType: FieldsV1
@@ -33,3 +33,8 @@
     - destination:
         host: reviews
         subset: v1
+      weight: 50
+    - destination:
+        host: reviews
+        subset: v3
+      weight: 50
```

The manifest will add another route to `reviews:v3` in reviews virtual service. Cool!

```
istio> kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml
virtualservice.networking.istio.io/reviews configured
```

Now we'll see close to exact 50:50 split of no ratings vs colored ratings
displayed on product page. If there's no problem with `reviews:v3`, we can
direct 100% of the traffic with
`samples/bookinfo/networking/virtual-service-reviews-v3.yaml `

```diff
vice.default.reviews /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/MERGED-723337435/networking.istio.io.v1alpha3.VirtualService.default.reviews
--- /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/LIVE-857487468/networking.istio.io.v1alpha3.VirtualService.default.reviews   2021-10-05 01:33:20.000000000 +0700
+++ /var/folders/5f/g2fc5rs5661g509_x0_7c8ch0000gn/T/MERGED-723337435/networking.istio.io.v1alpha3.VirtualService.default.reviews 2021-10-05 01:33:20.000000000 +0700
@@ -5,7 +5,7 @@
     kubectl.kubernetes.io/last-applied-configuration: |
       {"apiVersion":"networking.istio.io/v1alpha3","kind":"VirtualService","metadata":{"annotations":{},"name":"reviews","namespace":"default"},"spec":{"hosts":["reviews"],"http":[{"route":[{"destination":{"host":"reviews","subset":"v1"},"weight":50},{"destination":{"host":"reviews","subset":"v3"},"weight":50}]}]}}
   creationTimestamp: "2021-10-04T18:18:25Z"
-  generation: 2
+  generation: 3
   managedFields:
   - apiVersion: networking.istio.io/v1alpha3
     fieldsType: FieldsV1
@@ -32,9 +32,4 @@
   - route:
     - destination:
         host: reviews
-        subset: v1
-      weight: 50
-    - destination:
-        host: reviews
         subset: v3
-      weight: 50
```

Yup, just remove the `reviews:v1` route and the remaining weight. Let's apply the manifest.

```
istio> kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-v3.yaml
virtualservice.networking.istio.io/reviews configured
```

Now we'll just see colored ratings.

I only add these tasks for now. I'll add more task later `:fingers_crossed:`
