# Prinzip Parser
## Описание
Prinzip Parser - десктоп утилита, которая собирает актуальную информацию любой квартиры с сайта [prinzip.su](https://prinzip.su)
и отслеживает изменение цены выбранных квартир.

## Функционал
* Парсинг выбранной квартиры
* Сохранение результатов парсинга в БД
* Отслеживание изменений цены на квартиру
* Отображение логов
* Возможность сохранения Email адреса локально
* Отправка сообщений по Email
* Возможность изменять состояние квартиры в БД (отслеживать/не отслеживать)

## Инструкция
### Основной функционал
В поле "***Введите URL***" впишите ссылку на квартиру с сайта [prinzip.su](https://prinzip.su) самостоятельно,
или скопируйте и вставьте.
> Ссылка выглядит следующим образом:
> https:&#8204;//prinzip.su/apartments/`type`/`id`,
>  где `type` - набор символов на латинице, `id` - набор целых положительных чисел.

Нажмите кнопку "***Запарсить***", чтобы получить информацию о квартире с сайта.
Если выбранная квартира раннее не парсилась, она отобразится в программе.
> В ином случае, программа сообщит в логах, что квартира уже отслеживается.

Если вы хотите отслеживать изменение цены на эту квартиру, нажмите "***правую кнопку мыши***" по квартире
и выберите опцию "***Установить слежку***".

После того, как вы установите слежку за квартирой, она автоматически добавится в базу данных.
Прежде чем начать отслеживание изменения цены на квартиру, необходимо ввести свой Email адрес в поле "***Введите вашу почту***" и нажать кнопку "***Сохранить***".

Если вы правильно указали почту, программа отобразит ее в правом нижнем углу и предложит сохранить ее, чтобы при каждом входе в программу не вводить заново.

Вам на почту придет сообщение, что почта привязана, чтобы убедиться, что на указанный адрес могут приходить сообщения.

После манипуляций выше, в программе станет доступна кнопка "***Отслеживать***", которая запустит мониторинг состояния цены на квартиру. 
Если цена изменится, вам на почту придет сообщение с ссылкой на квартиру, у которой изменилась цена.
> [!WARNING]
>  Не закрывайте программу, если хотите отслеживать цены на квартиры. После закрытия программы мониторинг отключится.

### Дополнительно
Кнопка "***Показать сохраненные квартиры***" отображает все квартиры, которые находятся в базе данных.
Вы можете изменить состояние мониторинга у определенной квартиры, нажав "***правой кнопкой мыши***" по квартире.

Кнопка "***Обратно***" отобразит квартиры, которые были добавлены путем парсинга в текущей сессии.

## Как все работает?
### Parsing
* Программа отправляет POST запрос с указанным типом проекта и id квартиры на сайт с параметром `?ajax=1&similar=1`.
> Например, тип проекта `newtonpark`, `id == 62287`
* Сервер передает ответ на запрос, который является JSON файлом.
* Файл десериализируется в объект типа `Apartment`, который состоит из свойств `Id`, `Name`, `Price` и т.д.
* Объект Apartment добавляется в коллекцию `Apartments`.
* После выбора опции "Установить слежку" объект добавляется в базу данных PrinzipDB, свойство которого `IsMonitoring == true`.
### Email
* Программа получает данные, которые ввел пользователь.
* Если данные соотвествуют формату e-mail адреса, то они добавляются в переменную `Email`
* Происходит создание `EmailMessage`, в которое входит:
  * От кого (Prinzip Parser)
  * Кому (Email адрес пользователя)
  * Тема и само сообщение
* Происходит создание SMTP клиента (smpt.gmail.com)
  * Производится аутентификация Prinzip Parser
  * Отправляется сообщение пользователю
### Monitoring
* Программа каждые 5 минут выполняет метод `CheckPriceChanges`, который проверяет изменение цены.
* Метод обращается к базе данных и получает коллекцию объектов `DbApartments`, состояние которых `IsMonitoring == true`.
* Объект типа `Apartment` содержит ссылку на квартиру.
* Метод перебирает отслеживаемые квартиры и отправляет POST запрос на сервер.
* Сравнивает значение `Price` и `MortgageMonthly` из базы данных с значениями объекта `Apartment`, полученного в результатах парсинга.
* Если значения отличаются, метод отправляет [сообщение пользователю](README.md#email) с указанием ссылки на квартиру, старой цены и новой цены.

## Будущее проекта
### Развитие
* Реализовать подтверждение почты путем отправки кода доступа, который клиент должен ввести в программе.
* Расширить функционал парсинга:
  * Добавить возможность выбора района квартир.
  * Добавить возможность не указывать ссылку напрямую, а выбирать квартиру в приложении.
  * Добавить возможность парсинга n-го количества квартир.
  * Добавить возможность управлять парсингом.
* Добавить функции для работы с базой данных:
  * Удаление.
  * Изменение.
* Дать клиенту возможность свернуть приложение в трей.
* Сделать качественный UI
* Сделать на основе программы API
### Замечания
Prinzip-Parser возвращает пользователю отслеживаемые квартиры.
В случае изменения цены на квартиру,
программа, находясь в состоянии мониторинга, изменит цену квартиры из базы данных на актуальную,
полученную при проверке методом `CheckPriceChanges`.

По сути, пользователь будет получать актуальные цены на квартиры в программе, но:

Если программа не смогла по каким-либо причинам получить доступ к базе данных,
после выполнения метода `CheckPriceChanges` цена на квартиру в базе данных останется неактуальной.

Это не критическая проблема, ведь Prinzip-Parser будет пытаться изменить цену, пока происходит мониторинг и пока он не получит доступ к БД.
> [!TIP]
> При этом возникает, как говорится, не баг, а фича.
> 
> Пользователю постоянно будут приходить уведомления о том, что изменилась цена на http:&#8204;//xxx квартиру,
> соответственно, можно передавать пользователю предупреждение о том, что цена изменилась на сайте,
> но результат не сохранен в программе, что будет сообщать пользователю, что что-то не так.

## Ссылки
Проект реализован по [заданию](https://disk.yandex.ru/d/TMRMcmMdnXfvsw) от работодателя.

