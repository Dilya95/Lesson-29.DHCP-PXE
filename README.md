# Домашнее задание 29: Настройка PXE сервера для автоматической установки

## Задания
Настроить загрузку по сети дистрибутива Ubuntu 24<br>
Установка должна проходить из HTTP-репозитория.<br>
Настроить автоматическую установку c помощью файла user-data<br>
Задания со звёздочкой*<br>
Настроить автоматическую загрузку по сети дистрибутива Ubuntu 24 c использованием UEFI<br>
Задания со звёздочкой выполняются по желанию<br>


## Структура
├── README.md<br>
└── Vagrantfile



## Выполнение

### ставим dnsmasq, делаем конфигурацию pxe, делаем директорию tftp и загружаем туда ubuntu24
```
vagrant@pxeserver:~$ sudo -i

root@pxeserver:~# systemctl stop ufw

root@pxeserver:~# systemctl disable ufw
Synchronizing state of ufw.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install disable ufw
Removed '/etc/systemd/system/multi-user.target.wants/ufw.service'.

root@pxeserver:~# sudo apt update


root@pxeserver:~# sudo apt install dnsmasq -y


root@pxeserver:~# vim /etc/dnsmasq.d/pxe.conf


root@pxeserver:~# cat /etc/dnsmasq.d/pxe.conf
#Указываем интерфейс в на котором будет работать DHCP/TFTP
interface=eth1
bind-interfaces
#Также указаваем интерфейс и range адресов которые будут выдаваться по DHCP
dhcp-range=eth1,10.0.0.100,10.0.0.120
#Имя файла, с которого надо начинать загрузку для Legacy boot (этот пример рассматривается в методичке)
dhcp-boot=pxelinux.0
#Имена файлов, для UEFI-загрузки (не обязательно добавлять)
dhcp-match=set:efi-x86_64,option:client-arch,7
dhcp-boot=tag:efi-x86_64,bootx64.efi
#Включаем TFTP-сервер
enable-tftp
#Указываем каталог для TFTP-сервера
tftp-root=/srv/tftp/amd64

root@pxeserver:~# mkdir -p /srv/tftp

root@pxeserver:~# cd /srv/tftp


root@pxeserver:/srv/tftp# wget http://cdimage.ubuntu.com/ubuntu-server/noble/daily-live/current/noble-netboot-amd64.tar.gz
--2026-07-30 10:59:25--  http://cdimage.ubuntu.com/ubuntu-server/noble/daily-live/current/noble-netboot-amd64.tar.gz
Resolving cdimage.ubuntu.com (cdimage.ubuntu.com)... 91.189.91.124, 185.125.190.40, 185.125.190.37, ...
Connecting to cdimage.ubuntu.com (cdimage.ubuntu.com)|91.189.91.124|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 101052383 (96M) [application/x-gzip]
Saving to: ‘noble-netboot-amd64.tar.gz’

noble-netboot-amd64.tar. 100%[================================>]  96.37M  1.68MB/s    in 45s     

2026-07-30 11:00:11 (2.13 MB/s) - ‘noble-netboot-amd64.tar.gz’ saved [101052383/101052383]


root@pxeserver:/srv/tftp# tar -xzvf noble-netboot-amd64.tar.gz -C /srv/tftp
./
./amd64/
./amd64/initrd
./amd64/pxelinux.cfg/
./amd64/pxelinux.cfg/default
./amd64/ldlinux.c32
./amd64/grubx64.efi
./amd64/bootx64.efi
./amd64/pxelinux.0
./amd64/linux
./amd64/grub/
./amd64/grub/grub.cfg



root@pxeserver:/srv/tftp# tree ./amd64
./amd64
├── bootx64.efi
├── grub
│   └── grub.cfg
├── grubx64.efi
├── initrd
├── ldlinux.c32
├── linux
├── pxelinux.0
└── pxelinux.cfg
    └── default

3 directories, 8 files


root@pxeserver:/srv/tftp# systemctl restart dnsmasq

root@pxeserver:/srv/tftp# systemctl status dnsmasq
● dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server
     Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-07-30 11:02:50 UTC; 49s ago
 Invocation: 7d8c357e420a410fbf19985a0017cd76
       Docs: man:dnsmasq(8)
    Process: 17351 ExecStartPre=/usr/share/dnsmasq/systemd-helper checkconfig (code=exited, status=0/SUCCESS)
    Process: 17357 ExecStart=/usr/share/dnsmasq/systemd-helper exec (code=exited, status=0/SUCCESS)
    Process: 17363 ExecStartPost=/usr/share/dnsmasq/systemd-helper start-resolvconf (code=exited, status=0/SUCCESS)
   Main PID: 17362 (dnsmasq)
      Tasks: 1 (limit: 992)
     Memory: 3.1M (peak: 6.3M)
        CPU: 26ms
     CGroup: /system.slice/dnsmasq.service
             └─17362 /usr/sbin/dnsmasq -x /run/dnsmasq/dnsmasq.pid -u dnsmasq -r /run/dnsmasq/resolv.conf -7 /etc/dnsmasq.d,.dpkg-dist,.dpkg-old,.dpkg-new --local-service --trust-anchor=.,20326,8,2,E06D44B80B8F1D39A95C0B0D7C65D08458E8>

Jul 30 11:02:50 pxeserver dnsmasq-tftp[17362]: TFTP root is /srv/tftp/amd64
Jul 30 11:02:50 pxeserver dnsmasq[17362]: read /etc/hosts - 10 names
Jul 30 11:02:50 pxeserver systemd[1]: Started dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server.
Jul 30 11:02:50 pxeserver resolvconf[17370]: Dropped protocol specifier '.dnsmasq' from 'lo.dnsmasq'. Using 'lo' (ifindex=1).
Jul 30 11:02:50 pxeserver resolvconf[17370]: Failed to set DNS configuration: Link lo is loopback device.
Jul 30 11:03:21 pxeserver dnsmasq-dhcp[17362]: DHCPDISCOVER(eth1) 08:00:27:87:9c:74
Jul 30 11:03:21 pxeserver dnsmasq-dhcp[17362]: DHCPOFFER(eth1) 10.0.0.105 08:00:27:87:9c:74
Jul 30 11:03:21 pxeserver dnsmasq-dhcp[17362]: DHCPREQUEST(eth1) 10.0.0.105 08:00:27:87:9c:74
Jul 30 11:03:21 pxeserver dnsmasq-dhcp[17362]: DHCPACK(eth1) 10.0.0.105 08:00:27:87:9c:74 vagrant
Jul 30 11:03:21 pxeserver dnsmasq-dhcp[17362]: not giving name vagrant to the DHCP lease of 10.0.0.105 because the name exists in /etc/hosts with address 127.0.1.1

```

