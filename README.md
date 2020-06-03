# k8s Deployment strategies demo

This Demo shows how the different deployment mechanisms on how to deploy applications to a kubernetes cluster work.


## Basic Setup

Create a namespace

```bash
kubectl create namespace deployment-strategies-demo
```

Deploy the basic infrastructure() Prometheus, Grafana and our awesome application) using helm 3.

You'll need to set the proper Hostnames and Url

```bash
helm install \
--namespace deployment-strategies-demo \
--values ./deployment-strategies-demo/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo.yoururl.ch" \
--set grafana.ingress.hosts[0].[0]="app-deploymentstrategies-demo.yoururl.ch" \
--set grafana.'grafana\.ini'.grafana_net.url="grafana-deploymentstrategies-demo.yoururl.ch" \
basicsetup ./deployment-strategies-demo
```

where to find the grafana admin password

```bash
kubectl get secret --namespace deployment-strategies-demo basicsetup-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

To reset the admin password use the following command inside the grafana container

```bash
grafana-cli admin reset-admin-password <password>
```


### Upgrade Release

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./deployment-strategies-demo/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo.yoururl.ch" \
--set grafana.ingress.hosts[0]="grafana-deploymentstrategies-demo.yoururl.ch" \
--set grafana.'grafana\.ini'.grafana_net.url="grafana-deploymentstrategies-demo.yoururl.ch" \
basicsetup ./deployment-strategies-demo
```


### Update Dependencies

```bash
helm dependency update ./deployment-strategies-demo
```


## Recreate Update

Install the awesome app version 1.0.0

```bash
helm install \
--namespace deployment-strategies-demo \
--values ./recreate/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-recreate.yoururl.ch" \
recreate ./recreate
```

create load on the application

```bash
while sleep 0.1; do curl "https://app-deploymentstrategies-demo-recreate.yoururl.ch/pod/"; done
```

Trigger Deployment version 2.0.0

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./recreate/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-recreate.yoururl.ch" \
--set awesomeappversion="2.0.0" \
recreate ./recreate
```

Open the grafana dashboard and watch what happens

Remove the app

```bash
helm delete recreate --namespace deployment-strategies-demo
```


## Rolling Update

Install the awesome app version 1.0.0

```bash
helm install \
--namespace deployment-strategies-demo \
--values ./rolling-update/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-rolling.yoururl.ch" \
rolling ./rolling-update
```

create load on the application

```bash
while sleep 0.1; do curl "https://app-deploymentstrategies-demo-rolling.yoururl.ch/pod/"; done
```

Trigger Deployment version 2.0.0

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./rolling-update/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-rolling.yoururl.ch" \
--set awesomeappversion="2.0.0" \
rolling ./rolling-update
```

Open the grafana dashboard and watch what happens

Remove the app

```bash
helm delete rolling --namespace deployment-strategies-demo
```


## Blue Green Version selector Labels

This Version of the Blue Green Deployment mechanism uses selector Labels in the service definition to switch traffic from pods with Version 1 to Version 2 by updating the `version` label in the service definition:

```yaml
# Service
# From version: 1.0.0
  selector:
    app.kubernetes.io/instance: blue-green
    app.kubernetes.io/name: blue-green
    version: 1.0.0
# To version: 2.0.0
  selector:
    app.kubernetes.io/instance: blue-green
    app.kubernetes.io/name: blue-green
    version: 2.0.0
```

Install the awesome app Version 1

```bash
helm install \
--namespace deployment-strategies-demo \
--values ./blue-green/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-blue-green.yoururl.ch" \
blue-green ./blue-green
```

Also install the Version 2

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./blue-green/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-blue-green.yoururl.ch" \
--set version2.enabled=true \
blue-green ./blue-green
```

create load on the application

```bash
while sleep 0.1; do curl "https://app-deploymentstrategies-demo-blue-green.yoururl.ch/pod/"; done
```

switch from Version 1 to Version 2

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./blue-green/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-blue-green.yoururl.ch" \
--set version2.enabled=true \
--set serviceselectorversion=2.0.0 \
blue-green ./blue-green
```

Open the grafana dashboard and watch what happens

