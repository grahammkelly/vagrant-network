# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANT_API_VER = "2"
DEFAULT_BOX = "ubuntu/trusty64"
#DEFAULT_BOX = "centos/7"

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
        cpus: 2
    }
]
ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip

#Set up the boxes
Vagrant.configure(VAGRANT_API_VER) do |config|
    #Assuming you have the hostmanager plugin installed
    # If not, install with '$ vagrant plugin install vagrant-hostmanager'
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true   #Tell vagrant to manage the new VMs onto the host system's /etc/hosts
    config.hostmanager.manage_guest = true  #Tell vagrant to manage the new VMs onto the VM's /etc/hosts
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true

    servers.each do |svr|
        config.vm.define svr[:name] do |node|
            node.vm.box = svr[:box] || DEFAULT_BOX
            node.vm.hostname = svr[:name]
            node.hostmanager.aliases = ["#{svr[:name]}.localdomain"]

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
                vb.cpus = svr[:cpus] if svr[:cpus]
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
