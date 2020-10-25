# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  #config.vm.box = "jumperfly/kube-master-etcd-1.13"
  config.vm.box = "file://output-kube-master-etcd/package.box"
  #config.vm.box_version = "1.7"
  config.vm.box_check_update = false
  config.ssh.insert_key = false
  config.vm.provider "virtualbox" do |v|
    v.cpus = 1
    v.memory = 4096
  end

  config.vm.hostname = "worker1"

  config.vm.provision "ansible" do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.limit = "all"
    ansible.galaxy_role_file = "tests/requirements.yml"
    ansible.galaxy_command = "ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path}"
    ansible.playbook = "tests/test.yml"
    ansible.groups = {
      "etcd_nodes" => ["default"]
    }
  end
end
