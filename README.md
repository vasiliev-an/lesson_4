# **Загрузка системы**
### **Попасть в систему без пароля несколькими способами**

Для получения доступа необходимо открыть GUI VirtualBox (или другой системы виртуализации), запустить виртуальную машину и при выборе ядра для загрузки нажать e - в данном контексте edit. Попадаем в окно где мы можем изменить параметры загрузки. 

Способ 1. init=/bin/sh

В строке linux16 добавляем в конец строки init=/bin/sh, и нажимаем сtrl-x для загрузки в систему.

Рутовая файловая система при этом монтируется в режиме Read-Only, поэтому для перемонтирования ее в режим Read-Write можно воспользоваться командой:

```
# mount -o remount,rw /
```

Проверяем, что система в режиме rw:

```
touch testfile
```
Способ 2. rd.break

В строке linux16 добавляем в конец строки rd.break, и нажимаем сtrl-x для загрузки в систему.
Попадаем в emergency mode. Наша корневая файловая система смонтирована (опять же в режиме Read-Only, но мы не в ней. Далее будет пример как попасть в нее и поменять пароль администратора:

```
# mount -o remount,rw /sysroot
# chroot /sysroot
# passwd root
# touch /.autorelabel
# exit
# exit
```

Способ 3. rw init=/sysroot/bin/sh

В строке начинающейся с linux16 заменяем ro на rw init=/sysroot/bin/sh и нажимаем сtrl-x для загрузки в систему. Файловая система сразу
смонтирована в режим Read-Write.



### **Установить систему с LVM, после чего переименовать VG**

Текущее состояние системы:

```
# vgs
```

Переименовываем Volume Group на OtusRoot:

```
# vgrename VolGroup00 OtusRoot
```

Правим /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. Везде заменяем Volume Group на OtusRoot и пересоздаём initrd image для сохранения правок:

```
# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```

![network](https://github.com/vasiliev-an/lesson_4/blob/master/img/1.png)

Перезагружаемся и проверяем, что изменения применились:

```
# vgs
```

### **Добавить модуль в initrd**

Скрипты модулей хранятся в каталоге /usr/lib/dracut/modules.d/. Для того чтобы добавить свой модуль создаем там папку с именем 01test:

```
# mkdir /usr/lib/dracut/modules.d/01test
```

В нее поместим два скрипта:

```
# vi module-setup.sh
#!/bin/bash

check() {
    return 0
}

depends() {
    return 0
}

install() {
    inst_hook cleanup 00 "${moddir}/test.sh"
}
```


```
# vi test.sh
#!/bin/bash


exec 0<>/dev/console 1<>/dev/console 2<>/dev/console
cat <<'msgend'

Hello! You are in dracut module!

 ___________________
< I'm dracut module >
 -------------------
   \
    \
        .--.
       |o_o |
       |:_/ |
      //   \ \
     (|     | )
    /'\_   _/`\
    \___)=(___/
msgend
sleep 10
echo " continuing...."
```

Пересобираем образ initrd:

```
# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```

Можно проверить/посмотреть какие модули загружены в образ:

```
# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test
```


Редактируем grub.cfg, убирая опции rhgb и quiet и перезагружаемся:

```
# vi /boot/grub2/grub.cfg
# reboot
```
В итоге при загрузке будет пауза на 10 секунд и видим пингвина в выводе
терминала
