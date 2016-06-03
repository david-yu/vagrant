# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.

  config.vm.provider "virtualbox" do |v|
    v.memory = 2048
    v.cpus = 2
  end

  config.vm.define "master" do |master|
    master.vm.box = "ubuntu/trusty64"
    master.vm.network "private_network", type: "dhcp"
    master.vm.hostname = "master_node"
    master.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
     sudo echo "deb https://apt.dockerproject.org/repo ubuntu-trusty main" >> /etc/apt/sources.list.d/docker.list
     sudo apt-get update
     sudo apt-get purge lxc-docker
     sudo apt-get update
     sudo apt-get install -y linux-image-extra-$(uname -r)
     sudo apt-get install -y apparmor
     sudo apt-get install -y docker-engine
   SHELL
  end

  config.vm.define "node1" do |node1|
    node1.vm.box = "ubuntu/trusty64"
    node1.vm.network "private_network", type: "dhcp"
    node1.vm.hostname = "node1"
    node1.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
     sudo echo "deb https://apt.dockerproject.org/repo ubuntu-trusty main" >> /etc/apt/sources.list.d/docker.list
     sudo apt-get update
     sudo apt-get purge lxc-docker
     sudo apt-get update
     sudo apt-get install -y linux-image-extra-$(uname -r)
     sudo apt-get install -y apparmor
     sudo apt-get install -y docker-engine
   SHELL

  end

  config.vm.define "node2" do |node2|
    node2.vm.box = "ubuntu/trusty64"
    node2.vm.network "private_network", type: "dhcp"
    node2.vm.hostname = "node2"
    node2.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
     sudo echo "deb https://apt.dockerproject.org/repo ubuntu-trusty main" >> /etc/apt/sources.list.d/docker.list
     sudo apt-get update
     sudo apt-get purge lxc-docker
     sudo apt-get update
     sudo apt-get install -y linux-image-extra-$(uname -r)
     sudo apt-get install -y apparmor
     sudo apt-get install -y docker-engine
   SHELL

  end

  config.vm.define "ucp-node" do |ucp_node|
    ucp_node.vm.box = "ubuntu/trusty64"
    ucp_node.vm.network "private_network", type: "dhcp"
    ucp_node.vm.hostname = "ucp-node"
    ucp_node.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
     sudo echo "deb https://apt.dockerproject.org/repo ubuntu-trusty main" >> /etc/apt/sources.list.d/docker.list
     sudo apt-get update
     sudo apt-get purge lxc-docker
     sudo apt-get update
     sudo apt-get install -y linux-image-extra-$(uname -r)
     sudo apt-get install -y apparmor
     sudo apt-get install -y docker-engine
   SHELL
 end
   config.vm.define "dtr-node" do |dtr_node|
     dtr_node.vm.box = "ubuntu/trusty64"
     dtr_node.vm.network "private_network", type: "dhcp"
     dtr_node.vm.hostname = "dtr-node"
     dtr_node.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y apt-transport-https ca-certificates
      sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
      sudo echo "deb https://apt.dockerproject.org/repo ubuntu-trusty main" >> /etc/apt/sources.list.d/docker.list
      sudo apt-get update
      sudo apt-get purge lxc-docker
      sudo apt-get update
      sudo apt-get install -y linux-image-extra-$(uname -r)
      sudo apt-get install -y apparmor
      sudo apt-get install -y docker-engine
    SHELL
  end

  #config.vm.box = "ubuntu/trusty64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
end
