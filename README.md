#скачиваем Vagrantfile

           git clone https://github.com/MaximMiklyaev/loadsystem.git

#переходим в директорию:

           cd /loadsystem

#запускаем образ

           vagrant up

#Vagrantfile поднимает VM

#в консоли вводим

          virtualbox .vagrant/

#VM отобразится в VBox

#через графические инструменты перегружаем образ VM в VBox

#появится меню загрузчика GRUB (чёрный экран) нажимаем

          e

#далее найдем и отредактируем строку начальной загрузки, которая начинается с linux16 /boot/...

#Параметр ro (read-only) отвечает за загрузку ядра Linux в режиме только чтение. Чтобы после сброса root-пароля изменения были сохранены, необходимо заменить ro на rw (read-write) — режим чтение-запись.
 
#укажем запуск командной оболочки bash, необходимо удалить информацию от ro до rhgb quint 

#для отключения SELinux, сконфигурируйте SELINUX=disabled в /etc/selinux/config

          vi /etc/selinux/config


          # This file controls the state of SELinux on the system.
          # SELINUX= can take one of these three values:
          #       enforcing - SELinux security policy is enforced.
          #       permissive - SELinux prints warnings instead of enforcing.
          #       disabled - No SELinux policy is loaded.
          SELINUX=disabled
          # SELINUXTYPE= can take one of these two values:
          #       targeted - Targeted processes are protected,
          #       mls - Multi Level Security protection.
          SELINUXTYPE=targeted



   #1

#прописать вместо ro 

          rw init=/sysroot/bin/sh
  
#нажмите Ctrl+X и дождитесь загрузки операционной системы в однопользовательском режиме (single-user mode). 

#для установки нового root-пароля введите команду

          chroot /sysroot

#далее команду

          passwd root

#вводим новый root-пароль

           password

#cохраняем изменения командами

          exit

#перегружаем

          reboot 


   #2

#прописать вместо ro 

          rw init=/bin/sh
  
#нажмите Ctrl+X и дождитесь загрузки операционной системы в однопользовательском режиме (single-user mode). 

#работа с корнем

           mount -o remount,rw /

#вводим новый root-пароль

           passwd

#проверяем включен для SELUX 
  
           cat /etc/selinux/config

#если SELINUX=enforcing, то заменяем или на permissive или disabled и перезагружаемся

          vi /etc/selinux/config

#перегружаем

          reboot

   #3

#прописать вместо ro

          rd.break

#нажмите Ctrl+X и дождитесь загрузки операционной системы в однопользовательском режиме (single-user mode). 
    

#работа с корнем

           mount -o remount,rw /sysroot

           chroot /sysroot

           passwd root

#после смены пароля необходимо создать скрытый файл .autorelabel в /, выполнив

            touch /.autorelabel


#перегружаем

           reboot



   инф. server_script.sh

                         #!/bin/sh
                         set -eux
                         echo "HM4"
                         whoami
                         uname -a
                         hostname -f
                         ip addr show dev eth1
                         echo "commands"
                         #переходим под root права
                         sudo -i
                         #устанавка утилит на сервер
                         yum install -y nfs-utils
                         #включаем сервисы nfs сервера
                         systemctl enable rpcbind
                         systemctl enable nfs-server
                         systemctl enable rpc-statd
                         systemctl enable nfs-idmapd
                         #запуск сервисов nfs сервера
                         systemctl start rpcbind
                         systemctl start nfs-server
                         systemctl start rpc-statd
                         systemctl start nfs-idmapd
                         #создание дирректории для общего доступа
                         mkdir -p /usr/shared/
                         #назначение прав данной дирректории
                         chmod 0777 /usr/shared/
                         #конфигурируем exports
                         cat << EOF | sudo tee /etc/exports
                         /usr/shared/ 192.168.10.0/24(rw,sync)
                         EOF
                         #применяем изменения
                         exportfs -ra
                         #включаем и запускаем firewalld
                         systemctl enable firewalld
                         systemctl start firewalld
                         #включаем сервисы
                         firewall-cmd --permanent --add-service=nfs3
                         firewall-cmd --permanent --add-service=mountd
                         firewall-cmd --permanent --add-service=rpc-bind
                         firewall-cmd --reload
                         #создаем папку uploads в расшареной дирректории
                         mkdir /usr/shared/uploads
 
#при поднятие nfsClient запускается скрипт client_script.sh

   инф. server_script.sh

                          #!/bin/sh
                          set -eux
                          echo "HM4"
                          whoami
                          uname -a
                          hostname -f
                          ip addr show dev eth1
                          echo "mount NFSv3 UDP"
                          echo "test mount NFSv4"
                          #переходим под root права
                          sudo -i
                          #устанавка утилит на клиенте
                          yum install -y nfs-utils
                          #включаем и запускаем firewalld
                          systemctl enable firewalld
                          systemctl start firewalld
                          #добавляем запись в fstab для автоматического монтирования
                          echo "192.168.10.10:/usr/shared /mnt nfs rw,vers=3,sync,proto=udp,rsize=32768,wsize=32768 0 0" >> /etc/fstab
                          #монтируем указанный в fstab каталог в /mnt
                          mount /mnt/

#проверка примонтированного диска на клиенте

#подключаемся к клиентской VM

    vagrant ssh nfsClient

#

    lsblk

    вывод:
          NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
          sda      8:0    0  40G  0 disk
          `-sda1   8:1    0  40G  0 part /

#

    df -h

    вывод:
          Filesystem                 Size  Used Avail Use% Mounted on
          devtmpfs                   111M     0  111M   0% /dev
          tmpfs                      118M     0  118M   0% /dev/shm
          tmpfs                      118M  8.5M  110M   8% /run
          tmpfs                      118M     0  118M   0% /sys/fs/cgroup
          /dev/sda1                   40G  3.1G   37G   8% /
          192.168.10.10:/usr/shared   40G  3.1G   37G   8% /mnt
          tmpfs                       24M     0   24M   0% /run/user/1000
