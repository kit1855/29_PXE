Vagrant.configure("2") do |config|
  
  # Настройка PXE-сервера
  config.vm.define "pxeserver" do |server|
    server.vm.box = "bento/ubuntu-22.04"
    server.vm.box_url = "https://vagrant.elab.pro/bento/ubuntu-22.04.box"
    server.vm.host_name = "pxeserver"
    
    # Проброс порта для веб-сервера
    server.vm.network "forwarded_port", guest: 80, host: 8080
    
    # Внутренняя сеть для PXE (сетевой мост pxenet)
    server.vm.network "private_network", 
      ip: "10.0.0.20", 
      virtualbox__intnet: "pxenet"
    
    # Дополнительная сеть для доступа в интернет
    server.vm.network "private_network", 
      ip: "192.168.50.10", 
      adapter: 3
    
    server.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
    
    server.vm.provision "ansible" do |ansible|
      ansible.playbook = "ansible/provision.yml"
      ansible.inventory_path = "ansible/hosts"
      ansible.host_key_checking = "false"
      ansible.limit = "all"
    end
  end

  # Настройка PXE-клиента
  config.vm.define "pxeclient" do |client|
    client.vm.box = "bento/ubuntu-22.04"
    client.vm.box_url = "https://vagrant.elab.pro/bento/ubuntu-22.04.box"
    client.vm.host_name = "pxeclient"
    
    # Подключение к внутренней сети PXE
    client.vm.network "private_network", 
      ip: "10.0.0.21", 
      virtualbox__intnet: "pxenet"
    
    client.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = 2
      
      # Первый интерфейс — внутренняя сеть pxenet (для PXE)
      vb.customize ["modifyvm", :id, "--nic1", "intnet"]
      vb.customize ["modifyvm", :id, "--intnet1", "pxenet"]
      
      # Второй интерфейс — NAT (для доступа в интернет после установки)
      vb.customize ["modifyvm", :id, "--nic2", "nat"]
      
      # Отключаем звук
      vb.customize ["modifyvm", :id, "--audio", "none"]
      
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
  end
  
end
