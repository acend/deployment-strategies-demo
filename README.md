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

Install the awesome app

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

Trigger Deployment

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./recreate/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-recreate.yoururl.ch" \
--set awesomeappversion="2.0.0" \
recreate ./recreate
```

Open the grafana dashboard an watch what happens

Remove the app

```bash
helm delete recreate --namespace deployment-strategies-demo
```


## Rolling Update

Install the awesome app

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

Trigger Deployment

```bash
helm upgrade \
--namespace deployment-strategies-demo \
--values ./rolling-update/values.yaml \
--set ingress.hosts[0].host="app-deploymentstrategies-demo-rolling.yoururl.ch" \
--set awesomeappversion="2.0.0" \
rolling ./rolling-update
```

Open the grafana dashboard an watch what happens

Remove the app

```bash
helm delete rolling --namespace deployment-strategies-demo
```


## TODO

Maybe want to use something like that as a graph


sum(increase(flask_http_request_duration_seconds_count{app_name="acend-awesome-python", path="/pod/"}[30s])) by (app_version)
