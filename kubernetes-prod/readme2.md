## ДЗ#14
в YC  по непонятным причинам обрывалась связь между kubeapi и etcd, ноды не джойнились, а баланс уходил в 
поэтому этот вариант запуска кластера в Virtualbox

на всех нодах

sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

на всех нодах sudo apt-get install kubeadm=1.30.5-1.1

sudo apt-get update
sudo apt-get install kubelet
sudo apt-get install kubeadm=1.30.5-1.1
sudo apt-get install kubectl=1.30.5-1.1

чтобы версии не разъехались
sudo apt-mark hold kubeadm kubelet kubectl


sudo swapoff -a
sudo modprobe overlay
sudo modprobe br_netfilter
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system


sudo apt install -y docker.io



Changing Docker Cgroup Driver

It is crucial to change docker cgroup Driver after install or you get error regarding “Docker group driver detected cgroupfs instead systemd”

sudo cat <<EOF | sudo tee /etc/docker/daemon.json
{ "exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts":
{ "max-size": "100m" },
"storage-driver": "overlay2"
}
EOF


sudo systemctl restart docker



на мастере инит

sudo kubeadm init --apiserver-advertise-address 10.244.0.1 --control-plane-endpoint 10.244.0.1


на воркерах выполняем джоин
kubeadm join 10.244.0.1:6443 --token 99ydyf.zn5nl5ublrk0veql \
	--discovery-token-ca-cert-hash sha256:a4a247eb1d0879b4dd2cdfefa90abb5fe5749a47ea5126c634360f5466c1c3f6 



kubectl get nodes
NAME        STATUS     ROLES           AGE     VERSION
u1-master   NotReady   control-plane   4m23s   v1.30.5
u2-worker   NotReady   <none>          18s     v1.30.5
u3-worker   NotReady   <none>          7s      v1.30.5
u4-worker   NotReady   <none>          4s      v1.30.5

kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml


kubectl get nodes
NAME        STATUS   ROLES           AGE     VERSION
u1-master   Ready    control-plane   12m     v1.30.5
u2-worker   Ready    <none>          8m12s   v1.30.5
u3-worker   Ready    <none>          8m1s    v1.30.5
u4-worker   Ready    <none>          7m58s   v1.30.5