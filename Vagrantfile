VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "tkhinfra/debian10"
  config.ssh.insert_key = false
  config.vm.provider "virtualbox"
  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.vm.provider :virtualbox do |v|
    v.memory = 1684
    v.cpus = 2
    v.linked_clone = true
    v.customize ['modifyvm', :id, '--audio', 'none']
  end

  # Define three VMs with static IP addresses.
  cluster = {
    "cluster1" => { :ip => "192.168.2.2" },
    "cluster2" => { :ip => "192.168.2.3" },
    "cluster3" => { :ip => "192.168.2.4" }
  }

  # Configure each VM.
  cluster.each_with_index do |(hostname, info), index|
    config.vm.define hostname do |config|
      config.vm.hostname = hostname + ".kubernetes.local"
      config.vm.network :private_network, ip: "#{info[:ip]}"

      # Run ansible after last VM is running.
      if index == cluster.size - 1
        config.vm.provision "ansible" do |ansible|
          ansible.compatibility_mode = "2.0"
          ansible.playbook = "main.yml"
          ansible.inventory_path = "inventory"
          ansible.limit = "all"
        end
      end
    end
  end
end
