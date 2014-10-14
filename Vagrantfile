# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

DB_PRIVATE_IP = "192.168.60.4"
ATG_PRIVATE_IP = "192.168.60.5"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.define :db12c do |db12c_config|
    db12c_config.vm.box = "chef/centos-6.5"

    # change memory size
    db12c_config.vm.provider "virtualbox" do |v|
      v.memory = 2048
    end

    # static IP so we can configure machines to talk to each other
    db12c_config.vm.network "private_network", ip: DB_PRIVATE_IP

    # provision
    db12c_config.vm.provision "shell" do |s|
        s.path = "scripts/db12c/provision.sh"
        s.args = ENV['DB12C_PROVISION_ARGS']
    end
end

  # ==============================

  config.vm.define :atg do |atg_config|

    atg_config.vm.box = "chef/centos-6.5"

    # change memory size
    atg_config.vm.provider "virtualbox" do |v|
      v.memory = 6144
    end

    # static IP so we can configure machines to talk to each other
    atg_config.vm.network "private_network", ip: ATG_PRIVATE_IP

    # provision
    atg_config.vm.provision "shell" do |s|
        s.path = "scripts/atg/provision.sh"
        s.args = ENV['ATG_PROVISION_ARGS']
    end

  end

end
