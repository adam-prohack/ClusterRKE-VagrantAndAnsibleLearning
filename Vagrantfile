Vagrant.require_version ">= 2.2"

NETWORK_NAME = "rke-cluster"
NETWORK_BRIDGE_INTERFACE = "eno1"
NETWORK_IP_BASE = "192.168.0"
MACHINE_HOSTNAME_PREFIX = "rke"
WORKER_NODES = 2

Vagrant.configure("2") do |config|
    config.vm.box = "debian/buster64"

    # Configure virtualbox provider
    config.vm.provider "virtualbox" do |provider|
        provider.gui = false
        provider.linked_clone = true

        provider.customize ["modifyvm", :id, "--ostype", "Linux24_64"]
        provider.customize ["modifyvm", :id, "--ioapic", "on"]
        provider.customize ["modifyvm", :id, "--acpi", "on"]
        provider.customize ["modifyvm", :id, "--pae", "on"]
        provider.customize ["modifyvm", :id, "--chipset", "ich9"]
        provider.customize ["modifyvm", :id, "--graphicscontroller", "vboxvga"]
        provider.customize ["modifyvm", :id, "--accelerate3d", "off"]
        provider.customize ["modifyvm", :id, "--accelerate2dvideo", "off"]
    end

    # Configure vagrant
    config.vm.synced_folder '.', '/vagrant', disabled: true

    # Configure ssh keys
    config.ssh.private_key_path = ["keys/id_rsa", "~/.vagrant.d/insecure_private_key"]
    config.ssh.insert_key = false
    config.vm.provision "file", source: "keys/id_rsa.pub", destination: "~/.ssh/authorized_keys"
    config.vm.provision "file", source: "keys/id_rsa.pub", destination: "~/.ssh/id_rsa.pub"
    config.vm.provision "file", source: "keys/id_rsa", destination: "~/.ssh/id_rsa"

    # Configure workers
    (1..WORKER_NODES).each do |node_id|
        config.vm.define "node-#{node_id}" do |worker_node|
            worker_node.vm.provider "virtualbox" do |provider|
                provider.memory = 1024 * 4
                provider.cpus = 2
            end
            worker_node.vm.network "public_network", ip: "#{NETWORK_IP_BASE}.#{200+node_id}", bridge: "#{NETWORK_BRIDGE_INTERFACE}"
            worker_node.vm.network "forwarded_port", guest: 22, host: 3100+node_id, id: 'ssh'
            worker_node.vm.hostname = "#{MACHINE_HOSTNAME_PREFIX}-node-#{node_id}"
        end
    end

    # Configure master
    config.vm.define "master" do |master_node|
        master_node.vm.provider "virtualbox" do |provider|
            provider.memory = 1024 * 5
            provider.cpus = 2
        end
        master_node.vm.network "public_network", ip: "#{NETWORK_IP_BASE}.200", bridge: "#{NETWORK_BRIDGE_INTERFACE}"
        master_node.vm.network "forwarded_port", guest: 22, host: 3100, id: 'ssh'
        master_node.vm.hostname = "#{MACHINE_HOSTNAME_PREFIX}-master"
    end

    # Provision ansible
    config.vm.provision "ansible" do |ansible|
        ansible.verbose = "v"
        ansible.inventory_path = "./ansible/hosts"
        ansible.playbook = "./ansible/site.yml"
    end
end