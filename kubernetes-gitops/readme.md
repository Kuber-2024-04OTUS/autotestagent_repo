## ДЗ #10

создадим argocd-values.yaml

выполним  
```console
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd -n argocd --create-namespace -f argocd-values.yaml
```

создадим лейбл на инфра ноде
```console
kubectl label nodes cl1t1ck4o3s3n9km7bq7-eluv node-role.kubernetes.io/infra=""
```
создадим проект и приложение
otus-project.yaml
kubernetes-networks-app.yaml

добавим селектор толерейшн в ямлы в репозитории kubernetes-networks

локально установим argocd
```console
 curl -sSL -o argocd-darwin-amd64 https://github.com/argoproj/argo-cd/releases/download/v2.12.1/argocd-darwin-amd64
 chmod +x argocd-darwin-amd64
 sudo mv argocd-darwin-amd64 /usr/local/bin/argocd
 argocd login
 argocd app create kubernetes-networks-app \
  --repo https://github.com/Kuber-2024-04OTUS/autotestagent_repo \
  --path kubernetes-networks \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace homework \
  --revision kubernetes-networks --upsert
application 'kubernetes-networks-app' updated
```

применим обновленные манифесты к  кластеру 
```console
argocd app sync kubernetes-networks-app
```
создадим helm приложение 
kubernetes-templating-app.yaml

применим изменения
```console
kubectl apply -f kubernetes-templating-app.yaml
```
проверим его состояние
```console
argocd app get kubernetes-templating-app
```