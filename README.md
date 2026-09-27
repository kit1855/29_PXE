# 29_PXE

1. запустить ВМ pxeser и дождаться окончания:
vagrant up pxeser

2. запустить ВМ pxeсli:
vagrant up pxecli

3. дождаться отображения в терминале строки с текстом "==> pxecli: Booting VM..." и выключить ВМ. Зайти в настройки ВМ pxecli в виртуалбоксе и отключить второй сетевой адаптер (NAT). Это нужно для того, чтоб ВМ pxecli получала ip от dnsmasq на ВМ pxeser, а не от NAT виртуалбокса.

4. вручную в виртуалбоксе запустить ВМ pxecli, начнётся установка операционной системы по PXE.

5. дождаться момента, когда ВМ начнёт перезагружаться после установки операционной системы и выключить ВМ. В виртуалбоксе зайти в настройки ВМ, во вкладке "система" в блоке "порядок загрузки" отключить сеть, как источник загрузки операционной системы. Должен остаться единственный источник загрузки операционной системы - диск. Если этого не сделать, пойдёт повторная установка операционной системы через PXE.

6. у меня что-то с системой случилось и получил проблему с подключением к ВМ по ssh. Для решения проблемы выполнил в обычной командной строке виндовс следующие команды:
```C:\Users\Lenovo>icacls "D:\linux\Professional\29 DHCP, PXE\dz14\.vagrant\machines\pxeser\virtualbox\private_key" /inheritance:r
обработанный файл: D:\linux\Professional\29 DHCP, PXE\dz14\.vagrant\machines\pxeser\virtualbox\private_key
Успешно обработано 1 файлов; не удалось обработать 0 файлов

C:\Users\Lenovo>icacls "D:\linux\Professional\29 DHCP, PXE\dz14\.vagrant\machines\pxeser\virtualbox\private_key" /grant:r "%USERNAME%:R"
обработанный файл: D:\linux\Professional\29 DHCP, PXE\dz14\.vagrant\machines\pxeser\virtualbox\private_key
Успешно обработано 1 файлов; не удалось обработать 0 файлов
```
7. подключиться к ВМ pxeser, зайти по ssh на ВМ pxecli и сделать все проверки.

8. проверки:

