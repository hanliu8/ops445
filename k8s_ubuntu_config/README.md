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
overlay
br_netfilter
```

8. 

9. 