### ставим apache2, делаем директорию /srv/images и загружаем туда ubuntu24 iso, делаем конфигурацию apache2 и pxelinux.cfg
```

root@pxeserver:/srv/tftp# sudo apt install apache2 -y

root@pxeserver:/srv/tftp# mkdir /srv/images

root@pxeserver:/srv/tftp# cd /srv/images

root@pxeserver:/srv/images# wget http://cdimage.ubuntu.com/ubuntu-server/noble/daily-live/current/noble-live-server-amd64.iso
--2026-07-30 11:06:25--  http://cdimage.ubuntu.com/ubuntu-server/noble/daily-live/current/noble-live-server-amd64.iso
Resolving cdimage.ubuntu.com (cdimage.ubuntu.com)... 91.189.91.124, 185.125.190.40, 185.125.190.37, ...
Connecting to cdimage.ubuntu.com (cdimage.ubuntu.com)|91.189.91.124|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3807281152 (3.5G) [application/x-iso9660-image]
Saving to: ‘noble-live-server-amd64.iso’

noble-live-server-amd64.iso    100%[=================================================>]   3.54G  1.63MB/s    in 37m 7s  

2026-07-30 11:43:33 (1.63 MB/s) - ‘noble-live-server-amd64.iso’ saved [3807281152/3807281152]


root@pxeserver:/srv/images# vim /etc/apache2/sites-available/ks-server.conf

root@pxeserver:/srv/images# cat /etc/apache2/sites-available/ks-server.conf
#Указываем IP-адрес хоста и порт на котором будет работать Web-сервер
<VirtualHost 10.0.0.20:80>
DocumentRoot /
# Указываем директорию /srv/images из которой будет загружаться iso-образ
<Directory /srv/images>
Options Indexes MultiViews
AllowOverride All
Require all granted
</Directory>
</VirtualHost>

root@pxeserver:/srv/images# sudo a2ensite ks-server.conf
Enabling site ks-server.
To activate the new configuration, you need to run:
  systemctl reload apache2

root@pxeserver:/srv/images# systemctl reload apache2

root@pxeserver:/srv/images# vim /srv/tftp/amd64/pxelinux.cfg/default

root@pxeserver:/srv/images# cat /srv/tftp/amd64/pxelinux.cfg/default
DEFAULT install
PROMPT 0
TIMEOUT 10

LABEL install
  KERNEL linux
  INITRD initrd
  APPEND root=/dev/ram0 ramdisk_size=5000000 ip=:::::enp0s3:dhcp url=http://10.0.0.20/srv/images/noble-live-server-amd64.iso autoinstall ds=nocloud-net;s=http://10.0.0.20/srv/ks/



root@pxeserver:/srv/images# systemctl restart apache2


```