# подключение из ВМ pxeser по ssh к ВМ pxecli
```
vagrant@pxeser:~$ ssh otus@10.0.0.110
The authenticity of host '10.0.0.110 (10.0.0.110)' can't be established.
ED25519 key fingerprint is SHA256:voM9sjzBLvfKrZ9uNlbBgOsQqhZyCfNaWbV+xHl7QRQ.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.0.110' (ED25519) to the list of known hosts.
otus@10.0.0.110's password:
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-100-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sat Sep 26 04:26:14 PM UTC 2026

  System load:  0.37               Processes:               124
  Usage of /:   27.7% of 23.45GB   Users logged in:         1
  Memory usage: 2%                 IPv4 address for enp0s3: 10.0.0.110
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
```
# проверки ВМ pxecli
```
otus@ubuntu-pxe:~$ hostname
ubuntu-pxe
otus@ubuntu-pxe:~$ whoami
otus
otus@ubuntu-pxe:~$ cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=noble
LOGO=ubuntu-logo
otus@ubuntu-pxe:~$ ip -br a
lo               UNKNOWN        127.0.0.1/8 ::1/128
enp0s3           UP             10.0.0.110/24 metric 100 fe80::44:a4ff:fe14:4581/64
enp0s9           DOWN
otus@ubuntu-pxe:~$ systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/usr/lib/systemd/system/ssh.service; disabled; preset:>
     Active: active (running) since Sat 2026-09-26 16:26:08 UTC; 1min 18s a>
TriggeredBy: ● ssh.socket
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 1323 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUC>
   Main PID: 1325 (sshd)
      Tasks: 1 (limit: 9434)
     Memory: 4.0M (peak: 5.1M)
        CPU: 447ms
     CGroup: /system.slice/ssh.service
             └─1325 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startup>

Sep 26 16:26:07 ubuntu-pxe systemd[1]: Starting ssh.service - OpenBSD Secur>
Sep 26 16:26:08 ubuntu-pxe sshd[1325]: Server listening on 0.0.0.0 port 22.
Sep 26 16:26:08 ubuntu-pxe systemd[1]: Started ssh.service - OpenBSD Secure>
Sep 26 16:26:08 ubuntu-pxe sshd[1325]: Server listening on :: port 22.
Sep 26 16:26:13 ubuntu-pxe sshd[1326]: Accepted password for otus from 10.0>
Sep 26 16:26:13 ubuntu-pxe sshd[1326]: pam_unix(sshd:session): session open>
otus@ubuntu-pxe:~$ lsblk
NAME                      MAJ:MIN RM SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0  50G  0 disk
├─sda1                      8:1    0   1M  0 part
├─sda2                      8:2    0   2G  0 part /boot
└─sda3                      8:3    0  48G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0  24G  0 lvm  /
sdb                         8:16   0  10M  0 disk
otus@ubuntu-pxe:~$ df -h /
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv   24G  6.6G   16G  30% /
otus@ubuntu-pxe:~$ dpkg -l | grep -c ^ii
679
otus@ubuntu-pxe:~$ uname -r
6.8.0-100-generic
otus@ubuntu-pxe:~$

otus@ubuntu-pxe:~$
otus@ubuntu-pxe:~$ exit
logout
Connection to 10.0.0.110 closed.
```
# проверки на VM pxeser
# статус dnsmasq
```
vagrant@pxeser:~$ sudo systemctl status dnsmasq
● dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server
     Loaded: loaded (/lib/systemd/system/dnsmasq.service; enabled; vendor p>
     Active: active (running) since Sat 2026-09-26 16:02:11 UTC; 26min ago
    Process: 4164 ExecStartPre=/etc/init.d/dnsmasq checkconfig (code=exited>
    Process: 4172 ExecStart=/etc/init.d/dnsmasq systemd-exec (code=exited, >
    Process: 4181 ExecStartPost=/etc/init.d/dnsmasq systemd-start-resolvcon>
   Main PID: 4180 (dnsmasq)
      Tasks: 1 (limit: 2323)
     Memory: 1.3M
        CPU: 32.604s
     CGroup: /system.slice/dnsmasq.service
             └─4180 /usr/sbin/dnsmasq -x /run/dnsmasq/dnsmasq.pid -u dnsmas>

Sep 26 16:17:40 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.110 0>
Sep 26 16:17:40 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.110 02:44>
Sep 26 16:24:03 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 02:44:a4:14>
Sep 26 16:24:03 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.110 02:>
Sep 26 16:24:03 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.110 0>
Sep 26 16:24:03 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.110 02:44>
Sep 26 16:24:07 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 02:44:a4:14>
Sep 26 16:24:07 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.110 02:>
Sep 26 16:24:07 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.110 0>
Sep 26 16:24:07 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.110 02:44>
```
# порты, которые слушают ndsmasq и tftp
```
vagrant@pxeser:~$ sudo ss -tunlp | grep -E ":67|:69"
udp   UNCONN 0      0                         0.0.0.0%enp0s8:67        0.0.0.0:*    users:(("dnsmasq",pid=4180,fd=4))
udp   UNCONN 0      0                              127.0.0.1:69        0.0.0.0:*    users:(("dnsmasq",pid=4180,fd=7))
udp   UNCONN 0      0                              10.0.0.20:69        0.0.0.0:*    users:(("dnsmasq",pid=4180,fd=6))
udp   UNCONN 0      0                                  [::1]:69           [::]:*    users:(("dnsmasq",pid=4180,fd=9))
udp   UNCONN 0      0      [fe80::a00:27ff:fe7c:4378]%enp0s8:69           [::]:*    users:(("dnsmasq",pid=4180,fd=8))
```
# статус apache2
```
vagrant@pxeser:~$ sudo systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled; vendor p>
     Active: active (running) since Sat 2026-09-26 16:02:12 UTC; 26min ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 4198 ExecStart=/usr/sbin/apachectl start (code=exited, status=>
   Main PID: 4202 (apache2)
      Tasks: 55 (limit: 2323)
     Memory: 968.1M
        CPU: 21.837s
     CGroup: /system.slice/apache2.service
             ├─4202 /usr/sbin/apache2 -k start
             ├─4203 /usr/sbin/apache2 -k start
             └─4204 /usr/sbin/apache2 -k start

Sep 26 16:02:12 pxeser systemd[1]: Starting The Apache HTTP Server...
Sep 26 16:02:12 pxeser apachectl[4201]: AH00558: apache2: Could not reliabl>
Sep 26 16:02:12 pxeser systemd[1]: Started The Apache HTTP Server.
```
# наличие образа
```
vagrant@pxeser:~$ ls -la /srv/images/
total 3325668
drwxr-xr-x 2 root root       4096 Sep 26 15:51 .
drwxr-xr-x 5 root root       4096 Sep 26 16:02 ..
-rwxr-xr-x 1 root root 3405469696 Sep 26 16:02 ubuntu-24.04.4-live-server-amd64.iso
```
# нетбут файлы
```
vagrant@pxeser:~$ ls -la /srv/tftp/amd64/
total 89532
drwxr-xr-x 3 root root     4096 Sep 26 16:02 .
drwxr-xr-x 3 root root     4096 Sep 26 16:02 ..
-r--r--r-- 1 root root 76466818 Sep 26 16:02 initrd
-rw-r--r-- 1 root root   119284 Sep 26 16:02 ldlinux.c32
-r--r--r-- 1 root root 15030664 Sep 26 16:02 linux
-rw-r--r-- 1 root root    42584 Sep 26 16:02 pxelinux.0
drwxr-xr-x 2 root root     4096 Sep 26 16:02 pxelinux.cfg
```
# конфиг загрузчика pxelinux
```
vagrant@pxeser:~$ cat /srv/tftp/amd64/pxelinux.cfg/default
DEFAULT install
LABEL install
    KERNEL linux
    INITRD initrd
    APPEND root=/dev/ram0 ramdisk_size=8388608 ip=dhcp url=http://10.0.0.20/srv/images/ubuntu-24.04.4-live-server-amd64.iso autoinstall cloud-config-url=/dev/null ds=nocloud-net;s=http://10.0.0.20/srv/ks/
```
# файл автоустановки с логином, паролем, хостнэйм, сетью и ssh
```
vagrant@pxeser:~$ cat /srv/ks/user-data
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
```
# проверка наличия обязательного файла для cloud-init. Он пустой, но без него возможны проблемы.
```
vagrant@pxeser:~$ cat /srv/ks/meta-data
```
# проверка, что можно apache2 отдаёт файлы по http
```
vagrant@pxeser:~$ curl -I http://10.0.0.20/srv/images/ubuntu-24.04.4-live-server-amd64.iso
HTTP/1.1 200 OK
Date: Sat, 26 Sep 2026 16:29:38 GMT
Server: Apache/2.4.52 (Ubuntu)
Last-Modified: Sat, 26 Sep 2026 16:02:09 GMT
ETag: "cafb5800-65c64f4882851"
Accept-Ranges: bytes
Content-Length: 3405469696
Content-Type: application/x-iso9660-image
```
```
vagrant@pxeser:~$ curl -I http://10.0.0.20/srv/ks/user-data
HTTP/1.1 200 OK
Date: Sat, 26 Sep 2026 16:29:44 GMT
Server: Apache/2.4.52 (Ubuntu)
Last-Modified: Sat, 26 Sep 2026 15:51:28 GMT
ETag: "213-65c64ce5b4619"
Accept-Ranges: bytes
Content-Length: 531
```
# проверка логов журнала apache2
```
vagrant@pxeser:~$ sudo tail -20 /var/log/apache2/other_vhosts_access.log
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:15:50 +0000] "GET /srv/images/ubuntu-24.04.4-live-server-amd64.iso HTTP/1.1" 200 3405469974 "-" "Wget"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:44 +0000] "GET /srv/ks/meta-data HTTP/1.1" 200 256 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:44 +0000] "GET /srv/ks/user-data HTTP/1.1" 200 791 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:44 +0000] "GET /srv/ks/vendor-data HTTP/1.1" 404 488 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:45 +0000] "GET /srv/ks/vendor-data HTTP/1.1" 404 487 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:46 +0000] "GET /srv/ks/vendor-data HTTP/1.1" 404 487 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:47 +0000] "GET /srv/ks/vendor-data HTTP/1.1" 404 487 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:48 +0000] "GET /srv/ks/vendor-data HTTP/1.1" 404 487 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:49 +0000] "GET /srv/ks/vendor-data HTTP/1.1" 404 487 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:50 +0000] "GET /srv/ks/vendor-data HTTP/1.1" 404 487 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:51 +0000] "GET /srv/ks/vendor-data HTTP/1.1" 404 487 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:52 +0000] "GET /srv/ks/vendor-data HTTP/1.1" 404 487 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:53 +0000] "GET /srv/ks/vendor-data HTTP/1.1" 404 487 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.118 - - [26/Sep/2026:16:16:54 +0000] "GET /srv/ks/vendor-data HTTP/1.1" 404 487 "-" "Cloud-Init/25.2-0ubuntu1~24.04.1"
pxeser:80 10.0.0.20 - - [26/Sep/2026:16:29:38 +0000] "HEAD /srv/images/ubuntu-24.04.4-live-server-amd64.iso HTTP/1.1" 200 259 "-" "curl/7.81.0"
pxeser:80 10.0.0.20 - - [26/Sep/2026:16:29:44 +0000] "HEAD /srv/ks/user-data HTTP/1.1" 200 204 "-" "curl/7.81.0"
```
# проверка что dhcp и tftp работают
```
vagrant@pxeser:~$ sudo journalctl -u dnsmasq --since "30 minutes ago"
Sep 26 16:02:11 pxeser systemd[1]: Starting dnsmasq - A lightweight DHCP an>
Sep 26 16:02:11 pxeser dnsmasq[4180]: started, version 2.91 DNS disabled
Sep 26 16:02:11 pxeser dnsmasq[4180]: compile time options: IPv6 GNU-getopt>
Sep 26 16:02:11 pxeser dnsmasq-dhcp[4180]: DHCP, IP range 10.0.0.100 -- 10.>
Sep 26 16:02:11 pxeser dnsmasq-dhcp[4180]: DHCP, sockets bound exclusively >
Sep 26 16:02:11 pxeser dnsmasq-tftp[4180]: TFTP root is /srv/tftp/amd64
Sep 26 16:02:11 pxeser systemd[1]: Started dnsmasq - A lightweight DHCP and>
Sep 26 16:13:56 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 02:44:a4:14>
Sep 26 16:13:56 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.108 02:>
Sep 26 16:13:56 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 02:44:a4:14>
Sep 26 16:13:56 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.108 02:>
Sep 26 16:13:56 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 02:44:a4:14>
Sep 26 16:13:56 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.108 02:>
Sep 26 16:13:56 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.108 0>
Sep 26 16:13:56 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.108 02:44>
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: sent /srv/tftp/amd64/pxelinux.0 >
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: sent /srv/tftp/amd64/ldlinux.c32>
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: file /srv/tftp/amd64/pxelinux.cf>
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: file /srv/tftp/amd64/pxelinux.cf>
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: file /srv/tftp/amd64/pxelinux.cf>
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: file /srv/tftp/amd64/pxelinux.cf>
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: file /srv/tftp/amd64/pxelinux.cf>
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: file /srv/tftp/amd64/pxelinux.cf>
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: file /srv/tftp/amd64/pxelinux.cf>
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: file /srv/tftp/amd64/pxelinux.cf>
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: file /srv/tftp/amd64/pxelinux.cf>
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: file /srv/tftp/amd64/pxelinux.cf>
Sep 26 16:13:56 pxeser dnsmasq-tftp[4180]: sent /srv/tftp/amd64/pxelinux.cf>
Sep 26 16:14:10 pxeser dnsmasq-tftp[4180]: sent /srv/tftp/amd64/linux to 10>
Sep 26 16:15:27 pxeser dnsmasq-tftp[4180]: sent /srv/tftp/amd64/initrd to 1>
Sep 26 16:15:43 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 08:00:27:6b>
Sep 26 16:15:43 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.118 08:>
Sep 26 16:15:46 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 02:44:a4:14>
Sep 26 16:15:46 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.109 02:>
Sep 26 16:15:46 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.118 0>
Sep 26 16:15:46 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.118 08:00>
Sep 26 16:15:46 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.109 0>
Sep 26 16:15:46 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.109 02:44>
Sep 26 16:16:48 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 02:44:a4:14>
Sep 26 16:16:48 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.110 0>
Sep 26 16:16:48 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.110 02:44>
Sep 26 16:16:51 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 10.0.0.118 >
Sep 26 16:16:51 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.119 08:>
Sep 26 16:16:51 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.119 0>
Sep 26 16:16:51 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.119 08:00>
Sep 26 16:16:55 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 02:44:a4:14>
Sep 26 16:16:55 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.110 02:>
Sep 26 16:16:55 pxeser dnsmasq-dhcp[4180]: DHCPNAK(enp0s8) 10.0.0.118 08:00>
Sep 26 16:16:55 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.110 0>
Sep 26 16:16:55 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.110 02:44>
Sep 26 16:16:55 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 10.0.0.118 >
Sep 26 16:16:55 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.119 08:>
Sep 26 16:16:55 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.119 0>
Sep 26 16:16:55 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.119 08:00>
Sep 26 16:17:39 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 02:44:a4:14>
Sep 26 16:17:39 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.110 02:>
Sep 26 16:17:39 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.110 0>
Sep 26 16:17:39 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.110 02:44>
Sep 26 16:17:40 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 02:44:a4:14>
Sep 26 16:17:40 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.110 02:>
Sep 26 16:17:40 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.110 0>
Sep 26 16:17:40 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.110 02:44>
Sep 26 16:24:03 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 02:44:a4:14>
Sep 26 16:24:03 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.110 02:>
Sep 26 16:24:03 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.110 0>
Sep 26 16:24:03 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.110 02:44>
Sep 26 16:24:07 pxeser dnsmasq-dhcp[4180]: DHCPDISCOVER(enp0s8) 02:44:a4:14>
Sep 26 16:24:07 pxeser dnsmasq-dhcp[4180]: DHCPOFFER(enp0s8) 10.0.0.110 02:>
Sep 26 16:24:07 pxeser dnsmasq-dhcp[4180]: DHCPREQUEST(enp0s8) 10.0.0.110 0>
Sep 26 16:24:07 pxeser dnsmasq-dhcp[4180]: DHCPACK(enp0s8) 10.0.0.110 02:44>
```
