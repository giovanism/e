---
categories:
- technology
date: "2021-12-12T15:09:34+07:00"
description: ""
draft: false
resources:
- name: ingress-gw
  src: ingress.png
  title: Ingress Gateway
- name: ingress2
  src: ingress2.png
  title: Ingress 2
- images:
  - https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/ASR-33_at_CHM.agr.jpg/800px-ASR-33_at_CHM.agr.jpg
  name: reverse-proxy
  src: reverse-proxy.png
  title: Reverse Proxy
tags:
- computer
- kubernetes
- istio
title: 'Istio Series Part 3: Ingress Gateway'
toc: false
---

Hi, it's been a while since I have time to write something for this blog. Last
post about traffic management in Istio covers a lot of topics, a tad too much
now that I think about it again. Next time, I'll aim for under 5 minutes read
time for my weekly posts. Now that it's out of the way, let's talk about Istio
Ingress Gateway.

{{<resfigure
  alt="Ingress Gateway"
  src="ingress-gw"
  title="Ingress Gateway"
>}}

<!--more-->

At first, Ingress (just ingress without gateway) is a topic that's particularly
hard for me to understand. Furthermore, when all I knew before was just Apache
and Nginx web servers. What's the difference between reverse proxy, load
balancer and ingress? I've wondered about this the first time I heard about them
in local Kubernetes meetup and now I can confidently say that they are all the
same thing.

{{<figure
  alt="Wait, it's all just envoy proxies? Always has been."
  src="https://i.imgflip.com/63grdx.jpg"
  title="it's all just envoy proxies"
>}}

Well, at least in Istio they are made up of the same components, Envoy Proxies,
even including the Load Balancers if you are on Google Cloud. However there are
small difference between these terms that I want to point out.

{{<resfigure
  alt="Reverse Proxy"
  src="reverse-proxy"
  title="Reverse Proxy"
>}}

Reverse Proxy as a concept only describe proxy that sits in front of backend
server. People refer to both Apache and Nginx as examples of reverse proxy and
they provide the common functionality that we know such as TLS termination,
virtual hosts, and load balancing. However, in the Kubernetes and Istio world
reverse proxy rarely talked about again. Why? Because in service mesh everything
is behind some kind proxy and the term kind of lost its meaning. It makes
more sense to use other term that can more accurately describe it and *Ingress
Gateway* is just that.

In Istio, Ingress Gateway is envoy proxy deployment that sits at the edge of
Istio Mesh and acts as a gateway to our services. Istio mesh can have multiple
ingress and egress gateways. The gateways is configured through `IstioOperator`
API object and later `istiod` will deploy the gateways inside Istio system

{{<resfigure
  alt="Ingress 2"
  src="ingress2"
  title="Ingress 2"
>}}

Configuring the ingress gateway in Istio can be done through `Gateway` object,
or the new Kubernetes Ingress API. Here's an example of how `Gateway`
configuration looks like *yoinked* from Istio documentation.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: my-gateway
  namespace: some-config-namespace
spec:
  selector:
    app: my-gateway-controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - uk.bookinfo.com
    - eu.bookinfo.com
    tls:
      httpsRedirect: true # sends 301 redirect for http requests
  - port:
      number: 443
      name: https-443
      protocol: HTTPS
    hosts:
    - uk.bookinfo.com
    - eu.bookinfo.com
    tls:
      mode: SIMPLE # enables HTTPS on this port
      serverCertificate: /etc/certs/servercert.pem
      privateKey: /etc/certs/privatekey.pem
  - port:
      number: 9443
      name: https-9443
      protocol: HTTPS
    hosts:
    - "bookinfo-namespace/*.bookinfo.com"
    tls:
      mode: SIMPLE # enables HTTPS on this port
      credentialName: bookinfo-secret # fetches certs from Kubernetes secret
  - port:
      number: 9080
      name: http-wildcard
      protocol: HTTP
    hosts:
    - "*"
  - port:
      number: 2379 # to expose internal service via external port 2379
      name: mongo
      protocol: MONGO
    hosts:
    - "*"
```

You can find the complete reference guide for `Gateway` on
[Istio docs](https://istio.io/latest/docs/reference/config/networking/gateway/).

Any comment and feedback is always welcome. See you next time!

