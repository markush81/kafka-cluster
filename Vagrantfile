ENV["LC_ALL"] = "en_US.UTF-8"

# if sth. gets changed here, also adapt /ansible/inventories/vbox/hosts
KAFKA = 3

Vagrant.configure("2") do |config|

  required_plugins = %w( vagrant-hostsupdater )
  required_plugins.each do |plugin|
    system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
  end

  config.vm.box = "markush81/centos8-vbox-guestadditions"
  config.vm.box_check_update = true

  config.vm.synced_folder "exchange", "/home/vagrant/exchange", create: true, SharedFoldersEnableSymlinksCreate: true

  config.trigger.after :destroy do |trigger|
    trigger.run = { inline: 'rm -rf exchange/ssl && rm -rf exchange/ssl-client'}
  end

  config.vm.provision :shell, inline: "ifup eth1", run: "always"

  config.vm.define "mon-1" do |mon|
    mon.vm.hostname = "mon-1"
    mon.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = "2"
    end
    mon.vm.network :private_network, ip: "192.168.10.2", auto_config: true

    mon.vm.provision :ansible do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.limit = "elk"
      ansible.playbook = "ansible/cluster.yml"
      ansible.inventory_path = "ansible/inventories/vbox"
      ansible.raw_arguments  = ["-vv"]
    end
  end

  config.vm.define "mon-2" do |mon|
    mon.vm.hostname = "mon-2"
    mon.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = "1"
    end
    mon.vm.network :private_network, ip: "192.168.10.3", auto_config: true

    mon.vm.provision :ansible do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.limit = "mon"
      ansible.playbook = "ansible/network.yml"
      ansible.inventory_path = "ansible/inventories/vbox"
      ansible.raw_arguments  = [
        "-vv"
      ]
    end

    mon.vm.provision :ansible do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.limit = "grafana"
      ansible.playbook = "ansible/cluster.yml"
      ansible.inventory_path = "ansible/inventories/vbox"
      ansible.raw_arguments  = ["-vv"]
    end
  end

  (1..KAFKA).each do |i|
    config.vm.define "kafka-#{i}" do |kafka|
      kafka.vm.hostname = "kafka-#{i}"
      kafka.vm.provider "virtualbox" do |vb|
        vb.memory = "3072"
        vb.cpus = "1"
      end
      kafka.vm.network :private_network, ip: "192.168.10.#{3 + i}", auto_config: true

      if i == KAFKA

        kafka.vm.provision :ansible do |ansible|
          ansible.compatibility_mode = "2.0"
          ansible.limit = "network"
          ansible.playbook = "ansible/network.yml"
          ansible.inventory_path = "ansible/inventories/vbox"
          ansible.raw_arguments  = [
            "-vv"
          ]
        end

        kafka.vm.provision :ansible do |ansible|
          ansible.compatibility_mode = "2.0"
          ansible.limit = "kafka"
          ansible.playbook = "ansible/cluster.yml"
          ansible.inventory_path = "ansible/inventories/vbox"
          ansible.raw_arguments  = [
            "-vv"
          ]
        end
      end
    end
  end
end
