# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

  VAGRANT_VM_PROVIDER = "virtualbox"

  config.vm.box_check_update = true
  config.vm.synced_folder "./", "/vagrant", disabled:true
  config.vm.synced_folder "./", "/home/vagrant/sync", disabled:true

  config.vm.define "master" do |ubuntu|
    ubuntu.vm.box = "ubuntu/xenial64"
    ubuntu.vm.network "private_network", ip:"192.168.56.210"
    ubuntu.vm.hostname = "master"
    ubuntu.vm.provider "virtualbox" do |vb|
      vb.name = "master-esgi"
      vb.cpus = 2
      vb.memory = 4*1024
    end
    ubuntu.vm.provision "shell", inline: "sudo apt-get update && sudo apt-get install --assume-yes python-minimal"
  end

  config.vm.define "slave" do |centos|
    centos.vm.box = "centos/7"
    centos.vm.network "private_network", ip:"192.168.56.211"
    centos.vm.hostname = "slave"
    centos.vm.provider "virtualbox" do |vb|
      vb.name = "slave-esgi"
      vb.cpus = 2
      vb.memory = 4*1024
    end

    # install ansible
    centos.vm.provision :shell, :inline=>"
      sudo yum -y install epel-release 
      sudo yum -y install python python-pip 
      sudo pip install --upgrade pip 
      sudo yum -y groupinstall \"Development Tools\" 
      sudo yum -y install python-devel
      sudo pip install ansible==2.1.1.0
      echo 'restart network service (eth1 not started when vm started using vagrant)'
      sudo systemctl restart network.service
      "
  end

  config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "$HOME/.ssh/me.pub"
  config.vm.provision :shell, :inline =>"
    echo 'Copying public SSH Keys to the VM'
    cat $HOME/.ssh/me.pub >> $HOME/.ssh/authorized_keys
    chmod -R 600 ~/.ssh/authorized_keys
    echo 'Host 192.168.*.*' >> ~/.ssh/config
    echo 'StrictHostKeyChecking no' >> ~/.ssh/config
    echo 'UserKnownHostsFile /dev/null' >> ~/.ssh/config
    chmod -R 600 ~/.ssh/config
  ", privileged: false

end
