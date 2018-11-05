# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vars 
MANAGER_IP = "10.0.100.2"
BOX_IMAGE = "ubuntu/xenial64"
V_CPUS = 1
V_MEMORY = 512
WORKDER_NODES = 3
SUBNET = "10.0.100."
ANSIBLE_INVENTORY = "ansible/hosts"

Vagrant.configure("2") do |config|
  config.ssh.forward_agent = true

  # read ssh public key if exists
  if File.file?("#{Dir.home}/.ssh/id_rsa.pub")
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
  end

  #create ansible inventory
  Dir.mkdir(ANSIBLE_INVENTORY) unless Dir.exists?(ANSIBLE_INVENTORY)

  # configure manager
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
    
    # run shell provisioner to deploy public key to authorized keys
    manager.vm.provision "shell", run: "always", inline: <<-SHELL
      if [ -z "#{ssh_pub_key}"  ]; then
        echo "No public key found exiting."
        exit 0;
      else
        if grep -sq "#{ssh_pub_key}" /home/vagrant/.ssh/authorized_keys; then
          echo "SSH keys already provisioned."
          exit 0;
        else
          echo "Writing public key to authorized keys."
          echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
          echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
        fi
      fi
      apt-add-repository ppa:ansible/ansible
      apt-get update
      apt-get install -y software-properties-common ansible
      apt-get -y dist-upgrade    
    SHELL

    File.open("#{ANSIBLE_INVENTORY}/vagrant_manager" ,'w') do |f|
      f.write "manager ansible_connection=local\n"
    end
  end # Manager 

  (1..WORKDER_NODES).each do |i|
    config.vm.define "workernode-#{i}" do |node|
      node.vm.hostname = "workernode-#{i}"
      node.vm.box = BOX_IMAGE
      node.vm.network "private_network", ip: SUBNET+"#{i+2}" 
      
      node.vm.provider :virtualbox do |v|
        v.cpus = V_CPUS
        v.memory = V_MEMORY
      end
      
      # deploy public key
      node.vm.provision "shell", run: "always", inline: <<-SHELL
        if [ -z "#{ssh_pub_key}"  ]; then
          echo "No public key found exiting."
          exit 0;
        else
          if grep -sq "#{ssh_pub_key}" /home/vagrant/.ssh/authorized_keys; then
            echo "SSH keys already provisioned."
            exit 0;
          else
            echo "Writing public key to authorized keys."
            echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
            echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
          fi
        fi      
      SHELL
    end

    # write ansible inventory
    File.open("#{ANSIBLE_INVENTORY}/vagrant_node-#{i}" ,'w') do |f|
      f.write "workernode-#{i}      \
ansible_ssh_host=#{SUBNET}#{i+2} \
ansible_ssh_private_key_file=/vagrant/.vagrant/machines/workernode-#{i}/virtualbox/private_key \
ansible_python_interpreter=/usr/bin/python3\n"
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
