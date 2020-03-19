# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  glusterfs_version = "7"
  # We setup three nodes to be gluster hosts, and one gluster client to mount the volume
  (1..3).each do |i|
    config.vm.define vm_name = "gluster-server-#{i}" do |config|
      config.vm.hostname = vm_name
      ip = "172.21.12.#{i+10}"
      config.vm.network :private_network, ip: ip
      config.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -yq software-properties-common", :privileged => true
      config.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive add-apt-repository ppa:gluster/glusterfs-#{glusterfs_version}", :privileged => true
      config.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -yq glusterfs-server", :privileged => true
    end
  end
  config.vm.define vm_name = "gluster-client" do |config|
    config.vm.hostname = vm_name
    ip = "172.21.12.10"
    config.vm.network :private_network, ip: ip
    config.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -yq software-properties-common", :privileged => true
    config.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive add-apt-repository ppa:gluster/glusterfs-#{glusterfs_version}", :privileged => true
    config.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -yq glusterfs-client", :privileged => true
  end
end
