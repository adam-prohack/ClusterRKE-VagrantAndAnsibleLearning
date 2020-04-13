Vagrant.require_version ">= 2.2"

NETWORK_NAME = "rke-cluster"
NETWORK_IP_BASE = "10.101"
MACHINE_HOSTNAME_PREFIX = "rke"
WORKER_NODES = 3

Vagrant.configure("2") do |config|
    config.vm.box = "ubuntu/focal64"

    # Configure vagrant
    config.vm.synced_folder '.', '/vagrant', disabled: true
    config.vagrant.plugins = []

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
                provider.gui = false
                provider.memory = 1024 * 3
                provider.cpus = 1
            end
            worker_node.vm.network "private_network", ip: "#{NETWORK_IP_BASE}.1.#{100+node_id}", virtualbox__intnet: NETWORK_NAME
            worker_node.vm.network "forwarded_port", guest: 22, host: 3100+node_id, id: 'ssh'
            worker_node.vm.hostname = "#{MACHINE_HOSTNAME_PREFIX}-node-#{node_id}"
        end
    end

    # Configure master
    config.vm.define "master" do |master_node|
        master_node.vm.provider "virtualbox" do |provider|
            provider.gui = false
            provider.memory = 1024 * 4
            provider.cpus = 1
        end
        master_node.vm.network "private_network", ip: "#{NETWORK_IP_BASE}.1.100", virtualbox__intnet: NETWORK_NAME
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