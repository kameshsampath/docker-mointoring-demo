# -*- mode: ruby -*-
# vi: set ft=ruby :
# TODO - vagrant-registration plugin setup

Vagrant.configure(2) do |config|

  # Root RHEL7 CDKv2
  config.vm.define "rhel7-cdkv2", primary: true  do|node|
   node.vm.box = "cdkv2"
   node.vm.network "private_network", ip: "192.168.56.100"

   node.vm.host_name="cdkv2.example.com"

   node.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
   end


  end
  # All Atomic Hosts
  (1..2).each do | i |
       config.vm.define "node#{i}" do |node|
       # node.vm.box = "centos/atomic-host"
       node.vm.box ="centos/atomic-host"
       node.vm.hostname = "node#{i}.example.com"
       node.vm.network "private_network", ip: "192.168.56.10#{i}"
       node.vm.provider "virtualbox" do |vb|
         vb.memory = "1024"
       end
    end
  end

end
