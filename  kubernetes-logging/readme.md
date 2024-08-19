## ДЗ #9

Создадим в яндекс облаке две группы узлов, одну для воркеров, другую для инфраструктурных сервисов, по одному узлу в каждой.


```console
curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
brew install zsh-completion
#now in new terminal
yc managed-kubernetes cluster get-credentials otus-homework-cluster1 --external
kubectl config get-contexts

kubectl get nodes -l pool-name=infra-pool 
#команда выше почему-то не сработала несмотря на метку созданную в веб управлялке яндекса
kubectl taint nodes cl1t1ck4o3s3n9km7bq7-eluv node-role=infra:NoSchedule
#получим информацию о нодах
johnappleseed@WD-1011 kubernetes-templating % kubectl get node -o wide 
NAME                        STATUS   ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP     OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
cl1b22c97mtjtljf6lg8-icyr   Ready    <none>   29m   v1.30.1   172.16.0.31   51.250.78.131   Ubuntu 20.04.6 LTS   5.4.0-187-generic   containerd://1.6.28
cl1t1ck4o3s3n9km7bq7-eluv   Ready    <none>   28m   v1.30.1   172.16.0.33   62.84.124.96    Ubuntu 20.04.6 LTS   5.4.0-187-generic   containerd://1.6.28
johnappleseed@WD-1011 kubernetes-templating % kubectl get node --show-labels
NAME                        STATUS   ROLES    AGE   VERSION   LABELS
cl1b22c97mtjtljf6lg8-icyr   Ready    <none>   29m   v1.30.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=standard-v3,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/zone=ru-central1-a,kubernetes.io/arch=amd64,kubernetes.io/hostname=cl1b22c97mtjtljf6lg8-icyr,kubernetes.io/os=linux,node.kubernetes.io/instance-type=standard-v3,node.kubernetes.io/kube-proxy-ds-ready=true,node.kubernetes.io/masq-agent-ds-ready=true,node.kubernetes.io/node-problem-detector-ds-ready=true,topology.kubernetes.io/zone=ru-central1-a,yandex.cloud/node-group-id=cat7hvc1l66s0m1052u8,yandex.cloud/pci-topology=k8s,yandex.cloud/preemptible=false
cl1t1ck4o3s3n9km7bq7-eluv   Ready    <none>   28m   v1.30.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/instance-type=standard-v3,beta.kubernetes.io/os=linux,failure-domain.beta.kubernetes.io/zone=ru-central1-a,kubernetes.io/arch=amd64,kubernetes.io/hostname=cl1t1ck4o3s3n9km7bq7-eluv,kubernetes.io/os=linux,node.kubernetes.io/instance-type=standard-v3,node.kubernetes.io/kube-proxy-ds-ready=true,node.kubernetes.io/masq-agent-ds-ready=true,node.kubernetes.io/node-problem-detector-ds-ready=true,topology.kubernetes.io/zone=ru-central1-a,yandex.cloud/node-group-id=catnifi8ik4q58efkcb8,yandex.cloud/pci-topology=k8s,yandex.cloud/preemptible=false
johnappleseed@WD-1011 kubernetes-templating % kubectl get nodes -o custom-columns=NAME:.metadata.name,TAINTS:.spec.taints
NAME                        TAINTS
cl1b22c97mtjtljf6lg8-icyr   <none>
cl1t1ck4o3s3n9km7bq7-eluv   [map[effect:NoSchedule key:node-role value:infra]]
#создадим бакет в интерфейсе клауда
#создание сервисного акаунта

yc resource-manager folder list
yc iam service-account create --name otus-homework-sa --folder-id   b1g55e8c936vv7hnp0ff
done (1s)
id: aje2chfr8uv5f4hd475i
folder_id: b1g55e8c936vv7hnp0ff
created_at: "2024-08-18T18:52:06.198726328Z"
name: otus-homework-sa

yc resource-manager folder add-access-binding \
  --id b1g55e8c936vv7hnp0ff \
  --role storage.editor \
  --subject serviceAccount:aje2chfr8uv5f4hd475i

yc iam key create --service-account-id aje2chfr8uv5f4hd475i --output key.json     

#начнем устанавливать loki


helm repo add grafana https://grafana.github.io/helm-charts

helm repo update

```


Создадим конфигурационный файл values.yaml 

```console

kubectl create namespace monitoring
helm install loki grafana/loki-stack --namespace monitoring -f values.yaml
NAME: loki
LAST DEPLOYED: Sun Aug 18 22:39:13 2024
NAMESPACE: monitoring
STATUS: deployed
REVISION: 1
NOTES:
The Loki stack has been deployed to your cluster. Loki can now be added as a datasource in Grafana.

See http://docs.grafana.org/features/datasources/loki/ for more detail.

kubectl port-forward service/loki 30000:3100 \
        --namespace monitoring

```

дополнительно может понадобиться дать FULL_CONTROL ACL для otus-homework-sa в otus-homework-bucket
создадим promtail-values.yaml
создадим grafana-values.yaml

установим promtail и grafana 
```console

helm install promtail grafana/promtail --namespace monitoring -f promtail-values.yaml
helm install grafana grafana/grafana --namespace monitoring -f values-grafana.yaml
kubectl --namespace monitoring port-forward $POD_NAME 3000       

```
создадим loki-service.yaml


скачаем более старую графану
```console
helm pull grafana/grafana --version 6.28.0 --untar
rm ./grafana/templates/tests/test-podsecuritypolicy.yaml
rm ./grafana/templates/podsecuritypolicy.yaml
helm upgrade grafana ./grafana --namespace monitoring -f grafana-values.yaml
```

настроим promtail для отправки логов
отредактируем promtail-configmap.yaml
создадим promtail-deployment.yaml

проверим что логи отправляются в datasource loki и сохраним скриншот