# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

# Vagrant バージョン2を指定
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure("2") do |config|
   # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "bento/ubuntu-16.04"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  #config.vm.network "forwarded_port", guest: 22, host: 22
  config.vm.network "forwarded_port", guest: 80, host: 80
  config.vm.network "forwarded_port", guest: 443, host: 443
  config.vm.network "forwarded_port", guest: 1935, host: 1935

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  #config.vm.network "public_network", ip: "192.168.12.109"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  config.vm.synced_folder "./docker-composes", "/docker-composes"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    #vb.gui = true
    # Customize the amount of memory on the VM:
    vb.memory = "2048"
    vb.cpus = 2
    vb.name = "hls-streaming-server"
  end
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

  config.vm.provision "shell", inline: $setup
  config.vm.provision "shell", run: "always", inline: $start
end

# 初回・reload時ｍに実行される
$setup = <<SCRIPT
  #test -f /etc/bootstrapped && exit
  #sudo apt-get update -y
  #sudo apt-get install -y apt-transport-https ca-certificates
  #sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
  #sudo echo "deb https://apt.dockerproject.org/repo ubuntu-trusty main" > /etc/apt/sources.list.d/docker.list
  #sudo apt-get update -y
  #sudo apt-get purge lxc-docker
  #sudo apt-cache policy docker-engine
  #sudo apt-get install -y docker-engine
  
  sudo apt-get remove docker docker-engine
  sudo apt-get update -y
  sudo apt-get -y install wget linux-image-extra-$(uname -r) linux-image-extra-virtual
  wget -qO- https://get.docker.com/ | sh
  sudo usermod -aG docker vagrant
  
  sudo service docker restart
  
  sudo curl -s -S -L https://github.com/docker/compose/releases/download/1.8.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
  sudo chmod +x /usr/local/bin/docker-compose
  sudo date > /etc/bootstrapped
  
  ## create docker network
  sudo docker network create --driver bridge common_link
SCRIPT

# $setup = <<SCRIPT
#   ## update packages
#   apt-get update -y
#   apt-get install -y linux-image-generic-lts-raring linux-headers-generic-lts-raring
#   apt-get install -y curl
#   curl -s http://get.docker.io/ubuntu/ | sudo sh
#   
#   ## start docker
#   sudo service docker.service start
#   sudo systemctl enable docker
#   
#   apt-get install -y lsof wget nc
#   
#   ## docker-compose install
#   sudo curl -L "https://github.com/docker/compose/releases/download/1.8.1/docker-compose-$(uname -s)-$(uname -m)" >  ~/docker-compose
#   sudo mkdir -p /opt/bin
#   sudo mv ~/docker-compose /usr/local/bin
#   sudo chown root:root /usr/local/bin
#   sudo chmod +x /usr/local/bin
#   
#   ## create docker network
#   sudo docker network create --driver bridge common_link
# SCRIPT

# VM再起動時にコマンド実行する場合
$start = <<SCRIPT
  ## up nginx-proxy
  # sudo docker run -d -p 80:80 -e DEFAULT_HOST=foo.bar.com -v /var/run/docker.sock:/tmp/docker.sock:ro jwilder/nginx-proxy
  sudo /usr/local/bin/docker-compose -f /docker-composes/proxy/docker-compose.yml stop 2>&1
  sudo /usr/local/bin/docker-compose -f /docker-composes/proxy/docker-compose.yml rm -f 2>&1
  sudo /usr/local/bin/docker-compose -f /docker-composes/proxy/docker-compose.yml up -d
  
  ## up php-mysql
  sudo /usr/local/bin/docker-compose -f /docker-composes/php-mysql/docker-compose.yml stop 2>&1
  sudo /usr/local/bin/docker-compose -f /docker-composes/php-mysql/docker-compose.yml rm -f 2>&1
  sudo /usr/local/bin/docker-compose -f /docker-composes/php-mysql/docker-compose.yml up -d
SCRIPT
