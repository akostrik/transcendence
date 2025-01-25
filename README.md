
* docker-compose up --build
  + пересобрать образы -> Django подхватывает изменения (если настроен **hot-reload**), фронтенд тоже
* https://github.com/bakyt92/14_ft_transendence
* https://docs.google.com/document/d/14zC4f2D8vdh9cYKosDQxsjWYc9aax2hPGuh8Y7CoENI/edit?tab=t.0
* https://docs.google.com/document/d/1O1r9jEdxISjMV29lZgLXWNh-bgPzSlnZ6Nr8QuyP_Jc/edit?pli=1
* наш сайт
  + http://localhost:4444/ 
  + https://localhost:4443/
  + https://localhost:4443/chat
    - HTTP-запрос для загрузки страницы (HTML, CSS, JavaScript)
  + ws://localhost:4443/ws/chat/<roomName>/
    - Ws-запрос отправляется на URL /ws/chat/ (настроен в routing.py)
  + http://localhost:8000/admin
  + https://tr.naurzalinov.me/users/
  + http://95.217.129.132:8000/
* бэк, фронт, база данных, API https://www.youtube.com/watch?v=XBu54nfzxAQ
* REST API на DRF в Pycharm https://blog.jetbrains.com/pycharm/2023/09/building-apis-with-django-rest-framework/
* https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/WebSocket
* https://docs.djangoproject.com/en/5.1/ref/contrib/auth/
* Django Tutorial https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website
* https://www.djangoproject.com/start/
* https://www.django-rest-framework.org/tutorial/quickstart/
* django https://youtube.com/playlist?list=PLA0M1Bcd0w8xZA3Kl1fYmOH_MfLpiYMRs&si=mza4MvfFeRMqgU_B
* django https://youtube.com/playlist?list=PLA0M1Bcd0w8yU5h2vwZ4LO7h1xt8COUXl&si=ddHMMDnBVPiUuEXy
* The Browsable API - Django REST frameworkhttps://www.django-rest-framework.org/topics/browsable-api/
* design https://www.figma.com/design/aWDJYfDmaeCv2NKJ8bJ15n/FT_Transcendence?node-id=109-32&p=f
* // https://miro.com/app/board/uXjVLAphyh8=/

