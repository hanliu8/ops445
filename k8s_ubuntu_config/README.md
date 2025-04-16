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
```

