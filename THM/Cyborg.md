nmap приглашает исследовать веб-сервер

![](../images/Pasted%20image%2020260509135630.png)

На веб-сервере обычная страница, поэтому необходима разведка скрытых директорий

![](../images/Pasted%20image%2020260509140022.png)

Запускаем ffuf

![](../images/Pasted%20image%2020260509141937.png)

Нашлась директория admin и etc, проверяем их по очереди

![](../images/Pasted%20image%2020260509140250.png)

Теперь у нас есть материал для изучения.

```
############################################
############################################
  
[Yesterday at 4.32pm from Josh]
  
Are we all going to watch the football game at the weekend??
  
############################################
############################################
  
[Yesterday at 4.33pm from Adam]
  
Yeah Yeah mate absolutely hope they win!
  
############################################
############################################
  
[Yesterday at 4.35pm from Josh]
  
See you there then mate!
  
############################################
############################################
  
[Today at 5.45am from Alex]
  
Ok sorry guys i think i messed something up, uhh i was playing around with the squid proxy i mentioned earlier.
  
I decided to give up like i always do ahahaha sorry about that.
  
I heard these proxy things are supposed to make your website secure but i barely know how to use it so im probably making it more insecure in the process.
  
Might pass it over to the IT guys but in the meantime all the config files are laying about.
  
And since i dont know how it works im not sure how to delete them hope they don't contain any confidential information lol.
  
other than that im pretty sure my backup "music_archive" is safe just to confirm.
  
############################################
############################################
```

Находим такой диалог. Там упоминаются имена  Josh, Adam и Alex и прокси сервер squid

Также на этом сайте можно скачать архив, посмотрим что там.
Хорошо, что он не запаролен.

![](../images/Pasted%20image%2020260509141114.png)

Оставим пока что архив, исследуем директорию /etc

![](../images/Pasted%20image%2020260509141825.png)

Скорее всего это и есть тот самый squid
Первый файл который там есть, это passwd
```
music_archive:$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.
```

Это однозначно креды, но пароль захэширован, можно попробовать перебрать через rockyou, поставим перебор и будем изучать дальше.

Второй файл в squid - это squid.conf, при взаимодействии с сайтом возможно пригодиться знать эти настройки

```
auth_param basic program /usr/lib64/squid/basic_ncsa_auth /etc/squid/passwd
auth_param basic children 5
auth_param basic realm Squid Basic Authentication
auth_param basic credentialsttl 2 hours
acl auth_users proxy_auth REQUIRED
http_access allow auth_users
```

Креды сломаны
```
$apr1$BpZ.Q.1m$F0qqPwHSOG50URuOVQTTn.:squidward
music_archive:squidward
```

Зайти в систему с ними не получилось. Отсюда появляются три варианта, либо эти креды нужны для взаимодействия с архивом скачанным, потому что там всё зашифровано, либо для какой-нибудь CVE, где необходимы логин с паролем, либо вообще для реальной работы со squid прокси. Надо всё это проверить

Отлично, первое предположение оказалось верным, с архивом можно взаимодействовать без шифрования, используя найденную парольную фразу.

Судя по логике и здравому смыслу, в этом архиве мне надо найти логин и пароль для входа в систему.

Файлов, которые хранятся в снимке системы очень много. Интересные файлы secret.txt и note.txt.

```
└─$ cat secret.txt         
shoutout to all the people who have gotten to this stage whoop whoop!"

└─$ cat note.txt     
Wow I'm awful at remembering Passwords so I've taken my Friends advice and noting them down!

alex:S3cretP@s3
```

А вот и кажется креды для Алекса, пробуем зайти.
Заходим и получаем флаг пользователя

![](../images/Pasted%20image%2020260509145726.png)

Посмотрев какие действия можно выполнять от лица рут пользователя я нашел скрипт

![](../images/Pasted%20image%2020260509145933.png)

Мы еще и можем его поменять
```
-r-xr-xr--   1 alex alex  1083 Dec 30  2020 backup.sh
```

Редактируем права на скрипт и меняем скрипт, чтобы получить оболочку рут пользователя

Добавляем 
```
cp /bin/bash /tmp/rootshell
chmod 4777 /tmp/rootshell
```

Запускаем скрипт

![](../images/Pasted%20image%2020260509150402.png)

Отлично, мы получили права рут пользователя
Читаем последний флаг.

![](../images/Pasted%20image%2020260509150451.png)

