
## ДЗ #7

создадим crd.yaml
отредактируем serviceaccount.yaml
отредактируем clusterrole.yaml
отредактируем clusterrolebinding.yaml
отредактируем deployment.yaml
отредактируем mysql-custom-resource.yaml


проверим
```console
kubectl apply -f serviceaccount.yaml
kubectl apply -f clusterrole.yaml
kubectl apply -f clusterrolebinding.yaml
kubectl apply -f deployment.yaml
kubectl apply -f crd.yaml
kubectl apply -f mysql-custom-resource.yaml
kubectl get pods
```

