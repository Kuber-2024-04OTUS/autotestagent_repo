
## ДЗ #6 
Установим helm CLI и helmfile.

Создадим структуру helm chart'a
```console
helm create myapp
```
отредактируем values.yaml
отредактируем deployment.yaml
отредактируем service.yaml
отредактируем ingress.yaml
подключим зависимость в Chart.yaml
создадим NOTES.txt

проверим последнюю версию mysql с помощью команды
```console
helm search repo bitnami/mysql --versions
```

отредактируем hpa.yaml


для второй части задания создадим требуемые файлы в каталоге myapp2 и проверим что всё применяется

```console
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
kubectl create namespace prod
helm install kafka bitnami/kafka --namespace prod -f values-prod.yaml
helm plugin install https://github.com/databus23/helm-diff
export INTER_BROKER_PASSWORD=$(kubectl get secret --namespace "prod" kafka-user-passwords -o jsonpath="{.data.inter-broker-password}" | base64 -d)
helm upgrade kafka bitnami/kafka \
  --namespace prod \
  --version 26.0.0 \
  --set sasl.interbroker.password=$INTER_BROKER_PASSWORD \
  -f values-prod.yaml


```