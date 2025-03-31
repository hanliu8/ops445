# Lab 8: Ansible
## Objective
1. Set up a develop environment for Ansible
2. Explore and run Ansible's ad hoc commands
3. Explore and study a few Ansible's modules
4. Explore, create, and run a few Ansible playbooks

## Overview
Ansible is an agentless IT automation engine for automating cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT system administration tasks.

Ansible uses no additional custom security infrastructure, and it uses a very simple human readable language called 'YAML', to compose an Ansible Playbook which allows you to describe the tasks you want to automate.

## Reference
 - For more detail information about ansible, check out the ansible web site at www.ansible.com
 - Overview on how [ansible works](https://www.redhat.com/en/ansible-collaborative/how-ansible-works?intcmp=7015Y000003t7aWQAQ)
 - [Ansible Latest User Guide](https://docs.ansible.com/ansible/latest/user_guide/index.html)
 - [A System Administrator's guide to getting started with Ansible](https://www.redhat.com/en/blog/system-administrators-guide-getting-started-ansible-fast)
## System requirements
 - control machine Install Ansible on your Linux VM 
 - managed machine(s) - to be managed by the control machine

 - You should be able to ssh from your control machine as a regular user to your managed machine without supplying a login password.
 - Your account on your managed machine is a sudoer and can run sudo with/without password.
## Development Environment Setup
### Install Oracle VirtualBox

download the Oracle VirtualBox installation for Windows
```
curl https://download.virtualbox.org/virtualbox/7.1.6/VirtualBox-7.1.6-167084-Win.exe -o VirtualBox-7.1.6-167084-Win.exe
```
run the installation file from command line with administrative permission and follow the prompt to complete installation
```
VirtualBox-7.1.6-167084-Win.exe
```
verify your installation by launch the VirtualBox from Startup menu

### Install Vagrant 
download Vagrant installation package for Windows
```
curl https://releases.hashicorp.com/vagrant/2.4.3/vagrant_2.4.3_windows_amd64.msi -o vagrant_2.4.3_windows_amd64.msi
```
run the installation from command line with administrative permission and follow the prompt to complete installation (You may be required to restart your machine)
```
msiexec.exe /i "vagrant_2.4.3_windows_amd64.msi"
```
verify the installation from command prompt
```
vagrant --version
```
> Vagrant 2.4.3

### Provision VMs
create a project directory
```
mkdir -p C:\vagrant\ops445
```
change to the newly created directory and download the configuration files
```
cd C:\vagrant\ops445
curl https://raw.githubusercontent.com/hanliu8/ops445/refs/heads/main/Vagrantfile?token=GHSAT0AAAAAADBB3LJTTT4SUPAJA7UHTAOCZ7J2SVA -o Vagrantfile
curl https://raw.githubusercontent.com/hanliu8/ops445/refs/heads/main/common-dependencies.sh?token=GHSAT0AAAAAADBB3LJSUFUP4YZMXQLHTA46Z7J2TSQ -o common-dependencies.sh
vagrant up
```
once the vagrant completes provisioning VMs, launch the Oracle VirtualBox to verify 3 VMs up and running

### Configure Ansible
connect to control node
```
vagrant ssh control-center
ping -c 3 192.168.56.10
ping -c 3 192.168.56.11
```
once verified the VMs accessible, generate ssh rsa key pairs
**Answer: _'yes'_, and password: _'vagrant'_ all lower case** 
```
ssh-keygen -t rsa -b 4096
ssh-copyid vagrant@192.168.56.10
ssh-copyid vagrant@192.168.56.11
```
verify the ssh connection to the two VMs 
**Should not prompt for password**
```
ssh vagrant@192.168.56.10
ssh vagrant@192.168.56.10
```
verify Ansible installed
```
ansible --version
```
download the sample inventory file
```
curl https://raw.githubusercontent.com/hanliu8/ops445/refs/heads/main/hosts?token=GHSAT0AAAAAADBB3LJSQ3OEKR54L66SWI2KZ7J2O6A -o hosts
```

## Investigation 1: The Ansible Package
In this investigation, we explore the main components of the Ansible configuration management system and its operating environment. We also study a simple playbook for managing the configuration.

You have three Linux systems for this lab: one control machine and two VMs as the managed machine. 

### Important:
If you decide to use Matrix for this lab, please follow the steps [here](https://seneca-ictoer.github.io/OPS445/A-Labs/lab8)

### Key Concepts when using Ansible
 - YAML - a human-readable data serialization language used by Ansible's playbooks. To know more, your can check out the [wikipedia page here](https://en.wikipedia.org/wiki/YAML) or a simple introduction here

 - Control machine - the host on which you use Ansible to execute tasks on the managed machines

 - Managed machine - a host that is configured by the control machine

 - Hosts file - contains information about machines to be managed - click here for sample hosts file

 - Idempotency - is an operation that, if applied twice to any value, gives the same result as if it were applied once.

 - Ad hoc commands - a simple one-off task:

   - shell commands
```bash
ansible remote_machine_id [-i inventory] [--private-key id_rsa] [-u remote_user] -a 'date'
```
 - Ansible modules - code that performs a particular task such as copy a file, installing a package, etc:

   - copy module

```bash
ansible remote_machine_id -m copy -a "src=/ops445/ansible.txt dest=/tmp/ansible.txt"
```
   - Package management
```bash
ansible remote_machine_id -m yum -a "name=epel-release state=latest"
```
 - Playbooks - contains one or multiple plays, each play defines a set of repeatable tasks on one or more managed machines. Playbooks are written in YAML. Every play in the playbook is created with environment-specific parameters for the target machines:
```
ansible-playbook remote_machine_id [-i inventory] setup_webserver.yaml
ansible-playbook remote_machine_id [-i inventory] firstrun.yaml
```

### Part 1: The Ansible package installed on matrix
You only need to have the "ansible" package on your control VM.

To confirm that you have access to the Ansible package, try the following command:

```
$ ansible --help
usage: ansible [-h] [--version] [-v] [-b] [--become-method BECOME_METHOD]
               [--become-user BECOME_USER] [-K] [-i INVENTORY] [--list-hosts]
               [-l SUBSET] [-P POLL_INTERVAL] [-B SECONDS] [-o] [-t TREE] [-k]
               [--private-key PRIVATE_KEY_FILE] [-u REMOTE_USER]
               [-c CONNECTION] [-T TIMEOUT]
               [--ssh-common-args SSH_COMMON_ARGS]
               [--sftp-extra-args SFTP_EXTRA_ARGS]
               [--scp-extra-args SCP_EXTRA_ARGS]
               [--ssh-extra-args SSH_EXTRA_ARGS] [-C] [--syntax-check] [-D]
               [-e EXTRA_VARS] [--vault-id VAULT_IDS]
               [--ask-vault-pass | --vault-password-file VAULT_PASSWORD_FILES]
               [-f FORKS] [-M MODULE_PATH] [--playbook-dir BASEDIR]
               [-a MODULE_ARGS] [-m MODULE_NAME]
               pattern
```

Take a look of all the available command line options for the "ansible" command. There are a lots of options when running Ansible. Let's move on to try a few simple ones.

### Part 2: Sample runs for some of the Ad hoc commands
The following commands are based on the following entries in the ansible inventory file called "hosts" in the current working directory:

```
labvm1   ansible_host=192.168.56.10 ansible_port=22 ansible_ssh_private_key_file=~/.ssh/id_rsa

[webservers]
labvm2   ansible_host=192.168.56.11 ansible_port=22 ansible_ssh_private_key_file=~/.ssh/id_rsa
```
```
$ ansible labvm1 -i hosts --private-key ~/.ssh/id_rsa -m copy -a "src=/home/vagrant/hosts dest=/tmp/ansible_hosts"
labvm1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "changed": true,
    "checksum": "f800d7c90175d1b6f4a482575f748cb988c42d3d",
    "dest": "/tmp/ansible_hosts",
    "gid": 1000,
    "group": "vagrant",
    "md5sum": "2c6e21d11e3ac8050ff19bf11340c14e",
    "mode": "0664",
    "owner": "vagrant",
    "size": 210,
    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1743379689.9955702-30857-203528441420686/.source",
    "state": "file",
    "uid": 1000
}
```

**labvm1** is the remote machine ID.

**hosts** is the name of the ansible inventory file in the current working directory, you may also specify the inventory file with full path name, e.g. /home/raymond.chan/ops445/lab8/hosts.

**--private-key id_rsa** is the private key for ssh key-based authentication for connecting to the remote machine.

**-u** is for specifying the user account to be used to login to the remote machine.

**-m copy** is to tell ansible to use the "copy" module.

after **-a** is the arguments to the copy module, which specify the source file and the destination for the copy action.

If you got the same "SUCCESS" message, login to the remote machine and check the directory "/tmp" for the file ansible_hosts.

### Part 3: Sample runs for using some Ansible's modules
You can get a complete list of all the ansible modules installed on you system with the following command:

ansible-doc --list_files

**"apt"** is a stable ansible module. You can get the detail information about any ansible module with the ansible-doc, try the following commands to see the documentation and examples for using the copy and yum modules:

`ansible-doc copy`

ansible-doc apt

The following command demonstrates how to install the "epel-release" package with the "yum" module with different module arguments and under different remote user (your result may be differ from what is show below):

```
$ansible labvm1 -i hosts --private-key ~/.ssh/id_rsa -u vagrant -m apt -a "name=apache2 state=present"
[WARNING]: Platform linux on host labvm1 is using the discovered Python interpreter at
/usr/bin/python3.12, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
labvm1 | FAILED! => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "cache_update_time": 1743374086,
    "cache_updated": false,
    "changed": false,
    "msg": "'/usr/bin/apt-get -y -o \"Dpkg::Options::=--force-confdef\" -o \"Dpkg::Options::=--force-confold\"       install 'apache2=2.4.58-1ubuntu8.5'' failed: E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)\nE: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?\n",
    "rc": 100,
    "stderr": "E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)\nE: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?\n",
    "stderr_lines": [
        "E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)",
        "E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?"
    ],
    "stdout": "",
    "stdout_lines": []
}
```
**"E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)",** indicates permission denied because the package installation requires elevated permission.

Add the '-b' option to tell ansible to invoke "sudo" when running the apt command on the remote machine:

```
$  ansible labvm1 -i hosts --private-key ~/.ssh/id_rsa -u vagrant -b -m apt -a "name=apache2 state=present"
```
<details>
<summary>Output</summary>
[WARNING]: Platform linux on host labvm1 is using the discovered Python interpreter at
/usr/bin/python3.12, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
labvm1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "cache_update_time": 1743374086,
    "cache_updated": false,
    "changed": true,
    "stderr": "",
    "stderr_lines": [],
    "stdout": "Reading package lists...\nBuilding dependency tree...\nReading state information...\nThe following additional packages will be installed:\n  apache2-bin apache2-data apache2-utils libapr1t64 libaprutil1-dbd-sqlite3\n  libaprutil1-ldap libaprutil1t64 liblua5.4-0 ssl-cert\nSuggested packages:\n  apache2-doc apache2-suexec-pristine | apache2-suexec-custom www-browser\nThe following NEW packages will be installed:\n  apache2 apache2-bin apache2-data apache2-utils libapr1t64\n  libaprutil1-dbd-sqlite3 libaprutil1-ldap libaprutil1t64 liblua5.4-0 ssl-cert\n0 upgraded, 10 newly installed, 0 to remove and 120 not upgraded.\nNeed to get 2084 kB of archives.\nAfter this operation, 8094 kB of additional disk space will be used.\nGet:1 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 libapr1t64 amd64 1.7.2-3.1ubuntu0.1 [108 kB]\nGet:2 http://us.archive.ubuntu.com/ubuntu noble/main amd64 libaprutil1t64 amd64 1.6.3-1.1ubuntu7 [91.9 kB]\nGet:3 http://us.archive.ubuntu.com/ubuntu noble/main amd64 libaprutil1-dbd-sqlite3 amd64 1.6.3-1.1ubuntu7 [11.2 kB]\nGet:4 http://us.archive.ubuntu.com/ubuntu noble/main amd64 libaprutil1-ldap amd64 1.6.3-1.1ubuntu7 [9116 B]\nGet:5 http://us.archive.ubuntu.com/ubuntu noble/main amd64 liblua5.4-0 amd64 5.4.6-3build2 [166 kB]\nGet:6 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2-bin amd64 2.4.58-1ubuntu8.5 [1329 kB]\nGet:7 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2-data all 2.4.58-1ubuntu8.5 [163 kB]\nGet:8 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2-utils amd64 2.4.58-1ubuntu8.5 [97.1 kB]\nGet:9 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2 amd64 2.4.58-1ubuntu8.5 [90.2 kB]\nGet:10 http://us.archive.ubuntu.com/ubuntu noble/main amd64 ssl-cert all 1.1.2ubuntu1 [17.8 kB]\nPreconfiguring packages ...\nFetched 2084 kB in 1s (2086 kB/s)\nSelecting previously unselected package libapr1t64:amd64.\r\n(Reading database ... \r(Reading database ... 5%\r(Reading database ... 10%\r(Reading database ... 15%\r(Reading database ... 20%\r(Reading database ... 25%\r(Reading database ... 30%\r(Reading database ... 35%\r(Reading database ... 40%\r(Reading database ... 45%\r(Reading database ... 50%\r(Reading database ... 55%\r(Reading database ... 60%\r(Reading database ... 65%\r(Reading database ... 70%\r(Reading database ... 75%\r(Reading database ... 80%\r(Reading database ... 85%\r(Reading database ... 90%\r(Reading database ... 95%\r(Reading database ... 100%\r(Reading database ... 124309 files and directories currently installed.)\r\nPreparing to unpack .../0-libapr1t64_1.7.2-3.1ubuntu0.1_amd64.deb ...\r\nUnpacking libapr1t64:amd64 (1.7.2-3.1ubuntu0.1) ...\r\nSelecting previously unselected package libaprutil1t64:amd64.\r\nPreparing to unpack .../1-libaprutil1t64_1.6.3-1.1ubuntu7_amd64.deb ...\r\nUnpacking libaprutil1t64:amd64 (1.6.3-1.1ubuntu7) ...\r\nSelecting previously unselected package libaprutil1-dbd-sqlite3:amd64.\r\nPreparing to unpack .../2-libaprutil1-dbd-sqlite3_1.6.3-1.1ubuntu7_amd64.deb ...\r\nUnpacking libaprutil1-dbd-sqlite3:amd64 (1.6.3-1.1ubuntu7) ...\r\nSelecting previously unselected package libaprutil1-ldap:amd64.\r\nPreparing to unpack .../3-libaprutil1-ldap_1.6.3-1.1ubuntu7_amd64.deb ...\r\nUnpacking libaprutil1-ldap:amd64 (1.6.3-1.1ubuntu7) ...\r\nSelecting previously unselected package liblua5.4-0:amd64.\r\nPreparing to unpack .../4-liblua5.4-0_5.4.6-3build2_amd64.deb ...\r\nUnpacking liblua5.4-0:amd64 (5.4.6-3build2) ...\r\nSelecting previously unselected package apache2-bin.\r\nPreparing to unpack .../5-apache2-bin_2.4.58-1ubuntu8.5_amd64.deb ...\r\nUnpacking apache2-bin (2.4.58-1ubuntu8.5) ...\r\nSelecting previously unselected package apache2-data.\r\nPreparing to unpack .../6-apache2-data_2.4.58-1ubuntu8.5_all.deb ...\r\nUnpacking apache2-data (2.4.58-1ubuntu8.5) ...\r\nSelecting previously unselected package apache2-utils.\r\nPreparing to unpack .../7-apache2-utils_2.4.58-1ubuntu8.5_amd64.deb ...\r\nUnpacking apache2-utils (2.4.58-1ubuntu8.5) ...\r\nSelecting previously unselected package apache2.\r\nPreparing to unpack .../8-apache2_2.4.58-1ubuntu8.5_amd64.deb ...\r\nUnpacking apache2 (2.4.58-1ubuntu8.5) ...\r\nSelecting previously unselected package ssl-cert.\r\nPreparing to unpack .../9-ssl-cert_1.1.2ubuntu1_all.deb ...\r\nUnpacking ssl-cert (1.1.2ubuntu1) ...\r\nSetting up ssl-cert (1.1.2ubuntu1) ...\r\nCreated symlink /etc/systemd/system/multi-user.target.wants/ssl-cert.service → /usr/lib/systemd/system/ssl-cert.service.\r\r\nSetting up libapr1t64:amd64 (1.7.2-3.1ubuntu0.1) ...\r\nSetting up liblua5.4-0:amd64 (5.4.6-3build2) ...\r\nSetting up apache2-data (2.4.58-1ubuntu8.5) ...\r\nSetting up libaprutil1t64:amd64 (1.6.3-1.1ubuntu7) ...\r\nSetting up libaprutil1-ldap:amd64 (1.6.3-1.1ubuntu7) ...\r\nSetting up libaprutil1-dbd-sqlite3:amd64 (1.6.3-1.1ubuntu7) ...\r\nSetting up apache2-utils (2.4.58-1ubuntu8.5) ...\r\nSetting up apache2-bin (2.4.58-1ubuntu8.5) ...\r\nSetting up apache2 (2.4.58-1ubuntu8.5) ...\r\nEnabling module mpm_event.\r\nEnabling module authz_core.\r\nEnabling module authz_host.\r\nEnabling module authn_core.\r\nEnabling module auth_basic.\r\nEnabling module access_compat.\r\nEnabling module authn_file.\r\nEnabling module authz_user.\r\nEnabling module alias.\r\nEnabling module dir.\r\nEnabling module autoindex.\r\nEnabling module env.\r\nEnabling module mime.\r\nEnabling module negotiation.\r\nEnabling module setenvif.\r\nEnabling module filter.\r\nEnabling module deflate.\r\nEnabling module status.\r\nEnabling module reqtimeout.\r\nEnabling conf charset.\r\nEnabling conf localized-error-pages.\r\nEnabling conf other-vhosts-access-log.\r\nEnabling conf security.\r\nEnabling conf serve-cgi-bin.\r\nEnabling site 000-default.\r\nCreated symlink /etc/systemd/system/multi-user.target.wants/apache2.service → /usr/lib/systemd/system/apache2.service.\r\r\nCreated symlink /etc/systemd/system/multi-user.target.wants/apache-htcacheclean.service → /usr/lib/systemd/system/apache-htcacheclean.service.\r\r\nProcessing triggers for ufw (0.36.2-6) ...\r\nProcessing triggers for man-db (2.12.0-4build2) ...\r\nProcessing triggers for libc-bin (2.39-0ubuntu8.4) ...\r\n\nPending kernel upgrade!\nRunning kernel version:\n  6.8.0-51-generic\nDiagnostics:\n  The currently running kernel version is not the expected kernel version 6.8.0-56-generic.\n\nRestarting the system to load the new kernel will not be handled automatically, so you should consider rebooting.\n\nRestarting services...\n systemctl restart vboxadd-service.service\n\nService restarts being deferred:\n /etc/needrestart/restart.d/dbus.service\n systemctl restart getty@tty1.service\n systemctl restart systemd-logind.service\n systemctl restart unattended-upgrades.service\n\nNo containers need to be restarted.\n\nNo user sessions are running outdated binaries.\n\nNo VM guests are running outdated hypervisor (qemu) binaries on this host.\n",
    "stdout_lines": [
        "Reading package lists...",
        "Building dependency tree...",
        "Reading state information...",
        "The following additional packages will be installed:",
        "  apache2-bin apache2-data apache2-utils libapr1t64 libaprutil1-dbd-sqlite3",
        "  libaprutil1-ldap libaprutil1t64 liblua5.4-0 ssl-cert",
        "Suggested packages:",
        "  apache2-doc apache2-suexec-pristine | apache2-suexec-custom www-browser",
        "The following NEW packages will be installed:",
        "  apache2 apache2-bin apache2-data apache2-utils libapr1t64",
        "  libaprutil1-dbd-sqlite3 libaprutil1-ldap libaprutil1t64 liblua5.4-0 ssl-cert",
        "0 upgraded, 10 newly installed, 0 to remove and 120 not upgraded.",
        "Need to get 2084 kB of archives.",
        "After this operation, 8094 kB of additional disk space will be used.",
        "Get:1 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 libapr1t64 amd64 1.7.2-3.1ubuntu0.1 [108 kB]",
        "Get:2 http://us.archive.ubuntu.com/ubuntu noble/main amd64 libaprutil1t64 amd64 1.6.3-1.1ubuntu7 [91.9 kB]",
        "Get:3 http://us.archive.ubuntu.com/ubuntu noble/main amd64 libaprutil1-dbd-sqlite3 amd64 1.6.3-1.1ubuntu7 [11.2 kB]",
        "Get:4 http://us.archive.ubuntu.com/ubuntu noble/main amd64 libaprutil1-ldap amd64 1.6.3-1.1ubuntu7 [9116 B]",
        "Get:5 http://us.archive.ubuntu.com/ubuntu noble/main amd64 liblua5.4-0 amd64 5.4.6-3build2 [166 kB]",
        "Get:6 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2-bin amd64 2.4.58-1ubuntu8.5 [1329 kB]",
        "Get:7 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2-data all 2.4.58-1ubuntu8.5 [163 kB]",
        "Get:8 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2-utils amd64 2.4.58-1ubuntu8.5 [97.1 kB]",
        "Get:9 http://us.archive.ubuntu.com/ubuntu noble-updates/main amd64 apache2 amd64 2.4.58-1ubuntu8.5 [90.2 kB]",
        "Get:10 http://us.archive.ubuntu.com/ubuntu noble/main amd64 ssl-cert all 1.1.2ubuntu1 [17.8 kB]",
        "Preconfiguring packages ...",
        "Fetched 2084 kB in 1s (2086 kB/s)",
        "Selecting previously unselected package libapr1t64:amd64.",
        "(Reading database ... ",
        "(Reading database ... 5%",
        "(Reading database ... 10%",
        "(Reading database ... 15%",
        "(Reading database ... 20%",
        "(Reading database ... 25%",
        "(Reading database ... 30%",
        "(Reading database ... 35%",
        "(Reading database ... 40%",
        "(Reading database ... 45%",
        "(Reading database ... 50%",
        "(Reading database ... 55%",
        "(Reading database ... 60%",
        "(Reading database ... 65%",
        "(Reading database ... 70%",
        "(Reading database ... 75%",
        "(Reading database ... 80%",
        "(Reading database ... 85%",
        "(Reading database ... 90%",
        "(Reading database ... 95%",
        "(Reading database ... 100%",
        "(Reading database ... 124309 files and directories currently installed.)",
        "Preparing to unpack .../0-libapr1t64_1.7.2-3.1ubuntu0.1_amd64.deb ...",
        "Unpacking libapr1t64:amd64 (1.7.2-3.1ubuntu0.1) ...",
        "Selecting previously unselected package libaprutil1t64:amd64.",
        "Preparing to unpack .../1-libaprutil1t64_1.6.3-1.1ubuntu7_amd64.deb ...",
        "Unpacking libaprutil1t64:amd64 (1.6.3-1.1ubuntu7) ...",
        "Selecting previously unselected package libaprutil1-dbd-sqlite3:amd64.",
        "Preparing to unpack .../2-libaprutil1-dbd-sqlite3_1.6.3-1.1ubuntu7_amd64.deb ...",
        "Unpacking libaprutil1-dbd-sqlite3:amd64 (1.6.3-1.1ubuntu7) ...",
        "Selecting previously unselected package libaprutil1-ldap:amd64.",
        "Preparing to unpack .../3-libaprutil1-ldap_1.6.3-1.1ubuntu7_amd64.deb ...",
        "Unpacking libaprutil1-ldap:amd64 (1.6.3-1.1ubuntu7) ...",
        "Selecting previously unselected package liblua5.4-0:amd64.",
        "Preparing to unpack .../4-liblua5.4-0_5.4.6-3build2_amd64.deb ...",
        "Unpacking liblua5.4-0:amd64 (5.4.6-3build2) ...",
        "Selecting previously unselected package apache2-bin.",
        "Preparing to unpack .../5-apache2-bin_2.4.58-1ubuntu8.5_amd64.deb ...",
        "Unpacking apache2-bin (2.4.58-1ubuntu8.5) ...",
        "Selecting previously unselected package apache2-data.",
        "Preparing to unpack .../6-apache2-data_2.4.58-1ubuntu8.5_all.deb ...",
        "Unpacking apache2-data (2.4.58-1ubuntu8.5) ...",
        "Selecting previously unselected package apache2-utils.",
        "Preparing to unpack .../7-apache2-utils_2.4.58-1ubuntu8.5_amd64.deb ...",
        "Unpacking apache2-utils (2.4.58-1ubuntu8.5) ...",
        "Selecting previously unselected package apache2.",
        "Preparing to unpack .../8-apache2_2.4.58-1ubuntu8.5_amd64.deb ...",
        "Unpacking apache2 (2.4.58-1ubuntu8.5) ...",
        "Selecting previously unselected package ssl-cert.",
        "Preparing to unpack .../9-ssl-cert_1.1.2ubuntu1_all.deb ...",
        "Unpacking ssl-cert (1.1.2ubuntu1) ...",
        "Setting up ssl-cert (1.1.2ubuntu1) ...",
        "Created symlink /etc/systemd/system/multi-user.target.wants/ssl-cert.service → /usr/lib/systemd/system/ssl-cert.service.",
        "",
        "Setting up libapr1t64:amd64 (1.7.2-3.1ubuntu0.1) ...",
        "Setting up liblua5.4-0:amd64 (5.4.6-3build2) ...",
        "Setting up apache2-data (2.4.58-1ubuntu8.5) ...",
        "Setting up libaprutil1t64:amd64 (1.6.3-1.1ubuntu7) ...",
        "Setting up libaprutil1-ldap:amd64 (1.6.3-1.1ubuntu7) ...",
        "Setting up libaprutil1-dbd-sqlite3:amd64 (1.6.3-1.1ubuntu7) ...",
        "Setting up apache2-utils (2.4.58-1ubuntu8.5) ...",
        "Setting up apache2-bin (2.4.58-1ubuntu8.5) ...",
        "Setting up apache2 (2.4.58-1ubuntu8.5) ...",
        "Enabling module mpm_event.",
        "Enabling module authz_core.",
        "Enabling module authz_host.",
        "Enabling module authn_core.",
        "Enabling module auth_basic.",
        "Enabling module access_compat.",
        "Enabling module authn_file.",
        "Enabling module authz_user.",
        "Enabling module alias.",
        "Enabling module dir.",
        "Enabling module autoindex.",
        "Enabling module env.",
        "Enabling module mime.",
        "Enabling module negotiation.",
        "Enabling module setenvif.",
        "Enabling module filter.",
        "Enabling module deflate.",
        "Enabling module status.",
        "Enabling module reqtimeout.",
        "Enabling conf charset.",
        "Enabling conf localized-error-pages.",
        "Enabling conf other-vhosts-access-log.",
        "Enabling conf security.",
        "Enabling conf serve-cgi-bin.",
        "Enabling site 000-default.",
        "Created symlink /etc/systemd/system/multi-user.target.wants/apache2.service → /usr/lib/systemd/system/apache2.service.",
        "",
        "Created symlink /etc/systemd/system/multi-user.target.wants/apache-htcacheclean.service → /usr/lib/systemd/system/apache-htcacheclean.service.",
        "",
        "Processing triggers for ufw (0.36.2-6) ...",
        "Processing triggers for man-db (2.12.0-4build2) ...",
        "Processing triggers for libc-bin (2.39-0ubuntu8.4) ...",
        "",
        "Pending kernel upgrade!",
        "Running kernel version:",
        "  6.8.0-51-generic",
        "Diagnostics:",
        "  The currently running kernel version is not the expected kernel version 6.8.0-56-generic.",
        "",
        "Restarting the system to load the new kernel will not be handled automatically, so you should consider rebooting.",
        "",
        "Restarting services...",
        " systemctl restart vboxadd-service.service",
        "",
        "Service restarts being deferred:",
        " /etc/needrestart/restart.d/dbus.service",
        " systemctl restart getty@tty1.service",
        " systemctl restart systemd-logind.service",
        " systemctl restart unattended-upgrades.service",
        "",
        "No containers need to be restarted.",
        "",
        "No user sessions are running outdated binaries.",
        "",
        "No VM guests are running outdated hypervisor (qemu) binaries on this host."
    ]
}
</details>

If you run the same command the 2nd time:

```
$ ansible labvm1 -i hosts --private-key ~/.ssh/id_rsa -u vagrant -b -m apt -a "name=apache2 state=present"
```
<details>
 <summary>Ouput</summary>

[WARNING]: Platform linux on host labvm1 is using the discovered Python interpreter at
/usr/bin/python3.12, but future installation of another Python interpreter could change the
meaning of that path. See https://docs.ansible.com/ansible-
core/2.17/reference_appendices/interpreter_discovery.html for more information.
labvm1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "cache_update_time": 1743374086,
    "cache_updated": false,
    "changed": false
}
</details>

This is becuase Ansible is considered **idempotent**. It makes changes only if needed. Since Apache web server is already installed, it will not reinstall it.

We see there is warning message regarding the Python interpreter, which could be eliminated by setting up a configuration in Ansible
```
$ sudo vim /etc/ansible/ansible.cfg
[defaults]
interpreter_python=auto_silent
```
Now run the similar command but with "state=latest":

```
$  ansible labvm1 -i hosts --private-key ~/.ssh/id_rsa -u vagrant -b -m apt -a "name=apache2 state=latest"
labvm1 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3.12"
    },
    "cache_update_time": 1743374086,
    "cache_updated": false,
    "changed": false
}

```
The warning message dispeared.

Depending on the status of the packages installed on your VM, the output may not exactly the same as shown above. Please read and try to understanding the meaning of the text return by ansible. If it's been updated instead, then run the command again.

### Part 4: Gather software and hardware information available on remote machine
One of the core ansible module is called "setup", it is automatically called by ansible playbook to gather useful "facts" about remote hosts that can be used in ansible playbooks. It can also be executed directly by the ansible command (/usr/bin/ansible) to check out what "facts" are available on a remote host.

```
$ ansible labvm1 -i hosts -m setup

labvm1 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.0.2.15",
            "192.168.56.10"
        ],
... the output omitted
"changed": false
}
```

<details>
 <summary>Click here for complete sample contents of the above</summary>
 labvm1 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.0.2.15",
            "192.168.56.10"
        ],
        "ansible_all_ipv6_addresses": [
            "fd00::a00:27ff:fef0:f51d",
            "fe80::a00:27ff:fef0:f51d",
            "fe80::a00:27ff:fe89:69db"
        ],
        "ansible_apparmor": {
            "status": "enabled"
        },
        "ansible_architecture": "x86_64",
        "ansible_bios_date": "12/01/2006",
        "ansible_bios_vendor": "innotek GmbH",
        "ansible_bios_version": "VirtualBox",
        "ansible_board_asset_tag": "NA",
        "ansible_board_name": "VirtualBox",
        "ansible_board_serial": "NA",
        "ansible_board_vendor": "Oracle Corporation",
        "ansible_board_version": "1.2",
        "ansible_chassis_asset_tag": "NA",
        "ansible_chassis_serial": "NA",
        "ansible_chassis_vendor": "Oracle Corporation",
        "ansible_chassis_version": "NA",
        "ansible_cmdline": {
            "BOOT_IMAGE": "/vmlinuz-6.8.0-51-generic",
            "autoinstall": true,
            "ds": "nocloud-net",
            "ro": true,
            "root": "/dev/mapper/ubuntu--vg-ubuntu--lv"
        },
        "ansible_date_time": {
            "date": "2025-03-31",
            "day": "31",
            "epoch": "1743381724",
            "epoch_int": "1743381724",
            "hour": "00",
            "iso8601": "2025-03-31T00:42:04Z",
            "iso8601_basic": "20250331T004204730227",
            "iso8601_basic_short": "20250331T004204",
            "iso8601_micro": "2025-03-31T00:42:04.730227Z",
            "minute": "42",
            "month": "03",
            "second": "04",
            "time": "00:42:04",
            "tz": "UTC",
            "tz_dst": "UTC",
            "tz_offset": "+0000",
            "weekday": "Monday",
            "weekday_number": "1",
            "weeknumber": "13",
            "year": "2025"
        },
        "ansible_default_ipv4": {
            "address": "10.0.2.15",
            "alias": "enp0s3",
            "broadcast": "",
            "gateway": "10.0.2.2",
            "interface": "enp0s3",
            "macaddress": "08:00:27:f0:f5:1d",
            "mtu": 1500,
            "netmask": "255.255.255.0",
            "network": "10.0.2.0",
            "prefix": "24",
            "type": "ether"
        },
        "ansible_default_ipv6": {
            "address": "fd00::a00:27ff:fef0:f51d",
            "gateway": "fe80::2",
            "interface": "enp0s3",
            "macaddress": "08:00:27:f0:f5:1d",
            "mtu": 1500,
            "prefix": "64",
            "scope": "global",
            "type": "ether"
        },
        "ansible_device_links": {
            "ids": {
                "dm-0": [
                    "dm-name-ubuntu--vg-ubuntu--lv",
                    "dm-uuid-LVM-gcstO6NqhZ75oxWeFUHzfNu5vty9ZWgzM2LFMkZ9mF0Vpxe4kAYQs9OOtNTxIp2d"
                ],
                "sda": [
                    "ata-VBOX_HARDDISK_VBeb3c7999-cdc105c5",
                    "scsi-0ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5",
                    "scsi-1ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5",
                    "scsi-SATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5"
                ],
                "sda1": [
                    "ata-VBOX_HARDDISK_VBeb3c7999-cdc105c5-part1",
                    "scsi-0ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part1",
                    "scsi-1ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part1",
                    "scsi-SATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part1"
                ],
                "sda2": [
                    "ata-VBOX_HARDDISK_VBeb3c7999-cdc105c5-part2",
                    "scsi-0ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part2",
                    "scsi-1ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part2",
                    "scsi-SATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part2"
                ],
                "sda3": [
                    "ata-VBOX_HARDDISK_VBeb3c7999-cdc105c5-part3",
                    "lvm-pv-uuid-65jUTf-Kz8X-ps8G-1BMJ-iKhF-FBya-0YPXuo",
                    "scsi-0ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part3",
                    "scsi-1ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part3",
                    "scsi-SATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part3"
                ]
            },
            "labels": {},
            "masters": {
                "sda3": [
                    "dm-0"
                ]
            },
            "uuids": {
                "dm-0": [
                    "c49cc7e4-ea6e-470c-9ded-23ac5face980"
                ],
                "sda1": [
                    "6232-5EA6"
                ],
                "sda2": [
                    "ada47328-b19a-4188-8a88-7022a9ab5469"
                ]
            }
        },
        "ansible_devices": {
            "dm-0": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [
                        "dm-name-ubuntu--vg-ubuntu--lv",
                        "dm-uuid-LVM-gcstO6NqhZ75oxWeFUHzfNu5vty9ZWgzM2LFMkZ9mF0Vpxe4kAYQs9OOtNTxIp2d"
                    ],
                    "labels": [],
                    "masters": [],
                    "uuids": [
                        "c49cc7e4-ea6e-470c-9ded-23ac5face980"
                    ]
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "",
                "sectors": "20971520",
                "sectorsize": "512",
                "size": "10.00 GB",
                "support_discard": "0",
                "vendor": null,
                "virtual": 1
            },
            "loop0": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "none",
                "sectors": "0",
                "sectorsize": "512",
                "size": "0.00 Bytes",
                "support_discard": "4096",
                "vendor": null,
                "virtual": 1
            },
            "loop1": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "none",
                "sectors": "0",
                "sectorsize": "512",
                "size": "0.00 Bytes",
                "support_discard": "512",
                "vendor": null,
                "virtual": 1
            },
            "loop2": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "none",
                "sectors": "0",
                "sectorsize": "512",
                "size": "0.00 Bytes",
                "support_discard": "512",
                "vendor": null,
                "virtual": 1
            },
            "loop3": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "none",
                "sectors": "0",
                "sectorsize": "512",
                "size": "0.00 Bytes",
                "support_discard": "512",
                "vendor": null,
                "virtual": 1
            },
            "loop4": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "none",
                "sectors": "0",
                "sectorsize": "512",
                "size": "0.00 Bytes",
                "support_discard": "512",
                "vendor": null,
                "virtual": 1
            },
            "loop5": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "none",
                "sectors": "0",
                "sectorsize": "512",
                "size": "0.00 Bytes",
                "support_discard": "512",
                "vendor": null,
                "virtual": 1
            },
            "loop6": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "none",
                "sectors": "0",
                "sectorsize": "512",
                "size": "0.00 Bytes",
                "support_discard": "512",
                "vendor": null,
                "virtual": 1
            },
            "loop7": {
                "holders": [],
                "host": "",
                "links": {
                    "ids": [],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": null,
                "partitions": {},
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "none",
                "sectors": "0",
                "sectorsize": "512",
                "size": "0.00 Bytes",
                "support_discard": "512",
                "vendor": null,
                "virtual": 1
            },
            "sda": {
                "holders": [],
                "host": "SATA controller: Intel Corporation 82801HM/HEM (ICH8M/ICH8M-E) SATA Controller [AHCI mode] (rev 02)",
                "links": {
                    "ids": [
                        "ata-VBOX_HARDDISK_VBeb3c7999-cdc105c5",
                        "scsi-0ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5",
                        "scsi-1ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5",
                        "scsi-SATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5"
                    ],
                    "labels": [],
                    "masters": [],
                    "uuids": []
                },
                "model": "VBOX HARDDISK",
                "partitions": {
                    "sda1": {
                        "holders": [],
                        "links": {
                            "ids": [
                                "ata-VBOX_HARDDISK_VBeb3c7999-cdc105c5-part1",
                                "scsi-0ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part1",
                                "scsi-1ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part1",
                                "scsi-SATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part1"
                            ],
                            "labels": [],
                            "masters": [],
                            "uuids": [
                                "6232-5EA6"
                            ]
                        },
                        "sectors": "1906688",
                        "sectorsize": 512,
                        "size": "931.00 MB",
                        "start": "2048",
                        "uuid": "6232-5EA6"
                    },
                    "sda2": {
                        "holders": [],
                        "links": {
                            "ids": [
                                "ata-VBOX_HARDDISK_VBeb3c7999-cdc105c5-part2",
                                "scsi-0ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part2",
                                "scsi-1ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part2",
                                "scsi-SATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part2"
                            ],
                            "labels": [],
                            "masters": [],
                            "uuids": [
                                "ada47328-b19a-4188-8a88-7022a9ab5469"
                            ]
                        },
                        "sectors": "3670016",
                        "sectorsize": 512,
                        "size": "1.75 GB",
                        "start": "1908736",
                        "uuid": "ada47328-b19a-4188-8a88-7022a9ab5469"
                    },
                    "sda3": {
                        "holders": [
                            "ubuntu--vg-ubuntu--lv"
                        ],
                        "links": {
                            "ids": [
                                "ata-VBOX_HARDDISK_VBeb3c7999-cdc105c5-part3",
                                "lvm-pv-uuid-65jUTf-Kz8X-ps8G-1BMJ-iKhF-FBya-0YPXuo",
                                "scsi-0ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part3",
                                "scsi-1ATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part3",
                                "scsi-SATA_VBOX_HARDDISK_VBeb3c7999-cdc105c5-part3"
                            ],
                            "labels": [],
                            "masters": [
                                "dm-0"
                            ],
                            "uuids": []
                        },
                        "sectors": "35379200",
                        "sectorsize": 512,
                        "size": "16.87 GB",
                        "start": "5578752",
                        "uuid": null
                    }
                },
                "removable": "0",
                "rotational": "1",
                "sas_address": null,
                "sas_device_handle": null,
                "scheduler_mode": "mq-deadline",
                "sectors": "40960000",
                "sectorsize": "512",
                "size": "19.53 GB",
                "support_discard": "512",
                "vendor": "ATA",
                "virtual": 1
            }
        },
        "ansible_distribution": "Ubuntu",
        "ansible_distribution_file_parsed": true,
        "ansible_distribution_file_path": "/etc/os-release",
        "ansible_distribution_file_variety": "Debian",
        "ansible_distribution_major_version": "24",
        "ansible_distribution_release": "noble",
        "ansible_distribution_version": "24.04",
        "ansible_dns": {
            "nameservers": [
                "127.0.0.53"
            ],
            "options": {
                "edns0": true,
                "trust-ad": true
            },
            "search": [
                "cover-all.local"
            ]
        },
        "ansible_domain": "",
        "ansible_effective_group_id": 1000,
        "ansible_effective_user_id": 1000,
        "ansible_enp0s3": {
            "active": true,
            "device": "enp0s3",
            "features": {
                "esp_hw_offload": "off [fixed]",
                "esp_tx_csum_hw_offload": "off [fixed]",
                "fcoe_mtu": "off [fixed]",
                "generic_receive_offload": "on",
                "generic_segmentation_offload": "on",
                "highdma": "off [fixed]",
                "hsr_dup_offload": "off [fixed]",
                "hsr_fwd_offload": "off [fixed]",
                "hsr_tag_ins_offload": "off [fixed]",
                "hsr_tag_rm_offload": "off [fixed]",
                "hw_tc_offload": "off [fixed]",
                "l2_fwd_offload": "off [fixed]",
                "large_receive_offload": "off [fixed]",
                "loopback": "off [fixed]",
                "macsec_hw_offload": "off [fixed]",
                "netns_local": "off [fixed]",
                "ntuple_filters": "off [fixed]",
                "receive_hashing": "off [fixed]",
                "rx_all": "off",
                "rx_checksumming": "off",
                "rx_fcs": "off",
                "rx_gro_hw": "off [fixed]",
                "rx_gro_list": "off",
                "rx_udp_gro_forwarding": "off",
                "rx_udp_tunnel_port_offload": "off [fixed]",
                "rx_vlan_filter": "on [fixed]",
                "rx_vlan_offload": "on",
                "rx_vlan_stag_filter": "off [fixed]",
                "rx_vlan_stag_hw_parse": "off [fixed]",
                "scatter_gather": "on",
                "tcp_segmentation_offload": "on",
                "tls_hw_record": "off [fixed]",
                "tls_hw_rx_offload": "off [fixed]",
                "tls_hw_tx_offload": "off [fixed]",
                "tx_checksum_fcoe_crc": "off [fixed]",
                "tx_checksum_ip_generic": "on",
                "tx_checksum_ipv4": "off [fixed]",
                "tx_checksum_ipv6": "off [fixed]",
                "tx_checksum_sctp": "off [fixed]",
                "tx_checksumming": "on",
                "tx_esp_segmentation": "off [fixed]",
                "tx_fcoe_segmentation": "off [fixed]",
                "tx_gre_csum_segmentation": "off [fixed]",
                "tx_gre_segmentation": "off [fixed]",
                "tx_gso_list": "off [fixed]",
                "tx_gso_partial": "off [fixed]",
                "tx_gso_robust": "off [fixed]",
                "tx_ipxip4_segmentation": "off [fixed]",
                "tx_ipxip6_segmentation": "off [fixed]",
                "tx_lockless": "off [fixed]",
                "tx_nocache_copy": "off",
                "tx_scatter_gather": "on",
                "tx_scatter_gather_fraglist": "off [fixed]",
                "tx_sctp_segmentation": "off [fixed]",
                "tx_tcp6_segmentation": "off [fixed]",
                "tx_tcp_ecn_segmentation": "off [fixed]",
                "tx_tcp_mangleid_segmentation": "off",
                "tx_tcp_segmentation": "on",
                "tx_tunnel_remcsum_segmentation": "off [fixed]",
                "tx_udp_segmentation": "off [fixed]",
                "tx_udp_tnl_csum_segmentation": "off [fixed]",
                "tx_udp_tnl_segmentation": "off [fixed]",
                "tx_vlan_offload": "on [fixed]",
                "tx_vlan_stag_hw_insert": "off [fixed]",
                "vlan_challenged": "off [fixed]"
            },
            "hw_timestamp_filters": [],
            "ipv4": {
                "address": "10.0.2.15",
                "broadcast": "",
                "netmask": "255.255.255.0",
                "network": "10.0.2.0",
                "prefix": "24"
            },
            "ipv6": [
                {
                    "address": "fd00::a00:27ff:fef0:f51d",
                    "prefix": "64",
                    "scope": "global"
                },
                {
                    "address": "fe80::a00:27ff:fef0:f51d",
                    "prefix": "64",
                    "scope": "link"
                }
            ],
            "macaddress": "08:00:27:f0:f5:1d",
            "module": "e1000",
            "mtu": 1500,
            "pciid": "0000:00:03.0",
            "promisc": false,
            "speed": 1000,
            "timestamping": [],
            "type": "ether"
        },
        "ansible_enp0s8": {
            "active": true,
            "device": "enp0s8",
            "features": {
                "esp_hw_offload": "off [fixed]",
                "esp_tx_csum_hw_offload": "off [fixed]",
                "fcoe_mtu": "off [fixed]",
                "generic_receive_offload": "on",
                "generic_segmentation_offload": "on",
                "highdma": "off [fixed]",
                "hsr_dup_offload": "off [fixed]",
                "hsr_fwd_offload": "off [fixed]",
                "hsr_tag_ins_offload": "off [fixed]",
                "hsr_tag_rm_offload": "off [fixed]",
                "hw_tc_offload": "off [fixed]",
                "l2_fwd_offload": "off [fixed]",
                "large_receive_offload": "off [fixed]",
                "loopback": "off [fixed]",
                "macsec_hw_offload": "off [fixed]",
                "netns_local": "off [fixed]",
                "ntuple_filters": "off [fixed]",
                "receive_hashing": "off [fixed]",
                "rx_all": "off",
                "rx_checksumming": "off",
                "rx_fcs": "off",
                "rx_gro_hw": "off [fixed]",
                "rx_gro_list": "off",
                "rx_udp_gro_forwarding": "off",
                "rx_udp_tunnel_port_offload": "off [fixed]",
                "rx_vlan_filter": "on [fixed]",
                "rx_vlan_offload": "on",
                "rx_vlan_stag_filter": "off [fixed]",
                "rx_vlan_stag_hw_parse": "off [fixed]",
                "scatter_gather": "on",
                "tcp_segmentation_offload": "on",
                "tls_hw_record": "off [fixed]",
                "tls_hw_rx_offload": "off [fixed]",
                "tls_hw_tx_offload": "off [fixed]",
                "tx_checksum_fcoe_crc": "off [fixed]",
                "tx_checksum_ip_generic": "on",
                "tx_checksum_ipv4": "off [fixed]",
                "tx_checksum_ipv6": "off [fixed]",
                "tx_checksum_sctp": "off [fixed]",
                "tx_checksumming": "on",
                "tx_esp_segmentation": "off [fixed]",
                "tx_fcoe_segmentation": "off [fixed]",
                "tx_gre_csum_segmentation": "off [fixed]",
                "tx_gre_segmentation": "off [fixed]",
                "tx_gso_list": "off [fixed]",
                "tx_gso_partial": "off [fixed]",
                "tx_gso_robust": "off [fixed]",
                "tx_ipxip4_segmentation": "off [fixed]",
                "tx_ipxip6_segmentation": "off [fixed]",
                "tx_lockless": "off [fixed]",
                "tx_nocache_copy": "off",
                "tx_scatter_gather": "on",
                "tx_scatter_gather_fraglist": "off [fixed]",
                "tx_sctp_segmentation": "off [fixed]",
                "tx_tcp6_segmentation": "off [fixed]",
                "tx_tcp_ecn_segmentation": "off [fixed]",
                "tx_tcp_mangleid_segmentation": "off",
                "tx_tcp_segmentation": "on",
                "tx_tunnel_remcsum_segmentation": "off [fixed]",
                "tx_udp_segmentation": "off [fixed]",
                "tx_udp_tnl_csum_segmentation": "off [fixed]",
                "tx_udp_tnl_segmentation": "off [fixed]",
                "tx_vlan_offload": "on [fixed]",
                "tx_vlan_stag_hw_insert": "off [fixed]",
                "vlan_challenged": "off [fixed]"
            },
            "hw_timestamp_filters": [],
            "ipv4": {
                "address": "192.168.56.10",
                "broadcast": "192.168.56.255",
                "netmask": "255.255.255.0",
                "network": "192.168.56.0",
                "prefix": "24"
            },
            "ipv6": [
                {
                    "address": "fe80::a00:27ff:fe89:69db",
                    "prefix": "64",
                    "scope": "link"
                }
            ],
            "macaddress": "08:00:27:89:69:db",
            "module": "e1000",
            "mtu": 1500,
            "pciid": "0000:00:08.0",
            "promisc": false,
            "speed": 1000,
            "timestamping": [],
            "type": "ether"
        },
        "ansible_env": {
            "DBUS_SESSION_BUS_ADDRESS": "unix:path=/run/user/1000/bus",
            "HOME": "/home/vagrant",
            "LANG": "en_US.UTF-8",
            "LOGNAME": "vagrant",
            "PATH": "/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin",
            "PWD": "/home/vagrant",
            "SHELL": "/bin/bash",
            "SHLVL": "0",
            "SSH_CLIENT": "192.168.56.12 42788 22",
            "SSH_CONNECTION": "192.168.56.12 42788 192.168.56.10 22",
            "SSH_TTY": "/dev/pts/1",
            "TERM": "xterm-256color",
            "USER": "vagrant",
            "XDG_RUNTIME_DIR": "/run/user/1000",
            "XDG_SESSION_CLASS": "user",
            "XDG_SESSION_ID": "35",
            "XDG_SESSION_TYPE": "tty",
            "_": "/bin/sh"
        },
        "ansible_fibre_channel_wwn": [],
        "ansible_fips": false,
        "ansible_form_factor": "Other",
        "ansible_fqdn": "labvm1",
        "ansible_hostname": "labvm1",
        "ansible_hostnqn": "",
        "ansible_interfaces": [
            "lo",
            "enp0s3",
            "enp0s8"
        ],
        "ansible_is_chroot": false,
        "ansible_iscsi_iqn": "",
        "ansible_kernel": "6.8.0-51-generic",
        "ansible_kernel_version": "#52-Ubuntu SMP PREEMPT_DYNAMIC Thu Dec  5 13:09:44 UTC 2024",
        "ansible_lo": {
            "active": true,
            "device": "lo",
            "features": {
                "esp_hw_offload": "off [fixed]",
                "esp_tx_csum_hw_offload": "off [fixed]",
                "fcoe_mtu": "off [fixed]",
                "generic_receive_offload": "on",
                "generic_segmentation_offload": "on",
                "highdma": "on [fixed]",
                "hsr_dup_offload": "off [fixed]",
                "hsr_fwd_offload": "off [fixed]",
                "hsr_tag_ins_offload": "off [fixed]",
                "hsr_tag_rm_offload": "off [fixed]",
                "hw_tc_offload": "off [fixed]",
                "l2_fwd_offload": "off [fixed]",
                "large_receive_offload": "off [fixed]",
                "loopback": "on [fixed]",
                "macsec_hw_offload": "off [fixed]",
                "netns_local": "on [fixed]",
                "ntuple_filters": "off [fixed]",
                "receive_hashing": "off [fixed]",
                "rx_all": "off [fixed]",
                "rx_checksumming": "on [fixed]",
                "rx_fcs": "off [fixed]",
                "rx_gro_hw": "off [fixed]",
                "rx_gro_list": "off",
                "rx_udp_gro_forwarding": "off",
                "rx_udp_tunnel_port_offload": "off [fixed]",
                "rx_vlan_filter": "off [fixed]",
                "rx_vlan_offload": "off [fixed]",
                "rx_vlan_stag_filter": "off [fixed]",
                "rx_vlan_stag_hw_parse": "off [fixed]",
                "scatter_gather": "on",
                "tcp_segmentation_offload": "on",
                "tls_hw_record": "off [fixed]",
                "tls_hw_rx_offload": "off [fixed]",
                "tls_hw_tx_offload": "off [fixed]",
                "tx_checksum_fcoe_crc": "off [fixed]",
                "tx_checksum_ip_generic": "on [fixed]",
                "tx_checksum_ipv4": "off [fixed]",
                "tx_checksum_ipv6": "off [fixed]",
                "tx_checksum_sctp": "on [fixed]",
                "tx_checksumming": "on",
                "tx_esp_segmentation": "off [fixed]",
                "tx_fcoe_segmentation": "off [fixed]",
                "tx_gre_csum_segmentation": "off [fixed]",
                "tx_gre_segmentation": "off [fixed]",
                "tx_gso_list": "on",
                "tx_gso_partial": "off [fixed]",
                "tx_gso_robust": "off [fixed]",
                "tx_ipxip4_segmentation": "off [fixed]",
                "tx_ipxip6_segmentation": "off [fixed]",
                "tx_lockless": "on [fixed]",
                "tx_nocache_copy": "off [fixed]",
                "tx_scatter_gather": "on [fixed]",
                "tx_scatter_gather_fraglist": "on [fixed]",
                "tx_sctp_segmentation": "on",
                "tx_tcp6_segmentation": "on",
                "tx_tcp_ecn_segmentation": "on",
                "tx_tcp_mangleid_segmentation": "on",
                "tx_tcp_segmentation": "on",
                "tx_tunnel_remcsum_segmentation": "off [fixed]",
                "tx_udp_segmentation": "on",
                "tx_udp_tnl_csum_segmentation": "off [fixed]",
                "tx_udp_tnl_segmentation": "off [fixed]",
                "tx_vlan_offload": "off [fixed]",
                "tx_vlan_stag_hw_insert": "off [fixed]",
                "vlan_challenged": "on [fixed]"
            },
            "hw_timestamp_filters": [],
            "ipv4": {
                "address": "127.0.0.1",
                "broadcast": "",
                "netmask": "255.0.0.0",
                "network": "127.0.0.0",
                "prefix": "8"
            },
            "ipv6": [
                {
                    "address": "::1",
                    "prefix": "128",
                    "scope": "host"
                }
            ],
            "mtu": 65536,
            "promisc": false,
            "timestamping": [],
            "type": "loopback"
        },
        "ansible_loadavg": {
            "15m": 0.00537109375,
            "1m": 0.080078125,
            "5m": 0.0166015625
        },
        "ansible_local": {},
        "ansible_locally_reachable_ips": {
            "ipv4": [
                "10.0.2.15",
                "127.0.0.0/8",
                "127.0.0.1",
                "192.168.56.10"
            ],
            "ipv6": [
                "::1",
                "fd00::a00:27ff:fef0:f51d",
                "fe80::a00:27ff:fe89:69db",
                "fe80::a00:27ff:fef0:f51d"
            ]
        },
        "ansible_lsb": {
            "codename": "noble",
            "description": "Ubuntu 24.04.1 LTS",
            "id": "Ubuntu",
            "major_release": "24",
            "release": "24.04"
        },
        "ansible_lvm": "N/A",
        "ansible_machine": "x86_64",
        "ansible_machine_id": "5c8e04bf168b4792a2d2622d50ec8c47",
        "ansible_memfree_mb": 551,
        "ansible_memory_mb": {
            "nocache": {
                "free": 1604,
                "used": 346
            },
            "real": {
                "free": 551,
                "total": 1950,
                "used": 1399
            },
            "swap": {
                "cached": 0,
                "free": 1955,
                "total": 1955,
                "used": 0
            }
        },
        "ansible_memtotal_mb": 1950,
        "ansible_mounts": [
            {
                "block_available": 1127183,
                "block_size": 4096,
                "block_total": 2554693,
                "block_used": 1427510,
                "device": "/dev/mapper/ubuntu--vg-ubuntu--lv",
                "dump": 0,
                "fstype": "ext4",
                "inode_available": 519059,
                "inode_total": 655360,
                "inode_used": 136301,
                "mount": "/",
                "options": "rw,relatime",
                "passno": 0,
                "size_available": 4616941568,
                "size_total": 10464022528,
                "uuid": "c49cc7e4-ea6e-470c-9ded-23ac5face980"
            },
            {
                "block_available": 367400,
                "block_size": 4096,
                "block_total": 442014,
                "block_used": 74614,
                "device": "/dev/sda2",
                "dump": 0,
                "fstype": "ext4",
                "inode_available": 114375,
                "inode_total": 114688,
                "inode_used": 313,
                "mount": "/boot",
                "options": "rw,relatime",
                "passno": 0,
                "size_available": 1504870400,
                "size_total": 1810489344,
                "uuid": "ada47328-b19a-4188-8a88-7022a9ab5469"
            },
            {
                "block_available": 235719,
                "block_size": 4096,
                "block_total": 237859,
                "block_used": 2140,
                "device": "/dev/sda1",
                "dump": 0,
                "fstype": "vfat",
                "inode_available": 0,
                "inode_total": 0,
                "inode_used": 0,
                "mount": "/boot/efi",
                "options": "rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro",
                "passno": 0,
                "size_available": 965505024,
                "size_total": 974270464,
                "uuid": "6232-5EA6"
            }
        ],
        "ansible_nodename": "labvm1",
        "ansible_os_family": "Debian",
        "ansible_pkg_mgr": "apt",
        "ansible_proc_cmdline": {
            "BOOT_IMAGE": "/vmlinuz-6.8.0-51-generic",
            "autoinstall": true,
            "ds": "nocloud-net",
            "ro": true,
            "root": "/dev/mapper/ubuntu--vg-ubuntu--lv"
        },
        "ansible_processor": [
            "0",
            "GenuineIntel",
            "Intel(R) Xeon(R) Silver 4316 CPU @ 2.30GHz"
        ],
        "ansible_processor_cores": 1,
        "ansible_processor_count": 1,
        "ansible_processor_nproc": 1,
        "ansible_processor_threads_per_core": 1,
        "ansible_processor_vcpus": 1,
        "ansible_product_name": "VirtualBox",
        "ansible_product_serial": "NA",
        "ansible_product_uuid": "NA",
        "ansible_product_version": "1.2",
        "ansible_python": {
            "executable": "/usr/bin/python3.12",
            "has_sslcontext": true,
            "type": "cpython",
            "version": {
                "major": 3,
                "micro": 3,
                "minor": 12,
                "releaselevel": "final",
                "serial": 0
            },
            "version_info": [
                3,
                12,
                3,
                "final",
                0
            ]
        },
        "ansible_python_version": "3.12.3",
        "ansible_real_group_id": 1000,
        "ansible_real_user_id": 1000,
        "ansible_selinux": {
            "status": "disabled"
        },
        "ansible_selinux_python_present": true,
        "ansible_service_mgr": "systemd",
        "ansible_ssh_host_key_ecdsa_public": "AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBI6LE8rXLLtZ4RuKW3sSK88v5qD69t4za5Q54i02MK8Fs/E6ulAmfV5O1qVQJweoSWODyEQ61uTk12sUIqgQTZQ=",
        "ansible_ssh_host_key_ecdsa_public_keytype": "ecdsa-sha2-nistp256",
        "ansible_ssh_host_key_ed25519_public": "AAAAC3NzaC1lZDI1NTE5AAAAICP4+f5LT4EjFtsmcy319MJs+Oaff/kOySl1/bBBYLg5",
        "ansible_ssh_host_key_ed25519_public_keytype": "ssh-ed25519",
        "ansible_ssh_host_key_rsa_public": "AAAAB3NzaC1yc2EAAAADAQABAAABgQCdnQpqWn9p7uyQ5EsghMkfbrVuxaJlF4bVWteSv4+N5RkcbPvxfBCCbRo7GgpsIGApcl5kPIIV8nqsLLFivDVWiwTlMq1C+PIF8zU/lcLhJjlVpQCq5BBI29nlOyUZfpEa4W3vkCDu1rRqVRHDNB4dz1JbroJu8PwuA1BQZXFMH9dmdoVcVymLPHy2l+oVdr7oWqo2zA1X4qNW/t/CIjRb27EjoYPwpuPXknjgR7GaomvKbQlqy9bdbLR0knBgfyHIU5x/bKQLz8NBcd1agwPcJzx3TPRPPYjnEqKwdjuvC8A3S22Ghkemd6bFKSFV8+1AQxTVsewTCeOlM/3mpgRSkqfQ8+FlOuriVkFqdzxvP2URbXspwphzbtPKsDfE8Tjxz8kbO66Hlnucov6p7ciasINN2prZfM1nOhG1FJNwwPaDE3Ytw4yw3lZHiaJZwbsgMoRiheZdchb7Ilj2OT7dB51Sg/Id5V3jq5xyKk8mdrf2TFCJ4hwJGehJ7RND0Ns=",
        "ansible_ssh_host_key_rsa_public_keytype": "ssh-rsa",
        "ansible_swapfree_mb": 1955,
        "ansible_swaptotal_mb": 1955,
        "ansible_system": "Linux",
        "ansible_system_capabilities": [
            ""
        ],
        "ansible_system_capabilities_enforced": "True",
        "ansible_system_vendor": "innotek GmbH",
        "ansible_uptime_seconds": 7687,
        "ansible_user_dir": "/home/vagrant",
        "ansible_user_gecos": "vagrant",
        "ansible_user_gid": 1000,
        "ansible_user_id": "vagrant",
        "ansible_user_shell": "/bin/bash",
        "ansible_user_uid": 1000,
        "ansible_userspace_architecture": "x86_64",
        "ansible_userspace_bits": "64",
        "ansible_virtualization_role": "guest",
        "ansible_virtualization_tech_guest": [
            "virtualbox"
        ],
        "ansible_virtualization_tech_host": [],
        "ansible_virtualization_type": "virtualbox",
        "discovered_interpreter_python": "/usr/bin/python3.12",
        "gather_subset": [
            "all"
        ],
        "module_setup": true
    },
    "changed": false
}
</details>