![1-1](https://github.com/user-attachments/assets/e6a157ea-b278-493e-9649-6a361614deac)

### устройствыо проекта
* CSR client-side rendering / SSR server-side rendering
  + CSR данные загружаются на клиентскую сторону, HTML генерируется динамически с помощью JavaScript
  + SSR сервер генерирует и отправляет готовый HTML на клиентскую сторону, каждый запрос требует пересоздания всей страницы на сервере
  + server side rendering is not a good idea: генерация HTML на сервере не является хорошей идеей
  + SSR может быть медленным, сервер должен выполнить обработку данных и сгенерировать страницу каждый раз, когда поступает запрос 
  + if we do it in the front it's easy:
    - сервер отправляет данные (JSON, ...)
    - клиент с помощью JavaScript генерирует HTML на основе полученных данных
    - не требуется повторная генерация страниц на сервере для каждого запроса
  + it is almost front-end
  + if we do this module - power-ups for AI are obligatory (so it is back end part)
    - для модуля AI требуется бэкенд (сложная обработка данных, вычисления, доступ к серверным ресурсам)
* rendering
  + для фронтенда: = рендеринг HTML-шаблонов или динамически обновляемых данных через JavaScript, процесс генерации и отображения контента на веб-странице
  + для Django Views: обработка запросов и отправка ответов в разных форматах (JSON, ...) через API
  + в Django REST Framework (API): процесс обработки запросов и создания ответов в виде JSON, XML или других форматов
    - DRF автоматически обрабатывает рендеринг
  + для Django Channels: передача данных пользователю в реальном времени через протокол WebSocket

### frontend nginx server
* try using **bolt.new** it's better at frontend
  + the ui is fire here
* **game customization** it's just gonna be front
  + like custom colors custom map
* подписывается на **WebSocket-каналы**
* проверяет сертификаты SSL
* расшифровывает их с использованием SSL-сертификата
  + внутренний трафик не шифруется
* если запрос к статическому файлу, отдает его из /usr/share/nginx/html/static/
* запросы на /api/ (JSON, HTML, WebSocket) идут через Nginx на Django (аутентификаци, получения/отправки данных)
  + `proxy_set_header ...` передает заголовки (Host, IP-адрес клиента, протокол, ...), чтобы бэкенд понимал, откуда запрос
  + сохраняет заголовки WebSocket
* нет разделения маршрутов между ws и wss, wss работает через тот же путь, что и ws
* четыре server{}-блока = один процесс
* you should keep separate server blocks, if you are encountering errors after merging the two server blocks
* Vanilla JS (без фреймворков)
* фронт самописный или с использованием минимальных библиотек (jQuery, Bootstrap, ...), не на SPA-фреймворке
* балансировщик нагрузки (если много сообщений, **что будет**?)
* **Bootstrap toolkit**

### backend Daphne 
* runserver 0.0.0.0:8000 => запустили Django-приложение
* слушает внутри контейнера на порту 8000 
* слушает запросы фронтенда: аутентификация, чат, API для фронтенда, авторизация, игровая логика
* обработка API-запросов, HTTP-запросов через Django REST Framework (DRF) стандартным Django-приложением для выполнения бизнес-логики
* передаёт данные о запросе Python-приложению - Python-приложение обрабатывает запрос и возвращает ответ
* обработка ws-запросов через Django Channels framework
* `CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]`
  + не запускает Daphne 
  + запускает **встроенный сервер разработки Django**
    - **не поддерживает ASGI или WebSocket**, не сможет обрабатывать WebSocket-соединения или другие асинхронные протоколы
    - для разработки и тестирования
* хорошо интегрируется с Django Channels и поддерживает WebSocket из коробки
* ASGI сервер 
* asgi, wsgi
  + интерфейс между веб-сервером и веб-приложениями/фреймворками, используемый Django-приложениями
  + стандарт для взаимодействия между веб-серверами и Python-приложениями
  + в Python фреймворки, тулкиты, библиоти, для каждого собственный метод установки и настройки, не умеют взаимодействовать между собой
  + WSGI, Web Server Gateway Interface
    - поддерживает только синхронные запросы HTTP, не подходит для реального времени или протокола WebSocket
    - ывает нужен: некоторые инструменты поддерживают только синхронные HTTP-запросы; Django-приложений без асинхронных функций с WSGI стабильнее
  + ASGI, Asynchronous Server Gateway Interface - продолжение WSGI, когда есть асинхронные задачи
* перехожу на asgi only:
  + стандартные Django-функции (middleware, views, models) продолжают работать, не меняет Django-кода для HTTP
  + удалила файл wsgi.py
  + удалила строку `Gunicorn_APPLICATION = 'myproject.wsgi.application'`
  + nginx location /ws/
  + Тестирование
    - HTTP-запросы (HTML-страница, отправка данных формы)
    - функционал, связанный с WebSocket (чат, ...)
    - клиент подключаетсся к ws:// или wss://
* `ASGI_APPLICATION = "myproject.asgi.application"`
  + сервер ищет в `settings.py модуль `myproject.asgi` и внутри него переменную `application` 
  + Переменная `application` = точка входа для ASGI-сервера, ASGI-приложение, обрабатывает входящие запросы
  + `django_asgi_app = get_asgi_application()` создаёт объект ASGI-приложения, exposes the ASGI callable as a **module-level variable**
    - Initialize Django ASGI application: загрузка приложений из `INSTALLED_APPS`, конфигурация бд, **среда `AppRegistry`**
    - to ensure the AppRegistry is populated before importing ORM models
    - приложение Django инициализирется до использования **ORM**-моделей или других компонентов Django
    - приложения готовы к работе _до_ конфигурации маршрутов WebSocket
    - если снчала импортировать модели или др компоненты Django, будут ошибки, связанные с незарегистрированными приложениями или моделями
* AllowedHostsOriginValidator проверяет допустимые хосты для WebSocket-соединений
* DJANGO_SETTINGS_MODULE = mysite.settings **настройки** для компонент Django (**ORM**, middleware, ...)

### django в целом
* Бэкенд-фреймворк
* ключевые механики (аутентификация, управление базой данных, админка, API), функции и практики из коробки
* диктует архитектуру (приложения, модели, views, urls, ...)
* на базе Python
* `settings.py` конфиг Django на высоком уровне
  + `BASE_DIR` корневая папка проекта  
  + `SECRET_KEY` 
  + `ALLOWED_HOSTS` список доменов/IP, с которых разрешён доступ к этому Django-приложению (когда `DEBUG=False`)
  + `INSTALLED_APPS` список приложений: стандартные (admin, auth, sessions), кастомные: `myapp`, `chat`, `api_42`, ...
  + `ROOT_URLCONF` главный файл роутов `myproject/urls.py`, роутинг для всего проекта
  + `TEMPLATES` настройки для системы шаблонов Django (**HTML-шаблоны**, какие контекстные процессоры использовать)
  + `REST_FRAMEWORK` **рендеры**, по умолчанию JSON и **Browsable API**
  + `AUTH_PASSWORD_VALIDATORS` валидаторы сложности паролейл (минимальная длина, отсутствие совпадений с логином, ...)
* myproject/
  + конфигурация проекта Django, настройки, корневой маршрутизатор, запуск, управление проектом
  + `asgi.py` точка входа Django для запуска приложения
  + `__init__.py` делает папку пакетом Python
* `models.py` структура данных, связи (пользователи, профили, чаты, сообщения, статистика игры, ...)
* `url.py` какие функции обрабатывают какие пути, какие запросы обрабатывает по адресу endpoint
* `views.py` функции  для обработки endpoint, запросов
* `apps.py` (регистрация приложения в Django)
* **`migrations/`** (миграции для моделей)
* Управление пользователями и аутентификацией
  + встроенная система аутентификации и модель пользователей
  + Регистрацию и авторизацию (в том числе OAuth/SSO)
  + Разграничение доступа к страницам (приватные/публичные чаты, комнаты, ...)
  + Проверку токена/сессии при каждом запросе с фронтенда
* Администрирование
  + интерфейс для управления данными (пользователи, чаты, статистика, ...)
  + /admin встроено в django
* Безопасность и валидация
  + инструменты по защите от распространённых уязвимостей (**CSRF, XSS, SQL Injection**)
  + благодаря джанговским формам и сериализаторам + Django REST Framework, упрощается валидация данных, приходящих от фронтенда
* список команд django
  + `docker exec -it 423c6474989f python manage.py help` 
  + [auth]: changepassword createsuperuser
  + [contenttypes] remove_stale_contenttypes
  + [daphne] runserver
  + [django] check compilemessages createcachetable dbshell diffsettings dumpdata flush inspectdb loaddata makemessages makemigrations   migrate optimizemigration sendtestemail shell showmigrations sqlflush sqlmigrate sqlsequencereset squashmigrations startapp startproject test testserver
  + [rest_framework] **generateschema**
  + [sessions] clearsessions
  + [staticfiles] collectstatic findstatic
* DPR, Data Processing and Rendering (Обработка и отображение данных) - данные (JSON, ...) обрабатываются на серверной стороне с помощью Django и затем передаются в представление для рендеринга на клиентской стороне с использованием шаблонов и JavaScript
* `manage.py`
  + скрипт запуска Django-проекта
  + точка входа для административных задач Django (запуск сервера разработки, миграции базы данных, создание пользователей, ...)
  + `execute_from_command_line(sys.argv)` обрабатывает команды, переданные в `sys.argv` (`runserver`, `migrate`, `createsuperuser`, ...)
  + если в вашем проекте используются Docker и автоматизированные процессы, `manage.py` может не использоваться напрямую 
    - в продакшене Django запускается через daphne,   
    - daphne работает с конфигурацией проекта через `DJANGO_SETTINGS_MODULE`
    - `manage.py` не нужен для запуска
  + если миграции применяются автоматически при развёртывании, это делается через Docker, где команды `python manage.py migrate` уже интегрированы в скрипты, вызов `manage.py` встроен в процессы, не нужно вручную его запускать
  + daphne обращаtncw к проекту напрямую через или `mysite.asgi`
  + проверить, что миграции применяются: `python manage.py migrate`
  + проверить, что тесты запускаются: `python manage.py test`
  + проверить, что суперпользователь (администраторский аккаунт) создаётся: добавьте в скрипт развёртывания `python manage.py createsuperuser`
* суперпользователь
  + пользователь административной панели Django Admin
  + объект модели `User` в Django
  + имеет атрибуты `is_staff=True` и `is_superuser=True`, что даёт доступ к админке и все права в системе
  + полный доступ к системе управления данными
    - просмотр
    - редактирование, добавление, удаление записей во всех моделях, в базе через удобный интерфейс
    - Управлять пользователями и их правами
    - Следить за состоянием системы.
    - управлять приложением через графический интерфейс админки
    - помогает тестировать приложение и проверять доступ к админке
    - администраторы могут использовать суперпользователя для настройки данных или исправления ошибок прямо в продакшене
  + Суперпользователь базы данных - другое, пользователь (роль) в PostgreSQL, который имеет полный доступ к структуре базы данных, может изменять схемы, управлять пользователями базы, выполнять запросы напрямую (не связан с Django)
  + если админка не используется, можно отказаться от суперпользователя
* requirements.txt: **Python**-библиотеки для бэкенда
* **Django Debug Toolbar** отслеживание работы проекта, включая middleware
* manage.py django-утилита
  + интерфейс для выполнения административных задач, связанных с настройкой, разработкой и управлением проектом
  + `if __name__ == '__main__':` точка входа программы. Если файл запущен напрямую (а не импортирован как модуль), выполняется функция `main()`
  + `python manage.py runserver` запуск сервера разработки 
  + python manage.py migrate` применение **миграций** базы данных
  + `python manage.py createsuperuser` создание суперпользователя
  + можете создавать собственные команды (внутри приложения в папке `management/commands`) и запускать их через `manage.py`.

### django rest framework DRF
* создаёт HTTP API с поддержкой **сериализации**, аутентификации, прав доступа
* протокол HTTP(S) используется для создания RESTful API, которые обрабатывают стандартные HTTP-запросы (GET, POST, PUT, DELETE)
* ваимодействие с базами данных и другими внешними сервисами (API École 42)
* game logic, because we need it to do the multiplayer
* рендеринг **шаблонов** (**Server-Side Rendering**)
* js: нопка "создать турнир" -> вызывает 127.0.0.1/tour -> запускает DRF
* с ним: PUT = поменять поле в бд, без него запрос sql
* middlware
  + набор классов
  + не являются серверами
  + фильтры или обработчики, модифицирующими запросы/ответы
  + каждый middleware по очереди обрабатывает запрос и передаёт запрос следующему 
  + **для обеспечения доступа к объекту пользователя (request.user) и другим данным авторизации**
  + SecurityMiddleware
    - добавляет заголовки безопасности (`Strict-Transport-Security` для HTTPS)
    - перенаправляет HTTP-запросы на HTTPS (если настроено) **куда именно? какой-то порт?**
    - убирает потенциально опасные элементы из запросов (например, от уязвимости **XSS**)
  + SessionMiddleware
    - загружает данные сессии из хранилища
    - если сессии хранятся в Redis или базе данных, сохраняет данные сессии после выполнения запроса (Если пользователь **аутентифицирован**, сессия **сохраняет его идентификатор**)
    - для управления сессиями через куки
  + CommonMiddleware
    - базовая обработка (например, перенаправление с /)
    - **Перенаправление при добавлении/удалении `/` в конце URL** (если включено)
    - Возвращает стандартные заголовки HTTP (`Content-Type`, ...)
  + CsrfViewMiddleware
    - Проверяет наличие и правильность CSRF-токена
    - Если токен отсутствует или неверный, возвращает ошибку `403 Forbidden`
    - защита от атак типа **Cross-Site Request Forgery**
  + AuthenticationMiddleware
    - Проверяет сессию или **заголовки авторизации**
    - Добавляет **объект `request.user`**, чтобы представления могли определить, аутентифицирован ли пользователь
  + MessageMiddleware
    - обрабатывает систему сообщений (flash messages): временные уведомления или сообщений пользователю на веб-странице, которые исчезают после того, как пользователь обновит страницу или перейдет на другую
    - для отображения успешных действий, предупреждений или ошибок, например, после того как пользователь отправил форму, обновил профиль, или завершил какое-то действие в приложении
    - Когда пользователю нужно показать сообщение, это сообщение помещается в **сессии** с помощью системы сообщений
    - сообщения остаются в сессии только до следующего запроса, а затем удаляются автоматически
    - не имеет отношения к чату: чат требуют постоянного обновления страницы или использования WebSocket для обмена сообщениями
    - сообщения отображаются в верхней части страницы и исчезают через несколько секунд или при обновлении
    - **может быть настроен для обработки сообщений и добавления дополнительной информации или фильтрации на уровне WebSocket**
  + XFrameOptionsMiddleware
    - добавляет заголовок `X-Frame-Options` для защиты от атак **clickjacking** (например запрещает отображение сайта в **iframe** на других доменах)
* **ViewSet/APIView:** Обработчики HTTP-запросов, определяют, как данные обрабатываются для каждого типа запроса
* view обрабатывает запрос, формирует ответ
* middleware на обратном пути
  + XFrameOptionsMiddleware обрабатывает ответ (добавляет заголовок X-Frame-Options)
    - XFrameOptionsMiddleware добавление заголовков безопасности
  + MessageMiddleware (сохраняет flash-сообщения)
  + AuthenticationMiddleware (завершает обработку данных авторизации)
    - добавление объекта `request.user` для авторизованных пользователей 
  + CsrfViewMiddleware (может добавлять CSRF-токен в шаблон)
    - проверка CSRF-токена **ещё раз?**
  + CommonMiddleware (может модифицировать заголовки)
  + SessionMiddleware сохраняет сессии, если они изменились
  + SecurityMiddleware
    - проверка заголовков
    - добавление заголовков безопасности
* аутентификация реализуется через middleware, через стандартный механизм HTTP-запросов, например, с использованием **JWT или сессий**
* сессии управляются с помощью cookie-сессионных данных или токенов для аутентификации API
* каждый запрос обрабатывается отдельно, и клиент получает ответ сразу
* **среда `AppRegistry`** управляет регистрацией приложений и моделей
* использует сессии или JWT для аутентификации пользователей
  + эти данные доступны в middleware или обработчиках запросов
  + Session storage: Если включён SessionMiddleware, пользовательские данные будут сохраняться в сессиях и доступны в обработчиках.
  + JWT токен передаётся в заголовке авторизации (Authorization: Bearer <token>) и обрабатывается специальным классом аутентификации
* APIView 
  + инструмент для создания API
  + класс в DRF
  + более низкоуровневый способ для создания API, чем ViewSet
  + наследуется от класса View
  + добавляет поддержку сериализации, работы с JSON, вспомогательные методы для обработки запросов HTTP, `GET` `POST` `PUT` `DELETE`
  + сам контролируешь, что происходит в ответ на запросы
  + подходит для создания простых API с базовой логикой
  + если требуется большая гибкость в обработке запросов и собственная логика (сложные алгоритмы, особое поведение при получении или отправке данных)
  + если требуется больше контроля над процессом обработки запроса, нужно сделать что-то кастомизированное
* ViewSet 
  + более высокоуровневый класс
  + предоставляюет автоматическую обработку запросов для стандартных операций CRUD (Create, Read, Update, Delete)
  + часто используется для создания API для работы с моделями в Django
  + генерирует маршруты для операций с ресурсами (создание нового объекта, получение списка объектов, обновление, удаление)
  + позволяет избежать написания методов для каждого HTTP-запроса, таких как `GET`, `POST`, `PUT`, `DELETE`
  + использует методы по умолчанию, предоставляемые DRF
  + автоматически предоставляет стандартные действия для модели
  + меньше ручного кода для обработки CRUD-операций
  + позволяет легко настраивать маршруты для ресурсов API с помощью router
  + для стандартных операций, связанных с моделями (создание, получение, обновление, удаление объектов)
  + подходит для ресурсо-ориентированных API, где операции с объектами ("пользователи", "продукты") нужно реализовать с минимальными усилиями
  + быстро и эффективно настроить API для CRUD-операций над данными, которые уже есть в базе
   
### django channels framework
* Django Channels и Django REST Framework (DRF)
  + обрабатывают разные типы запросов
  + схожие задачи: обработка запросов и аутентификация пользователей
  + работают с разными протоколами
  + аналогичные концепции для работы с запросами и аутентификацией можно найти в обоих
  + протокол WebSocket, используется для двухсторонней связи в реальном времени (чаты, уведомления)
  + специальные middleware для обработки WebSocket-соединений:
    - SessionMiddleware – управляет сессионными данными (хранит информацию о пользователе, сессии, ...)
    - AuthenticationMiddleware – выполняет аутентификацию пользователей, связывая их с подключением WebSocket
  + Consumer обработчик для работы с ws-соединениями, аналогичный представлению в DRF, для работы с асинхронными запросами через WebSocket
* views.py **проверяет токен/подпись**
* django channel создает websocket, слушает запросы
  + отличия от html: непрерывное соединение, стрим
  + new Websocket(127.0.0.1:8000/game, wss)
+ Channels Middleware (WebSocket)
  + AuthenticationMiddlewareStack
    - проверяет, что WebSocket-соединение аутентифицировано: различные методы аутентификации, например, проверку токенов (JWT, Cookies и другие)
    - связывает пользователя с запросом.
  + SessionMiddleware
    - в случае использования аутентификации через сессии
    - для получения идентификатора сессии и связанного с ним пользователя
    - для сохранения информации о сессии WebSocket-подключения: идентификаторы сессии и связанные данные (информация о пользователе)
  +  Consumer (Channels Consumer)
    - обработчик сообщений ws в Django Channels (аналогичные представлениям в традиционном Django)
  + отправлены через Channel Layer (Redis)
    - для обмена сообщениями между различными экземплярами приложения, например, для отправки сообщений между WebSocket-соединениями
    - когда данные отправляются в channel layer, они могут быть опубликованы в опр. канал и доставлены тем, кто подписан на канал
* аутентификация осуществляется через `AuthenticationMiddleware`, который связывает подключение с пользователем
* сессии также обрабатываются через `SessionMiddleware`, и сессионные данные передаются через WebSocket-соединения
* Используется для асинхронной связи и поддерживает постоянные соединения, что позволяет получать и отправлять данные в реальном времени
   - После того как данные прошли через **backend (Redis)** и канал, ответ снова передается в ваш **consumer**, который отправляет его через WebSocket обратно пользователю.
* специфичных для Channels middleware, например, для ограничения скорости, управления правами доступа или обработки ошибок могут быть добавлены между слоями
* Путь запроса WebSocket 
2. **AuthenticationMiddlewareStack** — проверка аутентификации.
3. **SessionMiddleware** (если используется) — работа с сессией.
4. **Consumer** — обработка сообщения WebSocket.
5. **Backend Channel Layer (Redis)** — обмен данными через канал.
6. Ответ отправляется обратно в **Consumer** через WebSocket.
* сессии или JWT тоже могут использоваться для аутентификации пользователей, но их доступность обеспечивается через middleware Channels.
  + Сессии передаются через cookie, а Django Channels может использовать SessionMiddlewareStack для доступа к данным сессии.
  + JWT также может быть передан через заголовки WebSocket-сообщений или параметры URL, но нужно вручную обрабатывать его в consumer'е
* SessionMiddlewareStack: работать с данными сессии в WebSocket-сообщениях
  + обязателен, если вы используете сессии
    - Для авторизации через стандартный `django.contrib.auth`
    - Для хранения пользовательских данных между запросами.
    - Если вы работаете с `request.session`
  + обязателен в проектах, требующих взаимодействия с сессиями на сервере
    - Например, если используется сессионное хранилище в базе данных, Redis или файловой системе
  + обязателен, если вы используете стандартный механизм аутентификации Django `django.contrib.auth`
     - `django.contrib.auth` опирается на сессии для хранения данных о пользователе
  + можно без него, если вы используете вместо сессий  JWT токены или другие безсессионные методы аутентификации
    - Например, в REST API (DRF) сессии не нужны, авторизация проходит через токены
  + можно без него, если данные о сессии хранятся полностью в зашифрованных cookie (без серверного хранилища)
* AuthMiddlewareStack: предоставляет объект пользователя (scope["user"]) в consumer'ах, если пользователь аутентифицирован
* **не требует явного добавления в INSTALLED_APPS**
  + настроить ASGI 
  + установить Daphne
  + settings.py устанавливается ASGI-приложение
  + asgi.py конфигурация
        
### django cash framework
* инфраструктура для кэширования
* хранение кэша (запросы, объекты, шаблоны, ...)
* Django обращается к **redis**
* не требует добавления приложения в INSTALLED_APPS
* настраивается в файле settings.py, CACHES

### приложения Django app - отдельные модульные приложения внутри проекта INSTALLED_APPS
* myapp
  + логика пользовательских профилей, турниров, историй игр
  + модель `UserProfile`: инфо о пользователе, друзьях, аватаре, дате последней активности, ...
  + модель `Tournament`: список участников (many-to-many к `UserProfile`)
  + модель `Game` история матчей, кто участвовал, какой турнир, счёт, победитель
  + `friends = models.ManyToManyField("self")` пользователи могут быть друзьями 
  + расширить профили (рейтинг, биографию, статистику), турнирную логику (сетка турнира, раунды), механику игр:
     - добавлять поля
     - добавтиь **миграции** в соответствующие модели
    + **API** для работы с сообщениями
* 'django.contrib.messages'
* 'django.contrib.admin',
* 'django.contrib.auth',
* 'django.contrib.contenttypes',
* 'django.contrib.sessions',
* 'django.contrib.staticfiles'
* Используем **стандартные структуры юзера для авторизации и для моделей данных**
* **какие есть встроенные**
* логика в views.py

### обмен сообщениями
| **Method**                | **Real-Time** | **Message Persistence**  | **Use Case**                           | **Complexity** |
|---------------------------|---------------|--------------------------|----------------------------------------|----------------|
| WebSocket (chat)          | Yes           | No                       | Real-time chats, games                 | High           |
| Redis (Pub/Sub, WebSocket)| Yes           | No                       | Notifications, chats                   | High           |
| Django Messages           | No            | Yes                      | System notifications, confirmations    | Low            |
| REST API                  | No            | Yes                      | Simple notifications, data requests    | Low            |
| Email                     | No            | Yes                      | Important notifications, confirmations | Medium         |
| Push Notifications        | Yes           | Yes (by service)         | Mobile device notifications            | Medium         |

* REST API
  + **обмен данных через API в формате REST**
  + авторизация с помощью токенов (JWT) и другие механизмы для обработки запросов и откликов между клиентом и сервером
  + уведомления и сообщения могут быть частью бизнес-логики
  + реализуются
    - через события
    - через статусы в ответах API
    - не через механизм сообщений Django
* 'django.contrib.messages'
  + приложение
  + фреймворк
  + предоставляет API для работы с flash-сообщениями
  + сообщения удаляются после отображения
  + сообщения пользователю **между запросами** (об регистрации, ошибка при неверном вводе данных, ...)
  + реализуется через механизм сообщений Django
  + хранит сообщения временно (в сессии или cookies)
    - бэкэнд для хранения сообщений `MESSAGE_STORAGE = 'django.contrib.messages.storage.cookie.CookieStorage'`
  + уровни важности сообщений: `messages.debug`, `messages.info`, `messages.success`, `messages.warning`, `messages.error`
  + обмен данными через WebSocket соединения  
    - бывает и через HTTP-запросы, добавляя сообщения во время перенаправлений или в шаблоны
  + подключается к `MessageMiddleware`
    - `MessageMiddleware` автоматически сохраняет и извлекает сообщения из хранилища (cookie, сессии, ...)
    - сообщения сохраняются между запросами и автоматически удаляются после первого отображения
    - без `MessageMiddleware` только для сообщений **внутри одного запроса**, сообщения не сохраняются между запросами
    - `MessageMiddleware` без `django.contrib.messages` невозможно использовать
  + flash-messages
    - временные уведомления
    - отображаются пользователю после выполнения действия и исчезают после отображения
    - не используются для функций реального времени, как в чате
    - не асинхронны
    - одноразовые сообщения через **редиректы** (например, после POST-запросов)
    - работают благодаря связке `django.contrib.messages` + `MessageMiddleware`
    - результ действия: "Ваше сообщение отправлено", "Неверный пароль"
    - Операции в чате: уведомления о добавлении нового участника в группу
    - Управление игровыми процессами: Ваш запрос на матч отправлен
    - Администрирование: Настройки сохранены, Ошибка при обновлении параметров"
    - Передачи статуса операций между страницами: После успешного сохранения формы пользователю показывается сообщение на новой странице
* Реальное время (WebSocket/Push)
  + для отправки сообщений в чате
  + для **обновлений интерфейса** в режиме реального времени

### django app chat
* с реальным временем
* на WebSocket
* tuto https://channels.readthedocs.io/en/latest/index.html
* js обращается к rest api (post) endpoints /history, /users/, /send
* rest api строит и отдаёт html  
* js получает ответ (get)
* с каждым пользователем у бэкенда 2 вебсовета: чат, положение ракетки
* **история в бд**
* ws login new WS connexion
* ws system msg
* ws user communications
* INSTALLED_APPS 'channels'
* CHANNEL_LAYERS 'BACKEND': 'channels.layers.InMemoryChannelLayer', **in-memory ?**
* `chat/models.py` модели сообщений и комнат
* `chat/consumers.py` WebSocket consumer 
* `chat/routing.py маршрут
* `chat/urls.py` маршрут для комнаты чата
* `chat/views.py представление
* `python manage.py makemigrations`, `python manage.py migrate` создайте и примените миграции для моделей 
* Django Channels
  + библиотека, надстройка Django для вебсокетов, добавить поддержку протокола WebSocket
  + полагается на стандартную инициализацию Django для HTTP-запросов
  + использует дополнительную логику для обработки WebSocket-соединений и других типов протоколов
  + для приложений с функциями реального времени, выходящих за рамки стандартного HTTP-протокола: онлайн-игра, онлайн-статус пользователей
  + WebSocket - постоянное соединение между клиентом и сервером
  + WebSocket - передавать данные в режиме реального времени
  + подходит для чата
* RabbitMQ брокер сообщений для системных сообщений (уведомление о начале турнира, уведомление о добавлении в друзья), не нужен
* класс router обрабатывает перемещения по сайту
  + popstate кнопки назад вперёд в брузере
* на место .app подставляется div
  + div = homapage, profile, ... (все они наследуют от Component)
  + constructor() создаёт класс
  + class Component базовый, абстрактный - **понять**
  + render = вернуть html
  + state = переменные 
  + событие DomContactLoaded = html полностью загрузился (у нас только 1 раз)
  + фиксированная часть страницы доступна в js
  + % body % кберем, т.к. у нас SPA
  + login pages.js - запрос post к бэку
    - в заголовке запроса каждый раз CSRF токен, чтобы знать, что это не юзер с третьего сайта
  + **чат надо сделать компонентом**
  + fetch (в js) = запрос к бэку
  + функция render своя в каждом компоненте
    - например нужен username - делаем запрос к бэку fetchuserprofile
* websocket объект js
* send.msg
* receive = render msg
* websocket: кому пишем сообщение? в запросе
* prepMsg забирает инпут и делает ws запрос
  + взять список пользователей из базы
* @login_required только если залогинен, если нет токена или он неправильный - не пропустит
* fetch() запрос обычный (не websocket)
* websocket один для всех пользователей для чата
* **AbsTimeUser** (непралвьно написано) наш класс наследует
* johnResponse - временный
* view.py не html
  + jsonResponse или httpResponse
* createuser встроенная, т.к. наследуем от ... 
* **[2302]** в контейнере бэк: python manage.py make migrations
* если логин - присылает **session id token**
* первый логин - header CSRF токен
  + посмотреть это: F12 Applocation storage cookies
  + в js фенкция post - в header токен CSRF
  + session id приязывает сессия + юзер в приложении
* сначала выполняется header, потом подгружаются стили
* incognito
* в django новая функция: python manage.py statup, migrations, INSTALLED_APS ... 'chat'
* 1 эндпоинт слушает чат
  + в заголовке запроса - кому сообщение
* если сообщение не дошло, то такого пользователя нет
* в контейнере бэк: python manage.py superuser то ли python manage.py createsuperuser, с именем админ пародлем админ
  + после этого migrations
* pathname = часть адреса после корня
* login = new ws connexion
* ws system msgs
* ws user communications
* можно ли делать live chat с библиотекой channels или надо целиком писать? Какие библиотеки можно использовать?
* базовый функционал чата
  + `daphne` настроен для обработки вебсокетов, `asgi.py`, routing, WebSocket-соединения перенаправляюся в обработчики
  + `ChatConsumer` для обработки сообщений:
  + фронтенд, соединение с WebSocket-сервером
    ```javascript
    const chatSocket = new WebSocket(
        'ws://' + window.location.host + '/ws/chat/room_name/'
    );
    chatSocket.onmessage = function(e) {
        const data = JSON.parse(e.data);
        document.querySelector('#chat-log').value += (data.message + '\n');
    };
    chatSocket.onclose = function(e) {
        console.error('Chat socket closed unexpectedly');
    };
    document.querySelector('#chat-message-submit').onclick = function(e) {
        const messageInputDom = document.querySelector('#chat-message-input');
        const message = messageInputDom.value;
        chatSocket.send(JSON.stringify({
            'message': message
        }));
        messageInputDom.value = '';
    };
    ```
  + Обновите шаблоны HTML
      ```html
      <textarea id="chat-log" cols="100" rows="20" readonly></textarea><br>
      <input id="chat-message-input" type="text" size="100"><br>
      <button id="chat-message-submit">Send</button>
      ```
  + База данных: модель для хранения сообщений
    ```python
    class Message(models.Model):
        user = models.ForeignKey(User, on_delete=models.CASCADE)
        room = models.CharField(max_length=255)
        content = models.TextField()
        timestamp = models.DateTimeField(auto_now_add=True)
    ```
  + сохранение сообщений в `ChatConsumer` при получении данных
    ```python
    from .models import Message
    async def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json['message']
        user = self.scope['user']
        await database_sync_to_async(Message.objects.create)(         # Save to database
            user=user, room=self.room_name, content=message
        )
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message,
            }
        )
    ```
  + Проверьте соединения через WebSocket (например, с помощью браузера)
  + Убедитесь, что Redis корректно и сообщения передаются через каналы
  + Проверьте сохранение сообщений в базе данных
  + Развертывание `nginx.conf` location /ws/ 

### django REST API (Application Programming Interface)
* интерфейс
* набор правил, позволяет приложениям взаимодействовать друг с другом
* группа маршрутов (эндпоинтов)
* клиент использует для взаимодействия с сервером
* определяются в `urls.py`
* определяются в View-классах или функциях
  + классы, наследущие от `APIView`, `GenericViewSet`, `ViewSet`
* `routing.py` маршруты WebSocket
  + каждый WebSocket-путь = отдельное API
  + WebSocket - часть API
* могут быть
  + стандартными REST (GET, POST, PUT, DELETE)
  + асинхронными WebSocket (через Django Channels)
* несколько эндпоинтов реализуют функциональность API
  + приложение `api_42`
  + приложение `chat`
  + приложение `auth_app`, для каждого из них 5–10 эндпоинтов
  + для аутентификации/авторизации (École 42, ...)
  + для работы с чатом (отправка, получение сообщений)
  + для обработки пользовательских данных (профиль, друзья, статистика)
  + для игры (управление матчами, таблицы лидеров)
  + для управления пользователями 
    - маршрут `GET /users/` список пользователей
    - маршрут `POST /users/` создать нового пользователя
    - маршрут `GET /users/<id>/`
    - маршрут `DELETE /users/<id>/`
    - маршрут API чата = группа всех маршрутов, связанных с чатом
* **создаются с помощью Django REST Framework (DRF)**
* список API http://localhost:8000/swagger/, http://localhost:8000/redoc/
  + если настроена автоматическая документация (Swagger, Redoc)  

### Endpoint 
* конкретный маршрут, связанный с API
* выполняет определённое действие
* Endpoints чата:
  + `GET /chat/rooms/` — список комнат
  + `POST /chat/rooms/` — создать комнату
  + `GET /chat/rooms/<room_id>/messages/` — получить сообщения из комнаты
  + `POST /chat/rooms/<room_id>/messages/` — отправить сообщение
* endpoints в `urls.py`:
* команда Django `python manage.py show_urls` список эндпоинтов
* `grep -r "path(" backend/`, `grep -r "re_path(" backend/`

### F12 concole
* лучше всего в chrome
* colsole.log для отладки
* кнопка квадратик со стрелкой слево вверх - смотреть код элемента html
* js менять параметры html, class = стиль
* open chat
  + на странице login, profile, решгистрация нету
  + во время игры - статистика, другой user
  + приложение, отправлятть сообщение через js !!
  + django channel = framework для сокетов
* прямо в консоли можно писать js и пробовать
  + ndetermined - то, что ыернула функция
  + можно создать переменные (let)
    - они сохраняются в обхекте window (window = браузер)
    - x или window.x досутп к этой переменной
  + api браузера - геолокация, звук, 
* Application - хранилище:
  + куки
  + local storage
  + ... storage
* Netrwork
  + запросы от фронта
  + document = html

### autorisation
* При каждом запросе клиент отправляет токен в заголовке Authorization: Bearer <token>
* Сервер декодирует токен, проверяет его подпись, использует данные для авторизации (например, проверяет роль пользователя)
* виды авторизаций: Ecole 42, гугл, логин и пароль с базы данных
* задача на бэкэнде
* внедрить 2FA через электронную почту
  + Мы выполнили первую часть, генерируя код и все такое
  + были проблемы с последней частью, где мы на самом деле отправляем электронное письмо
    - попробовали sendgrid life, было сложно добиться успеха в отправке электронных писем
  + мы использовали Gmail, самое сложное — найти путь через пользовательский интерфейс Google для создания «mdp приложения» на стороне Django, все легко делается с помощью функции send_mail
    - сначала пришло как спам, а потом появилось само по себе после нескольких писем
    - со стороны Django это происходит, Gmail изменил условия две недели назад, с тех пор это стало проблемой

### db PostgreSQL
* СУБД для хранения пользователей, сообщений, данных о матчах в Pong, статистики, ...
* `psql -U myuser -d mydatabase` `\dt`
  ```
   Schema |                Name                | Type  | Owner  
  --------+------------------------------------+-------+--------
   public | auth_group                         | table | myuser
   public | auth_group_permissions             | table | myuser
   public | auth_permission                    | table | myuser
   public | django_admin_log                   | table | myuser
   public | django_content_type                | table | myuser
   public | django_migrations                  | table | myuser
   public | django_session                     | table | myuser
   public | myapp_game                         | table | myuser
   public | myapp_tournament                   | table | myuser
   public | myapp_tournament_players           | table | myuser
   public | myapp_userprofile                  | table | myuser
   public | myapp_userprofile_friends          | table | myuser
   public | myapp_userprofile_groups           | table | myuser
   public | myapp_userprofile_user_permissions | table | myuser
  ```

### redis (Remote Dictionary Server) 
* инструмент управления данными
* репликация и кластеризация для масштабирования и высокой доступности
* управление пользовательскими сессиями в веб-приложениях, хранение данных сессий
* работает в оперативной памяти => высокая скорость
  + важно следить за потреблением памяти
* быструю запись и чтение
* хранение данных о текущих авторизованных пользователях. Управление сессиями при подключении пользователей к WebSocket
* хранение сессий (если Django настроен)
* механизм списков или Pub/Sub для реализации очередей сообщений
* Быстрый подсчёт статистики, временных данных
* Реализация чата (Совместно с WebSocket для быстрой обработки сообщений. В **Pub/Sub** позволяет рассылать сообщения всем пользователям чата.)
* очередь задач (отправки email, push-уведомления, логирование, обработка фоновых задач (обновление статистики матчей))
* для матчей в реальном времени (текущее состояние игры, обмен данными между игроками,хранение лидербордов с помощью упорядоченных множеств (sorted sets))
  + база данных
  + брокер сообщений, быстрый обмен сообщениями
  + хранение Pub/Sub (для реального времени)
    - Pub/Sub для чата
    - Pub/Sub (механизм публикации/подписки) для обмена сообщениями
    - Pub/Sub в реальном времени (чат, игра Pong)
    - бекенд для Pub/Sub и WebSocket-сессий (чат, игра в реальном времени, ...)
  + Когда клиент отправляет сообщение через WebSocket, **оно сначала сохраняется в Redis**, после чего перенаправляется другим клиентам или серверу в зависимости от логики
  + канальный слой, для поддержки системы каналов через Django Channels
  + реализация реального времени через Django Channels (WebSocket)
  + Коммуникации между разными инстансами приложения
  + инструмент маршрутизации между подключёнными клиентами
  + Обработка событий WebSocket
  + Django Channels позволяет приложению обрабатывать протоколы реального времени (WebSocket, ...)
* Библиотеки абстрагируют низкоуровневую работу с Redis, скрывают сложность интеграции
  + `channels_redis` подключением и отправкой сообщений в Redis
  + `django-redis` сериализацию данных, запись, извлечение
  + легко интегрируется с Django через библиотеки `channels_redis` и `django-redis`
* хранение и маршрутизация данных
* **in-memory key-value store**
* значения могут быть: string, list, hash, set, sorted sets, bitmap, HyperLogLogs, геопространственные индексы и др  
* поддерживает множество языков программирования
* Транзакции и скрипты на языке Lua
* может сохранять данные на **диск**, чтобы избежать потери при сбоях
* пддержка Pub/Sub (публикации/подписки) => идеальный для каналов WebSocket
* Убедитесь, что сервер Redis защищён (настройка пароля, ограничение доступа, ...)
* Настройте Redis для работы с высоким количеством соединений
* настройки: время жизни кэша (`TIMEOUT`), ...
* Убедитесь, что Redis не перегружен `redis-cli info memory`
* очистить кэш `redis-cli -n 1 flushdb`
* один сервер Redis = кэширование + обслуживание channel layers
  + два процесса на одном сервере Redis, с разными ключами и настройками
* если WebSocket-запросы зависят от Redis (хранение данных о сессиях, ...), убедитесь, что Redis работает: `redis-cli ping`, Ожидаемый ответ: `PONG`
* логи Daphne или Redis - посмотреть соединения WebSocket создаются и работают стабильно?
* Redis WARNING Memory overcommit must be enabled!
  + `sudo sysctl vm.overcommit_memory=1` настройка системы, чтобы Redis мог эффективно сохранять данные в условиях низкой памяти
  + `vm.overcommit_memory = 1` в `/etc/sysctl.conf` сохранить настройку после перезагрузки
* используется механизм Pub/Sub (Publish/Subscribe) для реализации чата?
  + Поиск в коде `PubSub`, `publish`, `subscribe`, `event`, `broker`, `topic`
    - библиотеки  `Redis`, `RabbitMQ`, `Kafka`, WebSocket-библиотеки с поддержкой Pub/Sub
  + Если чат реализован через WebSocket, Pub/Sub может быть встроен в логику
    . Например, отправка сообщения (`publish`) уведомляет подписчиков (`subscribe`)
  + Redis = брокер сообщений в режиме Pub/Sub
     - Подключение к Redis (`redis.createClient()` или аналог)
     - Вызовы методов `publish()` и `subscribe()`
  + Если вы используете NestJS, в нем есть встроенная поддержка Pub/Sub через библиотеки, такие как `@nestjs/microservices`.
   - Проверьте папку `services`, `chat`, или `events` на наличие Pub/Sub-логики.
  + Найдите модули или файлы, связанные с чатом (`chat.service`, `chat.controller`)
    - если сообщения отправляются на брокер или WebSocket, это может быть признаком Pub/Sub
  + `package.json` зависимости, связанные с Pub/Sub `cat package.json | grep -E "redis|socket.io|nestjs/microservices"`
  + Отправьте сообщение в чат и проверьте:
  - Логирование на сервере. Часто Pub/Sub системы логируют публикацию и доставку сообщений.
  - Задержки при передаче сообщения. Если сообщение доходит мгновенно, вероятно используется WebSocket с логикой Pub/Sub

#### Redis для кэширования
* Кэширование использует Redis как хранилище данных в памяти с быстрым доступом
* Django кэширует данные, которые часто запрашиваются, но редко изменяются
  + временне данные для ускорения работы приложений (имя, аватар, статус
  + результатов сложных SQL-запросов
  + рендеринг HTML-шаблонов, чтобы повторные запросы быстрее обрабатывались
  + результаты REST API-запросов, чтобы не нагружать базу данных повторными запросами
  + API-ответов (данных профиля пользователя, данных статистики)
  + сеансы: **хранилище** сеансов для авторизованных пользователей
  + пользовательские сессии, что **ускоряет работу сессий в распределённых системах**
  + временный статус пользователя: **онлайн/офлайн**
  + результаты частых запросов (сложные вычисления, данные из базы)
* по умолчанию django кэширует в памяти `django.core.cache.backends.locmem.LocMemCache` 
  + кэш в процессе работы сервера
  + если приложение работает в пределах одного сервера и не требует масштабируемости
  + нет сильной нагрузки или большого объёма данных
  + не требует настройки и внешнего сервиса
  + доступ к данным в памяти быстрее
  + не нужно хранить кэш между перезапусками
* у нас Redis для кэширования
  + `CACHES`: `django_redis` = backend для кэширования
    - она берёт на себя работу с Redis, предоставляя **Django-friendly API**
    - **класс клиента определяет, как Django взаимодействует с Redis**
    - `OPTIONS` = `CLIENT_CLASS` настраивает использование `DefaultClient` из библиотеки `django-redis` для взаимодействия с Redis
  + если приложение масштабируется
  + если проект в распределённой среде, в многосерверных или контейнеризованных средах, несколько серверов обращаются к одному кэшированию, данные в процессе/сервисе (Redis-сервере), а не в памяти отдельного экземпляра приложения
  + несколько инстансов приложения могут одновременно использовать Redis как общий слой
  + сохранять кэшированные данные на диск (persistent storage), данные могут быть восстановлены после перезапуска или сбоя
  + кэшированные данные переживают перезапуск приложения или контейнера (например, для сессий)
  + сложные типы данных в кэше (списки, множества, хеши)
  + сложные стратегии кэширования: TTL (время жизни), евикция (удаление старых или неиспользуемых данных)
  + ограничение количества запросов от одного пользователя за определённое время (rate limiting)
* bakyt: только кэш сообщений, чат и **системные**
* Why the Redis container is set to "restart: always"?
* встроенные механизмы кэширования Django без Redis
  + Простое в настройке
  + Ограничено по производительности и функциональности
  + Подходит для небольших и локальных проектов
  + 1 типа: Локальное кэширование в памяти (`LocMemCache`)
    - кэш в оперативной памяти локального сервера
    - для небольших проектов
    - данные не разделяются между несколькими процессами или серверами
    - кэш очищается при перезапуске приложения.
  + 2 типа: Файловое кэширование (`FileBasedCache`)
    - кэш в файловой системе
    - для простых сценариев, где не требуется высокая скорость, медленнее, чем кэш в памяти или Redis
    - требует места на диске
  + 3 типа: База данных (`DatabaseCache`)
    - кэш в базе данных
    - если проект уже активно использует базу данных
    - замедляет производительность, так как кэш хранится в той же базе, что и основные данные, не подходит для высоконагруженных систем
* Redis является более мощным инструментом для кэширования
  + работает в оперативной памяти, что делает его чрезвычайно быстрым
  + поддерживает сложные операции (например, Pub/Sub), что полезно для систем в реальном времени
  + легко масштабируется и может использоваться в распределённых системах
  + данные кэша разделяются между всеми процессами и серверами, что делает его идеальным для **кластеров**
  + **сохраняет** данные на диск для предотвращения потери данных после перезапуска, чего нет в `LocMemCache`
  + Поддержка TTL (времени жизни записей)
  + Подходит не только для кэширования, но **и для других задач (сессии, Pub/Sub, очереди)**
* django cahce framework
  Django API для кэширования данных
  ```
  cache.set('my_key', 'some_value', timeout=60)
  value = cache.get('my_key')
  ```
* django REST Framework
  + кэшировать ответы API, особенно если API возвращает одинаковые данные для множества пользователей
  + можно использовать кэширование на уровне вьюх или сериализаторов
  + настроить кэширование для представлений API: если данные уже есть в кэше, они будут возвращены из Redis, если данных нет в кэше, они будут получены из базы данных и затем кэшированы
    ```
    class MyAPI(APIView):
        def get(self, request):
            data = cache.get('my_api_data')
            if not data:
                data = expensive_query_to_db()  # Долгий запрос к базе данных
                cache.set('my_api_data', data, timeout=60*15)  # Кэшируем на 15 минут
            return Response(data)
     ```
* django channels
  + redis = брокер сообщений для передачи сообщений между различными экземплярами Django
  + redis = брокер сообщений для обмена данными между клиентом и сервером
  + CHANNEL_LAYERS: позволяет Django Channels использовать Redis для обмена сообщениями через WebSocket или другие каналы
    - redis= очередь для сообщений, которые могут быть отправлены клиентам
    - reids = механизм синхронизации для разных процессов или серверов
  + redis для создания очередей задач (для обмена задачами между различными частями приложения, такими как обработка сообщений или фоновые задачи)
  + redis для кэширования промежуточных результатов в асинхронных операциях (промежуточные результаты обработки WebSocket-соединений)

### Channel Layers 
* использует Redis (или другой бэкенд) для обмена сообщениями между экземплярами Django

#### у нас redis для `CHANNEL_LAYERS` (канал-сервер) для Django Channels
* управляет обменом сообщений между **WebSocket-клиентами**
  + взаимодействие между различными **экземплярами** приложения, передача сообщений в реальном времени между пользователями
* Django Channels использует Redis для управления состоянием WebSocket-соединений и маршрутизации сообщений между различными клиентами
* Redis = брокер для обмена сообщениями между разными инстансами Django (если у вас несколько серверов или контейнеров)
* Redis = для работы с асинхронными задачами через Django Channels
* промежуточный уровень (channel layer) для обработки событий в реальном времени
* Django Channels использует Redis через установленный backend (`channels_redis`)
* для организации обмена сообщениями между различными инстансами приложения или даже между **различными процессами** (если их несколько)
  + Django Channels обрабатывает ws-соединения
  + Channel Layer для управления **сообщениями между клиентами**
  + Channel Layer для **связи между компонентами приложения**
* синхронизировать данные и события между серверами, передавая сообщения между серверами через Redis
  + если приложение работает на нескольких серверах (или контейнерах), каждый из которых может обрабатывать запросы пользователей
  + обработка сообщений чатов и уведомлений, если они должны быть доступны всем пользователям, подключенным к разным серверам: если два пользователя находятся на разных серверах, и один отправляет сообщение через WebSocket, то без Channel Layer это сообщение не будет доставлено на другой сервер
* Рассылка сообщений всем клиентам / группе клиентов
* механизм группировки пользователей (через **groups**)
  + можете легко передавать сообщения из одного WebSocket-соединения в несколько других, которые принадлежат одной группе
* для хранения состояния чата и управления уведомлениями
  + при отправке сообщения оно сначала отправлено в Channel Layer, затем пользователям через WebSocket

### 4. **Поддержка асинхронных операций**
Обработка сообщений через **Channel Layer** может происходить асинхронно, что помогает избежать блокировки основного потока приложения и позволяет эффективно обрабатывать большое количество запросов и сообщений от пользователей.

- Например, если вам нужно выполнить операцию в фоновом режиме, такую как обработка долгого запроса или отправка уведомлений, вы можете использовать **Channel Layer** для очередей сообщений и их последующей обработки.

### 5. **Распределенные задачи (например, через Redis)**
В сложных приложениях могут быть дополнительные требования для выполнения распределенных задач, таких как уведомления, обработка сообщений или других асинхронных операций. **Channel Layer** и Redis позволяют решать эти задачи в масштабируемом виде.

### Пример работы с WebSocket и **Channel Layer**:
1. Пользователь подключается к WebSocket.
2. **Django Channels** принимает запрос и создает **consumer**, который обрабатывает это соединение.
3. Сообщения от клиента передаются через **Channel Layer**, если необходимо (например, если они должны быть отправлены другим пользователям).
4. **Channel Layer** может передать сообщение другим серверам или пользователям, подключенным к одной и той же группе.
5. Ответ от сервера передается обратно через WebSocket-соединение.

### Итог:
**Channel Layer** нужен для распределения сообщений и управления группами пользователей в масштабируемых приложениях, когда запросы обрабатываются несколькими серверами. Он позволяет эффективно обмениваться сообщениями между инстансами приложения и клиентами, что особенно важно в приложениях с WebSocket- и реальным временем взаимодействием (например, чатах).

**Channel Layer** находит другие серверы благодаря использованию **брокера сообщений** (например, **Redis**), который позволяет передавать сообщения между различными инстансами приложения, даже если они работают на разных серверах. Вот как это работает:

### 1. **Подключение через брокер сообщений**
Для того чтобы **Channel Layer** мог находить другие серверы и передавать сообщения, он использует **брокер сообщений**, который в случае Django Channels чаще всего бывает **Redis**. **Redis** — это серверный компонент, который служит для хранения и передачи сообщений между различными инстансами вашего приложения.

- Когда вы используете Redis в качестве **Channel Layer**, каждый инстанс (или контейнер) приложения подключается к Redis-серверу, который работает как централизованный канал для передачи сообщений.
- Redis действует как "посредник", через который один инстанс приложения может отправить сообщение в канал, а другие инстансы, подписанные на этот канал, получат это сообщение.

### 2. **Как это работает в разрезе Django Channels**
Когда вы настраиваете **Channel Layer** с использованием Redis, вы указываете параметры подключения в конфигурации проекта Django. Например, в `settings.py`:

```python
CHANNEL_LAYERS = {
    "default": {
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [('127.0.0.1', 6379)],  # Подключение к Redis
        },
    },
}
```

Когда сервер запускается, он подключается к Redis-серверу (в данном примере — к локальному Redis-серверу по порту 6379). Все инстансы, подключенные к этому Redis, смогут обмениваться сообщениями через **Channel Layer**.

### 3. **Распределение сообщений через Redis**
Предположим, у вас есть несколько серверов (или контейнеров), каждый из которых обрабатывает WebSocket-запросы. Когда один сервер получает сообщение от клиента, он может отправить его в канал через **Channel Layer**, который подключен к Redis. Redis потом передает это сообщение всем другим инстансам, которые также подписаны на этот канал.

Вот пример, как это может быть реализовано в **consumer**:

```python
# В consumer, отправка сообщения в группу
async def send_chat_message(self, event):
    message = event["message"]

    # Отправляем сообщение в group
    await self.channel_layer.group_send(
        "chat_group",  # Название группы (канала)
        {
            "type": "chat.message",
            "message": message,
        }
    )
```

- **group_send** — это метод, который отправляет сообщение в группу, используя **Channel Layer**.
- Если несколько инстансов подписаны на `chat_group`, все они получат это сообщение.

### 4. **Как серверы "находят друг друга"**
1. Каждый сервер или контейнер подключается к одному и тому же Redis-серверу.
2. **Channel Layer** использует Redis как посредник для передачи сообщений между инстансами.
3. Когда сервер отправляет сообщение в Redis (например, через **group_send**), все другие серверы, которые подписаны на этот канал или группу, получают это сообщение через **Channel Layer**.

### Пример:
- Допустим, у вас есть два контейнера (или сервера), которые оба обрабатывают WebSocket-запросы.
- Когда пользователь на одном сервере отправляет сообщение в чат, это сообщение передается через **Channel Layer** в Redis.
- Второй сервер, который тоже подключен к Redis и подписан на ту же группу (например, `chat_group`), получит это сообщение и передаст его клиенту, подключенному через WebSocket.

Таким образом, серверы "находят друг друга" через Redis, потому что Redis выступает как централизованная точка для обмена сообщениями между всеми подключенными инстансами.

### 5. **Масштабируемость**
Если ваше приложение работает на нескольких серверах или контейнерах, использование Redis как **Channel Layer** позволяет масштабировать приложение горизонтально. Вы можете добавлять новые инстансы сервера, и они будут подключаться к тому же Redis-серверу, получая доступ к общим каналам и группам.

### Заключение
**Channel Layer** использует **Redis** (или другой брокер сообщений) для обмена сообщениями между различными инстансами вашего приложения. Каждый сервер или контейнер подключается к Redis, и все сообщения передаются через этот брокер, что позволяет распределять и синхронизировать данные между серверами в реальном времени.

Асинхронный обмен сообщениями **может работать без Channel Layers**, но с ними это гораздо проще, эффективнее и масштабируемо. Вот разбор:

---

### **Можно ли обойтись без Channel Layers?**
Да, можно. Например, вы можете напрямую использовать:
- **WebSockets без Django Channels**: Применяя библиотеки вроде `websockets` или `Django ASGI`, вы можете реализовать вебсокеты самостоятельно.
- **Прямое использование Redis или другого брокера**: Вы можете напрямую взаимодействовать с Redis (или другим брокером сообщений, например, RabbitMQ) для отправки и получения сообщений, но это потребует больше кода.
- **Долгие опросы (long polling)**: Использовать периодические запросы с клиента для получения новых данных (но это устаревший и менее эффективный метод).

---

### **Почему Channel Layers упрощают задачу**
Channel Layers предоставляют готовую инфраструктуру для асинхронного обмена сообщениями, интегрированную с Django Channels. Это снимает необходимость вручную управлять:
1. **Подключениями клиентов**: Channel Layers берут на себя маршрутизацию сообщений между пользователями или группами.
2. **Хранением состояния**: Вам не нужно вручную отслеживать, какие клиенты подключены к каким группам (например, в чате).
3. **Широковещательной отправкой сообщений**: Отправка сообщений сразу группе клиентов реализуется одной строкой кода.

---

### **Чем сложнее без Channel Layers**
1. **Ручная маршрутизация сообщений**: Без Channel Layers вам придется разрабатывать механизм для связи между клиентами (например, вручную управлять Redis Pub/Sub).
2. **Группы пользователей**: Channel Layers позволяют легко управлять группами (например, чаты, комнаты). Без них это потребует ручной работы.
3. **Масштабирование**: Channel Layers легко работают в кластере (через Redis или другой бекенд). Если вы реализуете всё самостоятельно, масштабируемость потребует дополнительных усилий.

---

### **Когда можно отказаться от Channel Layers?**
- **Простой проект**: Если вам нужно реализовать только базовый WebSocket без групп или сложного обмена сообщениями, Channel Layers могут быть избыточны.
- **Нет необходимости в реальном времени**: Если обновления происходят не часто и можно обойтись стандартным REST API с периодическими запросами от клиента.

---

### **Вывод**
Без Channel Layers **работать можно**, но это:
- Увеличивает объём кода и сложность разработки.
- Усложняет масштабирование и поддержку.
- Лишает вас преимуществ готовой интеграции с Django Channels.

Если ваш проект предполагает хотя бы минимальную сложность (например, чаты, уведомления или игры в реальном времени), использование Channel Layers — это лучший выбор.

### WebSockets
* сервер и клиент обмениваются данными в реальном времени: чата, игровой механики
* Браузер пользователя (JavaScript **WebSocket API**) ?
* создание 
  + клиент загружает страницу чата или игровой комнаты
  + клиентский код js инициирует соединение через **WebSocket API**: `const socket = new WebSocket("ws://example.com/ws/chat/room1/");`
  + daphne устанавливает WebSocket-соединение, **передавая его в ASGI-приложение**
  + `{ "type": "chat.message", "message": "Hello, World!" }` браузер -> nginx -> daphne -> механизм Channels -> Django consummer
  + `{ "type": "chat.message", "message": "Hi there!" }` от сервера к клиенту
* Django Channels
  + расширение Django
  + для обработки асинхронных протоколов (WebSocket), управления асинхронными соединениями внутри Django
  + не сервер, не принимает запросы от клиента 
  + обработка соединений внутри приложения 
  + `websocket_urlpatterns = [ re_path(r"ws/chat/(?P<room_name>\w+)/$", consumers.ChatConsumer.as_asgi()) ]` связывает URL, по которым поступают запросы, с обработчиками Consumers
    - `<room_name>` динамически извлекается из URL, тут любое слово (`\w+`) 
  + `consumers.ChatConsumer.as_asgi()` связывает запрос с Consumer-классом
    - `as_asgi()` позволяет Consumer работать в асинхронной среде 
  + Channel Layers для взаимодействия между Consumers и для передачи сообщений между **процессами**
* для работы с wss://
  + сервер использует HTTPS с действительным SSL-сертификатом
    - если сертификат отсутствует или самоподписан, браузер может блокировать подключение WebSocket

### подключить статические файлы
* CSS, JavaScript, изображения, шрифты
* CSS-OM = дереао как DOM
* bootstrap готовые стили
  - можно создавать кастомные на основе н их
* могут располагаться в разных местах
  + некоторые стили автоматически приносят сторонние **пакеты** (Django Admin, Django REST Framework)
  + некоторые — ваши файлы (bootstrap.min.css во фронтенде)
  + вместе с django из коробки идёт Django Admin и стандартные CSS-файлы для админки
    - при `python manage.py collectstatic` они копируются в staticfiles/admin/… (или admin/…)
  + Django REST Framework (DRF) имеет свои стат. файлы для **browsable API** или других админ-панелей
    - при `python manage.py collectstatic` они копируются в rest_framework/...
  + Ваши собственные стили (frontend)
    - frontend/static/css
    - например, bootstrap.min.css или ваши собственные файлы
  + `python manage.py collectstatic` Django собирает все статические файлы, включая админку и DRF, в общую директорию сборки STATIC_ROOT
    - внутри для каждого приложения (admin, rest_framework, ...) и для ваших собственных стилей, если вы настроили STATICFILES_DIRS
  + Django складывает их либо в подпапках staticfiles/…, либо хранит их отдельно, пока вы не соберёте их в одну директорию
  + разные пакеты = разные CSS, каждый пакет содержит свои статические ресурсы, не мешаясь с чужими
  + Автоматическая сборка — Django находит все static/ директории в установленных приложениях и складывает их в папку staticfiles (или то, что указано в STATIC_ROOT)
  + можете хранить файлы фронтенда отдельно и отдавать их Nginx-ом без участия Django
  + самое главное — чтобы все CSS-файлы находились/отдавались там, где ожидает браузер
    - физически файлы могут лежать в разных местах (при правильных настройках статики Django/Nginx)
  + настроить раздачу этих файлов (через Django в DEV, через Nginx или другую службу в PRODUCTION)
* `STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')`
  + директория на сервере
  + `python manage.py collectstatic` -> тут собираются все статические файлы приложений, настроенных для работы со статикой 
  + при подготовке проекта к продакшн
  + **не трекается гитом**
* INSTALLED_APPS = ...  'django.contrib.staticfiles', ... 
* `STATIC_URL = '/static/'`
  + `http://localhost:8000/static/` публичный 
  + `http://localhost:8000/static/css/style.css`
  + стат файлы доступны на сайте
  + при разработке можете напрямую ссылаться на файлы через URL
  ```
* STATICFILES_DIRS
  + Если используете нестандартную структуру (**папка статических файлов во фронтенде**), пропишите:
  BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
  + STATICFILES_DIRS = os.path.join(BASE_DIR, '../frontend/static') Путь к статическим файлам
  + STATICFILES_DIRS = родительская директория, содержит статические файлы
  + STATICFILES_DIRS = os.path.join(BASE_DIR, '..', 'frontend', 'static')  # путь до frontend/static
* {% load static %}, чтобы Django обработал ссылку {% static ... %}
  + статические файлы Django (table.css) настроены для загрузки через {% static %}
* `python manage.py collectstatic` файлы будут скопированы в backend/staticfiles
* `python manage.py findstatic css/popUpChat.css`
* `python manage.py runserver`
* сервер Django может обслуживать статические файлы: `urls.py`:
  ```
  from django.conf import settings
  from django.conf.urls.static import static
  urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
  if settings.DEBUG:
      urlpatterns += static(settings.STATIC_URL, document_root=settings.STATICFILES_DIRS[0])
  ```
* `frontend/static` доступна контейнеру:
  ```
  backend:
    volumes:
      - ./backend:/app
      - ./frontend/static:/app/frontend/static
  ```
* В разработке (DEBUG = True), статика обслуживается самим Django
  + достаточно запустить python manage.py runserver
* В **продакшене (DEBUG = False)**
  + настроить раздачу статики через collectstatic и веб-сервер Nginx
  + сделать collectstatic
  + настроить сервер для раздачи статических файлов
* DevTools → вкладка Network
  + какой путь к CSS-файлу пытается загрузить браузер
  + код ответа 200
  + код ответа 404: Django не может найти файл, конечный URL, по которому файл запрашивается = расположение файла?
* Иногда, если CSS-файл уже кэшировался, браузер может залипать на устаревшей версии
  + проверьте, обновляется ли версия CSS (можно добавить некий ?v=123 в конце ссылки или очистить кэш браузера)
* фронтенд 
  - хранятся и собираются логика, стили и ресурсы (CSS,...)
  - фронтенд — отдельное SPA
  - результат сборки (папка build или dist) может отдаёся отдельным сервером (Nginx), либо «прокидываться» через бэкенд, если вы используете Django для статики (чаще Nginx)
  - в бэкенде только делаете API (Django REST **или** Channels)
  - CSS лежат в папке фронта
  - после сборки CSS окажется в итоговых бандлах (или отдельных .css файлах)
  - Django будет их отдавать или (чаще) их будет отдавать Nginx
  - бэкенд отдает данные (API)
  - frontend/ (с package.json, src/, node_modules/ и т.п.)
  - CSS будет находиться внутри этой папки
  - а Django отдаёт или проксирует статику (или Nginx)
  - settings.py: STATIC_URL = '/static/', STATICFILES_DIRS = [ # ... ]: статика берётся из папки frontend/dist или build
  - Django подхватывает статические файлы из фронтенд
  - в Django-шаблоне либо нет упоминания о стилях, либо там просто <div id="root"></div> (для React)
  - стили приходят с фронтенда (собранный **бандл**)
  - Запустите приложение, DevTools, Network, запрос к файлу .css: http://localhost:3000/... = отдельный дев-сервер фронтенда  
* `{% load static %}` загружает статические файлы, помогает Django найти и сгенерировать путь к файлам
  + из STATICFILES_DIRS = [BASE_DIR / "static"] (у нас нету)
  + STATIC_ROOT = BASE_DIR / "staticfiles"
    - для продакшена
    - **nginx обслуживает статические файлы из этой директории**
  + в папке `app/static/` каждого зарегистрированного приложения Django
* если SPA, фронтенд отделен от бэкенда, а статические файлы обслуживаются через `/static`, то приложение **`django.contrib.staticfiles`** все равно может быть полезным
  + Django может собирать и обслуживать статические файлы (с помощью команды `collectstatic`), которые затем будут использоваться для обслуживания фронтенда
    - В продакшн-режиме, после того как файлы собраны в одну папку, Nginx может обслуживать их с помощью Django через `staticfiles`
  + в процессе разработки и сборки фронтенда можно добавлять версии или хеши в имена файлов, чтобы избежать кеширования старых версий. Django может помочь с этим через `staticfiles`.
  + если в проекте используется Django Admin, то `django.contrib.staticfiles` помогает корректно обслуживать статические файлы для admin*
* можно убрать `django.contrib.staticfiles`
  + проект не использует административную панель Django.
  + Статические файлы обслуживаются исключительно через Nginx или другой веб-сервер без использования Django для этого.
  + Нет потребности в использовании команд типа `collectstatic` для сборки статических файлов.
* для простоты настройки и администрирования часто оставляют `django.contrib.staticfiles`, даже если обслуживание статических файлов будет происходить через Nginx
* css admin
  + в `staticfiles/` после `collectstatic`  
Когда выполняется `python manage.py collectstatic`, все статические файлы собираются в **staticfiles/**. Проверь:  
```bash
ls staticfiles/admin/css/
```
или  
```bash
find staticfiles -name "*.css"
```

### **2. `static/admin/` (до `collectstatic`)**  
По умолчанию Django хранит **админские CSS** в `django.contrib.admin`, и они попадают в `static/admin/`. Проверь:  
```bash
ls $(python -c "import django; print(django.__path__[0])")/contrib/admin/static/admin/css/
```
Там должны быть файлы вроде:  
- `base.css`
- `dashboard.css`

### **3. `static/` в коде проекта**  
Если у вас кастомные стили для админки, проверьте `static/` внутри проекта:  
```bash
find . -path "*/static/admin/*.css"
```
Обычно путь такой:  
```
your_project/
    static/
        admin/
            custom.css
```

### **4. `settings.py` – где искать статические файлы?**  
Проверь, какие пути указаны в `settings.py`:  
```python
STATIC_URL = "/static/"
STATICFILES_DIRS = [
    BASE_DIR / "static",
]
STATIC_ROOT = BASE_DIR / "staticfiles"
```
Если `STATIC_ROOT` = `staticfiles/`, то все CSS будут там после `collectstatic`.  

### **Что делать, если стили не применяются?**
- Проверь, загружается ли CSS в браузере (`F12 → Network → CSS`).
- Попробуй `python manage.py collectstatic` внутри контейнера.  
- Перезапусти контейнер (`docker-compose down && docker-compose up -d`).  

Если стили админки Django отсутствуют, попробуй:  
```bash
python -m django --version
pip show django
```
Если версия **4.x**, то может потребоваться `python manage.py collectstatic` вручную.

### manage static files (e.g. images, JavaScript, CSS)
* https://docs.djangoproject.com/en/4.2/howto/static-files/
* Websites generally need to serve additional files such as images, JavaScript, CSS. In Django, we refer to these files as “static files”.
* Django provides django.contrib.staticfiles to help you manage the static files

#### Configuring static files
  + Make sure that django.contrib.staticfiles is included in your INSTALLED_APPS
  + settings file: `STATIC_URL = "static/"`
  + templates: use the static template tag to build the URL for the given relative path using the configured staticfiles STORAGES alias:
      ```
      {% load static %}
      <img src="{% static 'my_app/example.jpg' %}" alt="My image">
      ```
  + Store your static files in a folder called static in your app. For example my_app/static/my_app/example.jpg
  + You’ll also need to actually serve the static files.
    - During development, if you use django.contrib.staticfiles, this will be done automatically by runserver when DEBUG is set to True (see django.contrib.staticfiles.views.serve()). 
    - This method is inefficient and insecure, so it is unsuitable for production.
    - See How to deploy static files for proper strategies to serve static files in production environments.
* Your project will probably also have static assets that aren’t tied to a particular app
  + In addition to using a static/ directory inside your apps, you can define a list of directories (STATICFILES_DIRS) in your settings file where Django will also look for static files. For example:
  ```
  STATICFILES_DIRS = [
      BASE_DIR / "static",
      "/var/www/static/",
  ]
  ```
  + See the documentation for the STATICFILES_FINDERS setting for details on how staticfiles finds your files

#### Static file namespacing
  + Now we might be able to get away with putting our static files directly in my_app/static/ (rather than creating another my_app subdirectory), but it would actually be a bad idea. Django will use the first static file it finds whose name matches, and if you had a static file with the same name in a different application, Django would be unable to distinguish between them. We need to be able to point Django at the right one, and the best way to ensure this is by namespacing them. That is, by putting those static files inside another directory named for the application itself.
  + You can namespace static assets in STATICFILES_DIRS by specifying prefixes

#### Serving static files during development
* If you use django.contrib.staticfiles as explained above, runserver will do this automatically when DEBUG = True
* If you don’t have django.contrib.staticfiles in INSTALLED_APPS, you can manually serve static files using the django.views.static.serve() view
  + This is not suitable for production use
  + For some common deployment strategies, see How to deploy static files
  + if STATIC_URL = static/, you can do this by adding the following snippet to your urls.py:
    ```
    from django.conf import settings
    from django.conf.urls.static import static
    urlpatterns = [
        # ... the rest of your URLconf goes here ...
    ] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
    ```
  + This helper function works only in debug mode and only if the given prefix is local (e.g. static/) and not a URL (e.g. http://static.example.com/).
  + this helper function only serves the actual STATIC_ROOT folder; it doesn’t perform static files discovery like django.contrib.staticfiles
  + static files are served via a wrapper at the WSGI application layer. As a consequence, static files requests do not pass through the normal middleware chain

#### Serving files uploaded by a user during development
* During development, you can serve user-uploaded media files from MEDIA_ROOT using the django.views.static.serve() view
  + This is not suitable for production use
  + For some common deployment strategies, see How to deploy static files
  + For example, if MEDIA_URL = media/, you can do this by adding the following snippet to your ROOT_URLCONF:
    ```
    from django.conf import settings
    from django.conf.urls.static import static
    urlpatterns = [
        # ... the rest of your URLconf goes here ...
    ] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
    ```
  + This helper function works only in debug mode and only if the given prefix is local (e.g. media/) and not a URL (e.g. http://media.example.com/).

#### Testing
* When running tests that use actual HTTP requests instead of the built-in testing client (i.e. when using the built-in LiveServerTestCase) the static assets need to be served along the rest of the content so the test environment reproduces the real one as faithfully as possible, but LiveServerTestCase has only very basic static file-serving functionality: It doesn’t know about the finders feature of the staticfiles application and assumes the static content has already been collected under STATIC_ROOT.
* Because of this, staticfiles ships its own django.contrib.staticfiles.testing.StaticLiveServerTestCase, a subclass of the built-in one that has the ability to transparently serve all the assets during execution of these tests in a way very similar to what we get at development time with DEBUG = True, i.e. without having to collect them using collectstatic first.

#### Deployment
* django.contrib.staticfiles provides a convenience management command for gathering static files in a single directory
*  `STATIC_ROOT = "/var/www/example.com/static/"`where you’d like to serve these files
* `python manage.py collectstatic` copy all files from your static folders into the STATIC_ROOT
* Use a web server of your choice to serve the files
  + How to deploy static files covers some common deployment strategies for static files

### некоторые файлы
* .map (Source Map) для отладки кода в браузере
  + При минификации или транспиляции CSS/JS (например, при сборке проекта) ваш изначальный код (в данном случае CSS) превращается в более оптимизированную сжатую версию
  + Source Map хранит сопоставление (mapping) между сжатым (скомпилированным) кодом и исходным кодом
  + позволяет браузерным DevTools (Chrome, Firefox и др.) «понимать», какой изначальный CSS, SCSS или другой препроцессор соответствует конкретной строчке в минифицированном файле
  + bootstrap-grid.css — часть Bootstrap, отвечающая за сетку (Grid system)
  + соответствующий bootstrap-grid.css.map нужен, чтобы при отладке вы видели правильные номера строк и селекторы из исходных файлов Bootstrap, а не из минифицированной (или объединённой) версии
  + .map-файлы нужны при разработке или отладке, чтобы видеть исходный код
  + в продакшен версии многие отключают генерацию .map-файлов, чтобы уменьшить вес приложения и не раскрывать детали исходного кода
  + иногда их оставляют, чтобы упрощать диагностику проблем прямо на продакшен-сервере
  + если не хотите, чтобы пользователи имели доступ к отладочной информации, .map-файлы можно не заливать на сервер или закрывать их от внешнего доступа
  + в большинстве случаев там только информация о соответствии строк исходного и минифицированного кода

### запуск проекта
* For Ecole42 computers, I've updated settings of docker file in DEV branch 
  + порт, который нужен для django, занят
  + поменять номера портов в docker и в nginx
  + 6800  port for redis 
  + 4444 port frontend HTTP connections
  + 4443 port frontend for SSL connections over HTTPS
  + for local Ecole 42's network `ALLOWED_HOSTS = ['*']` in settings.py
* Секретный ключ от api раз в две недели примерно обновляется
* 'docker compose up --build' (BUILD нужно)
  - все скачается и распакуется
  - потом в терминале можно Ctrl + C (условное клавиатурное прерывание)
* `sudo ufw status` 80, 443, 8000 открыты
* `curl http://localhost:8000`
* `docker exec -it git-backend-1 ss -tuln` проверить порты бэкенда
  + `tcp LISTEN 0 511 0.0.0.0:8000 0.0.0.0:*`
* `docker exec -it git-frontend-1 curl http://git-backend-1:8000` подключиться к бэкенду из контейнера
* `docker logs git-backend-1`
* `docker logs git-frontend-1`
* через 42 на посте кластера, отличном от того, на котором он размещен
  + 1 способ: каждый раз менять URL-адрес перенаправления, чтобы он соответствовал IP-адресу хост-компьютера 
  + 2 саособ лучше:
    - каждый компьютер использует VM, в ней изменяете файл хостов, чтобы связать IP-адрес исходной станции с URL-адресами
    - подключиться к другому компьютеру в кластере через его внутреннее доменное имя школы, добавить это в nginx.conf, чтобы веб-сервер разрешал связываться с вами в этом домене, а также на стороне django

### Dockefiles
* `PYTHONPATH` переменная окружения - удалили
  + где Python ищет директории с модулями, кастомные библиотеки, пакеты при импорте
  + `PYTHONPATH` `export PYTHONPATH="/mnt/md0/42/14_ft_transendence/backend:$PYTHONPATH"`
+ PYTHONUNBUFFERED нужна
* не рекомендуется 
  ```dockerfile
  COPY requirements.txt .
  RUN pip install --no-cache-dir -r requirements.txt
  COPY . .
  ```
  + Docker кэширует каждый слой (команду). Если вы копируете только `requirements.txt` перед установкой зависимостей, `RUN pip install ...` будет повторно использовать кэш, если файл `requirements.txt` не изменился
  - Если вы измените код приложения (например, Python-файлы), установка зависимостей не будет повторяться.
  + Если вы сначала копируете весь код (`COPY . .`), то любые изменения в файлах приложения (например, в коде Python) приведут к тому, что Docker заново выполнит `RUN pip install ...`, даже если `requirements.txt` не изменялся.
  + Кэширование нарушается
   - Любые изменения в любом файле вашего приложения (даже в HTML, CSS или документации) приведут к повторной установке зависимостей.
   - Это замедлит сборку Docker-образа
  + Увеличение времени сборки: При каждой сборке будут тратиться ресурсы на переустановку всех пакетов, даже если зависимости остались неизменными
  + Ускоряет сборку благодаря кэшированию,
  + Избегает повторной установки зависимостей без необходимости
* `./context`
  + все файлы доступны для Docker в процессе сборки
  + только те файлы, которые явно указаны в Dockerfile (COPY, ADD, ...), будут скопированы в контейнер
* the volumes directive is only applied at runtime and does not affect the build stage
* If you want to avoid using COPY, you can change the pip install command to point to the mounted volume
  + this assumes that the requirements.txt file exists in the mounted /app folder at runtime
  + you need to skip installing dependencies during the build process
  + install them at runtime instead

### токены, безопасность
| **Характеристика**       | **Токен авторизации**            | **CSRF-ключ**                  | **Сессионный ключ**            |
|---------------------------|----------------------------------|--------------------------------|---------------------------------|
| **Назначение**           | Аутентификация                  |                                  | Идентификация сессии           |
| **Где хранится?**         | HTTP-заголовки, cookie, JS-хранилища | Cookie (обычно)               | Cookie (обычно)                |
| **Пример использования** | REST API, GraphQL, WebSockets   | Формы, AJAX-запросы           | Веб-приложения с авторизацией  |

| токен/ключ    | Назначение                                       | Где хранится                         | Особенности | пример |
|---------------|--------------------------------------------------|--------------------------------------|-------------|--------|
| CSRF          | от подделки запросов (CSRF-атак)                 | cookie csrftoken                     | проверяется сервером при POST/PUT запросах, передаётся в скрытом поле формы или заголовке            | Защита формы входа; обеспечение легитимности запросов от пользователя|
|ток авторизации| авторизация пользователя                         |HTTP-заг cookie locStorage sessStorage| Используется для REST API и WebSocket; может быть JWT                                          | авторизация REST API (`Authorization: Bearer <token>`) или ws-соединения |
|сессионный ключ| управление пользовательской сессией              | cookie sessionid                     | долговрем. связь пользователя с сервером; подходит для **веб-интерфейсов**; сохраняется на сервере | отслеживание состояния авторизации в веб-приложении; данные польз. (корзина)|
| WebSocket-ток | авторизация польз. при установке ws-соединения   | URL параметр / заголовок ws-запроса  | передаётся при установке соединения; проверка прав доступа                                              | авторизация чата, потоковой системы `wss://example.com/ws/chat/?token=<...>`|
| Email-токен   | подтверждение email-адреса, восстановление пароля| URL отправляемое пользователю        | одноразовый токен                     
                                                                   | подтверждение email ссылкой `https://example.com/verify-email/?token=<>`|
| API-ключи     | идентификация и авторизация сторонних приложений | HTTP заголовок запроса               | передаётся с каждым запросом; ограниченный доступ к API                                          | интеграция с внеш сервисом, предост. публ. API `Authorization: Api-Key <>`|
| щифров. ключи | шифрование сообщений                             | На сервере в хранилище               | не передаётся клиенту; для шифрования 
                                                                   | шифрование сообщений чата на стороне сервера|
| OAuth-токен   | Аутентификация и авторизация через Google        | в cookie или localStorage            | для OAuth 2.0.                        
                                                                   | авторизация Google OAuth API; доступ к данным польз. через API (email, ...)|

* **csrf MW проверяет токен, а до этого не надо его проверять?**
* можно вообще без сессий: если вся информация хранится в токенах или сессии вообще не нужны (например, чисто API-проект)
**если используется исключительно токен-авторизация (JWT), можно отключить django.contrib.sessions.middleware.SessionMiddleware и  django.middleware.csrf.CsrfViewMiddleware**
* CSRF (CROSS-SITE REQUEST FORGERY) ключ, токен
  + для защиты формы на странице входа
  + необходимы в stateful-приложениях, где используется sessionid или cookie для отслеживания пользователя
  + В stateless-приложениях с JWT-авторизацией CSRF-токен обычно не используется
  + для защиты от CSRF-атак
    - убедиться, что запрос исходит от легитимного пользователя
  + защищает формы и AJAX-запросы от подделки запросов
  + для защиты UI-приложений
  + Django по умолчанию сохраняет в cookie `csrftoken`
    - доступно только для чтения с клиентской стороны (не `HttpOnly`)
    - JavaScript может получить доступ к токену (например, для AJAX-запросов)
  + при отправке POST-запроса CSRF-токен передаётся серверу в заголовке или скрытом поле формы
  + django сравнивает токен из запроса с токеном из cookie, если совпадают, запрос легитимен
  + cookie устанавливается с параметром `Secure`, если используется HTTPS
  + Не должен быть доступен для сторонних доменов (используйте `SameSite=Lax` или `SameSite=Strict`).
  + токен **в settings.py**
  + чтобы никто не зашёл в БД
  + отдать клиенту при первом запросе
  + клиент вставлять токен в заголовок запроса к **api** и  при post/get запросах
    - **а при вебсокетах?**
  + views.py **набор проверок** добавить при запросе к api
  + https://docs.djangoproject.com/en/5.1/howto/csrf/
* ТОКЕН АВТОРИЗАЦИИ
  + для подтверждения того, что пользователь авторизован
  + идентифицировать пользователя без необходимости хранения состояния на сервере
  + содержит информацию о пользователе (например, его ID) и подписан сервером
  + для аутентификации пользователей Веб-приложения, авторизации, обмена информацией
  + для аутентификации в REST API или WebSocket
  + для API-запросов (например, токен JWT для мобильного приложения)
  + Часто применяется в RESTful API и WebSocket-соединениях
  + используется для API (например, для общения с мобильным приложением или фронтендом)
    - удобно для аутентификации в stateless-приложениях
  + в API и системах с аутентификацией без сохранения состояния (stateless authentication)
  + может храниться:
    - в HTTP заголовках (например, `Authorization: Bearer <token>`)
    - в cookie (реже, обычно для обеспечения защиты от XSS/CSRF)
    - в localStorage или sessionStorage (в клиентском JavaScript)
  + Пользователь получает токен после входа в систему
    - передаёт его с каждым запросом
    - Сервер проверяет, чтобы убедиться, что пользователь авторизован
  + если проект использует токены для авторизации, чаще всего это JWT (JSON Web Token)
    - с ним не будет необходимости в CSRF-токене
    - формат токена для передачи данных между сторонами
    - использует подпись, чтобы предотвратить изменения данных
    - все данные для проверки токена содержатся в нём, нет необходимости хранить их на сервере
    - nginx хранит jwt не в cookie, а в **локальном хранилище** (в переменной в js, обновляем как только он истечет)
    - безопасность доступа к API-ресурсам
    - Если украдён, злоумышленник может использовать его до истечения срока действия: устанавливайте короткий срок действия токена
    - отзыв токенов сложен, они самодостаточны (нет централизованного хранилища): реализуйте механизмы обновления и отзыва токенов
    - информация о пользователе
    - подписаны сервером
    - являются самодостаточными
    - имеют риски в случае компрометации
* СЕССИИОННЫЙ КЛЮЧ = ФАЙЛ СЕССИИ
  + для управления сессией после входа через веб-интерфейс
  + идентифицирует пользователя и связывает его сессии с данными на сервере
  + подходит для stateful-приложений (например, с веб-интерфейсом)
  + идентификатор для отслеживания пользовательской сессии
  + для сопоставления пользователя с данными, хранящимися на сервере (состояние авторизации, данные корзины)
  + хранится в cookie `sessionid`
    - cookie `HttpOnly`, чтобы предотвратить доступ к нему из JavaScript
    - cookie `Secure`
    - Настройте время жизни сессии (`SESSION_COOKIE_AGE`)
  + При каждом запросе браузер отправляет `sessionid` серверу
    - Django использует его для извлечения данных из хранилища сессий (база данных, кэш, файл)
* WEBSOCKET TOKEN
  + используемый для авторизации пользователя при установке WebSocket-соединения
  + при установке WebSocket-соединений клиент может передавать токен в URL или заголовках
    - `wss://example.com/ws/chat/?token=<your-token>`.
  + Проверка прав доступа и идентификация пользователя
  + могут быть основаны на JWT или другом формате, но их передача должна быть защищена (например, через HTTPS/WSS)
* EMAIL-ТОКЕН
  + для подтверждения email-адресов, восстановления пароля, ...
  + При отправке ссылок пользователю `https://example.com/verify-email/?token=<your-token>`
  + обеспечение одноразовой аутентификации для выполнения действий
* API-КЛЮЧИ
  + используемые для идентификации и авторизации сторонних приложений или интеграций
  + Для интеграции с внешними сервисами или предоставления публичного API
    - `Authorization: Api-Key <your-key>` в заголовке запроса
* ШИФРОВЛАЬНЫЕ КЛЮЧИ
  + для работы с зашифрованными данными (если шифруете сообщения чата на стороне сервера)
* OAUTH-ТОКЕНЫ
  + если проект интегрируется с внешними сервисами (Google)
* PUSH NOTIFICATION ТОКЕНЫ
  + Если приложение поддерживает уведомления, эти токены используются для отправки сообщений на устройства пользователей
* SSL-сертификат решает другие задачи, связанные с безопасностью
  + SSL-сертификаты шифруют соединение, защищая передаваемые токены (CSRF, авторизации, WebSocket)
  + токены (JWT, sessionid) передаются внутри защищённого соединения, чтобы их не могли перехватить злоумышленники
* Идентификация = Кто вы
  + Пользователь вводит логин и пароль
  + Сервер проверяет данные и создаёт **JWT**, содержащий информацию о пользователе, имя пользователя
  + Токен отправляется клиенту
  + служба аутентификации хранит закрытый ключ и может выдавать токены
  + другие микросервисы могут проверять их с помощью открытого ключа
  + Самый простой и самый безопасный — **cookie**
    - все микросервисы имеют одно и то же имя хоста => файл cookie будет автоматически отправляться браузером при каждом запросе
    - Со стороны  у вас есть расширение для JWT, но мне не удалось заставить его работать с RSA, проще создать собственное промежуточное программное обеспечение для аутентификации
  + https://dzone.com/articles/using-jwt-in-a-microservice-architecture
* Аутентификация = докажите, что это вы
  + система требует доказательства (пароль, одноразовый код (OTP) из SMS, отпечаток пальца, лицо
    - email + password или через 42
  + асимметричный алгоритм (например **RSA**)
  + https://django-rest-framework-simplejwt.readthedocs.io/en/latest/getting_started.html
  + https://openclassrooms.com/fr/courses/7192416-mise-en-place-une-api-avec-django-rest-framework/7424720-give-access-with-the-tokens
* Авторизация = что вам разрешено делать?
  + что вы можете делать, к каким ресурсам имеете доступ
  + пытаетесь открыть /admin - система проверяет, есть ли у вас права
* SSL session ID
  + низкоуровневая часть протокола SSL/TLS
  + для ускорения установления безопасного соединения
  + сохраняется на сервере
  + может быть сохранена в cookie, но это не обязательно
* sessionId cookies
  + веб-ориентированный идентификатор сессии
  + для отслеживания состояния пользователя на сервере
    - например, для аутентификации или сохранения данных между запросами
* **Stateful/Stateless**
* где backend хранит сессии?
  + настройки бекенда**
* хранятся (`settings.py` `SESSION_ENGINE`)
  + База данных в таблице django_session (по умолчанию)
    - `session_key`: Уникальный идентификатор сессии.
    - `session_data`: Зашифрованные данные сессии.
    - `expire_date`: Дата истечения срока действия сессии.
  + Файловая система (в виде файлов)
  + в кеше, настроенном в `CACHES`
  + кеш с fallback на базу данных: сначала пишутся в кеш, а при его недоступности — в базу данных
  + на стороне клиента в зашифрованных cookie

### cookie, localStorage, sessionStorage
| **Свойство**        | **Cookie**                            | **LocalStorage**      | **SessionStorage**             |
|---------------------|----------------------------------------|----------------------|-----------------------------------|
| **Хранение данных** | В браузере, отправляются серверу      | Только в браузере     | Только в браузере                    |
| **Срок действия**   | До истечения срока или закрытия браузера | Постоянное         | До закрытия вкладки/окна             |
| **Объём**           | ~4 KB                                 | 5-10 MB               | 5-10 MB                              |
| **Доступ к данным** | JavaScript и сервер (если `HttpOnly` нет) | Только JavaScript | Только JavaScript                   |
| **Область действия**| Общая для всего домена                | Общая для домена      | Изолирована для вкладки              |

* Cookie
  + текстовые данные
  + автоматически отправляются серверу при каждом HTTP-запросе на соответствующий домен.
  + использование:
    - хранение сессионных идентификаторов (`sessionid`)
    - настройки сайта (язык)
    - маркеры аутентификации
    - Используйте для передачи данных между клиентом и сервером (например, для аутентификации)
    - Подходит для сессионных данных, которые требуют взаимодействия с сервером
* localStorage
  + хранилище данных ключ-значение
  + для долговременного хранения данных
  + на устройстве пользователя, изолированно для каждого домена
  + сохраняются даже после закрытия браузера или перезагрузки устройства
  + не передаётся серверу
  + доступны только для того домена, который их создал.
  + доступны только через JavaScript
  + использование: 
    - хранение пользовательских предпочтений (тема сайта)
    - кеширование данных (JSON-ответы от API)
    - для долговременного хранения данных, доступных только клиенту (например, настройки интерфейса)
* SessionStorage**
  + Похож на localStorage
  + данные хранятся только на время текущей сессии браузера
  + изолированно для текущей вкладки или окна
  + данные удаляются, как только вкладка или окно браузера закрываются
  + доступны только через JavaScript
  + доступны только для вкладки или окна, где они были созданы
  + использование
    - хранение временных данных для одной сессии пользователя
    - статус пользователя на текущей странице (выбранные фильтры)
    - для временных данных, которые нужны только на время одной сессии пользователя

### протоколы
* **прикладные** протоколы: определяют, как данные кодируются и интерпретируются
  + HTTP (HyperText Transfer Protocol) для передачи данных между браузером и сервером
    - данные передаются в открытом виде
  + HTTPS (Secure)
    - шифрование через протокол SSL/TLS
  + Redis
  + PostgreSQL
* **транспортные**
  + TCP (Transmission Control Protocol)
    - сетевой протокол для передачи данных на порту
    - надежность соединения
    - без потерь, в нужном порядке, без дублирования
    - управление потоком: отправка данных регулируется, чтобы не перегружать получателя
    - контроль ошибок: если пакет теряется или повреждается, он будет переотправлен
    - Docker по умолчанию использует TCP, так как он подходит для большинства сетевых приложений
    - HTTP, HTTPS, подключение к PostgreSQL, redis  используют TCP
  + UDP (User Datagram Protocol)
    - менее надежен, но быстрее
  * SSL/TLS 
    + 1. шифровать соединение между браузером и nginx
      - браузер шифрует данные перед их отправкой на сервер
      - nginx отправляет ответ, зашифрованный для клиента
      - браузер расшифровывает с использованием **сессионного ключа**
    + 2. аутентификация сервера (подключаетесь к правильному серверу)
      - nginx подтвердает подлинность **сервера** (**браузер** проверяет, подписан ли сертификат доверенным центром)
    + 3. защита от **подделки данных**
    + браузер запрашивает установление защищённого **соединения** (HTTPS)
    + используется публичный ключ сервера (SSL-сертификат)
    + **сессионный ключ** согласуется в процессе установки соединения
    + nginx установливет **канал связи** SSL/TLS между сервером и браузером, чтобы шифровать трафик, использует сертификат и приватный ключ
    + nginx расшифровывает данные с использованием приватного ключа, связанного с SSL-сертификатом, использует сертификат и приватный ключ
    + `ssl_protocols TLSv1.2 TLSv1.3;` версии, TLS 1.1, SSLv3, ... не поддерживаются из соображений безопасности  
    + `ssl_ciphers 'HIGH:!aNULL:!MD5';` использовать надежные шифры, `!aNULL` и `!MD5` исключают небезопасные варианты  
    + `ssl_prefer_server_ciphers on;` браузер отдаёт предпочтение **списку шифров** сервера, а не клиента
    + генерируете `.csr` запрос на подпись сертификата (Certificate Signing Request) с приватным ключом локально, отправляете `.csr` в удостоверяющий центр (Let’s Encrypt, ...), получаете `.crt`  
    + `.key` приватный ключ для расшифровки соединения и подтверждения подлинности, хранится на **сервере**
    + `.crt` сертификат, выданный CA (Certificate Authority) или сгенерированный самостоятельно (self-signed)  
    + настраивается на стороне Nginx
    + SSL-сертификат (TLS-сертификат) для обеспечения защищённого соединения между клиентом и сервером
      - не является токенои или ключои в традиционном понимании
      - Шифрование данных, предотвращает перехват данных (паролей, сообщений) при их передаче
      - аутентификация сервера: подтверждает, что клиент общается с доверенным сервером
      - защита от MITM (Man-in-the-Middle): Исключает вмешательство третьих лиц в соединение
      - защиты HTTPS-запросов
      - шифрование WebSocket-соединений (wss://)
      - защищают транспортный уровень
      - не связаны с аутентификацией или авторизацией пользователя

### разное
* **GOOGLE_REDIRECT_URI=http://localhost:8000/auth/callback убрать**
* **42_REDIRECT_URI=http://127.0.0.1:8000/auth/callback убртаь**
* трекинг **когда пользователь был онлайн**
  + models.py: last online для индикатива
  + три решения 
    - через библиотеку channels - по вебсокетам следим пользователь онлайн или нет - НАМ ЭТОТ СПОСОБ
    - через Django sessions - как только юзер делает какое либо действие, Джанго сохраняет в базу данных дату этого действия 
    - через redis - но не понял как это работает
* **Карточка Bootstrap**  
* в модели пользователя сохраняю аватарки
  + у фронт энда есть требования какие нибудь к аватаркам или они могут сами отрисовывать?
  + вдруг пользователь сохранит свою фотографию 1000 х 1000 пикселей, на фронте вы сможете сами отрисовать аватарку ?
* с точки зрения реализации api есть **class based views** (не сильно сложнее) / functions based views (проще)
* предупредить клиента, что его ждет игра, даже если он в чате
* валидация данных
  + **js на фронте читает инпут, проверяет с помощью regex**
  + функция make password django шифрует на сервере ?
  + бэк ещё раз валидирует (проверяет пароль и почту, ...) **зачем два раза** 
* change the post requests to a websocket. The idea was to make the game and chat using websockets (native browser api), it’s beneficial in terms of continuous data streaming
* `frontend/static/app.js`
  + вызывает fetch к бэкенду и затем динамически обновляет DOM, это не делает приложение SPA-фреймворком — это обычная логика на чистом JS
  + связь между бекендом и фронтендом
  + получать информацию о пользователе (имя, статус, аватар, результаты игр)
  + динамическое обновления пользовательского интерфейса с использованием данных, полученных от бэка 
  + `fetch('http://backend:8000/api/user-data/')`: HTTP-запрос к эндпоинту бекенда `/api/user-data/`, получить данные JSON о пользователе
  + `response => response.json()` конвертирует ответ JSON от сервера в объект JavaScript
  + после получения данных пользовательский интерфейс (UI) (отображение имени пользователя, аватарка, настройки) обновляется без перезагрузки страницы
  + улушение: динамически изменять DOM:
    ```
    fetch('http://backend:8000/api/user-data/')
        .then(response => response.json())
        .then(data => {
            document.getElementById('username').innerText = data.username;
            document.getElementById('avatar').src = data.avatar_url;
        })
        .catch(error => console.error('Error:', error));
    ```
  + улучшене: `async/await` для упрощения чтения:
     ```
     async function fetchUserData() {
         try {
             const response = await fetch('http://backend:8000/api/user-data/');
             const data = await response.json();
             console.log('User Data:', data);
         } catch (error) {
             console.error('Error fetching user data:', error);
         }
     }
     fetchUserData();
* вы должны предупредить клиента о том, что его ждет игра, даже если он в чате
* single-page (SPA)
  + сайт одностраничный
  + один html файл
  + этот html меняется с помощью js
  + не надо: server side rendering подход
  + не надо: django html - server-side code
  + не надо: в завимисости от какого-то условия, показываем или нет какие-то части страницы
  + надо: js
  + надо: код внутри {} исполняется в django, если условие выполняется (например, есть юзер), он выполняет и заново отправляет html
* virtual environment не нужно, потому что у нас докер
* pass reset будет ли?
* change username, email будет ли?
* to set up the database **для чего?**
  + `python manage.py makemigrations`, `python manage.py migrate`
  + create a superuser `python manage.py createsuperuser`
* DRF - автоматичесая документация эндпоинтов
  + Browsable API (встроенная документация): в `http://localhost:8000/api/` или `http://localhost:8000/` список эндпоинтов

### test
* bakyt: Endpoint that are formed from views.py from different folders
  + `urls.py` связывает эндпоинты (URL-маршруты) с функциями/классами представлений из `views.py`
  + просматривать `views.py` в каждом приложении: какие представления и какие URL ассоциированы с функциями или классами в разных частях проекта
* если подключены библиотеки для документирования API, то `http://localhost:8000/swagger/` или `http://localhost:8000/redoc/`
* Postman: импортируйте коллекцию эндпоинтов, если она уже создана  
* Postman для изучения API, отправляя запросы на `/api/`, `/swagger/`, ... и исследуя доступные маршруты
* endpoints HTTP (API или страницы) with Postman:
  + Введите адрес вашего сервера, например:
     - `http://localhost:8000/api/endpoint/`
     - `https://example.com/api/endpoint/`
  + метод (GET, POST, PUT, DELETE и т. д.).
  + если требуется авторизация, добавьте токен или данные пользователя (если используете `Token` или `JWT`).
  + отправьте запрос и проверьте статус ответа (200 OK, 401 Unauthorized и т.д.) и тело ответа
* endpoints HTTP (API или страницы) with Curl:
  + `curl -X GET http://localhost:8000/api/endpoint/`
  + `curl -X POST http://localhost:8000/api/endpoint/ -H "Content-Type: application/json" -d '{"key": "value"}'`
* endpoints HTTP (API или страницы) with browser:
  + Для эндпоинтов, которые возвращают HTML (главная страница, панель администратора), просто откройте браузер и введите URL
* endpoints Websockets with Postman
  + меню `New Request` - `WebSocket`
  + Укажите URL WebSocket-соединения:
     - `ws://localhost:8000/ws/chat/room_name/`
     - `wss://example.com/ws/chat/room_name/`
  + Установите соединение и отправьте тестовые сообщения
  +  Посмотрите, возвращает ли сервер ответы.
* endpoints Websockets with Chrome + расширения [Smart WebSocket Client](https://chrome.google.com/webstore/detail/smart-websocket-client/kzhddgcmkfiimcdlddieeoemkbdmgkag) 
  + Укажите URL WebSocket: `ws://localhost:8000/ws/chat/room_name/`
  + Нажмите «Connect».
  + Отправьте тестовые сообщения и проверьте, получает ли сервер их.
* endpoints Websockets with Python + библиотека `websockets`
  ```python
  import asyncio
  import websockets
  async def test_websocket():
      uri = "ws://localhost:8000/ws/chat/room_name/"
      async with websockets.connect(uri) as websocket:
          await websocket.send("Hello, WebSocket!")
          response = await websocket.recv()
          print(f"Response: {response}")
  asyncio.run(test_websocket())
  ```
* Redis integration
  + `redis-cli`
  + `PING`
    - Ожидаемый ответ: `PONG`.
  + Проверьте, публикуются ли сообщения: `SUBSCRIBE my_channel`
    - отправьте тестовые сообщения в канал `my_channel` и убедитесь, что они принимаются
* HTTP-тесты с autrotests Django
  + Django предоставляет встроенные инструменты для тестирования HTTP:
    ```python
    from django.test import TestCase
    from django.urls import reverse
    class APITest(TestCase):
        def test_endpoint(self):
            url = reverse("api_endpoint_name")  # Используйте имя вашего маршрута
            response = self.client.get(url)
            self.assertEqual(response.status_code, 200)
    ```
* WebSocket-тесты с Django Channels + `pytest`:
  ```python
  from channels.testing import WebsocketCommunicator
  from myproject.asgi import application
  import pytest
  @pytest.mark.asyncio
  async def test_websocket():
      communicator = WebsocketCommunicator(application, "/ws/chat/room_name/")
      connected, _ = await communicator.connect()
      assert connected
      await communicator.send_to(text_data="Hello!")
      response = await communicator.receive_from()
      assert response == "Hello, WebSocket!"      await communicator.disconnect()
  ```
* basic functions of website
* websockets in room page
* websockets in the game
* connection
  + Connection from another computer is working (so local network is working) 
  + When Ivan tried to login with 42Auth from another computer (not server) - he got error 400; however basic sign up with email is working. 
  + My login with 42Auth from server computer worked.
* открываю 127.0.0.1:8000/chat/room1/ в двух разных местах и они получаются объдинены в одну комнату, оба видят все сообщения.
На данный момент только это надо проверять, потому что другое пока не реализовано.
* Убедитесь, что WebSocket работает: проверьте консоль браузера (F12) на наличие ошибок
* Проверьте `docker-compose logs`

### Organisation
* на всю линейку продуктов от JetBrains бесплатная студенческая лицензия, для фронта WebStorm
* the password for the django admin panel ...
* docker vscode extension 
  + launch bash inside a container from gui
* pop-up windows : login, chat, profile
* страница comptetition, profile, настройки
* Б
  + структуры данных
  + Django REST Framework
  + модуль с сокетами и чатом (live chat, уведомления о турнирах, и проч)
  + API, эндпоинты и методы для API
  + live chat
  + **rabbitMQ модуль сообщение от сервера**
  + google doc: status which modules are chosen and which modules are done
  + to finish APIS
  + live chat, ws
* Л
  + авторизация
  + фронт-энд
  + шаблон для фронт энда
  + profile
  + game page
  + auth issue
* Ан
  + фронт-энд
  + накидать в Figma шаблоны страничек (часть есть в Миро) страницы: страница с логином, с самой игрой (пока без игры), профиль пользователя, страница с турниром
  + писать код, делать шаблончики  
  + структуры страничек
  + API с бэкэнда
* Амин
  + вебсокеты для модуля remote players
    - обмен информацией между игроками и сервером о локации ракетки и мяча
  + game logic
    - whether we want to follow basic ping pong rules?
    - the ball should speed up when paddle hits the ball ?
    - https://stackoverflow.com/questions/54796089/python-ping-pong-game-speeding-up-the-ball-after-paddle-hit 
  + some sort of **anticheat** to be sure that the users mouvement are normal
    - my code will be easy, it'd just gonne output two players position and the ball and then you can render it however you want
  + I'll update you soon on the game websocket
* basic requirements 20.01.2025 
  + All pong game part will be done by Amine? Do you need help with front-end (table, paddles, ball, some activity of JS or someone will do it?
  + Tournament, registration and matchmaking system by Alexey? Do you need help? 
  + Basic front-end will be done by Alexey? (profile page, other pages) or you need help?
  + Security - probably we meet requirements by we need to validate input and follow some basic security rules on the front-end part.
* Modules (only that needs some response / comment)
  + User management - I did back-end (almost); but we need profile page with history of games, possibility to change profile data; see statistics of wins and loses - who will be responsible for this part? I can do it but I need template of javascript page (single page structure should be already applied) 
  + OAuth42 - almost finished by Alexey, please check that all requirements of this module are met/
  + Remote Players - will do Amine
  + LiveChat - is the proccess by me and Anna 
  + AI Opponent - will we do this module? Amine, can you implement it or you need help?
  + Game Customization Option - do we need this module? Who will do it? 
  + Multiple language supports - who will implement it? 
  + Server-side pong  - do we need this module? Is it implemented by Amine? For this module we need API for paddle, ball and other features
  + User and Game Stats Dashboards - do we need this module? Who will do it?
* settings.py logging.basicConfig
  + глобальные настройки логирования в Python
  + влияет на все логгеры, включая те, которые используются Django и Channels
  + Django и Channels используют свои логгеры (настроенные через LOGGING), может конфликтовать с их внутренними настройками
  + не предоставляет тонкого контроля над логгерами (раздельное управление для django и channels)
  + не рекомендуется для Django, так как может игнорировать встроенные настройки логирования
* settings.py LOGGING
  + настраивает Django и все его зависимости через встроенную систему логирования
  + можете настроить разные обработчики, уровни логирования и форматы для отдельных логгеров:
    - django — для стандартных событий Django (запросы, ответы, ошибки).
    - channels — для событий, связанных с WebSocket и канальным слоем.
  + гибкий подход, позволяет разделять логи
  + Позволяет управлять логированием для разных компонентов (например, django, channels, django.db)
  + Совместимо с встроенной системой Django, которая использует LOGGING
  + Рекомендуется для Django-проектов, особенно если вы хотите гибко управлять логами и выводить разные сообщения для компонентов
* Live pong game on website
  + users must have the ability to participate in a live Pong game against another player directly on the website
  + Both players will use the same keyboard
  + The Remote players module can enhance this functionality with remote players.
* Rules of Pong
  + All players must adhere to the same rules, which includes having identical paddle speed
  + This requirement also applies when using AI; the AI must exhibit the same speed as a regular player
* Tournament
  + A player must be able to play against another player
  + it should also be possible to propose a tournament
  + This tournament will consist of multiple players who can take turns playing against each other
  + it must display who is playing against whom and the order of the players
* A registration system
  + at the start of a tournament, each player must input their alias name
  + The aliases will be reset when a new tournament begins
  + this requirement can be modified using the Standard User Management module
* Matchmaking system for Tournament
  + the tournament system organize the matchmaking of the participants, and announce the next fight
* **DO NOT FORGET**
  + remove убрать settings.py SECRET_KEY, frontend/etc/private.key
  + close ports 800 and 6800 for outside
  + `http://backend:8000` хранить в перменной окружения
  + close http://localhost/backend:8000/chat
  + to justify your choices during the evaluation
  + настроить раздачу статики
  + remove volumes: - ./backend:/app
  + We set DJANGO_SETTINGS_MODULE in the .env, docker-compose and Dockerfile. Then, we set it again: os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings'). This appears to protect us from forgetting to set this variable in the .env, but it seems redundant in our case. May I remove os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings')?
