---
categories:
- technology
date: "2021-10-25T06:58:59+07:00"
description: ""
draft: false
images:
- https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/ASR-33_at_CHM.agr.jpg/800px-ASR-33_at_CHM.agr.jpg
tags:
- computer
- kubernetes
- helm
title: Writing Helm Chart
toc: false
---

Recently I wrote my first Helm chart, well technically copy pasted someone's 
else and make my modifications from there, but hey finally a good time to write
something about Helm.

<!--more-->

Helm chart in a nutshell is bunch of Kubernetes resource templates that we can
inject with values dynamically. Additionally, Helm also provides many killer
features that made it industry standard such as ability to
install/update/uninstall, chart repos/registries, JSON validation, tests,
dependency management, subchart, and vice versa.

Those sounds daunting, but let's try to write a simple chart for now. We can
start with `helm create NAME` command to generate the initial template.

```
k8s:minikube:default ~/tmp> helm create giovanism-chart
Creating giovanism-chart
k8s:minikube:default ~/tmp> tree
.
└── giovanism-chart
    ├── Chart.yaml
    ├── charts
    ├── templates
    │   ├── NOTES.txt
    │   ├── _helpers.tpl
    │   ├── deployment.yaml
    │   ├── hpa.yaml
    │   ├── ingress.yaml
    │   ├── service.yaml
    │   ├── serviceaccount.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml

4 directories, 10 files
k8s:minikube:default ~/tmp>
```

Here we have our `Chart.yaml` that contains information about the chart,
`values.yaml` that holds the default values that will be injected, `charts/` dir
and a bunch of things inside `templates/` dir. For now we can just delete the
`charts/` dir and other files inside `templates/` that aren't `NOTES.txt` or
`_helpers.tpl` so that our chart now looks like this.

```
.
└── giovanism-chart
    ├── Chart.yaml
    ├── templates
    │   ├── NOTES.txt
    │   └── _helpers.tpl
    └── values.yaml
```

This is pretty bare bone it does the job. Next we want to write our Kubernetes
resource inside the `templates/` dir. Let's start with a simple config map and
put it in `giovanism-chart/templates/configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: giovanism-chart-configmap
data:
  evil-dev: "Goodbye, World!"
```



```
k8s:minikube:default ~/tmp> helm install foo ./giovanism-chart
NAME: foo
LAST DEPLOYED: Mon Oct 25 10:14:02 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=giovanism-chart,app.kubernetes.io/instance=foo" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```

The leftover `NOTES.txt` still contains the generated message. We'll remove it
later. Now we can try to run `helm get manifest foo` this command to get the
applied manifests.

```yaml
---
# Source: giovanism-chart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: giovanism-chart-configmap
data:
  evil-dev: "Goodbye, World!"
```

We can also check the config map directly using `kubectl get configmaps
giovanism-chart-configmap -oyaml`.

```yaml
apiVersion: v1
data:
  evil-dev: Goodbye, World!
kind: ConfigMap
metadata:
  annotations:
    meta.helm.sh/release-name: foo
    meta.helm.sh/release-namespace: default
  creationTimestamp: "2021-10-25T03:14:02Z"
  labels:
    app.kubernetes.io/managed-by: Helm
  name: giovanism-chart-configmap
  namespace: default
  resourceVersion: "25683"
  uid: fe640659-dd02-4a1e-84dd-902f85c562fb
```

### Templating

We can make changes to our existing template to make it generate different names
for each installation using the templating syntax.

```diff
--- a/giovanism-chart/templates/configmap.yaml
+++ b/giovanism-chart/templates/configmap.yaml
@@ -1,6 +1,6 @@
 apiVersion: v1
 kind: ConfigMap
 metadata:
-  name: giovanism-chart-configmap
+  name: {{ .Release.Name }}-configmap
 data:
   evil-dev: "Goodbye, World!"
```

Then we do another install using different name using `helm install bar
./giovanism-chart` and see the result with `helm get manifest bar`.

```yaml
---
# Source: giovanism-chart/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: bar-configmap
data:
  evil-dev: "Goodbye, World!"
```

This way templating enable us to have unique Kubernetes resource names and have
both of our releases side by side.

```
k8s:minikube:default git:master ! ~/tmp> helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
bar     default         1               2021-10-26 03:02:16.77455 +0700 WIB     deployed        giovanism-chart-0.1.0   1.16.0
foo     default         1               2021-10-25 10:14:02.184255 +0700 WIB    deployed        giovanism-chart-0.1.0   1.16.0
```

I think that's all. For now I just want to keep up writing regularly rather than
focusing too much on the content. I do hope that in the future I can more
exciting and sophisticated contents.
