ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'

Vagrant.configure("2") do |config|
  
  # Настройка PXE-сервера
  config.vm.define "pxeserver" do |server|
    server.vm.box = "ubuntu/jammy64"
    server.vm.host_name = "pxeserver"
    
    # Проброс порта для веб-сервера
    server.vm.network "forwarded_port", guest: 80, host: 8080
    
    # Внутренняя сеть для PXE (сетевой мост pxenet)
    server.vm.network "private_network", 
      ip: "10.0.0.20", 
      virtualbox__intnet: "pxenet"
    
    # Дополнительная сеть для доступа в интернет (адаптер 3)
    server.vm.network "private_network", 
      ip: "192.168.50.10", 
      adapter: 3
    
    server.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
  end

  # Настройка PXE-клиента
  config.vm.define "pxeclient" do |client|
    client.vm.box = "ubuntu/jammy64"
    client.vm.host_name = "pxeclient"
    
    # Подключение к внутренней сети PXE
    client.vm.network "private_network", 
      ip: "10.0.0.21", 
      virtualbox__intnet: "pxenet"
    
    client.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.cpus = 2
      
      # НЕ ТРОГАЕМ первый интерфейс (NAT для SSH)!
      # Второй интерфейс настраиваем на внутреннюю сеть pxenet
      vb.customize ["modifyvm", :id, "--nic2", "intnet"]
      vb.customize ["modifyvm", :id, "--intnet2", "pxenet"]
      
      # Третий интерфейс — NAT (для доступа в интернет после установки)
      vb.customize ["modifyvm", :id, "--nic3", "nat"]
      
      # Отключаем звук
      vb.customize ["modifyvm", :id, "--audio", "none"]
      
      vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end
  end
  
end
