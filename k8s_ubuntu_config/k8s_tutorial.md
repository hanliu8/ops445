## ubuntu 24.0 Kubernetes
Verify the MAC address & product_uuid of the hosts that will participate in k8s cluster
```bash
ip address
sudo cat /sys/class/dmi/id/product_uuid
```
> Notes: make sure the output from each host unqiue

Verify TCP ports required by Kubernetes cluster not used by other processes
```bash
nc 127.0.0.1 6443 -zv -w 2
```
![image](https://github.com/user-attachments/assets/b9001aec-8af3-46b3-82d0-fcbd1d854ee0)



