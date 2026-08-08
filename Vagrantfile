# Указываем зеркало для скачивания боксов
ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'

Vagrant.configure("2") do |config|
  
  config.vm.define "pxeser" do |server|
    server.vm.box = 'ubuntu/jammy64'
    server.vm.box_version = '1.0.0'
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
    
    # Ansible отключён (закомментирован)
    # server.vm.provision "ansible" do |ansible|
    #   ansible.playbook = "ansible/provision.yml"
    #   ansible.inventory_path = "ansible/hosts"
    #   ansible.host_key_checking = "false"
    #   ansible.limit = "all"
    # end
  end
  
  config.vm.define "pxecli" do |pxeclient|
    pxeclient.vm.box = 'ubuntu/jammy64'
    pxeclient.vm.box_version = '1.0.0'
    pxeclient.vm.host_name = 'pxecli'
    
    pxeclient.vm.network :private_network, 
                          ip: "192.168.56.20",
                          adapter: 3
    
    pxeclient.vm.provider :virtualbox do |vb|
      vb.memory = "4096"
      vb.customize ["modifyvm", :id,"--natdnshostresolver1", "on"]
      # Убираем настройки адаптеров, т.к. Vagrant уже все настроил через vm.network
      # Оставляем только настройку загрузки с сети
      vb.customize ["modifyvm", :id, "--boot1", "net"]
      vb.customize ["modifyvm", :id, "--boot2", "none"]
      vb.customize ["modifyvm", :id, "--boot3", "none"]
      vb.customize ["modifyvm", :id, "--boot4", "none"]
      vb.customize ["modifyvm", :id,"--natdnshostresolver1", "on"]
    end
  end
  
end
