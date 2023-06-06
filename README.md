Для получения доступа необходимо открыть GUI VirtualBox (или другой системы виртуализации), запустить виртуальную машину и при выборе ядра для загрузки нажать e - в данном контексте edit. Попадаем в окно, где мы можем изменить параметры загрузки:


Открыл GUI VirtualBox и при выборе ядра для загрузки нажал e. 
1.В конце строки, начинающейся с linux16, добавил init=/bin/sh и нажал сtrl-x 
Система загрузилась, в mount / файловая система была в режиме Read-Only
2.В конце строки, начинающейся с linux16, добавил rd.break и нажал сtrl-x.
Попал в emergency mode, / файловая система смонтирована в режиме Read-Only.
Для смены пароля администратора:
mount -o remount,rw /sysroot
chroot /sysroot
passwd root
touch /.autorelabel
Перезагрузился и зашел с новым паролем.
3. В строке, начинающейся с linux16, заменил ro на rw init=/sysroot/bin/sh и нажал сtrl-x 
После загрузки файловая система в mount была в режим Read-Write
--
Установить систему с LVM, после чего переименовать VG

Script started on Tue 06 Jun 2023 08:30:40 AM UTC
[root@lvm ~]# vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   2   0 wz--n- <38.97g    0 
[root@lvm ~]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
├─sda1                    8:1    0    1M  0 part 
├─sda2                    8:2    0    1G  0 part /boot
└─sda3                    8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
[root@lvm ~]# lvs
  LV       VG         Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  LogVol00 VolGroup00 -wi-ao---- <37.47g                                                    
  LogVol01 VolGroup00 -wi-ao----   1.50g                                                    

[root@lvm ~]# vgrename VolGroup00 OtusRoot
  Volume group "VolGroup00" successfully renamed to "OtusRoot"

[root@lvm ~]# cat /etc/fstab | grep VolGroup00
/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
/dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0

[root@lvm ~]# vi /etc/fstab 
[root@lvm ~]# cat /etc/fstab | grep OtusRoot
/dev/mapper/OtusRoot-LogVol00 /                       xfs     defaults        0 0
/dev/mapper/OtusRoot-LogVol01 swap                    swap    defaults        0 0

[root@lvm ~]# cat /etc/default/grub | grep VolGroup00
GRUB_CMDLINE_LINUX="no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 rhgb quiet"

[root@lvm ~]# vi /etc/default/grub

[root@lvm ~]# cat /etc/default/grub | grep Vol
GRUB_CMDLINE_LINUX="no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=OtusRoot/LogVol00 rd.lvm.lv=OtusRoot/LogVol01 rhgb quiet"

[root@lvm ~]# cat /etc/default/grub | grep OtusRoot
GRUB_CMDLINE_LINUX="no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=OtusRoot/LogVol00 rd.lvm.lv=OtusRoot/LogVol01 rhgb quiet"

[root@lvm ~]# cat /boot/grub2/grub.cfg | grep VolGroup00
        linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 ro no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 rhgb quiet 

[root@lvm ~]# vi /boot/grub2/grub.cfg

[root@lvm ~]# cat /boot/grub2/grub.cfg | grep VolGroup00

reboot

[root@lvm ~]# vgs
  VG       #PV #LV #SN Attr   VSize   VFree
  OtusRoot   1   2   0 wz--n- <38.97g    0 
----

Добавить модуль в initrd

Создал двай файла module-setup.sh и  test.sh с содержимым из методички в дирректории /usr/lib/dracut/modules.d/01test

mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

[root@lvm ~]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test

Выключил опции rghb и quiet, нарисовался пингвин.
