# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    config.vm.box = "ubuntu/trusty64"
    config.vm.hostname = "vgb-dev"

    # Do some network configuration
    config.vm.network "private_network", ip: "192.168.100.100"

    # Assign a quarter of host memory and all available CPU's to VM
    # Depending on host OS this has to be done differently.
    config.vm.provider :virtualbox do |vb|
        host = RbConfig::CONFIG['host_os']

        if host =~ /darwin/
            cpus = `sysctl -n hw.ncpu`.to_i
            mem = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4

        elsif host =~ /linux/
            cpus = `nproc`.to_i
            mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4

        # Windows...
        else
            cpus = 4
            mem = 2048
        end

        vb.customize ["modifyvm", :id, "--memory", mem]
        vb.customize ["modifyvm", :id, "--cpus", cpus]
    end

    config.vm.provision "ansible" do |ansible|
        ansible.playbook = "ansible/setup.yml"
    end

    # Mount htdocs
    apache_vars = YAML.load_file('ansible/group_vars/all/main.yml')
    hostenty = ""
    apache_vars["projects"].each do |project|
        local_dir = "share/htdocs/#{project["vhost"]}"
        remote_dir = "/var/www/#{project["vhost"]}"
        Dir.mkdir(local_dir) unless File.exists?(local_dir)
        config.vm.synced_folder local_dir, remote_dir
        hostenty += "#{project["vhost"]} "
    end

    config.vm.post_up_message = "Host Entry:\n" +
                                "192.168.100.100 #{hostenty}"
end