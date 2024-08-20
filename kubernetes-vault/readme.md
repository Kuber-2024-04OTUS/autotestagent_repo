## ДЗ #11

докинем ещё воркер ноду в кластер
```console
kubectl create namespace consul
helm repo add hashicorp https://hashicorp-releases.yandexcloud.net
helm repo update
helm search repo hashicorp

 helm install consul bitnami/consul \  
  --namespace consul \
  --set global.name=consul \
  --set server.replicas=3
```
создадим consul-values.yaml

установим vault
```console
kubernetes-vault % helm install vault bitnami/vault \  
  --namespace vault \
  --set "server.ha.enabled=true" \
  --set "server.ha.raft.enabled=false" \
  --set "server.ha.replicas=3" \
  --set "server.ha.consul.address=consul.consul.svc.cluster.local:8500" \ 
  --set "server.ha.consul.path=vault/"
```

  инициализируем vault 
  ```console
  kubectl exec -it vault-server-0 -n vault -- vault operator init
```

(под vault получился один, возможно надо было брать другую инструкцию https://developer.hashicorp.com/vault/tutorials/raft/raft-storage?productSlug=vault&tutorialSlug=day-one-consul&tutorialSlug=ha-with-consul 
для начала подойдет конфигурация с тремя консулами)
 
  распечатаем три ключа

```console
kubectl exec -it vault-server-0 -n vault -- vault operator unseal ********************************************
kubectl exec -it vault-server-0 -n vault -- vault operator unseal ********************************************
kubectl exec -it vault-server-0 -n vault -- vault operator unseal ********************************************
```
аутентифицируемся с помощью initial root token, который был выдан на этапе инициализации:
```console
kubectl exec -it vault-server-0 -n vault -- vault login  ****************************
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.
```
настроим секретное хранилище
```console
kubectl exec -it vault-server-0 -n vault -- vault secrets enable -path=otus kv
```
создадим секрет
```console
kubectl exec -it vault-server-0 -n vault -- vault kv put otus/cred username='otus' password='asajkjkahs'
```
в namespace vault создадим serviceAccount с именем vault-auth и
ClusterRoleBinding для него с ролью system:auth-delegator
```console
kubectl apply -f serviceaccount.yaml
kubectl apply -f clusterrolebinding.yaml

kubectl exec -it vault-server-0 -n vault -- vault auth enable kubernetes
```
```console
kubectl exec -it vault-server-0 -n vault -- vault policy write otus-policy - <<EOF
path "otus/cred" {
  capabilities = ["read", "list"]
}
EOF
 ```
 создадим роль в Vault с использованием ServiceAccount vault-auth из namespace vault и политики otus-policy
```console
 kubectl exec -it vault-server-0 -n vault -- vault write auth/kubernetes/role/otus \
    bound_service_account_names=vault-auth \
    bound_service_account_namespaces=vault \
    policies=otus-policy \
    ttl=24h
```
установим Helm-чарт External Secrets Operator в namespace vault
```console
helm repo add external-secrets https://charts.external-secrets.io
helm install external-secrets external-secrets/external-secrets \
--namespace vault
```
созадим манифест для SecretStore, который будет использовать ранее созданную роль otus и сервис-аккаунт vault-auth
```console
kubectl apply -f secretstore.yaml
```
создадим externalsecret.yaml

проверим создание секрета
```console
kubectl get secret otus-cred -n vault -o yaml
```