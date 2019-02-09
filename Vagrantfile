# -*- mode: ruby -*-
NUMBER_OF_WEBSERVERS = 2
VM_VERSION= "ubuntu/trusty64"
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  groups = {
    "webservers" => ["10.0.15.2[1:#{NUMBER_OF_WEBSERVERS}]"],
    "loadbalancers" => ["load_balancer"],
    "all_groups:children" => ["webservers","loadbalancers"]
  }

  # create web servers
  (1..NUMBER_OF_WEBSERVERS).each do |i|
    config.vm.define "web#{i}" do |node|
        node.vm.box = VM_VERSION
        node.vm.hostname = "web#{i}"
		node.vm.synced_folder ".", "/vagrant"
        node.vm.network :private_network, ip: "10.0.15.2#{i}"
        node.vm.network "forwarded_port", guest: 80, host: "880#{i}"	
  # Execute on last web server after all web servers are up
     if i == NUMBER_OF_WEBSERVERS
	# setting host_key_checking = False in ansible.cfg didn't work, so I had to provision it manually.
		node.vm.provision "shell", inline: <<-SHELL
			sudo sed -i -e 's/#   StrictHostKeyChecking ask/  StrictHostKeyChecking no/g' /etc/ssh/ssh_config
		SHELL
	# ansible local provisioning
		node.vm.provision "ansible_local" do |ansible|
           ansible.playbook = "pb_web.yml"
           ansible.become = true
           ansible.limit = "all"
           ansible.inventory_path = "/vagrant/inventory"
		   ansible.groups = groups
         end
       end
      end
    end


    # create load balancer
	config.vm.define "load_balancer" do |lb_config|
        lb_config.vm.box = VM_VERSION
        lb_config.vm.hostname = "lb"
		lb_config.vm.synced_folder ".", "/vagrant"
        lb_config.vm.network :private_network, ip: "10.0.15.20"
        lb_config.vm.network "forwarded_port", guest: 80, host: 8800
		lb_config.ssh.password = "vagrant"
        lb_config.vm.provision "ansible_local" do |ansible|
          ansible.playbook = "pb_lb.yml"
		  ansible.become = true
		  ansible.limit = "all"
		  ansible.groups = groups
        end
    end
end
