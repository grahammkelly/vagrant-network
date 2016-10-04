# Vagrant network

Configures a network of Vagrant boxes.

You define the parameters of the servers being set up in the `server` array. This allows you to set up the VM name, memory, IP address and the base box the VM is based upon.

The Vagrant file configures the each VM to accept login to the 'vagrant' account
using the creating user's default public key. This is done by copying the
public key from the `.ssh/id_rsa.pub` file and appending it to the `.ssh/authorixed_keys` 
file under the vagrant user. 

```ruby
  ...
  config.vm.provision "shell" do |s|
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    s.inline = <<-SHELL
        echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
    SHELL
  end
  ...
```

## Local system configuration


Due to the nature of Vagrant, you can bring up and destroy these VMs at will. If you've 
SSH'ed into any of the VMs at all prior to destroying the VM, the SSH fingerprint will 
then stop you logging into that box again. In this case, you need to remove the VM's entry
from the `~/.ssh/known_hosts` file.

Additionally, as Vagrant creates (and assumes use of) the `vagrant` user as the default user
you probably want to login to that user by default (i.e. without needing to specify at the 
command line).

To do this, create (or edit) `~/.ssh/config` (on the _local_ system) and make sure the 
following is included (_note, this assumes I'll always create VM's on the `192.169.33.XXX` 
subnet. I could also have specified the exact IP._)

```
host 192.168.33.*
    StrictHostKeyChecking no
    UserKnownHostsFile=/dev/null
    User vagrant
```



