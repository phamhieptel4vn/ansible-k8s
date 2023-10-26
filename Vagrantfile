# -*- mode: ruby -*-
# vi: set ft=ruby :

NNODES=6

$script = <<-SCRIPT
echo "<CHANGE_ME>" >> /home/vagrant/.ssh/authorized_keys
SCRIPT
Vagrant.configure("2") do |config|
  (0..NNODES - 1).each do |i|
    config.vm.define "k8s-ubuntu-#{i}" do |node|
      #node.vm.box = "ubuntu/focal64"
      node.vm.box = "bento/ubuntu-20.04-arm64"
      node.vm.hostname = "k8s-ubuntu-#{i}"
      node.vm.provider "vmware_desktop" do |vmware|
        vmware.cpus = 4
        vmware.memory = 4096
        vmware.gui = true
        vmware.vmx["ethernet0.virtualdev"] = "vmxnet3"
      end
      node.vm.provision "shell", inline: $script
      node.vm.provision "shell", inline: "echo hello from node #{i}"
    end
  end
end