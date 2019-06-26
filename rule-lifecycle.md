### Rule lifecycle

Nastavení lifecycle v každém pravidlu (rule) rozhoduje o okamžiku, kdy bude danné pravidlo vykonáno.

#### Lifecycle fáze

- **FirstLine** - v tomto okamžiku cuberact-proxy přečetlo první řádek příchozího requestu a ví tedy pouze o `method`, `url` a `version`
- **Request** - v tomto okamžiku zná cuberact-proxy celý příchozí request, všechny `headers` i případné `body`
- **Response** - cuberact-proxy dokončilo komunikaci a chystá se poslat response klientovi.


 ![](https://raw.githubusercontent.com/cuberact/cuberact-proxy/master/images/rule-lifecycle/rule-lifecycle.png)