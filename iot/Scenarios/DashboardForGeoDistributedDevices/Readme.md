## Мониторинг состояния географически распределенных устройств

Данный сценарий выполнен на serverless сервисах, использует кластер PostgreSQL для хранения данных и Yandex DataLens для визуализации.

В качестве примера, сценарий реализует мониторинг вендинговых автоматов для продажи закусок, расположенных в разных районах города. 
Однако используя такой же подход можно реализовать мониторинг любых устройств. Например:

 - Мониторинг автотранспорта
 - Мониторинг кассовой техники
 - Мониторинг показателей качества воздуха
 - Мониторинг климатических установок и т.д.

Детальное описание сценария доступно в документации.

#### Описание сценария

Предположим, что у вас есть вендинговые автоматы для продажи закусок, расположенные в разных районах города.

Каждый автомат передает на сервер данные:

 - показания датчика открытия сервисной дверцы;
 - показатели напряжения сети (вольт);
 - данные об остатке товара типа 1 (процент от полной загрузки автомата);
 - данные об остатке товара типа 2 (процент от полной загрузки автомата);
 - данные об остатке товара типа 3 (процент от полной загрузки автомата);
 - данные об остатке товара типа 4 (процент от полной загрузки автомата);
 - показатели датчика температуры в отсеке выдачи товара;
 - данные о заполненности отсека купюр (процент от полной загрузки);
 - данные о заполненности отсека монет (процент от полной загрузки).
 
Вендинговые автоматы передают данные на контроллер. Контроллер один раз в минуту передает данные на сервер по протоколу MQTT в формате:

```
{
      "DeviceId":"e7a68b2d-464e-4222-88bd-c9e8d10a70cd",
      "TimeStamp":"2020-05-21T10:16:43Z",
      "Values":[
          {"Type":"Bool","Name":"Service door sensor","Value":"false"},
          {"Type":"Float","Name":"Power Voltage","Value":"24.456"},
          {"Type":"Float","Name":"Temperature","Value":"10.456"},
          {"Type":"Float","Name":"Cash drawer fullness","Value":"35"},
 	        {"Type":"Float","Name":"Coin drawer fullness","Value":"35"},
          {"Items":[
              {"Type":"Float", "Id":"1","Name":"Item  1","Fullness":"67.12"},
              {"Type":"Float", "Id":"2","Name":"Item 2","Fullness":"34.34"},
              {"Type":"Float", "Id":"3","Name":"Item 3","Fullness":"89.56"},
              {"Type":"Float", "Id":"4","Name":"Item 4","Fullness":"10.78"},
 	        ]}
      ]
}
```

Сценарий демонстрирует, как настроить мониторинг состояния устройств (например, вендинговых аппаратов), подключенных к сервису Yandex IoT Core и расположенных в разных точках города.
Можно использовать как уже существующие устройтсва, так и эмелятор на базе Yandex Cloud Functions.

#### Функция эмулятора устройства

Код функции находится в файле [device-emulator.js](device-emulator.js).

Точка входа функции - device-emulator.handler
Функцию необходимо запускать с помощью триггера-таймера с [CRON выражением](https://cloud.yandex.ru/docs/functions/concepts/trigger/timer#cron-expression) `* * * * ? *` (вызов раз в 1 минуту).
В функции должен использоваться сервисный аккаунт, имеющий роль iot.devices.writer, для того, чтобы можно было отправлять данные в Yandex IoT Core.

Функция использует следующие переменные окружения:
- REGISTRY_ID - уникальный ID реестра в котором мы создали наши устройства устройства
- CASH_DRAWER_SENSOR_VALUE - процент заполненности денежного ящика 
- POWER_SENSOR_VALUE - базовое значение показания датчика питания 
- SERVICE_DOOR_SENSOR_VALUE - показания датчика открытия сервисной дверцы 
- TEMPERATURE_SENSOR_VALUE - показания датчика открытия двери в серверную комнату
- ITEM1_SENSOR_VALUE - процент оставшегося товара типа 1 
- ITEM2_SENSOR_VALUE - процент оставшегося товара типа 2 
- ITEM3_SENSOR_VALUE - процент оставшегося товара типа 3 
- ITEM4_SENSOR_VALUE - процент оставшегося товара типа 4


#### Функция отправки данных в PostgreSQL

Код функции находится в файле [myfunction.py](myfunction.py).
Функцию необходимо запускать с помощью триггера Yandex IoT Core, который будет вызывать ее при получении новой посылки от устройства.
В функции должен использоваться сервисный аккаунт, имеющий роль editor, для того, чтобы можно было отправлять данные в PostgreSQL.

Функция использует следующие переменные окружения:
 - VERBOSE_LOG — если True, то функция выводит в журнал информацию о своем выполнении
 - DB_HOSTNAME — имя хоста в Yandex Managed Service for PostgreSQL	см. Yandex Managed Service for PostgreSQL
 - DB_PORT — порт подключения к кластеру в Yandex Managed Service for PostgreSQL	6432
 - DB_NAME — имя кластера в Yandex Managed Service for PostgreSQL	db1
 - DB_USER — имя пользователя для подключения к кластеру в Yandex Managed Service for PostgreSQL	user1
 - DB_PASSWORD — пароль подключения к кластеру в Yandex Managed Service for PostgreSQL	пароль, который вы создали в Yandex Managed Service for PostgreSQL