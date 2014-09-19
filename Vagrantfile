# -*- mode: ruby -*-
# vi: set ft=ruby :


# General project settings
#################################

  vagrant_root_path = File.dirname(__FILE__)

  # IP Address for the host only network(IPv4 private network range)
  ip_address = "172.16.10.10"

  # The project name is base for directories, hostname and alike
  project_name = "elasticsearch"


# Provision settings
#################################

$provision_script = <<SCRIPT
if [ -f "/var/vagrant_provision" ];
  then exit 0
fi

# DEPENDENCY INSTALL
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y default-jre default-jdk nginx zip irqbalance htop


# ELASTICSEARCH SETUP
sudo wget -q https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.3.2.zip -O /tmp/elasticsearch.zip
sudo unzip /tmp/elasticsearch.zip -d /tmp/elasticsearch

directory_name=`find /tmp/elasticsearch/ -type d -name "elasticsearch*" | sed -n '2p'`
sudo mv $directory_name /var/lib/elasticsearch && sudo rm -rf /tmp/elasticsearch*

export ES_CLASSPATH=/var/lib/elasticsearch
sudo touch /etc/init.d/elasticsearch
echo "sudo /var/lib/elasticsearch/bin/elasticsearch -d" | sudo tee --append /etc/init.d/elasticsearch
sudo chmod +x /etc/init.d/elasticsearch


# KIBANA SETUP
sudo wget -q https://download.elasticsearch.org/kibana/kibana/kibana-3.1.0.zip -O /tmp/kibana.zip
sudo unzip /tmp/kibana.zip -d /tmp/kibana

directory_name=`find /tmp/kibana/ -type d -name "kibana*" | sed -n '2p'`
sudo mv $directory_name /usr/share/nginx/html/kibana && sudo rm -rf /tmp/kibana*

sudo service nginx stop && sudo service nginx start & /etc/init.d/elasticsearch

sudo touch /var/vagrant_provision
SCRIPT

# Vagrant configuration
#################################

  Vagrant.configure("2") do |config|
    config.vm.box                  = "ubuntu/trusty64"
    config.ssh.private_key_path    = vagrant_root_path + "/keys/vagrant"
    config.hostmanager.enabled     = true
    config.hostmanager.manage_host = true

    config.vm.synced_folder "./" , vagrant_root_path, mount_options: ["dmode=777", "fmode=666"]

    config.vm.provider :virtualbox do |virtualbox|
      virtualbox.customize ["modifyvm", :id, "--memory", "1024"]
    end

    config.vm.define project_name do |node|
      node.vm.hostname = "#{ project_name }.local"
      node.vm.network :private_network, ip: ip_address
    end

    config.vm.provision :hostmanager
    config.vm.provision "shell", inline: $provision_script
  end