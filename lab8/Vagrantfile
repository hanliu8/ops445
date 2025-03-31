# Service configuration reference
SERVICES = {
  'labvm1' => {
    ip: '192.168.56.10'
  },
  'labvm2' => {
    ip: '192.168.56.11'
  },
  'control-center' => {
    ip: '192.168.56.12'
  }
}

Vagrant.configure("2") do |config|
  # Common configuration
  config.vm.box = "hashicorp-education/ubuntu-24-04"
  config.vm.box_version = "0.1.0"

  # Common provisioning script for all VMs
  config.vm.provision "shell", name: "common", path: "common-dependencies.sh"

  # labvm1 managed node
  config.vm.define "labvm1" do |labvm1|
    labvm1.vm.hostname = "labvm1"
    labvm1.vm.network "private_network", ip: SERVICES['labvm1'][:ip]
    labvm1.vm.synced_folder "./share", "/home/vagrant/share", create: true

  end

  # labvm2 managed node
  config.vm.define "labvm2" do |labvm2|
    labvm2.vm.hostname = "labvm2"
    labvm2.vm.network "private_network", ip: SERVICES['labvm2'][:ip]
    labvm2.vm.synced_folder "./share", "/home/vagrant/share", create: true

  end

  # control-center control node
  config.vm.define "control-center" do |control|
    control.vm.hostname = "control-center"
    control.vm.network "private_network", ip: SERVICES['control-center'][:ip]
    control.vm.synced_folder "./share", "/home/vagrant/share", create: true

    control.vm.provision "shell", name: "Install Ansible", inline: <<-SHELL
      apt update
      apt install -y software-properties-common
      apt-add-repository --yes --update ppa:ansible/ansible
      apt install ansible -y

      # Get labvm IP dynamically (with 1 minute timeout)
      for i in {1..30}; do
        if LABVM1_IP=$(getent hosts labvm1.local | awk '{print $1}'); then
          break
        fi

        if LABVM2_IP=$(getent hosts labvm2.local | awk '{print $1}'); then
          break
        fi

        echo "Waiting for labvm1.local to be resolvable..."
        sleep 2
      done
    SHELL

  end
end
