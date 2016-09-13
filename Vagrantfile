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
       vb.customize ["modifyvm", :id, "--memory", "2560"]
       vb.customize ["modifyvm", :id, "--cpus", "2"]
       vb.name = "ucp-node"
    end
    ucp_node.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     sudo apt-get install -y linux-generic-lts-vivid
     curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
     sudo apt-get update && sudo apt-get install -y apt-transport-https
     echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
     sudo apt-get update && sudo apt-get -y install docker-engine
     sudo usermod -a -G docker vagrant
     ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/ucp-ipaddr
     # Install UCP with license key
     docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock -v /vagrant/docker_subscription.lic:/docker_subscription.lic docker/ucp install --host-address $(cat "/vagrant/ucp-ipaddr") --admin-password admin
     # Retrieve UCP fingerprint
     docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp fingerprint | awk -F "=" '/SHA-256 Fingerprint/ {print $2}'  > /vagrant/ucp-fingerprint
     # Retrieve UCP id
     docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp id > /vagrant/ucp-id
     # Backup UCP to get certificates and keys
     docker run --rm -i --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp backup --root-ca-only --id $(cat "/vagrant/ucp-id") --passphrase "secret" > /vagrant/backup.tar
     # Get UCP cluster CA
     docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp dump-certs --cluster --ca > /vagrant/ucp-cluster-ca.pem
     # Install script for self-signed certs onto UCP
     cp /vagrant/install-certs-ucp.sh .
     chmod +x install-certs-ucp.sh
   SHELL
  end

  # UCP node 2 (HA) for UCP/DTR setup
  config.vm.define "ucp-node2" do |ucp_node2|
    ucp_node2.vm.box = "ubuntu/trusty64"
    ucp_node2.vm.network "private_network", type: "dhcp"
    ucp_node2.vm.hostname = "ucp-node2"
    config.vm.provider :virtualbox do |vb|
       vb.customize ["modifyvm", :id, "--memory", "2560"]
       vb.customize ["modifyvm", :id, "--cpus", "2"]
       vb.name = "ucp-node2"
    end
    ucp_node2.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     sudo apt-get install -y linux-generic-lts-vivid
     curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
     sudo apt-get update && sudo apt-get install -y apt-transport-https
     echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
     sudo apt-get update && sudo apt-get -y install docker-engine
     sudo usermod -a -G docker vagrant
     ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/ucp2-ipaddr
     # Setup Controller
     export UCP_IPADDR=$(cat /vagrant/ucp-ipaddr)
     export UCP2_IPADDR=$(cat /vagrant/ucp2-ipaddr)
     export UCP_FINGERPRINT=$(cat /vagrant/ucp-fingerprint)
     export UCP_ADMIN_USER=admin
     export UCP_ADMIN_PASSWORD=admin
     docker run --rm -it --name ucp -v /var/run/docker.sock:/var/run/docker.sock -v /vagrant/backup.tar:/backup.tar docker/ucp join --admin-username admin --admin-password admin --url https://${UCP_IPADDR} --replica --fingerprint ${UCP_FINGERPRINT} --host-address ${UCP2_IPADDR} --passphrase "secret"
   SHELL
  end

  # UCP node 3 (HA) for UCP/DTR setup
  config.vm.define "ucp-node3" do |ucp_node3|
    ucp_node3.vm.box = "ubuntu/trusty64"
    ucp_node3.vm.network "private_network", type: "dhcp"
    ucp_node3.vm.hostname = "ucp-node3"
    config.vm.provider :virtualbox do |vb|
       vb.customize ["modifyvm", :id, "--memory", "2560"]
       vb.customize ["modifyvm", :id, "--cpus", "2"]
       vb.name = "ucp-node3"
    end
    ucp_node3.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     sudo apt-get install -y linux-generic-lts-vivid
     curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
     sudo apt-get update && sudo apt-get install -y apt-transport-https
     echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
     sudo apt-get update && sudo apt-get -y install docker-engine
     sudo usermod -a -G docker vagrant
     ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/ucp3-ipaddr
     # Setup Controller
     export UCP_IPADDR=$(cat /vagrant/ucp-ipaddr)
     export UCP3_IPADDR=$(cat /vagrant/ucp3-ipaddr)
     export UCP_FINGERPRINT=$(cat /vagrant/ucp-fingerprint)
     docker run --rm -it --name ucp -v /var/run/docker.sock:/var/run/docker.sock -v /vagrant/backup.tar:/backup.tar docker/ucp join --admin-username admin --admin-password admin --url https://${UCP_IPADDR} --replica --fingerprint ${UCP_FINGERPRINT} --host-address ${UCP3_IPADDR} --passphrase "secret"
   SHELL
  end

  # DTR node for UCP/DTR setup
  config.vm.define "dtr-node" do |dtr_node|
     dtr_node.vm.box = "ubuntu/trusty64"
     dtr_node.vm.network "private_network", type: "dhcp"
     dtr_node.vm.hostname = "dtr-node"
     config.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "2560"]
        vb.customize ["modifyvm", :id, "--cpus", "2"]
        vb.name = "dtr-node"
     end
     dtr_node.vm.provision "shell", inline: <<-SHELL
      curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
      sudo apt-get update && sudo apt-get -y install apt-transport-https
      sudo apt-get install -y linux-generic-lts-vivid
      echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
      sudo apt-get update && sudo apt-get -y install docker-engine
      sudo usermod -a -G docker vagrant
      # Join UCP Swarm
      ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/dtr-ipaddr
      export UCP_IPADDR=$(cat /vagrant/ucp-ipaddr)
      export DTR_IPADDR=$(cat /vagrant/dtr-ipaddr)
      export UCP_FINGERPRINT=$(cat /vagrant/ucp-fingerprint)
      # Install registry certificates on client Docker daemon
      export DTR_IPADDR=$(cat /vagrant/dtr-ipaddr)
      openssl s_client -connect ${DTR_IPADDR}:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM | sudo tee /usr/local/share/ca-certificates/${DTR_IPADDR}.crt
      sudo update-ca-certificates
      sudo service docker restart
      curl -k "https://${UCP_IPADDR}/ca" > /vagrant/ucp-ca.pem
      docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp join --admin-username admin --admin-password admin --url https://${UCP_IPADDR} --host-address ${DTR_IPADDR} --fingerprint ${UCP_FINGERPRINT}
      # Install DTR
      docker run --rm docker/dtr install --ucp-url https://${UCP_IPADDR} --ucp-node dtr-node --dtr-external-url ${DTR_IPADDR} --ucp-username admin --ucp-password admin --ucp-ca "$(cat /vagrant/ucp-ca.pem)"
      # Configure DTR to trust UCP, excepted from https://raw.githubusercontent.com/alexmavr/ddc-eval/master/ddc_evaluation.sh
      echo -e "Configuring DTR to trust UCP"
      export DTR_CONFIG_DATA="{\"authBypassCA\":\"$(cat /vagrant/ucp-cluster-ca.pem | awk 'BEGIN{ORS="\\n"}1')\"}"
      echo -e "DTR_CONFIG_DATA = ${DTR_CONFIG_DATA}"
      curl -u admin:admin -k  -H "Content-Type: application/json" https://${DTR_IPADDR}/api/v0/meta/settings -X POST --data-binary "${DTR_CONFIG_DATA}"
      # Configure UCP to use DTR
      echo -e "Configuring DTR to use UCP"
      sudo apt-get -y install jq
      export UCP_AUTH_TOKEN=$(curl -k -c jar https://${UCP_IPADDR}/auth/login -d '{"username":"admin","password":"admin"}' -X POST | jq -r ".auth_token")
      export UCP_CONFIG_DATA="{\"url\":\"https://${DTR_IPADDR}\", \"insecure\":true }"
      curl -k -s -c jar -H "Authorization: Bearer ${UCP_AUTH_TOKEN}" https://${UCP_IPADDR}/api/config/registry -X POST --data "${UCP_CONFIG_DATA}"
      # Install Docker Compose
      sudo bash -c 'curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose'
      sudo chmod +x /usr/local/bin/docker-compose
      # Retrive Docker notary and bring up using Docker Compose
      git clone https://github.com/docker/notary.git
      cd /home/vagrant/notary
      docker-compose up -d
      # Setup DTR to use Notary
      curl -sk -o /dev/null -w "%{http_code}" -X POST --header "Content-Type: application/json" --header "Accept: application/json" -u admin:admin -d "{\"notaryServer\": \"https://${DTR_IPADDR}:4443\",\"notaryVerifyCert\": false}" "https://${DTR_IPADDR}/api/v0/meta/settings"
    SHELL
  end

  # Jenkins node for UCP/DTR setup - still need to setup project and install plugins
  config.vm.define "jenkins-node" do |jenkins|
    jenkins.vm.box = "ubuntu/trusty64"
    jenkins.vm.network "private_network", type: "dhcp"
    jenkins.vm.hostname = "jenkins-node"
    config.vm.provider :virtualbox do |vb|
       vb.customize ["modifyvm", :id, "--memory", "2560"]
       vb.customize ["modifyvm", :id, "--cpus", "2"]
       vb.name = "jenkins-node"
    end
    jenkins.vm.provision "shell", inline: <<-SHELL
      echo "deb http://zw.archive.ubuntu.com/ubuntu/ precise main restricted universe multiverse" | sudo tee -a /etc/apt/sources.list
      sudo apt-get update
      sudo apt-get install -y apt-transport-https ca-certificates
      sudo apt-get install -y linux-generic-lts-vivid
      sudo apt-get install -y default-jre default-jdk daemon curl jq unzip
      # Install Docker daemon
      curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
      sudo apt-get update && sudo apt-get install -y apt-transport-https
      echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee -a /etc/apt/sources.list.d/docker.list
      sudo apt-get update && sudo apt-get -y install docker-engine
      sudo usermod -a -G docker vagrant
      # Install registry certificates on client Docker daemon
      export DTR_IPADDR=$(cat /vagrant/dtr-ipaddr)
      openssl s_client -connect ${DTR_IPADDR}:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM | sudo tee /usr/local/share/ca-certificates/${DTR_IPADDR}.crt
      sudo update-ca-certificates
      # Install Jenkins
      wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
      echo "deb http://pkg.jenkins-ci.org/debian binary/" | sudo tee /etc/apt/sources.list
      sudo apt-get update
      sudo apt-get -y install jenkins
      sudo usermod -a -G docker jenkins
      sudo service docker restart
      # Set up new Jenkins home and copy configs over to home directory
      sudo cp -r /vagrant/jenkins ~/
      export JENKINS_HOME=~/jenkins/
      sudo service jenkins restart
      # Install Docker Compose
      sudo bash -c 'curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose'
      sudo chmod +x /usr/local/bin/docker-compose
      sudo -u jenkins docker login -u admin -p admin https://${DTR_IPADDR}
      # Join UCP Swarm
      ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/jenkins-ipaddr
      export UCP_IPADDR=$(cat /vagrant/ucp-ipaddr)
      export UCP_FINGERPRINT=$(cat /vagrant/ucp-fingerprint)
      export JENKINS_IPADDR=$(cat /vagrant/jenkins-ipaddr)
      docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp join --admin-username admin --admin-password admin --host-address ${JENKINS_IPADDR} --url https://${UCP_IPADDR} --fingerprint ${UCP_FINGERPRINT}
      # Get client bundle from UCP
      export AUTHTOKEN=$(curl -sk -d '{"username":"admin","password":"admin"}' https://${UCP_IPADDR}/auth/login | jq -r .auth_token)
      sudo mkdir ucp-bundle
      curl -k -H "Authorization: Bearer ${AUTHTOKEN}" https://${UCP_IPADDR}/api/clientbundle -o ucp-bundle/bundle.zip
      sudo cd /home/vagrant/ucp-bundle
      sudo unzip bundle.zip
      sudo chmod +x /home/vagrant/ucp-bundle/env.sh
      sudo chown -R jenkins /home/vagrant/ucp-bundle
      sudo chgrp -R jenkins /home/vagrant/ucp-bundle
   SHELL
  end

  # Swarm child practice node
 config.vm.define "app-node1" do |node1|
   node1.vm.box = "ubuntu/trusty64"
   node1.vm.network "private_network", type: "dhcp"
   node1.vm.hostname = "app-node1"
   config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024"]
      vb.customize ["modifyvm", :id, "--cpus", "2"]
      vb.name = "app-node1"
   end
   node1.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
     sudo apt-get update && sudo apt-get install -y apt-transport-https
     sudo apt-get install -y linux-image-extra-virtual
     sudo apt-get install -y linux-generic-lts-vivid
     echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
     sudo apt-get update && sudo apt-get -y install docker-engine
     sudo usermod -a -G docker vagrant
     # Join UCP Swarm
     ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/appnode1-ipaddr
     export UCP_IPADDR=$(cat /vagrant/ucp-ipaddr)
     export UCP_FINGERPRINT=$(cat /vagrant/ucp-fingerprint)
     export APPNODE1_IPADDR=$(cat /vagrant/appnode1-ipaddr)
     docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp join --admin-username admin --admin-password admin --host-address ${APPNODE1_IPADDR} --url https://${UCP_IPADDR} --fingerprint ${UCP_FINGERPRINT}
     echo 'DOCKER_OPTS="--label nodeType=app"' | sudo tee -a /etc/default/docker
     # Install registry certificates on client Docker daemon
     # https://docs.docker.com/docker-trusted-registry/configure/config-security/
     export DTR_IPADDR=$(cat /vagrant/dtr-ipaddr)
     openssl s_client -connect ${DTR_IPADDR}:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM | sudo tee /usr/local/share/ca-certificates/${DTR_IPADDR}.crt
     sudo update-ca-certificates
     sudo service docker restart
     # Install Docker Compose
     sudo bash -c 'curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose'
     sudo chmod +x /usr/local/bin/docker-compose
    SHELL
  end

  # Swarm child practice node
 config.vm.define "app-node2" do |node2|
   node2.vm.box = "ubuntu/trusty64"
   node2.vm.network "private_network", type: "dhcp"
   node2.vm.hostname = "app-node2"
   config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024"]
      vb.customize ["modifyvm", :id, "--cpus", "2"]
      vb.name = "app-node2"
   end
   node2.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
     sudo apt-get update && sudo apt-get install -y apt-transport-https
     sudo apt-get install -y linux-image-extra-virtual
     sudo apt-get install -y linux-generic-lts-vivid
     echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
     sudo apt-get update && sudo apt-get -y install docker-engine
     sudo usermod -a -G docker vagrant
     # Join UCP Swarm
     ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/appnode2-ipaddr
     export UCP_IPADDR=$(cat /vagrant/ucp-ipaddr)
     export UCP_FINGERPRINT=$(cat /vagrant/ucp-fingerprint)
     export APPNODE2_IPADDR=$(cat /vagrant/appnode2-ipaddr)
     docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp join --admin-username admin --admin-password admin --host-address ${APPNODE2_IPADDR} --url https://${UCP_IPADDR} --fingerprint ${UCP_FINGERPRINT}
     echo 'DOCKER_OPTS="--label nodeType=app"' | sudo tee -a /etc/default/docker
     # Install registry certificates on client Docker daemon
     export DTR_IPADDR=$(cat /vagrant/dtr-ipaddr)
     openssl s_client -connect ${DTR_IPADDR}:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM | sudo tee /usr/local/share/ca-certificates/${DTR_IPADDR}.crt
     sudo update-ca-certificates
     sudo service docker restart
     # Install Docker Compose
     sudo bash -c 'curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose'
     sudo chmod +x /usr/local/bin/docker-compose
    SHELL
  end

 # Swarm child practice node
 config.vm.define "app-node3" do |node3|
   node3.vm.box = "ubuntu/trusty64"
   node3.vm.network "private_network", type: "dhcp"
   node3.vm.hostname = "app-node3"
   config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "1024"]
      vb.customize ["modifyvm", :id, "--cpus", "2"]
      vb.name = "app-node3"
   end
   node3.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
     sudo apt-get update && sudo apt-get install -y apt-transport-https
     sudo apt-get install -y linux-image-extra-virtual
     sudo apt-get install -y linux-generic-lts-vivid
     echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
     sudo apt-get update && sudo apt-get -y install docker-engine
     sudo usermod -a -G docker vagrant
     # Join UCP Swarm
     ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/appnode3-ipaddr
     export UCP_IPADDR=$(cat /vagrant/ucp-ipaddr)
     export UCP_FINGERPRINT=$(cat /vagrant/ucp-fingerprint)
     export APPNODE3_IPADDR=$(cat /vagrant/appnode3-ipaddr)
     docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp join --admin-username admin --admin-password admin --host-address ${APPNODE3_IPADDR} --url https://${UCP_IPADDR} --fingerprint ${UCP_FINGERPRINT}
     echo 'DOCKER_OPTS="--label nodeType=app"' | sudo tee -a /etc/default/docker
     # Install registry certificates on client Docker daemon
     export DTR_IPADDR=$(cat /vagrant/dtr-ipaddr)
     openssl s_client -connect ${DTR_IPADDR}:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM | sudo tee /usr/local/share/ca-certificates/${DTR_IPADDR}.crt
     sudo update-ca-certificates
     sudo service docker restart
     # Install Docker Compose
     sudo bash -c 'curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose'
     sudo chmod +x /usr/local/bin/docker-compose
  SHELL
 end

 # Swarm lb practice node
 config.vm.define "lb-node" do |lbnode|
   lbnode.vm.box = "ubuntu/trusty64"
   lbnode.vm.network "private_network", type: "dhcp"
   lbnode.vm.hostname = "lb-node"
   config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "512"]
      vb.customize ["modifyvm", :id, "--cpus", "1"]
      vb.name = "lb-node"
   end
   lbnode.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
     sudo apt-get update && sudo apt-get install -y apt-transport-https
     sudo apt-get install -y linux-image-extra-virtual
     sudo apt-get install -y linux-generic-lts-vivid
     echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
     sudo apt-get update && sudo apt-get -y install docker-engine
     sudo usermod -a -G docker vagrant
     # Join UCP Swarm
     ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/lb-ipaddr
     export UCP_IPADDR=$(cat /vagrant/ucp-ipaddr)
     export UCP_FINGERPRINT=$(cat /vagrant/ucp-fingerprint)
     export LB_IPADDR=$(cat /vagrant/lb-ipaddr)
     docker run --rm --name ucp -v /var/run/docker.sock:/var/run/docker.sock docker/ucp join --admin-username admin --admin-password admin --host-address ${LB_IPADDR} --url https://${UCP_IPADDR} --fingerprint ${UCP_FINGERPRINT}
     echo 'DOCKER_OPTS="--label nodeType=lb"' | sudo tee -a /etc/default/docker
     # Install registry certificates on client Docker daemon
     export DTR_IPADDR=$(cat /vagrant/dtr-ipaddr)
     openssl s_client -connect ${DTR_IPADDR}:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM | sudo tee /usr/local/share/ca-certificates/${DTR_IPADDR}.crt
     sudo update-ca-certificates
     sudo service docker restart
     # Install Docker Compose
     sudo bash -c 'curl -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose'
     sudo chmod +x /usr/local/bin/docker-compose
     # Install lb software
     git clone https://github.com/yongshin/docker-lb.git
     cd /home/vagrant/docker-lb
     docker-compose up -d
  SHELL
 end

 config.vm.define "docker-client" do |docker_client|
   docker_client.vm.box = "ubuntu/trusty64"
   docker_client.vm.network "private_network", type: "dhcp"
   docker_client.vm.hostname = "docker-client"
   config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "512"]
      vb.customize ["modifyvm", :id, "--cpus", "1"]
      vb.name = "docker-client"
   end
   docker_client.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     curl -s 'https://sks-keyservers.net/pks/lookup?op=get&search=0xee6d536cf7dc86e2d7d56f59a178ac6c6238f52e' | sudo apt-key add --import
     sudo apt-get update && sudo apt-get install -y apt-transport-https
     sudo apt-get install -y linux-image-extra-virtual
     sudo apt-get install -y linux-generic-lts-vivid
     echo "deb https://packages.docker.com/1.11/apt/repo ubuntu-trusty main" | sudo tee /etc/apt/sources.list.d/docker.list
     sudo apt-get update && sudo apt-get -y install docker-engine
     # Install registry certificates on client Docker daemon
     export DTR_IPADDR=$(cat /vagrant/dtr-ipaddr)
     openssl s_client -connect ${DTR_IPADDR}:443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM | sudo tee /usr/local/share/ca-certificates/${DTR_IPADDR}.crt
     sudo update-ca-certificates
     sudo service docker restart
     sudo usermod -a -G docker vagrant
   SHELL
 end

 # Swarm child practice node
 config.vm.define "docker112-master" do |docker112master|
   docker112master.vm.box = "ubuntu/trusty64"
   docker112master.vm.network "private_network", type: "dhcp"
   docker112master.vm.hostname = "docker112-master"
   config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "512"]
      vb.customize ["modifyvm", :id, "--cpus", "1"]
      vb.name = "docker112-master"
   end
   docker112master.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     sudo apt-get update && sudo apt-get install -y apt-transport-https
     sudo apt-get install -y linux-image-extra-virtual
     sudo apt-get install -y linux-generic-lts-vivid
     curl -fsSL https://test.docker.com/ | sh
     sudo usermod -a -G docker vagrant
     ifconfig eth1 | grep 'inet addr:' | cut -d: -f2 | awk '{ print $1}' > /vagrant/docker112master-ipaddr
     export DOCKER_MASTER=$(cat /vagrant/docker112master-ipaddr)
     docker swarm init --listen-addr ${DOCKER_MASTER}:2377
     # install compose
     sudo bash -c 'curl -L https://github.com/docker/compose/releases/download/1.8.0-rc1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose'
     sudo chmod +x /usr/local/bin/docker-compose
  SHELL
 end

 # Swarm child practice node
 config.vm.define "docker112-child1" do |docker112child1|
   docker112child1.vm.box = "ubuntu/trusty64"
   docker112child1.vm.network "private_network", type: "dhcp"
   docker112child1.vm.hostname = "docker112-child1"
   config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "512"]
      vb.customize ["modifyvm", :id, "--cpus", "1"]
      vb.name = "docker112-child1"
   end
   docker112child1.vm.provision "shell", inline: <<-SHELL
     sudo apt-get update
     sudo apt-get install -y apt-transport-https ca-certificates
     sudo apt-get update && sudo apt-get install -y apt-transport-https
     sudo apt-get install -y linux-image-extra-virtual
     sudo apt-get install -y linux-generic-lts-vivid
     curl -fsSL https://test.docker.com/ | sh
     sudo usermod -a -G docker vagrant
     # initialize swarm
     export DOCKER_MASTER=$(cat /vagrant/docker112master-ipaddr)
     docker swarm join ${DOCKER_MASTER}:2377
  SHELL
end

# Swarm child practice node
config.vm.define "docker112-child2" do |docker112child2|
  docker112child2.vm.box = "ubuntu/trusty64"
  docker112child2.vm.network "private_network", type: "dhcp"
  docker112child2.vm.hostname = "docker112-child2"
  config.vm.provider :virtualbox do |vb|
     vb.customize ["modifyvm", :id, "--memory", "512"]
     vb.customize ["modifyvm", :id, "--cpus", "1"]
     vb.name = "docker112-child2"
  end
  docker112child2.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update
    sudo apt-get install -y apt-transport-https ca-certificates
    sudo apt-get update && sudo apt-get install -y apt-transport-https
    sudo apt-get install -y linux-image-extra-virtual
    sudo apt-get install -y linux-generic-lts-vivid
    curl -fsSL https://test.docker.com/ | sh
    sudo usermod -a -G docker vagrant
    export DOCKER_MASTER=$(cat /vagrant/docker112master-ipaddr)
    docker swarm join ${DOCKER_MASTER}:2377
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
