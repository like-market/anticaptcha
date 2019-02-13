# Антикапча
Антикапча для сервиса anti-captcha.com на nodejs

## Пример использования

__Пример продакшн-кода для решения капчи при попытке авторизовать аккаунт в vk.com__  
  
При высокой частоте запросов vk.com попросит ввести капчу, данные от которой содержатся в ответе на запрос.  
В этом случае скачиваем картинку, и отправляем её на сервис anti-captcha.com  
После получения капчи - вызываем функцию авторзации с теме же параметрами, добавив капчу

```js
authorize = async function(login, password, captcha_sid = null, captcha_key = null) {
    // Стандартные параметры запроса
    var params = {
        client_id:     2274003,                // Данные от приложения андроид
        client_secret: 'hHbZxrka2uZ6jB1inYsH', // Данные от приложения андроид
        grant_type:    'password',
        username:      login,
        password:      password,
        scope:         'pages,status,messages,wall,docs,groups,stats',
        v: 5.56
    }

    // Если нужно ввести капчу (т.е. есть данные капчи), то добавляем их
    if (captcha_sid && captcha_key) {
        params.captcha_sid = captcha_sid
        params.captcha_key = captcha_key // текст капчи
    }

    try {
        const response = await axios.get('https://oauth.vk.com/token', {params});
        return response.data;
    }catch (error) {
        // Если нужно ввести капчу
        if (error.response.data.error == 'need_captcha') {
            var captcha_img = error.response.data.captcha_img // Ссылка на капчу
            var captcha_sid = error.response.data.captcha_sid // ID капчи

            // Запрашиваем капчу с сервиса anti-captcha.com
            var [error, captcha_key] = await utils.anticaptcha.getCaptcha(captcha_img)
            if (!error) {
                // Вызываем эту же функцию, добавляя капчу в параметры
                const response = await exports.authorize(login, password, captcha_sid, captcha_key);
                return response;
            }
        }

        return error.response.data;
    }
}

```
