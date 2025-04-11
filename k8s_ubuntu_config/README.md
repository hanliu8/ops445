# Configure k8s
## System Preparation
1. check the port availability  
- on control node
```bash
sudo netstat -tulpn | grep "6334\|2379\|2380\|10250\|10257\|10259"
```  
- on worker node
```bash
sudo netstat -tulpn | grep 10250
```
- 2379/2380  - etcd API
2. 
