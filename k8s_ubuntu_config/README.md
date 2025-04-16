# Configure k8s
## System Preparation (Ubuntu)
1. check the port availability  
- on control node / control plane
```bash
sudo netstat -tulpn | grep "6443\|2379\|2380\|10250\|10257\|10259"
```  
- on worker node
```bash
sudo netstat -tulpn | grep 10250
```
> The output should be empty which means that ports are not being used by any processes
> notes
- 2379/2380  - etcd API
- 6443 - Kubernetes API server
- 10250 - kubelet API
- 10257 - kube-controller-manager
- 10259 - kube-scheduler
- 30000 - 32767 - NodePort range

![Control Plane Ports](https://github.com/user-attachments/assets/87175153-8cb9-4773-ab2c-dcd140ef3198)
![Workder Node Ports](https://github.com/user-attachments/assets/a710e319-ff95-4215-a5f3-9350af3e93a3)



>> if the netstat is not installed, it is part of the net-tools package
```bash
sudo apt update && sudo apt install -y net-tools
```  
  
2. configure ufw or disable ufw (disabling ufw should be done in pre-production environment only)
check if the ufw is active
```bash
systemctl status ufw
sudo ufw status
```
disable ufw
```bash
sudo ufw disable
```
enable ufw
```bash
sudo ufw enable
```
install ufw
```bash
sudo apt update
sudo apt install -y ufw
```
> allow specific ports through ufw
```bash
sudo ufw allow 6443
```

3. update & upgrade system
```bash
sudo apt update && sudo apt -y full-upgrade
```

4. enable time sync
```bash
sudo apt install -y systemd-timesyncd
sudo timedatectl set-ntp true
sudo timedatectl status
```
> output similar to below
NTP Service: active
  
5. turn off swap
```bash
sudo swapoff -a
sudo sed -i.bak 's/^\/swap*/#\/swap/' /etc/fstab
```
verify the changes
```bash
free -m
cat /etc/fstab | grep swap
```

6. configure kernel modules
create '/etc/modules-load.d/k8s.conf` file and add following lines
```
sudo vim /etc/modules-load.d/k8s.connf
```
```
overlay
br_netfilter
```
load the module
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```
verify the modules loaded
```
lsmod | grep "overlay\|br_netfilter"
```

8. configure network parameters
create `/etc/sysctl.d/k8s.conf' file
```bash
sudo vim /etc/sysctl.d/k8s.conf
```
add lines
```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_foward = 1
```
apply the changes
```bash
sudo sysctl --system
```

9. install software tools
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg gnupg2 software-properties-common
```

## Install Kubernetes Tools
1. add Kubernetes repository and keys
create `/etc/apt/keyrings` director if it does not exist
```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
```
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list
```
2. install packages
```bash
sudo apt-get update
sudo apt-get install -y kubectl kubelet kubeadm
sudo apt-mark hold kubelet kubeadm kubectl  # depends on if you have container runtime installed
```

## Install container runtime
> containerd is used
add containerd repository
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
enable read access of the gpg file
```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
sudo ls -al /etc/apt/keyrings
```
```bash
echo "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | sudo tee /etc/apt/sources.list.d/docker.list

sudo cat /etc/apt/sources.list.d/docker.list
```

install containerd
```bash
sudo apt update
sudo apt install -y containerd.io
```
configure containerd
```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
verify the configuration
```bash
sudo cat /etc/containerd/config.toml
```
> Output must contain the following
```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]  
  runtime_type = "io.containerd.runc.v2"  
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]  
     SystemdCgroup = true
```
restart & enable the containerd service
```bash
sudo systemctl status containerd.service
sudo systemctl restart containerd.service
sudo systemctl enable containerd.service
```
install crictl if not installed & configure the configuration
```bash
sudo crictl ps
sudo apt install crictl
sudo vim /etc/crictl.yaml
```
copy & paste the following content
```
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
debug: true # <- if you don't want to see debug info you can set this to false
pull-image-on-create: false
```
pull images
```bash
sudo kubeadm config images pull --cri-socket unix:///var/run/containerd/containerd.sock
```
initialize the control plane
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0./16 --cri-socket unix://var/run/containerd/containerd.sock
```
> you will see the output similar to below
```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.146.220:6443 --token gpkwa8.shu07vqma0i39ppd \
        --discovery-token-ca-cert-hash sha256:fdaae780b5426ffd84d796f2532b6c258bb6ad2a515b6b3cd5b8af8b530bf598
```
configure environment for running kubectl
```bash
kubectl get nodes

mkdir ~/.kube
sudo cp -i /etc/kubernetes/admin.conf ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config

kubectl get nodes

sudo crictl ps
```
install CNI plugin, weave net
```bash
kubectl apply -f https://reweave.azurewebsites.net/k8s/v1.32/net.yaml
```
- replace v1.32 with the installed kubernetes version







  





