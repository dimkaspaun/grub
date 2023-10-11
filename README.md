## Попасть в систему без пароля несколькими способами

Для получения доступа необходимо открыть GUI VirtualBox (или другой системы
виртуализации), запустить виртуальную машину и при выборе ядра для загрузки нажать e - в
данном контексте edit. Попадаем в окно где мы можем изменить параметры загрузки:

### Способ 1. init=/bin/sh

**● В конце строки начинающейся с linux16 добавляем init=/bin/sh и нажимаем сtrl-x для
загрузки в систему**

![](https://github.com/dimkaspaun/grub/blob/main/screen/1_1.png)

**● В целом на этом все, Вы попали в систему. Но есть один нюанс. Рутовая файловая
система при этом монтируется в режиме Read-Only. Если вы хотите перемонтировать ее в
режим Read-Write можно воспользоваться командой:**

    [root@otuslinux ~]# mount -o remount,rw /

**● После чего можно убедиться записав данные в любой файл или прочитав вывод
команды:**

    [root@otuslinux ~]# mount | grep root

![](https://github.com/dimkaspaun/grub/blob/main/screen/1_2.png)

### Способ 2. rd.break
**● В конце строки начинающейся с linux16 добавляем rd.break и нажимаем сtrl-x для
загрузки в систему**

![](https://github.com/dimkaspaun/grub/blob/main/screen/2_1.png)

**● Попадаем в emergency mode. Наша корневая файловая система смонтирована (опять же
в режиме Read-Only, но мы не в ней. Далее будет пример как попасть в нее и поменять
пароль администратора:**

    [root@otuslinux ~]# mount -o remount,rw /sysroot

    [root@otuslinux ~]# chroot /sysroot

    [root@otuslinux ~]# passwd root

    [root@otuslinux ~]# touch /.autorelabel

![](https://github.com/dimkaspaun/grub/blob/main/screen/2_2.png)

**● После чего можно перезагружаться и заходить в систему с новым паролем. Полезно
когда вы потеряли или вообще не имели пароль администратор.**

![](https://github.com/dimkaspaun/grub/blob/main/screen/2_3.png)

### Способ 3. rw init=/sysroot/bin/sh

**● В строке начинающейся с linux16 заменяем ro на rw init=/sysroot/bin/sh и нажимаем сtrl-x
для загрузки в систему**

**● В целом то же самое что и в прошлом примере, но файловая система сразу
смонтирована в режим Read-Write**

**● В прошлых примерах тоже можно заменить ro на rw**

![](https://github.com/dimkaspaun/grub/blob/main/screen/3_1.png)
![](https://github.com/dimkaspaun/grub/blob/main/screen/3_2.png)

## Установить систему с LVM, после чего переименовать VG

**● Первым делом посмотрим текущее состояние системы:**

    [root@otuslinux ~]# vgs

![](https://github.com/dimkaspaun/grub/blob/main/screen/4_1.png)

**● Приступим к переименованию:**

    [root@otuslinux ~]# vgrename VolGroup00 OtusRoot

![](https://github.com/dimkaspaun/grub/blob/main/screen/4_2.png)

**● Далее правим /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. Везде заменяем старое
название на новое. По ссылкам можно увидеть примеры получившихся файлов.**

![](https://github.com/dimkaspaun/grub/blob/main/screen/4_3.png)
![](https://github.com/dimkaspaun/grub/blob/main/screen/4_4.png)
![](https://github.com/dimkaspaun/grub/blob/main/screen/4_5.png)

**● Пересоздаем initrd image, чтобы он знал новое название Volume Group**

    [root@otuslinux ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
    
![](https://github.com/dimkaspaun/grub/blob/main/screen/4_7.png)

**● После чего можем перезагружаться и если все сделано правильно успешно грузимся с
новым именем Volume Group и проверяем:**

    [root@otuslinux ~]# vgs

![](https://github.com/dimkaspaun/grub/blob/main/screen/4_9.png)

## Добавить модуль в initrd

**Скрипты модулей хранятся в каталоге /usr/lib/dracut/modules.d/. Для того чтобы
добавить свой модуль создаем там папку с именем 01test:**

    [root@otuslinux ~]# mkdir /usr/lib/dracut/modules.d/01test
    
**В нее поместим два скрипта:**

    1. module-setup.sh - который устанавливает модуль и вызывает скрипт test.sh

    2. test.sh - собственно сам вызываемый скрипт, в нём у нас рисуется пингвинчик

![](https://github.com/dimkaspaun/grub/blob/main/screen/5_1.png)
![](https://github.com/dimkaspaun/grub/blob/main/screen/5_2.png)

**● Пересобираем образ initrd**

    [root@otuslinux ~]# dracut -f -v

![](https://github.com/dimkaspaun/grub/blob/main/screen/5_3.png)
![](https://github.com/dimkaspaun/grub/blob/main/screen/5_4.png)

**● Можно проверить/посмотреть какие модули загружены в образ:**

    [root@otuslinux ~]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test   

![](https://github.com/dimkaspaun/grub/blob/main/screen/5_5.png)

**● После чего можно пойти двумя путями для проверки:**

    ○ Перезагрузиться и руками выключить опции rghb и quiet и увидеть вывод

![](https://github.com/dimkaspaun/grub/blob/main/screen/5_6.png)

**● В итоге при загрузке будет пауза на 10 секунд и вы увидите пингвина в выводе
терминала**

![](https://github.com/dimkaspaun/grub/blob/main/screen/5_7.png)

