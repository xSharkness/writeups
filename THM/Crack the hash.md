Для решения этого задания я решил выбрать тактику, сначала проверять все хэши в crackstation.net, если это не даст результатов, то перебирать через hashcat

---
Хэш:
```
48bb6e862e54f2a795ffc4e541caed4d
```

Отлично, результат есть
![](../images/Pasted%20image%2020260506121840.png)

---
Хэш:
```
CBFDAC6008F9CAB4083784CBD1874F76618D2A97
```

![](../images/Pasted%20image%2020260506121934.png)

---
Хэш:
```
1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032
```

![](../images/Pasted%20image%2020260506122010.png)

---
Хэш:
```
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
```

![](../images/Pasted%20image%2020260506123338.png)

Результат не найден из-за того, что тип хэша brypt, поэтому перейдем к hashcat. В качестве словаря будем использовать rockyou, отфильтрованный по словам  из 4 символов (из условия задачи)
Через минут 5 хэш подобрался.

![](../images/Pasted%20image%2020260506124922.png)

---
Хэш:
```
279412f945939ba78ce0758d3fd83daa
```

![](../images/Pasted%20image%2020260506123111.png)

---

*Далее автор задания советует использовать только hashcat, а не какие-то онлайн инструменты.*

---
Хэш:
```
F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
```

![](../images/Pasted%20image%2020260506125237.png)

---
Хэш:
```
1DFECA0C002AE40B8619ECF94819CC1B
```

![](../images/Pasted%20image%2020260506125736.png)

---
Хэш:
```
$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.
```

Соль:
```
aReallyHardSalt
```

![](../images/Pasted%20image%2020260506133613.png)

---
Хэш:
```
e5d8870e5bdd26602cab8dbe07a942c8669e56d6
```

Соль:
```
tryhackme
```

![](../images/Pasted%20image%2020260506130640.png)
