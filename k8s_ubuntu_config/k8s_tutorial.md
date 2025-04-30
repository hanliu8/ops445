## ubuntu 24.0 Kubernetes
#### Verify the MAC address & product_uuid of the hosts that will participate in k8s cluster
```bash
ip address
sudo cat /sys/class/dmi/id/product_uuid
```
> Notes: make sure the output from each host unqiue

![image](https://github.com/user-attachments/assets/31330975-f457-4cac-be7b-49b2550ee4ed)



#### Verify TCP ports required by Kubernetes cluster not used by other processes
```bash
nc 127.0.0.1 2379 -zv -w 2
nc 127.0.0.1 2380 -zv -w 2
nc 127.0.0.1 6443 -zv -w 2
nc 127.0.0.1 10250 -zv -w 2
nc 127.0.0.1 10257 -zv -w 2
nc 127.0.0.1 10259 -zv -w 2

nc 127.0.0.1 10256 -zv -w 2
```
![image](https://github.com/user-attachments/assets/b23b5ee2-6d28-403f-bc83-322e965f8e68)

![image](https://github.com/user-attachments/assets/d348d2a4-a0be-4a24-ab20-92a3eb17e86f)


#### Disable the swap 
```bash
swapoff -a
sudo sed -i 's/^\/swap*/#\/swap/' /etc/fstab
```
#### Install a container runtime, containerd
```bash
curl -fsSL https://github.com/containerd/containerd/releases/download/v2.0.5/containerd-2.0.5-linux-amd64.tar.gz -o /tmp/containerd-2.0.5-linux-amd64.tar.gz






