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
grafana-cli admin reset-admin-password --homepath /usr/share/grafana
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


## Demo

Run the following commands for the demo