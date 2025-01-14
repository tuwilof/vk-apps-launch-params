# vk-apps-launch-params
Информация о параметрах запуска на платформе VK Mini Apps.

## Оглавление
- [Основная информация](#intro)
    - [Как параметры запуска попадают в приложение](#how-params-are-passed)
    - [Аутентификация пользователей на сервере](#auth)
- [Передача параметров запуска на сервер](#how-to-send-launch-params)
    - [Cons](#cons)
    - [Pros](#pros)
- [Примеры проверки подписи на различных языках](#examples)
    - [PHP](#php)  
    - [Java (1.8)](#java1p8)  
    - [Python 3](#python3)  
    - [Node](#node)
    - [TypeScript](#typescript)
    - [Ruby](#ruby)
  
<a name="intro"/>
  
## Основная информация
Приложение VK Mini Apps получает от ВКонтакте параметры запуска. Они могут содержать различную информацию: место запуска (каталог приложений, сообщество, обычное открытие по ссылке и т.д.), идентификаторы пользователя и приложения, включены ли у пользователя уведомления, какой выбран язык и многие другие. С полным списком параметров запуска можно ознакомиться в [официальной
документации](https://vk.com/dev/vk_apps_docs3?f=6.%2B%D0%9F%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D1%8B%2B%D0%B7%D0%B0%D0%BF%D1%83%D1%81%D0%BA%D0%B0).

<a name="how-params-are-passed"/>

### Как параметры запуска попадают в приложение

Каждый раз, когда мини-приложение запускается, ВКонтакте берёт указанный в настройках URL (или URL для разработки, если вы являетесь администратором приложения) и добавляет в конец строку поиска вместе с query-параметрами запуска. Таким образом, URL, доступный изнутри вашего приложения, будет иметь примерно такой вид:

`https://example.com/?vk_app_id=111&vk_user_id=222&sign=mvkasjdl22Ds&...`

> **Примечание:**
>
> Стоит помнить, что параметры запуска мини-приложения начинаются с префикса `vk_`. Но есть и дополнительный параметр — `sign`. Он отвечает за то, что все переданные параметры запуска являются валидными, то есть не подделаны. Как использовать `sign` рассмотрим ниже.

<a name="auth"/>

### Аутентификация пользователей на сервере

Параметры запуска имеют важную и полезную особенность — их можно использовать как аутентификационные данные на разработанном вами backend-сервисе. Это позволяет сократить время разработки и не утруждать себя написанием собственной системы аутентификации.

Вместе с параметрами запуска, как мы уже писали, передаётся `sign` — подпись, гарантирующая серверу корректность и правдивость параметров.

Безопасность подписи обеспечивается алгоритмом хеширования SHA-256, использующим секретный ключ вашего мини-приложения. Таким образом, не зная ключа, злоумышленник не сможет подделать параметры запуска.

<a name="how-to-send-launch-params"/>

## Передача параметров запуска на сервер

Для того чтобы получить список параметров запуска в строковом виде, достаточно
обратиться к `window.location.search`:

```javascript
// Используем slice(1), для того чтобы отбросить начальный знак вопроса.
const params = window.location.search.slice(1);
```

Если необходимо конвертировать параметры из строкового вида в объект, воспользуемся встроенной в node библиотекой `querystring`:

```javascript
import qs from 'querystring';
// или
const qs = require('querystring');

const params = window.location.search.slice(1);
const paramsAsObject = qs.parse(params);

// Теперь мы можем использовать эти параметры как нам заблагорассудится.
```

<a name="cons"/>

### Cons

Разработчики зачастую допускают ошибку, используя неявный и интуитивно непонятный explicit-метод передачи, — прикрепляемый браузером заголовок Referer, совпадающий с текущим адресом страницы.

Стоит запрещать браузеру прикреплять этот заголовок, иначе при запросе на какой-либо сторонний сервер вы можете, сами того не подозревая, передать ему свои параметры запуска. После этого злоумышленник получит возможность представиться вашему серверу другим пользователем, используя его аутентификационные данные. Как решить эту проблему, читайте [здесь](https://stackoverflow.com/a/32014225).

<a name="pros"/>

### Pros

Самым простым и корректным решением является явная передача своего заголовка и
проверка его на серверной стороне.

```javascript
import axios from 'axios';

// Создаём инстанс axios.
const http = axios.create({
  headers: {
    // Прикрепляем заголовок, отвечающий за параметры запуска.
    Authorization: `Bearer ${window.location.search.slice(1)}`,
  }
});

// Теперь при попытке сделать запросы при помощи ранее созданного инстанса
// axios (именуемого "http"), он будет автоматически прикреплять необходимый 
// нам заголовок, который мы сможем проверить на серверной стороне.
```

После того, как заголовок успешно прикрепляется, необходимо добавить его 
проверку на серверной стороне.

<a name="examples"/>

## Примеры проверки подписи на различных языках

- [PHP](/examples/php.php)
- [Java 1.8](/examples/java1.8.java)
- [Python 3](/examples/python3.py)
- [Node JS](/examples/node.js)
- [TypeScript](/examples/typescript.ts)
- [GoLang](/examples/golang.go)
- [Ruby](/examples/ruby.rb)

