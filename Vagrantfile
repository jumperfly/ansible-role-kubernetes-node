# -*- mode: ruby -*-
# vi: set ft=ruby :

unless Vagrant.has_plugin?("vagrant-hostmanager")
  raise 'vagrant-hostmanager plugin is not installed!'
end

$node_count = 2
$default_memory = 384
$master_memory = 1280
$ip_prefix = "192.168.99"

node_memory = $default_memory
host_vars = {}
kube_master_nodes = []
kube_nodes = []

Vagrant.configure("2") do |config|
  config.vm.box = "jumperfly/centos-7"
  config.vm.box_version = "1804.02.01"
  config.vm.box_check_update = false
  config.ssh.insert_key = false
  config.hostmanager.include_offline = true
  config.vm.provision :hostmanager

  (1..$node_count).each do |i|
    config.vm.define vm_name = "node#{i}" do |config|
      config.vm.provision "shell", run: "always", inline: <<-SHELL
        swapoff -a
        sed -i'' '/^127.0.0.1\\t#{vm_name}\\t#{vm_name}$/d' /etc/hosts
      SHELL
      config.vm.hostname = vm_name
      ip = "#{$ip_prefix}.10#{i}"
      config.vm.network "private_network", ip: ip
      host_vars[vm_name] = {
        "ip": ip,
        "kube_iface": "eth1",
        "kube_node_cidr_range": "10.201.#{i}.1/24",
        "ca_host": "node1"
      }

      if i == 1
        kube_master_nodes << vm_name
        config.hostmanager.aliases = %w(kubernetes)
        node_memory = $master_memory
      else
        kube_nodes << vm_name
      end

      config.vm.provider "virtualbox" do |v|
        v.memory = node_memory
        v.cpus = 2
      end

      config.vm.provision "shell", inline: <<-SHELL
        cp /vagrant/vagrant_insecure_key /home/vagrant/.ssh/id_rsa
        chmod 600 /home/vagrant/.ssh/id_rsa
        chown vagrant:vagrant /home/vagrant/.ssh/id_rsa
      SHELL
      if i == $node_count
        config.vm.box = "jumperfly/centos-7-ansible"
        config.vm.box_version = "1804.02.01"
        config.vm.provision "shell", inline: <<-SHELL
          mkdir -p /etc/ansible/roles
          if [ -e /vagrant/ansible-role-kubernetes-master ]; then
            ln -snf /vagrant/ansible-role-kubernetes-master /etc/ansible/roles/jumperfly.kubernetes_master
          fi
          ln -snf /vagrant/ /etc/ansible/roles/jumperfly.kubernetes_node
          ansible-galaxy install --ignore-errors -r /vagrant/tests/requirements.yml -p /etc/ansible/roles
        SHELL
        config.vm.provision "ansible_local" do |ansible|
          ansible.compatibility_mode = "2.0"
          ansible.limit = "all"
          ansible.playbook = "tests/test.yml"
          ansible.host_vars = host_vars
          ansible.groups = {
            "root_ca_nodes": [ "node1" ],
            "intermediate_ca_nodes": [ "node1" ],
            "kube_master_nodes": kube_master_nodes,
            "etcd_nodes": kube_master_nodes,
            "kube_nodes": kube_nodes
          }
        end
      end
    end
  end

end
