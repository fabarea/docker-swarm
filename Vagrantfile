# -*- mode: ruby -*-
# vi: set ft=ruby :

$install_docker_script = <<SCRIPT

  # install
  sudo apk add --update docker openrc

  # Add the user 'vagrant' to the 'docker' group
  sudo adduser vagrant docker

  # to start the Docker daemon at boot, run rc-update add docker boot.
  rc-update add docker boot

  # start the Docker daemon if not yet running
  if ! rc-service docker status >/dev/null 2>&1; then
    rc-service docker start
  fi

SCRIPT

$manager_script = <<SCRIPT
  echo Swarm Init...
  sudo docker swarm init --listen-addr 10.100.199.200:2377 --advertise-addr 10.100.199.200:2377
  sudo docker swarm join-token --quiet worker > /vagrant/worker_token
SCRIPT

$worker_script = <<SCRIPT
  echo Swarm Join...
  sudo docker swarm join --token $(cat /vagrant/worker_token) 10.100.199.200:2377
SCRIPT

BOX_NAME = "generic/alpine319" #, "generic/ubuntu2204"
MEMORY = "512"
MANAGERS = 2
WORKERS = 2
CPUS = 2
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure("2") do |config|

  #config.vm.provider "libvirt"
  config.vm.provider "virtualbox"

  # Connexion
  config.ssh.username = 'vagrant'
  config.ssh.password = 'vagrant'
  #config.ssh.insert_key = false

  # Just to speed things up
  if Vagrant.has_plugin?("vagrant-vbguest")
      config.vbguest.auto_update = false
  end

  # plugin: hostmanager
  # must comes at the end, so that the network is up and running
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true

  # Fix the /etc/hosts file
  config.vm.provision :shell, inline: 'sed -i "/^127.0.1.1 $(hostname)/c\#127.0.1.1 $(hostname)" /etc/hosts'

  # Common setup
  config.vm.box = BOX_NAME
  config.vm.synced_folder ".", "/home/vagrant"
  config.vm.provision "shell",inline: $install_docker_script, privileged: true
  config.vm.provider "virtualbox" do |vb|
    vb.memory = MEMORY
    vb.cpus = CPUS
  end

  #     #Setup Manager Nodes
  (1..MANAGERS).each do |i|
    config.vm.define "manager#{i}" do |node|
      node.vm.hostname = "m#{i}"

      node.vm.network "private_network", type: "dhcp"
      # node.vm.network :private_network, ip: "10.100.199.10#{i}"
      if i == 1

        # Make the first manager node accessible on the lan
        node.vm.network :public_network, bridge: "wlp110s0"
        #node.vm.provision "shell", inline: $manager_script, privileged: true

        # For local development
        # node.vm.network :forwarded_port, guest: 8080, host: 8080
        # node.vm.network :forwarded_port, guest: 5000, host: 5000
        # node.vm.network :forwarded_port, guest: 9000, host: 9000
      end
    end
  end

  #Setup Woker Nodes
  (1..WORKERS).each do |i|
    config.vm.define "w#{i}" do |node|
      node.vm.network "private_network", type: "dhcp"
      # node.vm.network :private_network, ip: "10.100.199.20#{i}"
      #node.vm.provision "shell", inline: $manager_script, privileged: true
      node.vm.hostname = "w#{i}"
    end
  end

  # ###################################
  # Helper to resolve the ip address bound to eth1
  # ###################################
  config.hostmanager.ip_resolver = proc do |machine|
    result = ""
    machine.communicate.execute("ip address show eth1") do |type, data|
      result << data if type == :stdout
    end

    ip_match = /inet (\d+\.\d+\.\d+\.\d+)/.match(result)
    if ip_match
      ip_match[1]
      #resolved_ip
    else
      machine.ui.warn("Failed to resolve IP address for eth1.")
      nil
    end
  end

end