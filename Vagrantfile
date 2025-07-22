# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.define "serverwp" do |serverwp|
    serverwp.vm.box = "bento/ubuntu-24.04"
    serverwp.vm.hostname = "serverwp"
    serverwp.vm.network :private_network, ip: "192.168.56.13", auto_config: true
    # server01.vm.provision :shell, path: "resources/provision.sh"
    # serverwp.vm.synced_folder "./wordpress", "/var/www/html"
  end

end
