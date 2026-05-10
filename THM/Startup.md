nmap показал открытые порты 21, 22, 80.
Начнем изучение сервисов, на ftp доступен анонимный вход и там есть файлы, смотрим что за файлы.

![](../images/Pasted%20image%2020260510122259.png)

Скачиваем файлы и смотрим что там

![](../images/Pasted%20image%2020260510122923.png)

```
└─$ cat notice.txt   
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY. People downloading documents from our website will think we are a joke! Now I dont know who it is, but Maya is looking pretty sus.
```

Также картинка приложена.

![](../images/Pasted%20image%2020260510123436.png)

Внутри картинки находится архив
```
└─$ binwalk important.jpg  

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 735 x 458, 8-bit/color RGBA, non-interlaced
57            0x39            Zlib compressed data, compressed
```

Очень странный файл, я попытался как то извлечь, понять, что внутри, но всё очень смутно и непонятно, поэтому просто запомним это и перейдем к исследованию веб-сервера.

![](../images/Pasted%20image%2020260510124711.png)

Через фаззинг директорий можно найти директорию files

![](../images/Pasted%20image%2020260510125450.png)

Которая является той же директорией, что и в ftp.
Тогда надо проверить возможность записи файлов в любой каталог ftp.

![](../images/Pasted%20image%2020260510140350.png)

Удалось загрузить свой файл.. Проверим работоспособность.

![](../images/Pasted%20image%2020260510140428.png)

Всё работает. Тогда прокидываем реверс shell.
С нескольких попыток нашелся реверс который подходит под эту систему и мы видим входящее подключение.

![](../images/Pasted%20image%2020260510141335.png)

Находим в системе рецепт изготовления блюда - главный ингридиент
```
cat recipe.txt

Someone asked what our main ingredient to our spice soup is today. I figured I can't keep it a secret forever and told him it was love.
```
Запустив linpeas я увидел несколько крутых векторов для атаки

![](../images/Pasted%20image%2020260510153504.png)

Были подозрения на некоторые CVE, но в системе нет компилятора, это усложняет задачу. Исследовав интересные файлы, которые я могу открыть от www-data, я нашел /incidents/suspicious.pcapng - это дамп сетевого трафика, который мы можем открыть в wireshark. Через ftp я его перекинул к себе и открыл.

![](../images/Pasted%20image%2020260510153735.png)

Среди прочих пакетов можно найти одну запись со строкой, в это время судя по всему идут неудачные попытки зайти в систему и после этой строки всё получается.

По каталогу /home я нашел пользователя lennie, скорее всего это пароль от него, пробуем зайти через ssh

```
c4ntg3t3n0ughsp1c3
```

![](../images/Pasted%20image%2020260510154210.png)

Действительно всё получилось. Осталось получить root права.

Исследуем папку Documents

![](../images/Pasted%20image%2020260510154708.png)

Также есть папки scripts
```
$ ls -la
total 16
drwxr-xr-x 2 root   root   4096 Nov 12  2020 .
drwx------ 7 lennie lennie 4096 May 10 11:52 ..
-rwxr-xr-x 1 root   root     77 Nov 12  2020 planner.sh
-rw-r--r-- 1 root   root      1 May 10 12:26 startup_list.txt
lennie@startup:~/scripts$ cat planner.sh
#!/bin/bash
echo $LIST > /home/lennie/scripts/startup_list.txt
/etc/print.sh
lennie@startup:~/scripts$ 
```

в ней мы видим, что мы можем запустить скрипт от лица рута, который перекинет содержимое переменной в txt файл.

Дальше я вернулся к выводу linpeas и нашел там CVE, которую пытался реализовать через реверс шел. Тут должно всё получиться.

```
pkexec version 0.105
Potentially vulnerable to CVE-2021-4034 (PwnKit)
```

И действительно, всё сработало.

![](../images/Pasted%20image%2020260510163700.png)

Читаем флаг рута