## Investigation 2: Ansible Playbook
What is a playbook?
- Playbook is one of the core features of Ansible.
- Playbook tells Ansible what to execute by which user on the remote machine.
- Playbook is like a to-do list for Ansible
- Playbook is written in "YAML".
- Playbook links a task to an ansible module and provide needed arguments to the module which requires them.

### Part 1: A playbook to update the /etc/motd file
Name: motd-play.yml

```
$ curl https://raw.githubusercontent.com/hanliu8/ops445/refs/heads/main/motd.yml -o motd.yml

$ cat motd-play.yml 
---
- name: update motd file
  hosts: vmlab
  user: instructor
  vars:
    apache_version: 2.6
    motd_warning: "WARNING: used by ICT faculty/students only.\n"
    testserver: yes
  tasks:
    - name: setup a MOTD
      copy:
        dest: /etc/motd
        content: "{{ motd_warning }}"
```
Sample Run:

```
$  ansible-playbook -i hosts -b motd.yml

PLAY [update motd file] **********************************************************************

TASK [Gathering Facts] ***********************************************************************
ok: [labvm1]

TASK [setup a Message Of The Day] ************************************************************
changed: [labvm1]

PLAY RECAP ***********************************************************************************
labvm1                     : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```
>Try to run it the 2nd time and pay attention to the result. What conclusion can you draw?

