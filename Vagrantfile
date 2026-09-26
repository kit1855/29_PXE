# -*- mode: ruby -*-
# vim: set ft=ruby :

# Указываем зеркало для скачивания образов
ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'

Vagrant.configure("2") do |config|
  config.vm.box_check_update = false
  # ============================================
  # 1. pxeser (Ubuntu 22.04)
  # ============================================
  config.vm.define "pxeser" do |pxeser|
    pxeser.vm.box = "ubuntu/jammy64"
    pxeser.vm.box_version = "1.0.0"
    pxeser.vm.hostname = "pxeser"

    pxeser.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    end

    # Проброс порта веб-сервера (для проверки с хоста)
    pxeser.vm.network "forwarded_port", guest: 80, host: 8080

    # Внутренняя сеть PXE (та же, что у клиента)
    pxeser.vm.network "private_network", ip: "10.0.0.20", adapter: 2, netmask: "255.255.255.0", virtualbox__intnet: "pxenet"

    # Дополнительная сеть (для доступа к хосту / интернету)
    pxeser.vm.network "private_network", ip: "192.168.56.10", adapter: 3, netmask: "255.255.255.0"

    pxeser.vm.provision "shell",
      run: "always",
      inline: <<-SHELL
        echo "pxeser is up"
        sudo ufw allow 22/tcp       # открываю порт для SSH
        sudo ufw allow 67/udp       # открываю порт для DHCP сервера
        sudo ufw allow 69/udp       # открываю порт для TFTP
        sudo ufw allow 80/tcp       # открываю порт для apache2
        sudo ufw --force enable
        sudo apt update
        sudo apt install -y dnsmasq apache2 syslinux pxelinux

sudo mkdir -p /srv/ks
sudo tee /etc/apache2/sites-available/ks-server.conf > /dev/null <<'EOF'
<VirtualHost 10.0.0.20:80>
    DocumentRoot /
    <Directory /srv/images>
        Options Indexes MultiViews
        AllowOverride All
        Require all granted
    </Directory>
    <Directory /srv/ks>
        Options Indexes MultiViews
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOF
sudo a2ensite ks-server.conf
sudo systemctl reload apache2

sudo tee /srv/ks/user-data > /dev/null <<'EOF'
#cloud-config
autoinstall:
  version: 1
  apt:
    primary:
      - arches: [amd64, i386]
        uri: http://archive.ubuntu.com/ubuntu
  identity:
    hostname: ubuntu-pxe
    username: otus
    password: "$6$xyz$73Q3Z.l5kN5BNAGMmP5IKozhqw3Zhj8bqQuJy3.Wf44.I3/nkSnzPMeX6rozvFiDHgi2DIt/BOc/lt14/2PH91"
  keyboard:
    layout: us
  locale: en_US.UTF-8
  network:
    version: 2
    ethernets:
      enp0s3:
        dhcp4: true
      enp0s8:
        dhcp4: true
  ssh:
    install-server: true
    allow-pw: true
  updates: security
EOF
sudo touch /srv/ks/meta-data


        sudo tee /etc/dnsmasq.d/pxe.conf > /dev/null <<'EOF'
interface=enp0s8
bind-interfaces
port=0
dhcp-range=enp0s8,10.0.0.100,10.0.0.120,255.255.255.0,12h
dhcp-boot=pxelinux.0
enable-tftp
tftp-root=/srv/tftp/amd64
EOF


    echo "=== Начинаю копирование ISO (3.17 ГБ). Это может занять 1-3 минуты ==="
    # Копируем ISO, если его ещё нет
    if [ ! -f /srv/images/ubuntu-24.04.4-live-server-amd64.iso ]; then
      sudo mkdir -p /srv/images
      sudo cp /vagrant/images/ubuntu-24.04.4-live-server-amd64.iso /srv/images/
      echo "=== ISO скопирован ==="
    else
      echo "=== ISO уже на месте, копирование не требуется ==="
    fi 

    sudo mkdir -p /srv/tftp/amd64
    sudo cp /usr/lib/PXELINUX/pxelinux.0 /srv/tftp/amd64/
    sudo cp /usr/lib/syslinux/modules/bios/ldlinux.c32 /srv/tftp/amd64/

    sudo mkdir -p /mnt/iso
    sudo mount -o loop /srv/images/ubuntu-24.04.4-live-server-amd64.iso /mnt/iso
    sudo cp /mnt/iso/casper/vmlinuz /srv/tftp/amd64/linux
    sudo cp /mnt/iso/casper/initrd /srv/tftp/amd64/initrd
    sudo umount /mnt/iso


    sudo mkdir -p /srv/tftp/amd64/pxelinux.cfg
    sudo tee /srv/tftp/amd64/pxelinux.cfg/default > /dev/null <<'EOF'
DEFAULT install
LABEL install
    KERNEL linux
    INITRD initrd
    APPEND root=/dev/ram0 ramdisk_size=8388608 ip=dhcp url=http://10.0.0.20/srv/images/ubuntu-24.04.4-live-server-amd64.iso autoinstall cloud-config-url=/dev/null ds=nocloud-net;s=http://10.0.0.20/srv/ks/
EOF
      sudo systemctl restart dnsmasq
      sudo systemctl restart apache2
      SHELL
  end

  # ============================================
  # 2. pxecli (Ubuntu 22.04, загрузка по сети)
  # ============================================
  config.vm.define "pxecli" do |pxecli|
    pxecli.vm.box = "ubuntu/jammy64"
    pxecli.vm.box_version = "1.0.0"
    pxecli.vm.hostname = "pxecli"
    pxecli.vm.disk :disk, size: "50GB", primary: true
    pxecli.vm.provider "virtualbox" do |v|
      v.memory = 8192
      v.cpus = 2
      v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]

      # Порядок загрузки: сначала сеть (PXE), затем диск, остальное выключено
      v.customize ["modifyvm", :id, "--boot1", "net"]
      v.customize ["modifyvm", :id, "--boot2", "disk"]
      v.customize ["modifyvm", :id, "--boot3", "none"]
      v.customize ["modifyvm", :id, "--boot4", "none"]

      # ВАЖНО: привязываем первый сетевой адаптер к внутренней сети pxenet
      v.customize ["modifyvm", :id, "--nic1", "intnet"]
      v.customize ["modifyvm", :id, "--intnet1", "pxenet"]

      # Второй адаптер — NAT (для интернета после установки)
      v.customize ["modifyvm", :id, "--nic2", "nat"]
    end

    # Внутренняя сеть PXE — та же, что у сервера
    pxecli.vm.network "private_network", ip: "10.0.0.21", adapter: 3, netmask: "255.255.255.0", virtualbox__intnet: "pxenet"

  end

end
