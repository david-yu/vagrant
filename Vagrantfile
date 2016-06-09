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

  # UCP node for UCP/DTR setup
  config.vm.define "ucp-node" do |ucp_node|
    ucp_node.vm.box = "ubuntu/trusty64"
    ucp_node.vm.network "private_network", type: "dhcp"
    ucp_node.vm.hostname = "ucp-node"
    config.vm.provider :virtualbox do |vb|
       vb.customize ["modifyvm", :id, "--memory", "4096"]
       vb.customize ["modifyvm", :id, "--cpus", "2"]
    end
    ucp_node.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
     sudo apt-get update && sudo apt-get install -y apt-transport-https
     sudo apt-get install -y linux-image-extra-virtual
     echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
     sudo apt-get update && sudo apt-get -y install docker-engine
     sudo usermod -a -G docker vagrant
     ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/ucp-ipaddr
     #wget https://dl.eff.org/certbot-auto
     #chmod a+x certbot-auto
     # Install UCP with license key
     docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock -v /vagrant/docker_subscription.lic:/docker_subscription.lic docker/ucp install --host-address $(cat "/vagrant/ucp-ipaddr") --admin-password admin
     # Retrieve UCP fingerprint
     docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp fingerprint | awk -F "=" '/SHA-256 Fingerprint/ {print $2}'  > /vagrant/ucp-fingerprint
   SHELL
  end

  # DTR node for UCP/DTR setup
  config.vm.define "dtr-node" do |dtr_node|
     dtr_node.vm.box = "ubuntu/trusty64"
     dtr_node.vm.network "private_network", type: "dhcp"
     dtr_node.vm.hostname = "dtr-node"
     config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "4096"]
        vb.customize ["modifyvm", :id, "--cpus", "2"]
     end
     dtr_node.vm.provision "shell", inline: <<-SHELL
      curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
      sudo apt-get update && sudo apt-get -y install apt-transport-https
      sudo apt-get install -y linux-image-extra-virtual
      echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
      sudo apt-get update && sudo apt-get -y install docker-engine
      sudo usermod -a -G docker vagrant
      ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/dtr-ipaddr
      export UCP_IPADDR=$(cat /vagrant/ucp-ipaddr)
      export DTR_IPADDR=$(cat /vagrant/dtr-ipaddr)
      export UCP_FINGERPRINT=$(cat /vagrant/ucp-fingerprint)
      curl -k "https://${UCP_IPADDR}/ca" > /vagrant/ucp-ca.pem
      # Join UCP Swarm
      docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp join --admin-username admin --admin-password admin --url https://${UCP_IPADDR} --host-address ${DTR_IPADDR} --fingerprint ${UCP_FINGERPRINT}
      # Install DTR
      docker run --rm docker/dtr install --ucp-url https://${UCP_IPADDR} --ucp-node dtr-node --dtr-external-url ${DTR_IPADDR} --ucp-username admin --ucp-password admin --ucp-ca "$(cat /vagrant/ucp-ca.pem)"
      # Configure DTR to trust UCP
      export DTR_CONFIG_DATA="{\"authBypassCA\":\"$(cat /vagrant/ucp-ca.pem | sed ':begin;$!N;s|\n|\\n|;tbegin')\"}"
      curl -u admin:admin -k  -H "Content-Type: application/json" https://${DTR_IPADDR}/api/v0/meta/settings -X POST --data-binary "${DTR_CONFIG_DATA}"
      # Configure UCP to use DTR
      sudo apt-get -y install jq
      export UCP_AUTH_TOKEN=$(curl -sk -d '{"username":"admin","password":"admin"}' https://${UCP_IPADDR}/auth/login | jq -r .auth_token)
      export UCP_CONFIG_DATA="{\"url\":\"https://${DTR_IPADDR}\", \"insecure\":true }"
      curl -k -s -c jar -H "Authorization: Bearer ${UCP_AUTH_TOKEN}" https://${UCP_IPADDR}/api/config/registry -X POST --data "${UCP_CONFIG_DATA}"
    SHELL
  end

  # Jenkins node for UCP/DTR setup - still need to setup project and install plugins
  config.vm.define "jenkins-node" do |jenkins|
    jenkins.vm.box = "ubuntu/trusty64"
    jenkins.vm.network "private_network", type: "dhcp"
    jenkins.vm.hostname = "jenkins-node"
    config.vm.provider :virtualbox do |vb|
       vb.customize ["modifyvm", :id, "--memory", "4096"]
       vb.customize ["modifyvm", :id, "--cpus", "2"]
    end
    jenkins.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y apt-transport-https ca-certificates
      sudo apt-get install -y default-jre default-jdk daemon
      curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
      sudo apt-get update && sudo apt-get install -y apt-transport-https
      sudo apt-get install -y linux-image-extra-virtual
      echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
      sudo apt-get update && sudo apt-get -y install docker-engine
      sudo usermod -a -G docker vagrant
      wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
      echo "deb http://pkg.jenkins-ci.org/debian binary/" | sudo tee /etc/apt/sources.list
      sudo apt-get update
      sudo apt-get -y install jenkins
      ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/jenkins-ipaddr
      export UCP_IPADDR=$(cat /vagrant/ucp-ipaddr)
      export UCP_FINGERPRINT=$(cat /vagrant/ucp-fingerprint)
      export JENKINS_IPADDR=$(cat /vagrant/jenkins-ipaddr)
      docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp join --admin-username admin --admin-password admin --host-address ${JENKINS_IPADDR} --url https://${UCP_IPADDR} --fingerprint ${UCP_FINGERPRINT}
      sudo cp -r /vagrant/jenkins /home/vagrant
      export JENKINS_HOME=/home/vagrant/jenkins
      sudo service jenkins restart
   SHELL
  end

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

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

end
