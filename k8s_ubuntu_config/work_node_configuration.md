## OS Preparation
### Ubuntu  
#### check port availability  
optional: if netstat is not available, install the net-tools package
```bash
sudo apt update && sudo apt install -y net-tools
```
check the required ports are available  
> the command should produce no output, which means that the ports are not being used
```bash
netstat -tulpn | grep "10250\|10256"
```
alternatively using netcap, the output should be similar to below  
> *nc: connect to 127.0.0.1 port 10250 (tcp) failed: Connection refused*
```bash
nc 127.0.0.1 10250 -zv -w 2
nc 127.0.0.1 10256 -zv -w 2
```
#### configure firewall rule
update && upgrade system
```bash
sudo apt update && sudo apt -y full-upgrade
```
check the ufw status
```bash
systemctl status ufw
```
if the ufw.service is disabled, enable it with the command
```bash
sudo ufw enable
```
allow the required ports coming through the firewall
```bash
sudo ufw allow 10250,10256/tcp
```
#### enable time synchronization
if timedatectl is not install, install the package systemd-timesyncd
```bash
sudo apt update && sudo apt install -y systemd-timesyncd
```
enable time synchronizing
```bash
sudo timedatectl set-ntp true
sudo timedatectl set-timezone America/Toronto
```
> assume the timezone is Toronto
verify the ntp status
```bash
timedatectl status
```
> output should be similar to below.
```
               Local time: Wed 2025-04-16 19:22:48 EDT
           Universal time: Wed 2025-04-16 23:22:48 UTC
                 RTC time: Wed 2025-04-16 23:22:48
                Time zone: America/Toronto (EDT, -0400)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```
#### turn off swap
```bash
sudo swapoff -a
sudo sed -i.bak 's/^\/swap/#\/swap/' /etch/fstab
```
verify the swap is off
```bash
free -m
sudo cat /etc/fstab | grep swap
```
> output 
```
# swap should be all zero
               total        used        free      shared  buff/cache   available
Mem:            7941         588        3708           1        3952        7353
Swap:              0           0           0

# swap should be comment
#/swap.img      none    swap    sw      0       0
```
#### configure kernel modules
create k8s configuration file
```bash
sudo vim /etc/kubernetes/k8s.conf
```
copy & paste the following contet
```
overlay
br_netfilter
```
load the modules
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```
verify the modules loaded
```bash
lsmod | grep "overlay\|br_netfilter"
```
> output should be similar to below
```
br_netfilter           32768  0
bridge                421888  1 br_netfilter
overlay               212992  0
```
#### configure network parameter
create k8s.conf file
```bash
sudo vim /etc/sysctl.d/k8s.conf
```
copy & paste the following content
```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```
apply the changes
```bash
sudo sysctl --system
```
#### install prerequisite software
```bash
sudo apt update && sudo apt install -y apt-transport-https ca-certificates curl gpg gnupg2 software-properties-common
```
#### install container runtime, containerd
download gpg signature
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
enable read access of the docker.gpg
```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
add repository
```bash
echo "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable"
```



