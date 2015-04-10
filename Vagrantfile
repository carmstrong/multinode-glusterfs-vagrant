# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  # We setup three nodes to be gluster hosts, and one gluster client to mount the volume
  (1..3).each do |i|
    config.vm.define vm_name = "gluster-server-#{i}" do |config|
      config.cache.scope = :box
      config.vm.hostname = vm_name
      config.vm.network :private_network, ip: "172.21.12.#{i+10}"
      config.vm.provision "shell", privileged: true, inline: <<-SHELL

        apt-get install -y python-software-properties
        add-apt-repository -y ppa:gluster/glusterfs-3.5
        apt-get update
        apt-get install -y glusterfs-server
      SHELL
    end
  end

  config.vm.define vm_name = "gluster-client" do |config|
    config.cache.scope = :box
    config.vm.hostname = vm_name
    config.vm.network :private_network, ip: "172.21.12.10"
    config.vm.provision "shell", privileged: true, inline: <<-SHELL

      # Install glusterfs, see http://www.gluster.org/community/documentation/index.php/Getting_started_overview
      apt-get install -y python-software-properties
      add-apt-repository -y ppa:gluster/glusterfs-3.5
      apt-get update
      apt-get install -y glusterfs-client
    SHELL

    # Install latest docker
    config.vm.provision :docker do |d|
    end
  end
end
