## ДЗ#14
в YC  по непонятным причинам обрывалась связь с kubeapi, во время дебага баланс ушел в большой минус
поэтому этот вариант запуска кластера в Virtualbox, для этого заранее создадим в настройках Virtualbox HostedNetwork без DHCP, где виртуальные машины моглибы общаться друг с другом, а также отдельный интерфейс на каждой машине со статическим адресом.

На всех нодах добавим репозитории и ключи для компонентов kubernetes
```console
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

На всех нодах sudo apt-get install kubeadm=1.30.5-1.1
```console
sudo apt-get update
sudo apt-get install kubelet
sudo apt-get install kubeadm=1.30.5-1.1
sudo apt-get install kubectl=1.30.5-1.1
```
Чтобы версии не разъехались
```console
sudo apt-mark hold kubeadm kubelet kubectl
```

Отключим своп, включим форвардинг
```console
sudo swapoff -a
sudo modprobe overlay
sudo modprobe br_netfilter
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
```

Установим докер
```console
sudo apt install -y docker.io
```


Изменим драйвер docker cgroup после установки, чтобы не словить ошибку «Docker group driver found cgroupfs instead systemd»
```console
sudo cat <<EOF | sudo tee /etc/docker/daemon.json
{ "exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts":
{ "max-size": "100m" },
"storage-driver": "overlay2"
}
EOF

sudo systemctl restart docker
```


Произведем инициализацию кластера на мастере
```console
sudo kubeadm init --apiserver-advertise-address 10.244.0.1 --control-plane-endpoint 10.244.0.1
```

На воркерах выполняем джоин
```console
kubeadm join 10.244.0.1:6443 --token 99ydyf.zn5nl5ublrk0veql \
	--discovery-token-ca-cert-hash sha256:a4a247eb1d0879b4dd2cdfefa90abb5fe5749a47ea5126c634360f5466c1c3f6 
```

Проверим что все заджойнились
```console
kubectl get nodes
NAME        STATUS     ROLES           AGE     VERSION
u1-master   NotReady   control-plane   4m23s   v1.30.5
u2-worker   NotReady   <none>          18s     v1.30.5
u3-worker   NotReady   <none>          7s      v1.30.5
u4-worker   NotReady   <none>          4s      v1.30.5
```

Установим flannel и проверим состояние узлов ещё раз 
```console
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml

kubectl get nodes
NAME        STATUS   ROLES           AGE     VERSION
u1-master   Ready    control-plane   12m     v1.30.5
u2-worker   Ready    <none>          8m12s   v1.30.5
u3-worker   Ready    <none>          8m1s    v1.30.5
u4-worker   Ready    <none>          7m58s   v1.30.5
```
