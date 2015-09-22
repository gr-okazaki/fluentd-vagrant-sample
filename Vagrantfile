# -*- mode: ruby -*-
# vi: set ft=ruby :

$initialize = <<SCRIPT
mv /etc/localtime /etc/localtime.org
ln -s /usr/share/zoneinfo/Asia/Tokyo localtime
SCRIPT

$fluentd_install = <<SCRIPT
curl -L https://td-toolbelt.herokuapp.com/sh/install-redhat-td-agent2.sh | sh
td-agent-gem install fluent-plugin-elasticsearch
chkconfig td-agent on
SCRIPT

$apache_install = <<SCRIPT
yum install -y httpd
chkconfig httpd on
chmod -R oug+rx /var/log/httpd
chmod -R oug+rx /var/log
SCRIPT

$elasticsearch_install = <<SCRIPT
yum install -y java-1.8.0-openjdk
wget https://download.elastic.co/elasticsearch/elasticsearch/elasticsearch-1.7.1.noarch.rpm
rpm -i elasticsearch-1.7.1.noarch.rpm
cp /etc/elasticsearch/elasticsearch.yml /etc/elasticsearch/elasticsearch.yml.org
cp /vagrant/configs/elasticsearch.yml /etc/elasticsearch/.
chkconfig elasticsearch on
service elasticsearch restart

wget https://download.elastic.co/kibana/kibana/kibana-4.1.1-linux-x64.tar.gz
tar xvfz kibana-4.1.1-linux-x64.tar.gz
mv kibana-4.1.1-linux-x64 kibana
/home/vagrant/kibana/bin/kibana &
SCRIPT

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define :client do |cl|
    cl.vm.box = "CentOS65"
    cl.vm.box_url = "http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_centos-6.5_chef-provisionerless.box"

    cl.vm.network "private_network", ip: "192.168.33.11"
    cl.vm.network :forwarded_port, guest: 24224, host: 24224
    cl.vm.network :forwarded_port, guest: 80, host: 8088

    cl.vm.provider "virtualbox" do |vb|
      # Don't boot with headless mode
      vb.gui = false

      # Use VBoxManage to customize the VM. For example to change memory:
      vb.customize ["modifyvm", :id, "--memory", "1024"]
    end
    cl.vm.provision "shell", inline: $initialize
    cl.vm.provision "shell", inline: $fluentd_install
$script=<<SCRIPT
cp /etc/td-agent/td-agent.conf /etc/td-agent/td-agent.conf.org
cp /vagrant/configs/td-agent.conf.client /etc/td-agent/td-agent.conf
service td-agent restart
SCRIPT
    cl.vm.provision "shell", inline: $script
    cl.vm.provision "shell", inline: $apache_install
  end

  config.vm.define :server do |sv|
    sv.vm.box = "CentOS65"
    sv.vm.box_url = "http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_centos-6.5_chef-provisionerless.box"

    sv.vm.network "private_network", ip: "192.168.33.12"
    sv.vm.network :forwarded_port, guest: 24225, host: 24225
    sv.vm.network :forwarded_port, guest: 5601, host: 5601

    sv.vm.provider "virtualbox" do |vb|
      # Don't boot with headless mode
      vb.gui = false

      # Use VBoxManage to customize the VM. For example to change memory:
      vb.customize ["modifyvm", :id, "--memory", "2048"]
    end
    sv.vm.provision "shell", inline: $initialize
    sv.vm.provision "shell", inline: $fluentd_install
$script=<<SCRIPT
cp /etc/td-agent/td-agent.conf /etc/td-agent/td-agent.conf.org
cp /vagrant/configs/td-agent.conf.server /etc/td-agent/td-agent.conf
service td-agent restart
SCRIPT
    sv.vm.provision "shell", inline: $script
    sv.vm.provision "shell", inline: $elasticsearch_install
  end
end
