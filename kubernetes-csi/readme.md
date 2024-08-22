## ДЗ#12

создадим бакет
```console
yc storage bucket create --name my-k8s-bucket
```
создадим IAM Service Account
```console
yc iam service-account create --name s3-access-sa
```
создадим необходимые права
```console
yc iam role create \
  --role-name storage.editor \
  --service-account-id aje16hj9qttmkferl4i5
```
  назначим необходимые права ( через веб зададим роль которая сможет создавать каталоги, например admin)

  создадим статический ключ доступа 
  ```console
  yc iam access-key create --service-account-id aje16hj9qttmkferl4i5
```

  создадим и применим yaml для хранения ключей доступа 
  ```console
  kubectl apply -f secret.yaml
```
  StorageClass для хранения ключей
```console
  kubectl apply -f storageclass.yaml
```
  установим драйвер из репозитория
```console
  helm repo add yandex-s3 https://yandex-cloud.github.io/k8s-csi-s3/charts

helm install csi-s3 yandex-s3/csi-s3
```
создадим и применим PersistentVolumeClaim 
```console
kubectl apply -f pvc.yaml
```
создадим под который будет использовать этот pvc
```console
kubectl apply -f pod.yaml
```
запишем файл из пода
```console
kubectl exec -it csi-s3-test-nginx -- /bin/sh
echo "Hello Yandex Cloud!" > /usr/share/nginx/html/s3/dsdfsd.txt 
```