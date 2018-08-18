# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/xenial64"

  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo apt-get -y install language-pack-en
    sudo apt-get -y install python-minimal
  SHELL

  N = 2
  
  (1..N).each do |machine_id|
    config.vm.define "app.#{machine_id}" do |machine|
      machine.vm.network "private_network", ip: "192.168.77.20#{machine_id}"

      if machine_id == N
        machine.vm.provision "ansible" do |ansible|
          ansible.groups = {
            "apps" => ["app.[1:#{N}]"],
          }
          ansible.limit = "apps"
          ansible.become = true
          ansible.playbook = "app.yml"
          ansible.galaxy_role_file = "requirements.yml"
          ansible.galaxy_roles_path = "roles_vendor"
          ansible.galaxy_command = "ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path} --force"
        end
      end
    end
  end

  config.vm.define "lb" do |machine|
    machine.vm.network "forwarded_port", guest: 80, host: 8080
    machine.vm.network "private_network", ip: "192.168.77.200"
    machine.vm.provision "ansible" do |ansible|
      ansible.limit = "lb"
      ansible.become = true
      ansible.playbook = "lb.yml"
      ansible.tags = "haproxy_install"
      ansible.galaxy_role_file = "requirements.yml"
      ansible.galaxy_roles_path = "roles_vendor"
      ansible.galaxy_command = "ansible-galaxy install --role-file=%{role_file} --roles-path=%{roles_path} --force"
    end

    # The reason for an additional Ansible provisable is so that it can gather facts from all hosts
    # Required to configure HAProxy with multiple apps
    machine.vm.provision "ansible" do |ansible|
      # Inventory file is overwritten upon each Ansible provision - Need to add groups again..
      ansible.groups = {
        "apps" => ["app.[1:#{N}]"],
      }
      ansible.limit = "all"
      ansible.become = true
      ansible.playbook = "lb.yml"
      ansible.tags = "haproxy_configure"
      ansible.galaxy_role_file = "requirements.yml"
    end
  end
end
