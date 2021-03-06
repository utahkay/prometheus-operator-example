# Example use of Prometheus operator
This is a working example showing the use of the prometheus operator to set up Prometheus and Alertmanager.

## Install Prometheus operator
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack
```

## Create the namespace to use for this example
```
kubectl create namespace monitoring
```

## Slack webhook secret
Make a file named `slack-webhook` that contains your slack webhook.

## The app to monitor
```
kubectl apply -f app.yaml -n monitoring
```

## Prometheus
```
mkdir generated
kubectl create secret generic my-prometheus-additional-scrape-configs --from-file=scrape-config.yaml --dry-run -oyaml > generated/prometheus-additional-scrape-configs.yaml
kubectl apply -f generated/prometheus-additional-scrape-configs.yaml -n monitoring
kubectl apply -f prometheusrule.yaml -n monitoring
kubectl apply -f prometheus.yaml -n monitoring
```

## Alertmanager
```
mkdir generated
kubectl create secret generic my-alertmanager-slack-webhook --from-file slack-webhook --dry-run -oyaml > generated/alertmanager-slack-webhook.yaml
kubectl apply -f generated/alertmanager-slack-webhook.yaml -n monitoring
kubectl apply -f alertmanager-config.yaml -n monitoring
kubectl apply -f alertmanager.yaml -n monitoring
```

## Notes
* Outside of local development you'll probably need a clusterrole, clusterrolebinding, and service account for Prometheus. 
* The storage class in prometheus.yaml is `hostpath` for Docker Desktop. If you're not on Docker Desktop you may need to use a different storage class.
