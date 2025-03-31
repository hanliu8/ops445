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
$ ansible labvm1 -i hosts --private-key ~/.ssh/id_rsa -u vagrant -m apt -a "name=httpd state=present"
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
    "msg": "'/usr/bin/apt-get -y -o \"Dpkg::Options::=--force-confdef\" -o \"Dpkg::Options::=--force-confold\"       install 'httpd'' failed: E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)\nE: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?\n",
    "rc": 100,
    "stderr": "E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)\nE: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?\n",
    "stderr_lines": [
        **"E: Could not open lock file /var/lib/dpkg/lock-frontend - open (13: Permission denied)",**
        "E: Unable to acquire the dpkg frontend lock (/var/lib/dpkg/lock-frontend), are you root?"
    ],
    "stdout": "",
    "stdout_lines": []
}
```

Add the '-b' option to tell ansible to invoke "sudo" when running the yum command on the remote machine:
```
$ ansible vmlab -i hosts --private-key ~/.ssh/id_rsa -u vagrant -b -m -a "name=software-properties-common state=present"
```

> If you run the same command the 2nd time:

```
$ ansible vmlab -i hosts --private-key ~/.ssh/id_rsa -u instructor -b -m yum -a "name=epel-release state=present"
```

Now run the similar command but with "state=latest":

```
$ ansible vmlab -i hosts --private-key ~/.ssh/id_rsa -u instructor -b -m yum -a "name=epel-release state=latest"
```

Depending on the status of the packages installed on your VM, the output may not exactly the same as shown above. Please read and try to understanding the meaning of the text return by ansible. If it's been updated instead, then run the command again.

### Part 4: Gather software and hardware information available on remote machine
One of the core ansible module is called "setup", it is automatically called by ansible playbook to gather useful "facts" about remote hosts that can be used in ansible playbooks. It can also be executed directly by the ansible command (/usr/bin/ansible) to check out what "facts" are available on a remote host.

```
$ ansible vmlab -i hosts --private-key ~/.ssh/id_rsa -u instructor -m setup
vmlab | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "10.102.114.140"
        ], 
        "ansible_all_ipv6_addresses": [
            "fe80::21d:d8ff:feb7:20cc"
        ], 
        "ansible_apparmor": {
            "status": "disabled"
        }, 
        "ansible_architecture": "x86_64", 
        "ansible_bios_date": "11/26/2012", 

...

        "ansible_userspace_bits": "64", 
        "ansible_virtualization_role": "guest", 
        "ansible_virtualization_type": "VirtualPC", 
        "discovered_interpreter_python": "/usr/bin/python", 
        "gather_subset": [
            "all"
        ], 
        "module_setup": true
    }, 
    "changed": false
}
```

Click here for complete sample contents of the above

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
$ ansible-playbook -i hosts -b motd-play.yml 

PLAY [update motd file] *******************************************************************

TASK [Gathering Facts] ********************************************************************
ok: [vmlab]

TASK [setup a MOTD] ***********************************************************************
changed: [vmlab]

PLAY RECAP ********************************************************************************
vmlab   ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
> Try to run it the 2nd time and pay attention to the result. What conclusion can you draw?

### Part 2: A playbook to install and start Apache Server
Name: httpd-play.yml

---
- hosts: vmlab
  user: instructor
  become: yes
  vars:
    apache_version: 2.6
    motd_warning: "WARNING: used by ICT faculty/students only.\n"
    testserver: yes
  tasks:
    - name: install apache
      action: yum name=httpd state=installed
    
    - name: restart apache
      service: 
        name: httpd
        state: restarted

Sample Run:

[rchan@centos7 playbooks]$ ansible-playbook -i hosts httpd-play.yml

PLAY [vmlab] ********************************************************************

TASK [Gathering Facts] **********************************************************
ok: [vmlab]

TASK [install apache] ***********************************************************
changed: [vmlab]

TASK [restart apache] ***********************************************************
changed: [vmlab]

PLAY RECAP **********************************************************************
vmlab : ok=3  changed=2  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0

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

update Apache (httpd) installed in the Investigation 2 - Part 2
install extra packages repository for enterprise Linux (EPEL) if it is not already installed
remove 'tree' package
set the hostname to your Seneca username (Seneca ID)
create a new user with your Seneca_ID with sudo access
configure the new user account you created above so that you can ssh to it without password
setup a directory structure using a loop for completing and organizing labs as shown below:
      /home/[seneca_id]/ops445/lab1
      /home/[seneca_id]/ops445/lab2
      /home/[seneca_id]/ops445/lab3
      /home/[seneca_id]/ops445/lab4
      /home/[seneca_id]/ops445/lab5
      /home/[seneca_id]/ops445/lab6
      /home/[seneca_id]/ops445/lab7
      /home/[seneca_id]/ops445/lab8

when it's ready, run your playbook
in order to test it, log into the VM with the newly created user (your Seneca_ID), install the 'tree' package with sudo, and check the directory structure with the 'tree' command
if everything is correct, capture its output for a successful run of your playbook to a file named "lab8_[seneca_id].txt"
Lab 8 Sign-off (Show Instructor)
Have the following items ready to show your instructor:
The updated inventory file called "hosts" which you used to access your VM.
The Ansible playbook called "config_ops445.yml" for configuring the VM.
The result of running the playbook "config_ops445.yml". Save the result in a file called "lab8_[seneca_id].txt"
Upload the following files to blackboard
hosts
config_ops445.yml
lab8_[seneca_id].txt