### 
```

root@pxeserver:/srv/images# mkdir /srv/ks


root@pxeserver:/srv/images# vim /srv/ks/user-data

root@pxeserver:/srv/images# cat /srv/ks/user-data
#cloud-config
autoinstall:
  version: 1
  apt:
    disable_components: []
    geoip: true
    preserve_sources_list: false
    primary:
      - arches:
          - amd64
          - i386
        uri: http://us.archive.ubuntu.com/ubuntu
      - arches:
          - default
        uri: http://ports.ubuntu.com/ubuntu-ports
  drivers:
    install: false
  identity:
    hostname: linux
    password: $6$sJgo6Hg5zXBwkkI8$btrEoWAb5FxKhajagWR49XM4EAOfO/Dr5bMrLOkGe3KkMYdsh7T3MU5mYwY2TIMJpVKckAwnZFs2ltUJ1abOZ.
    realname: otus
    username: otus
  kernel:
    package: linux-generic
  keyboard:
    layout: us
    toggle: null
    variant: ''
  locale: en_US.UTF-8
  network:
    ethernets:
      enp0s3:
        dhcp4: true
      #enp0s8:
      #  dhcp4: true
    version: 2
  ssh:
    allow-pw: true
    authorized-keys: []
    install-server: true
  updates: security


root@pxeserver:/srv/images# touch /srv/ks/meta-data

root@pxeserver:/srv/images# vim /etc/apache2/sites-available/ks-server.conf

root@pxeserver:/srv/images# cat /etc/apache2/sites-available/ks-server.conf
#Указываем IP-адрес хоста и порт на котором будет работать Web-сервер
<VirtualHost 10.0.0.20:80>
DocumentRoot /
<Directory /srv/ks>
Options Indexes MultiViews
AllowOverride All
Require all granted
</Directory>

# Указываем директорию /srv/images из которой будет загружаться iso-образ
<Directory /srv/images>
Options Indexes MultiViews
AllowOverride All
Require all granted
</Directory>
</VirtualHost>


root@pxeserver:/srv/images# vim /srv/tftp/amd64/pxelinux.cfg/default

root@pxeserver:/srv/images# cat /srv/tftp/amd64/pxelinux.cfg/default
DEFAULT install
LABEL install
  KERNEL linux
  INITRD initrd
  APPEND root=/dev/ram0 ramdisk_size=3000000 ip=dhcp iso-url=http://10.0.0.20/srv/images/noble-live-server-amd64.iso autoinstall ds=nocloud-net;s=http://10.0.0.20/srv/ks/



root@pxeserver:/srv/images# systemctl restart dnsmasq

root@pxeserver:/srv/images# systemctl restart apache2

root@pxeserver:/srv/images# systemctl status dnsmasq
● dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server
     Loaded: loaded (/usr/lib/systemd/system/dnsmasq.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-07-30 11:56:50 UTC; 12s ago
 Invocation: c8898c7e7d9044e4b0933e5a68aa37ed
       Docs: man:dnsmasq(8)
    Process: 25356 ExecStartPre=/usr/share/dnsmasq/systemd-helper checkconfig (code=exited, status=0/SUCCESS)
    Process: 25363 ExecStart=/usr/share/dnsmasq/systemd-helper exec (code=exited, status=0/SUCCESS)
    Process: 25369 ExecStartPost=/usr/share/dnsmasq/systemd-helper start-resolvconf (code=exited, status=0/SUCCESS)
   Main PID: 25368 (dnsmasq)
      Tasks: 1 (limit: 992)
     Memory: 748K (peak: 3.4M)
        CPU: 25ms
     CGroup: /system.slice/dnsmasq.service
             └─25368 /usr/sbin/dnsmasq -x /run/dnsmasq/dnsmasq.pid -u dnsmasq -r /run/dnsmasq/resolv.conf -7 /etc/dnsmasq.d,.dpkg-dist,.dpkg->

Jul 30 11:56:50 pxeserver dnsmasq[25368]: started, version 2.92 cachesize 150
Jul 30 11:56:50 pxeserver dnsmasq[25368]: compile time options: IPv6 GNU-getopt DBus no-UBus i18n IDN2 DHCP DHCPv6 no-Lua TFTP conntrack ipse>
Jul 30 11:56:50 pxeserver dnsmasq-dhcp[25368]: DHCP, IP range 10.0.0.100 -- 10.0.0.120, lease time 1h
Jul 30 11:56:50 pxeserver dnsmasq-dhcp[25368]: DHCP, sockets bound exclusively to interface eth1
Jul 30 11:56:50 pxeserver dnsmasq-tftp[25368]: TFTP root is /srv/tftp/amd64
Jul 30 11:56:50 pxeserver dnsmasq[25368]: read /etc/hosts - 10 names
Jul 30 11:56:50 pxeserver dnsmasq-dhcp[25368]: not giving name vagrant to the DHCP lease of 10.0.0.105 because the name exists in /etc/hosts >
Jul 30 11:56:50 pxeserver systemd[1]: Started dnsmasq.service - dnsmasq - A lightweight DHCP and caching DNS server.
Jul 30 11:56:50 pxeserver resolvconf[25377]: Dropped protocol specifier '.dnsmasq' from 'lo.dnsmasq'. Using 'lo' (ifindex=1).
Jul 30 11:56:50 pxeserver resolvconf[25377]: Failed to set DNS configuration: Link lo is loopback device.


root@pxeserver:/srv/images# systemctl status apache2
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/apache2.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-07-30 11:56:55 UTC; 18s ago
 Invocation: bd36caf698cb41a296e997efd3927d4d
       Docs: https://httpd.apache.org/docs/2.4/
   Main PID: 25397 (apache2)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 55 (limit: 992)
     Memory: 65.4M (peak: 65.4M)
        CPU: 49ms
     CGroup: /system.slice/apache2.service
             ├─25397 /usr/sbin/apache2 -k start -DFOREGROUND
             ├─25400 /usr/sbin/apache2 -k start -DFOREGROUND
             └─25401 /usr/sbin/apache2 -k start -DFOREGROUND

Jul 30 11:56:55 pxeserver systemd[1]: Starting apache2.service - The Apache HTTP Server...
Jul 30 11:56:55 pxeserver apachectl[25397]: AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 12>
Jul 30 11:56:55 pxeserver systemd[1]: Started apache2.service - The Apache HTTP Server.


```

