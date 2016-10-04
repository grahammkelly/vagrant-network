# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANT_API_VER = "2"

#Configuration
servers = [
    {   
        name: "server1",
        ip: "192.168.33.101",
        box: "ubuntu/trusty64",
        memory: "4098",
        expose_ports: [
            {vm: "8080", local: "80"}
        ],
        external_script_cfg: ["blah.sh"]
    },
    {  
        name: "server2",
        ip: "192.168.33.102",
    }
]
ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
server_host_entries = servers.map {|svr| "#{svr[:ip]}  #{svr[:name]}"}

#Set up the boxes
Vagrant.configure(VAGRANT_API_VER) do |config|
    servers.each do |svr|
        config.vm.define svr[:name] do |node|
            node.vm.box = svr[:box] || "ubuntu/trusty64"
            node.vm.hostname = svr[:name]

            #node.vm.box_check_update = false

            node.vm.network "private_network", ip: svr[:ip]
            #node.vm.network "public_network"

            #Open each exposed port from the VM as a local port on the host system
            if (svr[:expose_ports] || false) then
                svr[:expose_ports].each do |port|
                    node.vm.network "forwarded_port", guest: port[:local], host: port[:vm]
                end
            end

            node.vm.provider "virtualbox" do |vb|
                #vb.gui = true
                vb.memory = svr[:memory] if svr[:memory]
                vb.name = svr[:name]
            end

            node.vm.provision "shell" do |sh|
                sh.inline = <<-SHELL
                    echo "Ensuring the English locale is available"
                    sudo locale-gen en_IE.UTF-8

                    # Create the SSH keys to allow direct SSH onto the box (not through vagrant)
                    echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
                    echo #{ssh_pub_key} >> /root/.ssh/authorized_keys

                    #Add the other servers to the 'hosts' file
                    echo -e "Adding the following entries to /etc/hosts\n\t#{server_host_entries.join("\n\t")}"
                    sudo echo -e "\n# VMs on the same network\n#{server_host_entries.join("\n")}\n" >> /etc/hosts
                SHELL
            end

            if ((svr[:external_script_cfg] || false)) then
                svr[:external_script_cfg].each {|scriptname|
                    if (File.file?(scriptname)) then
                        node.vm.provision "shell", path: scriptname
                    end
                }
            end

            #Set up a shared location. Files placed here will be available on ALL VMs
            node.vm.synced_folder "#{Dir.pwd}/shared", "/vagrant_data"
        end
    end
   end
