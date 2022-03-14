# Стенд Vagrant с NFS
## Подготовка виртуальных машин
Локально загруженный box добавляем в vagrant (удалив предыдущий, его использовали для выполнения другого дз):
```
sudo vagrant box remove centos7
sudo vagrant box add centos7 CentOS-7-x86_64-Vagrant-2004_01.VirtualBox.box
```
Подготавливаем Vagrantfile с двумя машинами (сервер NFS и клиент):
```
Vagrant.configure("2") do |config|
  config.vm.box = "centos7"
  config.vm.provider "virtualbox" do |v|
        v.memory = 256
        v.cpus = 1
  end
  config.vm.define "nfss" do |nfss|
        nfss.vm.network "private_network", ip: "192.168.50.10",
    virtualbox__intnet: "net1"
        nfss.vm.hostname = "nfss"
  end
  config.vm.define "nfsc" do |nfsc|
        nfsc.vm.network "private_network", ip: "192.168.50.11",
    virtualbox__intnet: "net1"
        nfsc.vm.hostname = "nfsc"
end
end
```
## Настройка сервера NFS
После vagrant up подключаемся к серверу, переходим в режим суперпользователя и доустанавливаем утилиты (сервер NFS уже установлен в CentOS 7 как часть дистрибутива)
```
sam@yarkozloff:/otus/nfs$ sudo vagrant ssh nfss
[vagrant@nfss ~]$ sudo -i
[root@nfss ~]# yum install -y nfs-utils
```
Включаем firewall, проверяем, разрешаем доступ к сервисам NFS:
```
[root@nfss ~]# systemctl enable firewalld --now
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.
[root@nfss ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2022-03-14 00:34:30 UTC; 5s ago
     Docs: man:firewalld(1)
 Main PID: 22161 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─22161 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid
[root@nfss ~]# firewall-cmd --add-service="nfs3" \--add-service="rpc-bind" \--add-service="mountd" \--permanent
success
[root@nfss ~]# firewall-cmd --reload
success
```
Включаем сервер NFS, проверяем наличие слушаемых портов 2049/udp, 2049/tcp, 20048/udp,
20048/tcp, 111/udp, 111/tcp:
```
[root@nfss ~]# systemctl enable nfs --now
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.
[root@nfss ~]# ss -tnplu
Netid  State      Recv-Q Send-Q                                      Local Address:Port                                                     Peer Address:Port
udp    UNCONN     0      0                                                       *:55231                                                               *:*                   users:(("rpc.statd",pid=22316,fd=8))
udp    UNCONN     0      0                                                       *:2049                                                                *:*
udp    UNCONN     0      0                                               127.0.0.1:323                                                                 *:*                   users:(("chronyd",pid=348,fd=5))
udp    UNCONN     0      0                                                       *:68                                                                  *:*                   users:(("dhclient",pid=2634,fd=6))
udp    UNCONN     0      0                                                       *:20048                                                               *:*                   users:(("rpc.mountd",pid=22323,fd=7))
udp    UNCONN     0      0                                               127.0.0.1:868                                                                 *:*                   users:(("rpc.statd",pid=22316,fd=5))
udp    UNCONN     0      0                                                       *:48493                                                               *:*
udp    UNCONN     0      0                                                       *:111                                                                 *:*                   users:(("rpcbind",pid=342,fd=6))
udp    UNCONN     0      0                                                       *:927                                                                 *:*                   users:(("rpcbind",pid=342,fd=7))
udp    UNCONN     0      0                                                    [::]:51663                                                            [::]:*
udp    UNCONN     0      0                                                    [::]:2049                                                             [::]:*
udp    UNCONN     0      0                                                    [::]:60968                                                            [::]:*                   users:(("rpc.statd",pid=22316,fd=10))
udp    UNCONN     0      0                                                   [::1]:323                                                              [::]:*                   users:(("chronyd",pid=348,fd=6))
udp    UNCONN     0      0                                                    [::]:20048                                                            [::]:*                   users:(("rpc.mountd",pid=22323,fd=9))
udp    UNCONN     0      0                                                    [::]:111                                                              [::]:*                   users:(("rpcbind",pid=342,fd=9))
udp    UNCONN     0      0                                                    [::]:927                                                              [::]:*                   users:(("rpcbind",pid=342,fd=10))
tcp    LISTEN     0      128                                                     *:22                                                                  *:*                   users:(("sshd",pid=608,fd=3))
tcp    LISTEN     0      100                                             127.0.0.1:25                                                                  *:*                   users:(("master",pid=702,fd=13))
tcp    LISTEN     0      64                                                      *:2049                                                                *:*
tcp    LISTEN     0      64                                                      *:40078                                                               *:*
tcp    LISTEN     0      128                                                     *:54030                                                               *:*                   users:(("rpc.statd",pid=22316,fd=9))
tcp    LISTEN     0      128                                                     *:111                                                                 *:*                   users:(("rpcbind",pid=342,fd=8))
tcp    LISTEN     0      128                                                     *:20048                                                               *:*                   users:(("rpc.mountd",pid=22323,fd=8))
tcp    LISTEN     0      128                                                  [::]:22                                                               [::]:*                   users:(("sshd",pid=608,fd=4))
tcp    LISTEN     0      100                                                 [::1]:25                                                               [::]:*                   users:(("master",pid=702,fd=14))
tcp    LISTEN     0      64                                                   [::]:2049                                                             [::]:*
tcp    LISTEN     0      64                                                   [::]:46666                                                            [::]:*
tcp    LISTEN     0      128                                                  [::]:49292                                                            [::]:*                   users:(("rpc.statd",pid=22316,fd=11))
tcp    LISTEN     0      128                                                  [::]:111                                                              [::]:*                   users:(("rpcbind",pid=342,fd=11))
tcp    LISTEN     0      128                                                  [::]:20048                                                            [::]:*                   users:(("rpc.mountd",pid=22323,fd=10))
```
Cоздаём и настраиваем директорию, которая будет экспортирована в будущем
```
[root@nfss ~]# mkdir -p /srv/share/upload
[root@nfss ~]# chown -R nfsnobody:nfsnobody /srv/share
[root@nfss ~]# chmod 0777 /srv/share/upload
```
Cоздаём в файле __/etc/exports__ структуру, которая позволит экспортировать ранее созданную директорию
```
[root@nfss ~]# cat > /etc/exports << EOF 
> /srv/share 192.168.50.11/32(rw,sync,root_squash) 
EOF
[root@nfss ~]# cat /etc/exports
/srv/share 192.168.50.11/32(rw,sync,root_squash)
```
Экспортируем заранее созданную директорию и проверяем:
```
[root@nfss ~]# exportfs -r
[root@nfss ~]# exportfs -s
/srv/share  192.168.50.11/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```
## Настройка клиента NFS
Подключаемся к клиенту NFS, доустанавливаем вспомогательные утилиты (под супрпользователем), включаем firewall:
```
sam@yarkozloff:/otus/nfs$ sudo vagrant ssh nfsc
[sudo] password for sam:
[vagrant@nfsc ~]$ sudo -i
[root@nfsc ~]# yum install -y nfs-utils
...
Updated:
  nfs-utils.x86_64 1:1.3.0-0.68.el7.2

Complete!
[root@nfsc ~]# systemctl enable firewalld --now
Created symlink from /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service to /usr/lib/systemd/system/firewalld.service.
Created symlink from /etc/systemd/system/multi-user.target.wants/firewalld.service to /usr/lib/systemd/system/firewalld.service.
[root@nfsc ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2022-03-14 01:01:59 UTC; 7s ago
     Docs: man:firewalld(1)
 Main PID: 22182 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─22182 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid

Mar 14 01:01:59 nfsc systemd[1]: Starting firewalld - dynamic firewall daemon...
Mar 14 01:01:59 nfsc systemd[1]: Started firewalld - dynamic firewall daemon.
Mar 14 01:01:59 nfsc firewalld[22182]: WARNING: AllowZoneDrifting is enabled. This is considered an insecure configuration option. It will be removed in a fut...ing it now.
Hint: Some lines were ellipsized, use -l to show in full.
```
Добавляем в __/etc/fstab__ строку, перезапускаем демона и группу юнитов:
```
[root@nfsc ~]# echo "192.168.50.10:/srv/share/ /mnt nfs vers=3,proto=udp,noauto,x-systemd.automount 0 0" >> /etc/fstab
[root@nfsc ~]# systemctl daemon-reload
[root@nfsc ~]# systemctl restart remote-fs.target
```
Заходим в директорию /mnt/ и проверяем успешность монтирования:
```
[root@nfsc ~]# cd /mnt/
[root@nfsc mnt]# mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=46,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=46041)
192.168.50.10:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=32768,wsize=32768,namlen=255,hard,proto=udp,timeo=11,retrans=3,sec=sys,mountaddr=192.168.50.10,mountvers=3,mountport=20048,mountproto=udp,local_lock=none,addr=192.168.50.10)
```
## Проверка работоспособности:
На сервере создаем файл check_file, затем проверяем его на клиенте в соответсвующем каталоге:
```
sam@yarkozloff:/otus/nfs$ sudo vagrant ssh nfss
Last login: Mon Mar 14 01:10:49 2022 from 10.0.2.2
[vagrant@nfss ~]$ sudo -i
[root@nfss ~]# cd /srv/share/upload/
[root@nfss upload]# touch check_file
[root@nfss upload]# ls -a
.  ..  check_file
[root@nfss upload]# exit
logout
[vagrant@nfss ~]$ exit
logout
Connection to 127.0.0.1 closed.
sam@yarkozloff:/otus/nfs$ sudo vagrant ssh nfsc
Last login: Mon Mar 14 00:58:45 2022 from 10.0.2.2
[vagrant@nfsc ~]$ cd /mnt/upload/
[vagrant@nfsc upload]$ ls -a
.  ..  check_file
```
Перезагружаем клиент проверяем еще раз:
```
[vagrant@nfsc upload]$ reboot
==== AUTHENTICATING FOR org.freedesktop.login1.reboot ===
Authentication is required for rebooting the system.
Authenticating as: root
Password:
==== AUTHENTICATION COMPLETE ===
Connection to 127.0.0.1 closed by remote host.
Connection to 127.0.0.1 closed.
sam@yarkozloff:/otus/nfs$ sudo vagrant ssh nfsc
Last login: Mon Mar 14 01:13:39 2022 from 10.0.2.2
[vagrant@nfsc ~]$ cd /mnt/upload/
[vagrant@nfsc upload]$ ls -a
.  ..  check_file
```
Проверяем сервер. Перезагружаем, проверяем наличие файлов, работу служб, экспортов, RPC:
```
[vagrant@nfss ~]$ sudo reboot
Connection to 127.0.0.1 closed by remote host.
Connection to 127.0.0.1 closed.
sam@yarkozloff:/otus/nfs$ sudo vagrant ssh nfss
Last login: Mon Mar 14 01:17:08 2022 from 10.0.2.2
[vagrant@nfss ~]$ ls -a /srv/share/upload/
.  ..  check_file
[vagrant@nfss ~]$ systemctl status nfs
● nfs-server.service - NFS server and services
   Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
  Drop-In: /run/systemd/generator/nfs-server.service.d
           └─order-with-mounts.conf
   Active: active (exited) since Mon 2022-03-14 01:18:05 UTC; 1min 48s ago
  Process: 806 ExecStartPost=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy ; fi (code=exited, status=0/SUCCESS)
  Process: 785 ExecStart=/usr/sbin/rpc.nfsd $RPCNFSDARGS (code=exited, status=0/SUCCESS)
  Process: 780 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
 Main PID: 785 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/nfs-server.service
[vagrant@nfss ~]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2022-03-14 01:18:00 UTC; 1min 59s ago
     Docs: man:firewalld(1)
 Main PID: 403 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─403 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid
[vagrant@nfss ~]$ sudo exportfs -s
/srv/share  192.168.50.11/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
[vagrant@nfss ~]$ showmount -a 192.168.50.10
All mount points on 192.168.50.10:
192.168.50.11:/srv/share
```
Проверяем клиент.  Проверяем статус монтирования, создаём и проверяем тестовый файл final_check:
```
sam@yarkozloff:/otus/nfs$ sudo vagrant ssh nfsc
Last login: Mon Mar 14 01:25:32 2022 from 10.0.2.2
[vagrant@nfsc ~]$ cd /mnt/upload/
[vagrant@nfsc upload]$ touch final_check
[vagrant@nfsc upload]$ ls -a
.  ..  check_file  final_check
[vagrant@nfsc upload]$ mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=25,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=10882)
192.168.50.10:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=32768,wsize=32768,namlen=255,hard,proto=udp,timeo=11,retrans=3,sec=sys,mountaddr=192.168.50.10,mountvers=3,mountport=20048,mountproto=udp,local_lock=none,addr=192.168.50.10)
```
## Создание автоматизированного Vagrantfile
Выходим из машин, останавливаем машины, удаляем:
```
sam@yarkozloff:/otus/nfs$ sudo vagrant halt
==> nfsc: Attempting graceful shutdown of VM...
==> nfss: Attempting graceful shutdown of VM...
sam@yarkozloff:/otus/nfs$ sudo vagrant destroy
    nfsc: Are you sure you want to destroy the 'nfsc' VM? [y/N] y
==> nfsc: Destroying VM and associated drives...
    nfss: Are you sure you want to destroy the 'nfss' VM? [y/N] y
==> nfss: Destroying VM and associated drives...
```
## Проверка работоспособности
### На сервере создаем файл check_file, затем проверяем его на клиенте в соответсвующем каталоге. Перезагружаем клиент, еще раз проверяем:
```
sam@yarkozloff:/otus/nfs$ sudo vagrant ssh nfss
Last login: Mon Mar 14 21:08:14 2022 from 10.0.2.2
[vagrant@nfss ~]$ sudo -i
[root@nfss ~]# cd /srv/share/upload/
[root@nfss upload]# touch check_file
[root@nfss upload]# ls -a
.  ..  check_file
[root@nfss upload]# exit
logout
[vagrant@nfss ~]$ exit
logout
Connection to 127.0.0.1 closed.

sam@yarkozloff:/otus/nfs$ sudo vagrant ssh nfsc
Last login: Mon Mar 14 21:02:27 2022 from 10.0.2.2
[vagrant@nfsc ~]$ cd /mnt/upload/
[vagrant@nfsc upload]$ ls -a
.  ..  check_file
[vagrant@nfsc upload]$ sudo reboot
Connection to 127.0.0.1 closed by remote host.
Connection to 127.0.0.1 closed.

sam@yarkozloff:/otus/nfs$ sudo vagrant ssh nfsc
Last login: Mon Mar 14 21:15:00 2022 from 10.0.2.2
[vagrant@nfsc ~]$ ls /mnt/upload/
check_file
```
### Проверяем сервер. Перезагружаем, проверяем наличие файлов, работу служб, экспортов, RPC:
```
[vagrant@nfss ~]$ sudo reboot
Connection to 127.0.0.1 closed by remote host.
Connection to 127.0.0.1 closed.
sam@yarkozloff:/otus/nfs$ sudo vagrant ssh nfss
Last login: Mon Mar 14 21:22:40 2022 from 10.0.2.2
[vagrant@nfss ~]$ ls -a /srv/share/upload/
.  ..  check_file
[vagrant@nfss ~]$ systemctl status nfs
● nfs-server.service - NFS server and services
   Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
  Drop-In: /run/systemd/generator/nfs-server.service.d
           └─order-with-mounts.conf
   Active: active (exited) since Mon 2022-03-14 21:23:21 UTC; 42s ago
  Process: 810 ExecStartPost=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy ; fi (code=exited, status=0/SUCCESS)
  Process: 784 ExecStart=/usr/sbin/rpc.nfsd $RPCNFSDARGS (code=exited, status=0/SUCCESS)
  Process: 780 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
 Main PID: 784 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/nfs-server.service
[vagrant@nfss ~]$ systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2022-03-14 21:23:14 UTC; 58s ago
     Docs: man:firewalld(1)
 Main PID: 403 (firewalld)
   CGroup: /system.slice/firewalld.service
           └─403 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid
[vagrant@nfss ~]$ sudo exportfs -s
/srv/share  192.168.50.11/32(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
[vagrant@nfss ~]$ showmount -a 192.168.50.10
All mount points on 192.168.50.10:
192.168.50.11:/srv/share
```
### Проверяем клиент.  Проверяем статус монтирования, создаём и проверяем тестовый файл final_check:
```
sam@yarkozloff:/otus/nfs$ sudo vagrant ssh nfsc
Last login: Mon Mar 14 21:18:29 2022 from 10.0.2.2
[vagrant@nfsc ~]$ cd /mnt/upload/
[vagrant@nfsc upload]$ touch final_check
[vagrant@nfsc upload]$ ls
check_file  final_check
[vagrant@nfsc upload]$ mount | grep mnt
systemd-1 on /mnt type autofs (rw,relatime,fd=33,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=11231)
192.168.50.10:/srv/share/ on /mnt type nfs (rw,relatime,vers=3,rsize=32768,wsize=32768,namlen=255,hard,proto=udp,timeo=11,retrans=3,sec=sys,mountaddr=192.168.50.10,mountvers=3,mountport=20048,mountproto=udp,local_lock=none,addr=192.168.50.10)
```



## Сложности
- Небольшие ресурсы главной виртуальной машиной Ubuntu. Закончилась память, разбирался с увеличением ресурсов для VMWare и удалением лишних вм (разрегистрировать, удалить), работа с локальными боксами вагрант
- Особенности при работе с EOF (в новинку), не страшно, можно спокойно добавлять в sh
- Привыкаю к синтаксису Vagranfile (end лишний/нелишний, скобки и тд)
- Пока не разобрался с kerberos