### удаляем ВМ pxeclient и создаем заново
```
[root@otus-homework pxe]# vagrant destroy -f pxeclient
==> pxeclient: Destroying VM and associated drives...

[root@otus-homework pxe]# vagrant up pxeclient
Bringing machine 'pxeclient' up with 'virtualbox' provider...

root@pxeserver:/tmp# journalctl -u dnsmasq -f
Jul 31 11:52:07 pxeserver dnsmasq-dhcp[7454]: DHCPDISCOVER(eth1) 08:00:27:0a:4e:27
Jul 31 11:52:07 pxeserver dnsmasq-dhcp[7454]: DHCPOFFER(eth1) 10.0.0.107 08:00:27:0a:4e:27
Jul 31 11:52:08 pxeserver dnsmasq-dhcp[7454]: DHCPDISCOVER(eth1) 08:00:27:0a:4e:27
Jul 31 11:52:08 pxeserver dnsmasq-dhcp[7454]: DHCPOFFER(eth1) 10.0.0.107 08:00:27:0a:4e:27
Jul 31 11:52:10 pxeserver dnsmasq-dhcp[7454]: DHCPREQUEST(eth1) 10.0.0.107 08:00:27:0a:4e:27
Jul 31 11:52:10 pxeserver dnsmasq-dhcp[7454]: DHCPACK(eth1) 10.0.0.107 08:00:27:0a:4e:27
Jul 31 11:52:10 pxeserver dnsmasq-tftp[7454]: sent /srv/tftp/amd64/pxelinux.0 to 10.0.0.107
Jul 31 11:52:11 pxeserver dnsmasq-tftp[7454]: file /srv/tftp/amd64/pxelinux.cfg/5abb3516-1c91-4a3d-8699-0b40ffcbd9cc not found for 10.0.0.107
Jul 31 11:52:11 pxeserver dnsmasq-tftp[7454]: sent /srv/tftp/amd64/ldlinux.c32 to 10.0.0.107
Jul 31 11:52:11 pxeserver dnsmasq-tftp[7454]: file /srv/tftp/amd64/pxelinux.cfg/01-08-00-27-0a-4e-27 not found for 10.0.0.107
Jul 31 11:52:11 pxeserver dnsmasq-tftp[7454]: file /srv/tftp/amd64/pxelinux.cfg/0A00006B not found for 10.0.0.107
Jul 31 11:52:11 pxeserver dnsmasq-tftp[7454]: file /srv/tftp/amd64/pxelinux.cfg/0A00006 not found for 10.0.0.107
Jul 31 11:52:11 pxeserver dnsmasq-tftp[7454]: file /srv/tftp/amd64/pxelinux.cfg/0A0000 not found for 10.0.0.107
Jul 31 11:52:11 pxeserver dnsmasq-tftp[7454]: file /srv/tftp/amd64/pxelinux.cfg/0A000 not found for 10.0.0.107
Jul 31 11:52:11 pxeserver dnsmasq-tftp[7454]: file /srv/tftp/amd64/pxelinux.cfg/0A00 not found for 10.0.0.107
Jul 31 11:52:11 pxeserver dnsmasq-tftp[7454]: file /srv/tftp/amd64/pxelinux.cfg/0A0 not found for 10.0.0.107
Jul 31 11:52:11 pxeserver dnsmasq-tftp[7454]: file /srv/tftp/amd64/pxelinux.cfg/0A not found for 10.0.0.107
Jul 31 11:52:11 pxeserver dnsmasq-tftp[7454]: file /srv/tftp/amd64/pxelinux.cfg/0 not found for 10.0.0.107
Jul 31 11:52:11 pxeserver dnsmasq-tftp[7454]: sent /srv/tftp/amd64/pxelinux.cfg/default to 10.0.0.107
Jul 31 11:53:04 pxeserver dnsmasq-tftp[7454]: sent /srv/tftp/amd64/linux to 10.0.0.107
Jul 31 11:57:10 pxeserver dnsmasq-tftp[7454]: sent /srv/tftp/amd64/initrd to 10.0.0.107
Jul 31 11:57:32 pxeserver dnsmasq-dhcp[7454]: DHCPDISCOVER(eth1) 08:00:27:0a:4e:27
Jul 31 11:57:32 pxeserver dnsmasq-dhcp[7454]: DHCPOFFER(eth1) 10.0.0.108 08:00:27:0a:4e:27
Jul 31 11:57:32 pxeserver dnsmasq-dhcp[7454]: DHCPREQUEST(eth1) 10.0.0.108 08:00:27:0a:4e:27
Jul 31 11:57:32 pxeserver dnsmasq-dhcp[7454]: DHCPACK(eth1) 10.0.0.108 08:00:27:0a:4e:27


Jul 31 12:11:20 pxeserver dnsmasq-dhcp[7454]: DHCPDISCOVER(eth1) 08:00:27:0a:4e:27
Jul 31 12:11:20 pxeserver dnsmasq-dhcp[7454]: DHCPOFFER(eth1) 10.0.0.109 08:00:27:0a:4e:27
Jul 31 12:11:20 pxeserver dnsmasq-dhcp[7454]: DHCPREQUEST(eth1) 10.0.0.109 08:00:27:0a:4e:27
Jul 31 12:11:20 pxeserver dnsmasq-dhcp[7454]: DHCPACK(eth1) 10.0.0.109 08:00:27:0a:4e:27 linux



root@pxeserver:/var/log/apache2# tail -f /var/log/apache2/other_vhosts_access.log
pxeserver:80 10.0.0.108 - - [31/Jul/2026:11:28:00 +0000] "GET /srv/images/noble-live-server-amd64.iso HTTP/1.1" 200 3807281430 "-" "Wget"
pxeserver:80 10.0.0.107 - - [31/Jul/2026:11:31:32 +0000] "GET /srv/ks/meta-data HTTP/1.1" 200 256 "-" "Cloud-Init/26.1-0ubuntu1~24.04.1"
pxeserver:80 10.0.0.107 - - [31/Jul/2026:11:31:32 +0000] "GET /srv/ks/user-data HTTP/1.1" 200 1086 "-" "Cloud-Init/26.1-0ubuntu1~24.04.1"
pxeserver:80 10.0.0.107 - - [31/Jul/2026:11:31:32 +0000] "GET /srv/ks/vendor-data HTTP/1.1" 200 256 "-" "Cloud-Init/26.1-0ubuntu1~24.04.1"
pxeserver:80 10.0.0.108 - - [31/Jul/2026:11:57:37 +0000] "GET /srv/images/noble-live-server-amd64.iso HTTP/1.1" 200 3807281430 "-" "Wget"

```

