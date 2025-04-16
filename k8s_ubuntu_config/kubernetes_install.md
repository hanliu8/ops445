## install kubernetes with curl

download specific version of kubectl  
> [View available versions](https://kubernetes.io/releases/)
```bash
curl -LO "https://dl.k8s.io/release/v1.30.9/bin/linux/amd64/kubectl"
```
For the latest version
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```
download checksum file
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```
validate the download file
```bash
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```
