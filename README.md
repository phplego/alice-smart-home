# alice-smart-home
Простейший фреймвок для управления умным домом с помощью Алисы от Яндекса.

## Требования
* Веб-сервер (достаточно какого-нибудь одноплатного компьютера, например Raspberry Pi)
* Доменное имя
* SSL сертификат для работы HTTPS (можно использовать бесплатный от Let's Entrypt)
* Этот проект основан на Flask, поэтому нужен Python 3.x.x и установленный Flask

## Установка
* Склонируйте репозиторий к себе на сервер
* Отредактируйте __alice.wsgi__ и введите корректный путь к проекту
* Разверните проект на веб-сервере с помощью WSGI, не забудьте разрешить заголовки с авторизацией
* Идём на https://dialogs.yandex.ru/ и нажимаем "Создать навык" -> "Создать диалог" -> "Умный дом"
* Заполняем название (не принципиально)
* Заполняем Endpoint URL: https://_ваш-домен_/
* Не показывать в каталоге -> ставим галочку
* Официальный навык -> нет
* Заполняем остальные поля и загружаем иконку - всё это абсолютно неважно для приватного навыка
* Нажимаем "Авторизация" -> "Создать"
* Придумываем, запоминаем и вписываем идентификатор приложения и секрет
* URL авторизации: https://_ваш-домен_/auth/
* URL для получения токена: https://_ваш-домен_/token/
* "Сохранить"->"Cохранить"->"На модерацию" - модерация должна пройти мгновенно в случае приватного навыка
* "Опубликовать"
* Отредактируйте __config.py__ и введите __CLIENT_ID__ и __CLIENT_SECRET__, которые вы указали, также укажите __USERS_DIRECTORY__, __TOKENS_DIRECTORY__, and __DEVICES_DIRECTORY__ - пути к директориям __users__, __tokens__ and __devices__
* Рекомендуется сделать __chmod go-rwx tokens users__

## Как использовать
* Создайте файл _имя-пользователя_.json в директории __users__ и напишите JSON, в котором должны быть пароль пользователя и доступные ему устройства, например:
```json
{
    "password": "test",
    "devices": [
        "pc"
    ]
}
```

* Создайте файл _имя-устройства_.json в директории __devices__ и напишите JSON с описанием устройства в соответствии с документацией Яндекса: https://yandex.ru/dev/dialogs/alice/doc/smart-home/concepts/device-types-docpage/

Например:
```json
{
    "name": "Компьютер",
    "description": "Основной компьютер",
    "room": "Моя комната",
    "type": "devices.types.switch",
    "capabilities": [
        {
            "type": "devices.capabilities.on_off",
            "retrievable": true
        }
    ],
    "device_info": {
        "manufacturer": "Cluster",
        "model": "0",
        "hw_version": "1.0",
        "sw_version": "1.0"
    }
}
```
* Создайте файл _имя-устройства_.py в директории __devices__ и напишите Python скрипт с двумя методами: *query(capability_type, instance) и *command(capability_type, instance, value, relative)

Пример скрипта для включения/выключения компьютера:
```python
import subprocess

def pc_query(capability_type, instance):
    if capability_type == "devices.capabilities.on_off":
        p = subprocess.run(["ping", "-c", "1", "192.168.0.2"], stdout=subprocess.PIPE)
        state = p.returncode == 0
        # Возвращаем состояние и опционально instance 
        return state, "on"

def pc_action(capability_type, instance, value, relative):
    if capability_type == "devices.capabilities.on_off":
        if value:
            subprocess.run(["wakeonlan", "-i", "192.168.0.255", "00:11:22:33:44:55"])
        else:
            subprocess.run(["sh", "-c", "echo shutdown -h | ssh clust@192.168.0.2"])
        return "DONE"
```
Первая функция должна возвращать текущее состояние устройства и опционально __instance__ (если он не указан ни в запросе, ни в описании устройства), а вторая используется для управления им. В параметрах __capability_type__ и __instance__ передаётся, чем мы управляем, а в параметрах __value__ и __relative__ само значение. Подробности опять же смотрите в документации Яндекса.

* Откройте вкладку "Тестирование" в панели управления Яндекс диалогами и попробуйте связать аккаунты, используя ваши имя пользователя и пароль
* Проверяйте, должно работать как в панели для тестирования, так и на всех устройствах привязанных к вашему аккаунту
