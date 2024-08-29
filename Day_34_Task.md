# Day 34:

### In this task, I have installed kubeadm using 2 EC2 instances.
### For control plane, I have used t2.medium and for data plane I have used t2.micro.
### The commands, I ran on control plane are as below:

```
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get install -y kubelet kubeadm kubectl

sudo apt-get install -y kubelet

sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo systemctl enable --now kubelet

kubectl get nodes

kubeadm init

sudo kubeadm init --pod-network-cidr 192.168.0.0/16 --kubernetes-version 1.27.11

sudo apt-get install -y kubelet=1.27.11-00 kubeadm=1.27.11-00 kubectl=1.27.11-00

sudo apt-mark hold kubelet kubeadm kubectl

sudo kubeadm init --pod-network-cidr 192.168.0.0/16

sudo apt-get install -y containerd

sudo systemctl enable containerd --now

sudo sysctl -w net.ipv4.ip_forward=1

echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf

sudo sysctl -p

sudo kubeadm init --pod-network-cidr 192.168.0.0/16

kubectl get nodes

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

kubectl get nodes

```

### The commands, I ran on data plane are as below:
```
sudo apt-get update

sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.31/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.31/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    
sudo apt-get update

sudo apt-get install -y kubelet kubeadm kubectl

sudo systemctl enable --now kubelet
    
kubectl get nodes

kubeadm join 172.31.16.219:6443 --token 5eo92k.2j079qbms0wwvw2n 	--discovery-token-ca-cert-hash sha256:fbae27c7df64e11ecd49d0b5980128e6aec48ab5ebedd4f5ffe2ac371010cae0

sudo kubeadm join 172.31.16.219:6443 --token 5eo92k.2j079qbms0wwvw2n 	--discovery-token-ca-cert-hash sha256:fbae27c7df64e11ecd49d0b5980128e6aec48ab5ebedd4f5ffe2ac371010cae0

sudo apt-get update

sudo apt-get install -y containerd

sudo systemctl enable containerd --now

sudo sysctl -w net.ipv4.ip_forward=1

echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf

sudo sysctl -p

sudo kubeadm join 172.31.16.219:6443 --token 5eo92k.2j079qbms0wwvw2n     --discovery-token-ca-cert-hash sha256:fbae27c7df64e11ecd49d0b5980128e6aec48ab5ebedd4f5ffe2ac371010cae0

ping 172.31.16.219

sudo kubeadm join 172.31.16.219:6443 --token 5eo92k.2j079qbms0wwvw2n     --discovery-token-ca-cert-hash 

sha256:fbae27c7df64e11ecd49d0b5980128e6aec48ab5ebedd4f5ffe2ac371010cae0
```
### The successful creation of kubeadm cluster can be seen in the below images:

![alt text](images/Day_34_Images/Image_2)

![alt text](images/Day_34_Images/Image_1)

# Grafana Installation
### Below are the commands, I used to install grafana:

```
sudo apt-get update

sudo apt-get install -y software-properties-common

sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"

wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

sudo apt-get update

sudo apt-get install grafana

sudo systemctl start grafana-server

sudo systemctl start grafana

sudo systemctl start grafana-server

sudo systemctl status grafana-server.service

sudo systemctl start grafana-server.service

sudo systemctl enable grafana-server.service
```

### Grafana was successfully installed as can be seen in the below image:

![alt text](images/Day_34_Images/Image_3)