
##  service account monitoring и доступ к эндпоинту /metrics 
создадим yaml файлы для аккаунта monitoring с биндингом к роли admin
и применим изменения
```console
kubectl apply -f monitoring-serviceaccount.yaml
kubectl apply -f monitoring-rolebinding.yaml
```

внесем изменения в deployment.yaml
```yaml
    spec:
      serviceAccountName: monitoring  
      containers:
```

проверим что новые поды деплоятся с Service Account monitoring

```console
kubectl describe pod homework-deployment-b5ddfb455-ldqk6 -n homework

Name:             homework-deployment-b5ddfb455-ldqk6
Namespace:        homework
Priority:         0
Service Account:  monitoring
```

##создание Service Account cd с ролью admin  

```console
kubectl apply -f cd-serviceaccount.yaml
kubectl apply -f cd-rolebinding.yaml   
```
сгенерируем токен

```console
kubectl create token cd --duration=24h -n homework > token
TOKEN=$(cat token)
```

получим путь к сертификату УЦ
```console
CA_PATH=$(kubectl config view --raw --minify -o jsonpath='{.clusters[0].cluster.certificate-authority}')
```

создадим Kubeconfig
```console
CLUSTER_NAME=$(kubectl config view --minify -o jsonpath='{.clusters[0].name}')
CLUSTER_SERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
CLUSTER_CA=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.certificate-authority-data}')
cat <<EOF > cd-kubeconfig
apiVersion: v1
kind: Config
clusters:
- name: $CLUSTER_NAME
  cluster:
    certificate-authority-data: $CLUSTER_CA
    server: $CLUSTER_SERVER
contexts:
- name: cd-context
  context:
    cluster: $CLUSTER_NAME
    namespace: homework
    user: cd
current-context: cd-context
users:
- name: cd
  user:
    token: $TOKEN
EOF
```


проверим что мы можем получить список подов с этим kubeconfig

```console
kubectl --kubeconfig=cd-kubeconfig get pods -n homework
NAME                                  READY   STATUS    RESTARTS   AGE
homework-deployment-b5ddfb455-ldqk6   0/1     Running   0          73m
homework-deployment-b5ddfb455-nrg5l   0/1     Running   0          73m
homework-deployment-b5ddfb455-pn7r5   0/1     Running   0          73m
```


если изменить токен на некорректный, то вывод будет
```console
error: You must be logged in to the server (Unauthorized)
```


поменять команду для init сонтейнера
```yaml
        command: ['sh', '-c', 'echo "<h1>Hello, Kubernetes!</h1>" > /usr/share/nginx/html/index.html && wget -qO- http://homework.otus/metrics &> /usr/share/nginx/html/metrics.html']
```

```console
kubectl apply -f deployment.yaml  
```
перейти в один из контейнеров пода и убедиться что metrics.html возвращается
```console
kubectl exec -n homework homework-deployment-6f496bbbfb-8crf2  -it -- /bin/bash
Defaulted container "web-server" out of: web-server, init-container (init)
root@homework-deployment-6f496bbbfb-8crf2:/# curl localhost/metrics.html
wget: bad address 'homework.otus'
```