### Смотрим список ВМ, выключаем ВМ pxeclient, меняем порядок загрузки на загрузку с диска и включаем ВМ обратно
```

[root@otus-homework pxe]# VBoxManage list vms
"pxe_pxeserver_1785410649678_23872" {d73f4b61-3b9b-4441-8a2d-4b3b7e83196b}
"pxe_pxeclient_1785496906878_71866" {5abb3516-1c91-4a3d-8699-0b40ffcbd9cc}


[root@otus-homework pxe]# VBoxManage controlvm "pxe_pxeclient_1785496906878_71866" poweroff
0%...10%...20%...30%...40%...50%...60%...70%...80%...90%...100%

[root@otus-homework pxe]# vagrant status
Current machine states:

pxeserver                 running (virtualbox)
pxeclient                 poweroff (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.


[root@otus-homework pxe]# VBoxManage modifyvm "pxe_pxeclient_1785496906878_71866" \
>   --boot1 disk \
>   --boot2 net \
>   --boot3 none \
>   --boot4 none

[root@otus-homework pxe]# VBoxManage startvm "pxe_pxeclient_1785496906878_71866" --type headless
Waiting for VM "pxe_pxeclient_1785496906878_71866" to power on...
VM "pxe_pxeclient_1785496906878_71866" has been successfully started.

[root@otus-homework pxe]# vagrant status
Current machine states:

pxeserver                 running (virtualbox)
pxeclient                 running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.

```

### Дожидаемся полного включения ВМ и проверяем подключение по ssh и видим версию ОС 
**Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-136-generic x86_64)**
```

root@pxeserver:/var/log/apache2# ssh otus@10.0.0.109
The authenticity of host '10.0.0.109 (10.0.0.109)' can't be established.
ED25519 key fingerprint is: SHA256:ZP6O7Gx6jptwETpMK73Q/33fZoKI8y4+8aDxz5bLAxY
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.0.109' (ED25519) to the list of known hosts.
otus@10.0.0.109's password: 
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-136-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri Jul 31 12:12:44 PM UTC 2026

  System load:  0.99               Processes:               114
  Usage of /:   21.3% of 30.34GB   Users logged in:         0
  Memory usage: 3%                 IPv4 address for enp0s3: 10.0.0.109
  Swap usage:   0%

otus@linux:~$ 
```




