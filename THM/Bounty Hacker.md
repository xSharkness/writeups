Стандартная разведка открытых портов через nmap

![](../images/Pasted%20image%2020260504124216.png)

Заглянем на веб сервер, там раскрывается ход повествования задания поподробнее

![](../images/Pasted%20image%2020260504124418.png)


Проверяем ftp сервер, на нем находятся два файла, скачиваем их и читаем

![](../images/Pasted%20image%2020260504140923.png)

Из файла task.txt пользователь lin составил список задач, первая задача - защитать Vicious, это скорее всего имя пользователя, а второй файл - скорее всего пароли. Так как веб-сервер я проверил и ничего интересного не нашел, остается вариант брута ssh с этими данными, будем использовать гидру.

```
root:~# cat task.txt
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
root:~# cat locks.txt 
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e

```

Брут Vicious ничем не закончился, как и всех пользователей, которые упоминались на сайте, оставался только lin

![](../images/Pasted%20image%2020260504141652.png)

К нему пароль подошел, поэтому заходим в систему и читаем user флаг

![](../images/Pasted%20image%2020260504141852.png)

Далее необходимо понять как повысить себе привилегии. Проверяем какие команды мы можем выполнить от лица root пользователя. Находим там tar. Tar предусматривает выполнение произвольных команд в рамках своей работы, поэтому создаем оболочку. Запуская оболочку проверяем, получили ли мы права и читаем флаг.

![](../images/Pasted%20image%2020260504142451.png)

