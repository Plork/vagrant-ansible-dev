# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"
LOCAL_HTTP_PROXY = 'http://proxy.binckbank.nv:8080'

#require './vagrant-provision-reboot-plugin'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  if Vagrant.has_plugin?('vagrant-proxyconf')
    config.apt_proxy.http     = LOCAL_HTTP_PROXY
    config.apt_proxy.https    = LOCAL_HTTP_PROXY
    config.proxy.no_proxy = 'localhost,127.0.0.1,192.168.56.*,*binckbank.nv,*.otas.nv'
  end

  config.vm.define :ansible  do |ansible|

    ansible.vm.box = "ubuntu/trusty64"
    ansible.vm.network "private_network",
      ip: "192.168.56.4"
    ansible.vm.hostname = "ansible"

    ansible.vm.box_check_update = false

    ansible.vm.provider "virtualbox" do |vb|
      #vb.gui = true
      vb.name = "ansible"
      vb.cpus = 1
      vb.memory = 512
    end

    ansible.ssh.forward_agent = true
    ansible.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

    ansible.vm.provision :shell, path: "provision/install-ansible.sh"

    ansible.vm.synced_folder "./", "/vagrant", disabled: true
    ansible.vm.synced_folder "ansible/", "/etc/ansible", mount_options: ["dmode=777","fmode=666"]
  end

  config.vm.define :server2012  do |server2012|

    server2012.vm.box = "boxes/windows2012r2min-wmf5-virtualbox.box"
    server2012.vm.network "private_network",
      ip: "192.168.56.11"
    server2012.vm.hostname = "server2012"
    server2012.vm.guest = :windows
    server2012.vm.communicator = "winrm"
    server2012.vm.box_check_update = false
    server2012.vm.boot_timeout = 600

    server2012.winrm.username = "vagrant"
    server2012.winrm.password = "vagrant"
    server2012.winrm.timeout = 21600
    server2012.winrm.max_tries = 20

    server2012.vm.provider "virtualbox" do |vb|
      #vb.gui = true
      vb.name = "server2012"
      vb.memory = 1024
    end

    server2012.vm.synced_folder "./", "/vagrant", disabled: true
    server2012.vm.synced_folder "dsc/", "/dsc"

    server2012.vm.provision :shell, path: "provision/Configure-RemotingForAnsible.ps1"
    server2012.vm.provision :shell, inline: "cinst install vboxguestadditions.install -y -source https://www.myget.org/F/jmonsorno-choco/api/v2"
  end

end
