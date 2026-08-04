Vagrant.configure("2") do |config|
  
  config.vm.define "pxeser" do |server|
    server.vm.box = 'bento/ubuntu-22.04'
    server.vm.host_name = 'pxeser'
    server.vm.network "forwarded_port", guest: 80, host: 8080
    server.vm.network :private_network,
                       ip: "10.0.0.20",
                       virtualbox__intnet: 'pxenet'
    server.vm.network :private_network, ip: "192.168.56.10", adapter: 3
    server.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.customize ["modifyvm", :id,"--natdnshostresolver1", "on"]
    end
  # server.vm.provision "ansible" do |ansible|
  # ansible.playbook = "ansible/provision.yml"
  # ansible.inventory_path = "ansible/hosts"
  # ansible.host_key_checking = "false"
  # ansible.limit = "all"
  # end
  end
  config.vm.define "pxecli" do |pxeclient|
    pxeclient.vm.box = 'bento/ubuntu-22.04'
    #pxeclient.vm.box = 'seskion/ubuntu-20.04-efi'
    pxeclient.vm.host_name = 'pxecli'
    pxeclient.vm.network :private_network, 
                          ip: "192.168.56.20"
    pxeclient.vm.provider :virtualbox do |vb|
      vb.memory = "4096"
      vb.customize ["modifyvm", :id,"--natdnshostresolver1", "on"]
      vb.customize [
          'modifyvm', :id,
          '--nic1', 'intnet',
          '--intnet1', 'pxenet',
          '--nic2', 'nat',
          '--boot1', 'net',
          '--boot2', 'none',
          '--boot3', 'none',
          '--boot4', 'none'
        ]
      vb.customize ["modifyvm", :id,"--natdnshostresolver1", "on"]
    end
  end
end