### Part 2: A playbook to install and start Apache Server
Name: httpd-play.yml

---
- hosts: [webservers]
  user: vagrant
  become: yes
  vars:
    apache_version: 2.6
    motd_warning: "WARNING: used by ICT faculty/students only.\n"
    testserver: yes
  tasks:
    - name: Install Apache
      action: apt name=httpd state=installed
    
    - name: restart Apache
      service: 
        name: httpd
        state: restarted

Sample Run:
```
$ ansible-playbook -i hosts httpd-play.yml

PLAY [webservers] ****************************************************************************

TASK [Gathering Facts] ***********************************************************************
ok: [labvm2]

TASK [Install Apache web service] ************************************************************
changed: [labvm2]

TASK [restart Apache service] ****************************************************************
changed: [labvm2]

PLAY RECAP ***********************************************************************************
labvm2                     : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Investigation 3: Using Playbook to configure an OPS445 Linux VM machine
Assume you have just installed the latest version of CentOS 7.x on a VM with GNOME Desktop. You need to configure it so that you can use it for doing the Labs for OPS445.

Study the documentation and examples of following ansible modules:

- copy
- file
- hostname
- template
- user
- apt

Create an ansible playbook named "config_ops445.yml" using the appropriate modules to perform the following configuration tasks on your assigned VM:

update Apache (apache2) installed in the **Investigation 2 - Part 2**

install _'tree'_ package

set the hostname to your Seneca username (Seneca ID) [hostname module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/hostname_module.html#ansible-collections-ansible-builtin-hostname-module)

create a new user with your Seneca_ID with sudo access  [user module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html#ansible-collections-ansible-builtin-user-module)

configure the new user account you created above so that you can ssh to it without password

setup a directory structure using a loop for completing and organizing labs as shown below: [Ansible Loops](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html)

      /home/[seneca_id]/ops445/lab1
      /home/[seneca_id]/ops445/lab2
      /home/[seneca_id]/ops445/lab3
      /home/[seneca_id]/ops445/lab4
      /home/[seneca_id]/ops445/lab5
      /home/[seneca_id]/ops445/lab6
      /home/[seneca_id]/ops445/lab7
      /home/[seneca_id]/ops445/lab8
      

> when it's ready, run your playbook
> in order to test it, log into the VM with the newly created user (your Seneca_ID), install the 'tree' package with sudo, and check the directory structure with the 'tree' command
> if everything is correct, capture its output for a successful run of your playbook to a file named "lab8_[seneca_id].txt"

## Lab 8 Sign-off (Show Instructor)
#### Have the following items ready to show your instructor:
- The updated inventory file called "hosts" which you used to access your VM.
- The Ansible playbook called "config_ops445.yml" for configuring the VM.
- The result of running the playbook "config_ops445.yml". Save the result in a file called "lab8_[seneca_id].txt"
  
#### Upload the following files to blackboard
- hosts
- config_ops445.yml
- lab8_[seneca_id].txt
