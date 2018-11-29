# -*- mode: ruby -*-

hosts = {
    "master" => { "memory" => 1536, "ip" => "192.168.10.10"},
    "worker1" => { "memory" => 1536, "ip" => "192.168.10.30"},
    "worker2" => { "memory" => 1536, "ip" => "192.168.10.31"},
    "worker3" => { "memory" => 1024, "ip" => "192.168.10.32"},
    "nfs" => { "memory" => 512, "ip" => "192.168.10.20"}
}

Vagrant.configure("2") do |config|
    config.vm.box = "centos/7"
    config.vm.provider "libvirt"
    config.vm.synced_folder ".", "/vagrant", disabled: true

    # Loop over all machine names
    hosts.each_key do |host|
        config.vm.define host, primary: host == "master" do |node|
            node.vm.hostname = host
            node.vm.network :private_network, ip: hosts[host]["ip"]

            node.vm.provider :libvirt do |lv|
                lv.memory = hosts[host]["memory"]
            end

            # Run ansible in parallel when all hosts are up and running
            if host == hosts.keys.last
                # ----------------------------------------------
                # Install kubernetes on BOTH master and workers
                # ----------------------------------------------
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