Выполняем разведку через nmap. Обнаружили два открытых порта, на 22 порту работает ssh, на 80 http.

![](../images/Pasted%20image%2020260503150047.png)

Узнаем какие там версии сервисов.

22 порт SSH: OpenSSH 8.2p1

80 порт HTTP: Apache 2.4.41

![](../images/Pasted%20image%2020260503150352.png)

Теперь зайдем на веб-сервер, на первый взгляд это обычный одностраничник, поэтому обязательно необходимо проверить скрытые директории

![](../images/Pasted%20image%2020260503150720.png)

Для поиска скрытых директорий я буду использовать утилиту ffuf

![](../images/Pasted%20image%2020260503151849.png)

Найдена скрытое место на сайте - panel, перейдем туда. Это оказалось формой загрузки файлов. Попробуем прокинуть туда шел. Для этого загрузим туда легитимный файл, посмотрим как проходит процесс загрузки, можно ли достучаться потом до файла.

![](../images/Pasted%20image%2020260503151925.png)

Отлично, можно открыть файл после загрузки, поэтому пробуем сразу загрузить туда реверс шелл.

Мимо тайп проходит любой, расширение файла - всё кроме php, содержимое файла не любое, бывает тоже закидывает в блок запросы. Простейший реверс шелл по итогу заблокировался антивирусом

![](../images/Pasted%20image%2020260503155340.png)

base64 Тоже заблокирован антивирусом

Расширение php5 сработало и обработалось как php файл. Реверс шелл прокинулся. Далее изучаем систему.

![](../images/Pasted%20image%2020260503160414.png)

Находим и получаем флаг userа

![](../images/Pasted%20image%2020260503160539.png)

Далее просматриваем какие файлы можно запустить с максимальными привилегиями

```
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/newuidmap
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/python2.7
/usr/bin/at
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/pkexec
/snap/core/8268/bin/mount
/snap/core/8268/bin/ping
/snap/core/8268/bin/ping6
/snap/core/8268/bin/su
/snap/core/8268/bin/umount
/snap/core/8268/usr/bin/chfn
/snap/core/8268/usr/bin/chsh
/snap/core/8268/usr/bin/gpasswd
/snap/core/8268/usr/bin/newgrp
/snap/core/8268/usr/bin/passwd
/snap/core/8268/usr/bin/sudo
/snap/core/8268/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/8268/usr/lib/openssh/ssh-keysign
/snap/core/8268/usr/lib/snapd/snap-confine
/snap/core/8268/usr/sbin/pppd
/snap/core/9665/bin/mount
/snap/core/9665/bin/ping
/snap/core/9665/bin/ping6
/snap/core/9665/bin/su
/snap/core/9665/bin/umount
/snap/core/9665/usr/bin/chfn
/snap/core/9665/usr/bin/chsh
/snap/core/9665/usr/bin/gpasswd
/snap/core/9665/usr/bin/newgrp
/snap/core/9665/usr/bin/passwd
/snap/core/9665/usr/bin/sudo
/snap/core/9665/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core/9665/usr/lib/openssh/ssh-keysign
/snap/core/9665/usr/lib/snapd/snap-confine
/snap/core/9665/usr/sbin/pppd
/snap/core20/2599/usr/bin/chfn
/snap/core20/2599/usr/bin/chsh
/snap/core20/2599/usr/bin/gpasswd
/snap/core20/2599/usr/bin/mount
/snap/core20/2599/usr/bin/newgrp
/snap/core20/2599/usr/bin/passwd
/snap/core20/2599/usr/bin/su
/snap/core20/2599/usr/bin/sudo
/snap/core20/2599/usr/bin/umount
/snap/core20/2599/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core20/2599/usr/lib/openssh/ssh-keysign
/bin/mount
/bin/su
/bin/fusermount
/bin/umount
```
Среди всех файлов бросается в глаза питон 2.7

![](../images/Pasted%20image%2020260503164640.png)

Отлично, всё работает.
Немного изучаем систему и находим рут флаг

![](../images/Pasted%20image%2020260503164749.png)

