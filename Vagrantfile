MANAGER_IP = "10.0.100.2"
BOX_IMAGE = "ubuntu/xenial64"
V_CPUS = 1
V_MEMORY = 512
WORKDER_NODES = 3
SUBNET = "10.0.100."

Vagrant.configure("2") do |config|
  config.ssh.forward_agent = true

  config.vm.define "manager", primary: true do |manager|
    manager.vm.hostname = "manager"
    manager.vm.box = BOX_IMAGE
    manager.vm.network "private_network", ip: MANAGER_IP

    manager.vm.provider :virtualbox do |v|
      v.cpus = V_CPUS
      v.memory = V_MEMORY
    end

    # Forward port 8080-8089
    for i in 8080..8089
      manager.vm.network :forwarded_port, guest: i, host: i
    end
  end

  (1..WORKDER_NODES).each do |i|
    config.vm.define "workernode-#{i}" do |node|
      node.vm.hostname = "workernode-#{i}"
      node.vm.box = BOX_IMAGE
      node.vm.network "private_network", ip: SUBNET+"#{i+2}" 
      
      node.vm.provider :virtualbox do |v|
        v.cpus = V_CPUS
        v.memory = V_MEMORY
      end
    end
  end


  # install docker and add user vagrant to group docker
  # config.vm.provision "shell", inline: <<-SHELL
  #    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
  #    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
  #    apt-get update
  #    apt-get -y dist-upgrade
  #    apt-get install -y \
  #       apt-transport-https \
  #       ca-certificates \
  #       software-properties-common \
  #       docker-ce
  #    usermod -aG docker vagrant
  # SHELL

  # # run test container
  # config.vm.provision "shell",run: "always", inline: <<-SHELL
  #   chmod a+x /vagrant/test_script.sh
  #   /vagrant/test_script.sh
  # SHELL
end
