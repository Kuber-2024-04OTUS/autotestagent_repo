
## ДЗ #8

Создадим Dockerfile для кастомного nginx с экспортером и nginx.conf
В линукс машине соберем кастомный образ и запушим его в репозиторий
```console
sudo docker build -t custom-nginx .
sudo docker push autotestagent/containerimages:latest
```

 создадим и применим nginx-deployment.yaml

```console
 kubectl apply -f nginx-deployment.yaml
```
установим CRD для Prometheus Operator через helm
```console
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus-operator prometheus-community/kube-prometheus-stack
kubectl get crds | grep servicemonitor
```

создадим ServiceMonitor nginx-servicemonitor.yaml
```console
kubectl apply -f nginx-servicemonitor.yaml
```