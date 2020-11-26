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


   #1

#прописать вместо ro 

          rw init=/sysroot/bin/sh
  
#нажмите 

         Ctrl+X 

#и дождитесь загрузки операционной системы в однопользовательском режиме (single-user mode). 

#для отключения SELinux, сконфигурируйте SELINUX=disabled в /etc/selinux/config

          chroot /sysroot

          nano /etc/selinux/config


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

#нажмите 

          Ctrl+X

#подтверждаем изменения 

          y

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
  
#нажмите 

         Ctrl+X 

#и дождитесь загрузки операционной системы в однопользовательском режиме (single-user mode). 

#для отключения SELinux, сконфигурируйте SELINUX=disabled в /etc/selinux/config

          chroot /sysroot

          nano /etc/selinux/config


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

#нажмите 

          Ctrl+X

#подтверждаем изменения 

          y

#далее команду

          passwd root

#вводим новый root-пароль

          password

#cохраняем изменения командами

          exit

#перегружаем

          reboot

   #3

#прописать вместо ro

          rd.break

#нажмите 

         Ctrl+X 

#и дождитесь загрузки операционной системы в однопользовательском режиме (single-user mode). 

#для отключения SELinux, сконфигурируйте SELINUX=disabled в /etc/selinux/config

          chroot /sysroot

          nano /etc/selinux/config


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

#нажмите 

          Ctrl+X 

#подтверждаем изменения 

          y

#далее команду

          passwd root

#вводим новый root-пароль

          password

#cохраняем изменения командами

          exit

#перегружаем

          reboot

#установить систему с LVM, после чего переименовать VG

#первым делом посмотрим на текущее состояние системы

        vgs
#вывод

          VG         #PV #LV #SN Attr   VSize   VFree
          VolGroup00   1   2   0 wz--n- <38.97g    0


#далее можем поменять название VG

        vgrename VolGroup00 loadRoot

#смотрим новое название

       ls -l /dev/mapper

#вывод

          total 0
          crw------- 1 root root 10, 236 Nov 26 03:40 control
          lrwxrwxrwx 1 root root       7 Nov 26 03:43 loadRoot-LogVol00 -> ../dm-0
          lrwxrwxrwx 1 root root       7 Nov 26 03:43 loadRoot-LogVol01 -> ../dm-1
#запаминаем

          loadRoot-LogVol00
          loadRoot-LogVol01

#далее будем править файлы, чтобы в дальнейшем мы могли загрузиться.(/etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg) 

#заходим в эти файлы и меняем старое название на новое которое запаминали

       nano /etc/fstab

       nano /etc/default/grub

       nano /boot/grub2/grub.cfg

#далее пересоздаем initrd image

       mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

#перезагружаем ПК, теперь у нас система загружается с новым названием VG, проверяем:

       vgs

#скрипты модулей хранятся в каталоге /usr/lib/dracut/modules.d/. Для того чтобы добавить свой модуль создаем там папку с именем 01test.

        mkdir /usr/lib/dracut/modules.d/new

#переходим в директорию

        cd /usr/lib/dracut/modules.d/new/

#далее создаем 2 skripts - module-setup.sh и test.sh

       touch module-setup.sh

       touch module-new.sh

#прописываем

         nano module-setup.sh

       #!/bin/bash

       check() {
       return 0
       }

       depends() {
       return 0
       }

       install() {
       inst_hook cleanup 00 "${moddir}/module-test.sh"
       }

#прописываем

         nano module-new.sh


          #!/bin/bash

          exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
          cat <<'msgend'

          Hello! You are in dracut module!

          ___________________
         < I'm dracut module >
          -------------------
          \
          \
             \---/  \\
            |<> <>| ||
            |  *  |//
           //'''''')
           \\     |
             ======
             ||  ||
             --  --
         msgend
         sleep 10
         echo " continuing...."

#делаем их исполняемыми

       chmod +x module-setup.sh

       chmod +x module-new.sh

#далее выполняем команду

        dracut -f -v

#Можно посмотреть какие модули загружены в образ

        lsinitrd -m /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img | grep new

#После чего можно пойти двумя путями для проверки: 

#1.перезагрузиться и руками выключить опции rghb и quiet и увидеть вывод 

#2.либо отредактировать grub.cfg убрав эти опции

#Редактируем grub.cfg

       nano /boot/grub2/grub.cfg

       reboot