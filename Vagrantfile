# -*- mode: ruby -*-
# vi: set ft=ruby :

ADAPTER_COUNT = 8
N = 4
N_ROUTERS = 4

Vagrant.configure("2") do |config|

  config.vm.define "webserver" do |ws|
    ws.vm.hostname = "webserver"
    ws.vm.box = "generic/ubuntu2010"
    ws.vm.network :private_network, ip: "100.30.1.10"
    
    ws.vm.provider :libvirt do |libvirt|
      libvirt.title = "webserver"
      libvirt.cpus = 1
      libvirt.memory = 1024
      libvirt.nic_adapter_count = 1
      libvirt.nic_model_type = "e1000"
    end
  end

  (1..4).each do |i|
    config.vm.define "rt#{i}" do |node|
      node.vm.box = "cisco/iosv"

      # Turn off shared folders
      node.vm.synced_folder ".", "/vagrant", disabled: true

      # Do not try to insert new SSH key
      node.ssh.insert_key = false

      # Give VM time to boot
      node.vm.boot_timeout = 180

      # Set guest type to prevent guest type detection
      node.vm.guest = :freebsd

      # Network configuration
      (1..ADAPTER_COUNT).each do |j|
        node.vm.network :private_network, ip: "1.1.1.1"
      end
      
      # Provider-specific configuration
      node.vm.provider :libvirt do |domain|
        domain.nic_adapter_count = 8
        domain.memory = 512
        domain.cpus = 1
        domain.driver = "kvm"
        domain.nic_model_type = "e1000"
      end

      if i == N
        config.vm.provision "ansible" do |ansible|
          ansible.playbook = "provisioning/playbook.yml"
          ansible.host_key_checking = false
          ansible.groups = {
            "webservers" => "webserver",
            "routers" => ["rt[1:4]"],
            "area0_routers" => ["rt[1:2]"],
            "other_routers" => ["rt[3:4]"],
            "routers:vars" => {"ansible_connection" => "network_cli",
                               "ansible_network_os" => "ios"},
          }
          ansible.host_vars = {
            "rt1" => {"addr_loopback" => "1.1.1.1/32", "hostname" => "rt1",
                      "int1_ip" => "100.10.0.1/24", "int2_ip" => "100.10.1.1/24", "int3_ip" => "100.10.2.1/24",
                      "area0_network" => "100.10.0.0", "area0_mask" => "0.0.0.255",
                      "area1_network" => "100.10.1.0", "area1_mask" => "0.0.0.255",
                      "area2_network" => "100.10.2.0", "area2_mask" => "0.0.0.255"},

            "rt2" => {"addr_loopback" => "2.2.2.2/32", "hostname" => "rt2", 
                      "int1_ip" => "100.10.0.2/24", "int2_ip" => "100.10.10.2/24", "int3_ip" => "100.10.20.2/24",
                      "area0_network" => "100.10.0.0", "area0_mask" => "0.0.0.255",
                      "area1_network" => "100.10.10.0", "area1_mask" => "0.0.0.255",
                      "area2_network" => "100.10.20.0", "area2_mask" => "0.0.0.255"},

            "rt3" => {"addr_loopback" => "3.3.3.3/32", "hostname" => "rt3", "area" => "1",
                      "int1_ip" => "100.10.1.3/24", "int2_ip" => "100.10.10.3/24", "int3_ip" => "100.30.1.1/24",
                      "areax_network" => "100.10.1.0", "areax_mask" => "0.0.0.255",
                      "areax_network2" => "100.10.10.0", "areax_mask2" => "0.0.0.255"},

            "rt4" => {"addr_loopback" => "4.4.4.4/32", "hostname" => "rt4", "area" => "2",
                      "int1_ip" => "100.10.2.4/24", "int2_ip" => "100.10.20.4/24", "int3_ip" => "100.40.1.1/24",
                      "areax_network" => "100.10.2.0", "areax_mask" => "0.0.0.255",
                      "areax_network2" => "100.10.20.0", "areax_mask2" => "0.0.0.255"},
          }
        end
      end
    end
  end
end
