Сначала применяешь манифесты приложения:

```bash
kubectl apply -f Task2/01_deployment.yaml
kubectl apply -f Task2/03_metrics.yaml
kubectl apply -f Task2/02_scaling.yaml
```

Потом запускаешь мониторинг:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
kubectl create namespace monitoring
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring

helm upgrade --install prom-adapter prometheus-community/prometheus-adapter \
  -n monitoring \
  -f Task2/04_prometheus_adapter_values.yaml

# проверка, что custom metrics API поднят
kubectl get apiservice | grep custom.metrics

# проверка, что метрика отдается в k8s API
kubectl get --raw "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/pods/*/http_requests_total"

# убедись, что в кластере действительно применен актуальный HPA
kubectl apply -f Task2/02_scaling.yaml
kubectl describe hpa insuretech-deployment -n default

# после запуска мониторинга можно открыть UI
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090
```
