# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # Specify your boxes below for DHCP server and clietns. MORE THAN ONE!
  # Please pay attention. First host in the list will configured as DHCP server!
  BOX_IMAGES = ["debian11","debian10","debian11","debian12"]
  # Specify a desired DHCP range.
  # It works with /24 subnets only. Please avoid to use 10.0.2.0/24 subnet!
  DHCP_RANGE = "192.168.0.137-192.168.0.145"
  DHCP_NETMASK = "255.255.255.0"
  DHCP_LEASETIME = "12h"

  N = BOX_IMAGES.length()-1
  HOSTS = []
  
  # Disabling the default /vagrant share
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
    vb.cpus = "2"
    # Skip the guest additions check
    vb.check_guest_additions = false
  end
    
  (0..N).each do |i|

    HOST = "dhcp-client#{i}"
      if i == 0
        HOST = "dhcp-server"
      end
    HOSTS << HOST

    config.vm.define HOSTS[i] do |node|
      node.vm.box = BOX_IMAGES[i]
      node.vm.hostname = HOSTS[i]
      node.vm.network "private_network", ip: DHCP_RANGE.split('-', 2)[0].split('.')[0,3].join('.') + ".#{i+2}", virtualbox__intnet: true

      if i > 0
        # Genetare new machine-id for each VM (requred for update DHCP client-id)
        node.vm.provision "shell", inline: <<-SHELL
          sudo rm -f /etc/machine-id && sudo dbus-uuidgen --ensure=/etc/machine-id
          sudo systemctl restart systemd-networkd
        SHELL
      end

      # Only execute once the Ansible provisioner, when all the machines are up and ready.
      if i == N
        node.vm.provision :ansible, run: "always" do |ansible|
          ansible.compatibility_mode = "2.0"  
          # Disable default limit to connect to all the machines
          ansible.limit = "all"

          ansible.groups = {
            "dhcp_servers" => [HOSTS[0]],
            "dhcp_clients" => HOSTS[1,N],
            "all:vars" => {
              "dhcp_start_range" => DHCP_RANGE.split('-', 2)[0],
              "dhcp_end_range" => DHCP_RANGE.split('-', 2)[1],
            },
            "dhcp_servers:vars" => {
              "dhcp_subnet_mask" => DHCP_NETMASK,
              "dhcp_lease_time" => DHCP_LEASETIME
            }
          }

          #puts ansible.groups     

          ansible.playbook = "provisioning/main.yml"
        end
      end

    end
  end

end