Remove Version 1

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./blue-green/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-blue-green.yoururl.ch" \
--set version1.enabled=false \
--set version2.enabled=true \
--set serviceselectorversion=2.0.0 \
blue-green ./blue-green
```

Remove the app

```bash
helm delete blue-green --namespace deployment-strategies-demo
```


## Blue Green version ingress backend

This Version of the Blue Green Deployment mechanism uses the backend configuration in the ingress definition to switch traffic from Service of Version 1 to Version 2 by updating the `version` label in the ingress definition:

```yaml
# Ingress
# From version: 1.0.0
spec:
  rules:
  - host: app-deploymentstrategies-demo-blue-green-ingress.yoururl.ch
    http:
      paths:
      - backend:
          serviceName: blue-green-ingress-1
# To version: 2.0.0
spec:
  rules:
  - host: app-deploymentstrategies-demo-blue-green-ingress.yoururl.ch
    http:
      paths:
      - backend:
          serviceName: blue-green-ingress-2
```

Install the awesome app Version 1

```bash
helm install \
--namespace deployment-strategies-demo \
--values ./blue-green-ingress/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-blue-green-ingress.yoururl.ch" \
blue-green-ingress ./blue-green-ingress
```

Also install the Version 2

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./blue-green-ingress/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-blue-green-ingress.yoururl.ch" \
--set version2.enabled=true \
blue-green-ingress ./blue-green-ingress
```

create load on the application

```bash
while sleep 0.1; do curl "https://app-deploymentstrategies-demo-blue-green-ingress.yoururl.ch/pod/"; done
```

switch from Version 1 to Version 2

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./blue-green-ingress/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-blue-green-ingress.yoururl.ch" \
--set version2.enabled=true \
--set ingressbackendservicename=blue-green-ingress-2 \
blue-green-ingress ./blue-green-ingress
```

Open the grafana dashboard and watch what happens

Remove Version 1

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./blue-green-ingress/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-blue-green-ingress.yoururl.ch" \
--set version1.enabled=false \
--set version2.enabled=true \
--set ingressbackendservicename=blue-green-ingress-2 \
blue-green-ingress ./blue-green-ingress
```

Remove the app

```bash
helm delete blue-green-ingress --namespace deployment-strategies-demo
```


## Canary Releasing

The Canary Releasing Deployment mechanism uses the nginx configuration in the ingress definition to let a specific part of the traffic to the backend service 2, while the rest, still goes to version 1:

```yaml
# Ingress
# Canary annotations
metadata:
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"
    nginx.ingress.kubernetes.io/canary-weight: "10"
```

Install the awesome app Version 1

```bash
helm install \
--namespace deployment-strategies-demo \
--values ./canary/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-canary.yoururl.ch" \
canary ./canary
```

Also install the Version 2

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./canary/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-canary.yoururl.ch" \
--set version2.enabled=true \
canary ./canary
```

create load on the application

```bash
while sleep 0.1; do curl "https://app-deploymentstrategies-demo-canary.yoururl.ch/pod/"; done
```

Deploy Canary Ingress, 10% of the traffic gets routed to the new version

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./canary/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-canary.yoururl.ch" \
--set version2.enabled=true \
--set ingresscanary.enabled=true \
--set ingresscanary.hosts[0].host="app-deploymentstrategies-demo-canary.yoururl.ch" \
canary ./canary
```

Open the grafana dashboard and watch what happens

After successful verifying of the new version, remove canary and switch all traffic to version 2

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./canary/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-canary.yoururl.ch" \
--set version2.enabled=true \
--set ingresscanary.enabled=false \
--set ingresscanary.hosts[0].host="app-deploymentstrategies-demo-canary.yoururl.ch" \
--set ingressbackendservicename="canary-2" \
canary ./canary
```

Remove Version 1

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./canary/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-canary.yoururl.ch" \
--set version1.enabled=false \
--set version2.enabled=true \
--set ingresscanary.enabled=false \
--set ingressbackendservicename="canary-2" \
canary ./canary
```

Remove the app

```bash
helm delete canary --namespace deployment-strategies-demo
```


## TODO

Maybe want to use something like that as a graph

```
sum(increase(flask_http_request_duration_seconds_count{app_name="acend-awesome-python", path="/pod/"}[30s])) by (app_version)
```

Add the following mechanisms:

* AB Testing
* Shadow

Create Slidedeck
