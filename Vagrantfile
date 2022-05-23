# -*- mode: ruby -*-

hosts = {
    "control-plane1" => { "memory" => 2048, "ip" => "192.168.10.10"},
    # "control-plane2" => { "memory" => 2048, "ip" => "192.168.10.11"},
    # "control-plane3" => { "memory" => 2048, "ip" => "192.168.10.12"},
    "worker1" => { "memory" => 2048, "ip" => "192.168.10.30"},
    "worker2" => { "memory" => 2048, "ip" => "192.168.10.31"},
    # "worker2" => { "memory" => 1536, "ip" => "192.168.10.31", "box" => "generic/ubuntu2004"},
    # "worker3" => { "memory" => 1024, "ip" => "192.168.10.32", "box" => "generic/cenots8s"},
    "nfs" => { "memory" => 512, "ip" => "192.168.10.20"}
}

Vagrant.configure("2") do |config|
    # Choose which box you want below
    # config.vm.box = "generic/centos8s"
    config.vm.box = "generic/ubuntu2004"

    config.vm.synced_folder ".", "/vagrant", disabled: true

    config.vm.provider :libvirt do |libvirt|
      # QEMU system connection is required for private network configuration
      libvirt.qemu_use_session = false
    end

    # Loop over all machine names
    hosts.each_key do |host|
        config.vm.define host, primary: host == hosts.keys.first do |node|
            # Use custom box if set
            if hosts[host]["box"]
                node.vm.box = hosts[host]["box"]
            end

            node.vm.hostname = host
            node.vm.network :private_network, ip: hosts[host]["ip"]

            node.vm.provider :libvirt do |lv|
                lv.memory = hosts[host]["memory"]
                lv.cpus = 2
            end

            node.vm.provider :virtualbox do |vbox|
                vbox.customize ["modifyvm", :id, "--memory", hosts[host]["memory"]]
                vbox.cpus = 2
            end

            # Run ansible in parallel when all hosts are up and running
            if host == hosts.keys.last
                # ----------------------------------------------------
                # Install kubernetes on BOTH control-plane and workers
                # ----------------------------------------------------
                config.vm.provision :ansible do |ansible|
                    # Run on all hosts in parallel
                    ansible.limit = "all"
                    ansible.playbook = "playbooks/provision-all.yml"
                    ansible.inventory_path = "inventory.ini"
                end
            end
        end
    end

end
