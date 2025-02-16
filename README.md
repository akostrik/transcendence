### see
* не может играть сам с собой


### modules
module                        | front   | back 
------------------------------|---------|------   
basic front-end               | L       | ---
live pong game on website     | L       | Amine
rules of Pong                 | Amine   | Amine     
remote players 1              | ---     | Amine  
AI Opponent 1                 | ---     | Amine  
tournament                    | L       | L       
registration                  | L       | L       
matchmaking system Tournament | L       | L      
OAuth 1                       | ---     | L
user management 1             | L       | L  
live chat 1                   | Ann     | B
security                      |         | 
django 1                      | ---     | +
bootstrap 0.5                 | +       | ---
database 0.5                  | ---     | +


### TEST
* `docker-compose logs frontend`: `172.21.0.1 "GET /static/css/chat.css HTTP/1.1" 200 3189  "https://localhost:4443/static/chat.html" "Chrome"`
  + 172.21.0.1 IP-адрес клиента в контейнерной сети, внутренний адрес Docker
  + `-` идентификация (RFC 1413), по умолчанию отсутствует
  + `-` **имя пользователя (Basic Auth)**, по умолчанию отсутствует
  + https://localhost:4443/` откуда пользователь перешёл
* endpoints
  + `views.py`: какие представления и какие URL ассоциированы с функциями или классами в разных частях проекта
  + postman
    - импортируйте коллекцию эндпоинтов, если она уже создана  
    - отправляйте запросы на `/api/`, `/swagger/`, ... исследуя доступные маршруты
    - `http://localhost:8000/api/endpoint/` endpoints HTTP (API или страницы)
    - метод (GET, POST, PUT, DELETE и т. д.)
    - если требуется авторизация, добавьте токен или данные пользователя (если используете `Token` или `JWT`)
* Django Channels + `pytest` : ws-тесты
* запрос `GET /ws/chat/rr/` обрабатывается как обычный, не как WebSocket‐handshake - проверить:
  + `frontend "GET /ws/chat/rr/ HTTP/1.1" 404 ...`: js‐код формирует `ws://` или `wss://`, но браузер блокирует/не делает Upgrade
  + консоль `wss://localhost:4443/ws/chat/rr/` OK
  + location `/ws/`: `proxy_pass http://backend:8000;`, проксирует `/ws/chat/` OK
  + nginx передаёт Upgrade‐заголовки `Upgrade: websocket` и `Connection: upgrade` OK
  + браузер делает WebSocket‐handshake
  + F12 Network WS: `101 Switching Protocols` (но `GET /ws/chat/rr/ 404` значит браузер не отправляет Upgrade или Nginx вырезает заголовок)
  + `backend Not Found: /ws/chat/rr/` django не находит `/ws/chat/rr/` в `urlpatterns` => 404
  + log Daphne `WebSocket CONNECT /ws/chat/rr/` (но 404 = значит ws‐route не сработал, Channels ничего не получил)
  + нет WebSocket‐Upgrade. Решайте с помощью исправления JS (`wss://` при HTTPS) и location `/ws/` с Upgrade-заголовками
    - Channels получит `WebSocket CONNECT`
  + проверить, что Nginx видит заголовок Upgrade
    - `curl -i -N -H "Connection: Upgrade" -H "Upgrade: websocket" -H "Host: localhost:4443" -H "Origin: https://localhost:4443" --insecure  https://localhost:4443/ws/chat/test/`
    - Если Nginx проксирует правильно, то `HTTP/1.1 101 Switching Protocols, Upgrade: websocket, Connection: Upgrade`
    - Если сразу HTTP/1.1 404: nginx не зашёл в нужный location


### LIVE CHAT
* **чат сделать компонентом**
* зачем общий чат
* the user sends direct messages to other users (subject)
* the user blocks other users, they see no more messages from the account they blocked (subject)
* the user invites other users to play through the chat interface (subject)
* the tournament warns users expected for the next game (subject)
* the user access other players profiles through the chat interface (subject)
* с каждым пользователем у бэкенда 2 вебсовета: чат, положение ракетки
* 1 эндпоинт слушает чат
  + в заголовке запроса - кому сообщение
* если сообщение не дошло, пользователя нет
* сделать базовый функционал чата
  + Обновите шаблоны HTML
  + База данных: модель для хранения сообщений
  + сохранение сообщений в `ChatConsumer` при получении данных
  + сообщения передаются через каналы
  + проверьте сохранение сообщений в базе данных
* аутентификация: проверка перед подключением к ws, в ChatConsumer self.scope['user'] для доступа к объекту пользователя
* | Method                | Real-Time | Message Persistence  | Use Case                           | Complexity |
  |-----------------------|-----------|----------------------|------------------------------------|------------|
  | WebSocket (chat)      | Yes       | No                   | Real-time chats, games             | High       |
  | Redis (Pub/Sub, ws)   | Yes       | No                   | Notifications, chats               | High       |
  | Django Messages       | No        | Yes                  | System notifications, confirmations| Low        |
  | REST API              | No        | Yes                  | Simple notifications, data requests| Low        |
  | Email                 | No        | Yes                  | Important notifications, confirmations| Medium  |
  | Push Notifications    | Yes       | Yes (by service)     | Mobile device notifications         | Medium     |
* варианты хранения сообщений чата  
  + Оптимальный вариант для большинства проектов: PostgreSQL (основное хранилище) + Redis (для кеша и ускорения)
  + База данных: надёжное долговременное хранение, полный контроль и история сообщений
  + Redis: быстрое временное хранение в памяти, можно использовать как кэш.  
    - Если чат временный (например, анонимный чат, данные стираются со временем).  
  + Файл на диске (лог-файлы, JSON, CSV): простое хранение, сложно организовать быстрый поиск, редко удобно  
  + Elasticsearch (если нужен поиск по сообщениям) Хорош для чатов с большим объёмом данных.
    - Если требуется поиск по сообщениям → Elasticsearch + база данных  
  + Kafka (или RabbitMQ) Если нужно передавать сообщения между сервисами, но не для долговременного хранения.  
 

### AVATARS
+ picture_url = models.URLField(max_length=300, blank=True, null=True) #if avatar is not available - we can use picture URL
+ avatar = models.ImageField(**upload_to='avatars/'**, blank=True, null=True) #if avatar is not available - we can use picture URL
+ в отдельном томе `media_django_volume`
  - статические файлы не изменяются после деплоя  
  - загружаемые пользователями аватарки меняются и не должны попадать в систему контроля версий
  - статика кешируется и может раздаваться напрямую через Nginx  
  - аватарки требуют авторизации, что удобнее обрабатывать через Django
  - `volumes: media_django_volume`
  - backend: volumes: media_django_volume:/app/media, nginx: volumes: media_django_volume:/usr/share/nginx/media
  - MEDIA_URL = '/media/', MEDIA_ROOT = os.path.join(BASE_DIR, 'media')
  - location /media/ { alias /usr/share/nginx/media/; }
+ локальное хранение (Media Storage)
  - для Малых и средних проектов  
  - простота настройки и отладки  
  - `settings.py`: MEDIA_URL MEDIA_ROOT 
  - **чистить старые файлы, если аватарка обновляется**
+ в базе данных (Base64, BLOB) НЕ рекомендуется
  - Долгая загрузка аватарок
  - Сложно кэшировать
+ S3-совместимое хранилище (AWS S3, DigitalOcean Spaces, MinIO)
  - Для продакшена лучший вариант
  - Подходит для Масштабируемых проектов  
  - pip install django-storages[boto3]
  - `settings.py`: DEFAULT_FILE_STORAGE, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_STORAGE_BUCKET_NAME, AWS_S3_REGION_NAME, AWS_S3_CUSTOM_DOMAIN, MEDIA_URL
  - Не нагружает сервер
  - Простая интеграция с CDN
+ внешних сервисов (Gravatar, Cloudinary)
  - Быстрого решения     
  - Минимального кода  
  - Не нагружает сервер
  - Автоматическая оптимизация изображений
  - Данные хранятся у сторонних сервисов

| Вариант | Простота | Скорость | Масштабируемость | Рекомендуется? |
|---------|---------|----------|-----------------|---------------|
| Локальное хранение | ✅ Простое | ⚡ Быстрое | ❌ Ограничено сервером | 👍 Подходит для небольших проектов |
| S3 (AWS, DigitalOcean) | ❌ Сложнее | ⚡ Быстрое | ✅ Хорошо масштабируется | 🔥 Лучший вариант для продакшена |
| База данных | ❌ Плохо | ❌ Медленно | ❌ Плохо масштабируется | 🚫 Не рекомендуется |
| Gravatar / Cloudinary | ✅ Очень просто | ⚡ Быстрое | ✅ Хорошо масштабируется | 👍 Отлично, если вас устраивает внешний сервис |


### BROWSER
* внутри браузера нет отдельного сервера, который перехватывает и обрабатывает запросы от js
* браузер сам инициирует запросы и обрабатывает их через HTTP(S) протокол, направляя их на сервер
* использует встроенные механизмы и API:
  + XMLHttpRequest или Fetch API: Эти технологии позволяют js-ом на странице браузера отправлять запросы к серверу
  + WebSocket: если приложение использует WebSocket для постоянного соединения, то браузер также инициирует это соединение с сервером, используя встроенный **WebSocket API**, который поддерживается всеми современными браузерами
* последовательность:
  + js на странице делает запрос (например, к Daphne через HTTP или WebSocket)
  + браузер инициирует этот запрос.
  + Запрос отправляется на сервер Django или Daphne
  + сервер обрабатывает запрос (маршрутизирует его в соответствующий обработчик или API, ...)
  + ответ от сервера возвращается в браузер
  + js обрабатывает ответ в браузере


### FRONTEND NGINX
* compatible with the latest stable up-to-date version of Google Chrome (subject)
* The user should be able to use the Back and Forward buttons of the browser (subject)
* try using **bolt.new** it's better at frontend
  + Bolt.new – инструмент или библиотеку для фронтенда
  + the ui is fire here  Дизайн пользовательского интерфейса выглядит очень круто
* **game customization** it's just gonna be front
  + like custom colors custom map
* проверяет сертификаты SSL, расшифровывает с использованием SSL-сертификата
  + внутренний трафик не шифруется
  + если сертификат самоподписанный, браузер может выдавать предупреждение «Not secure»
    - у вас: «Non sécurisé»
    - иногда это мешает wss://‐подключению
    - в большинстве случаев всё равно WebSocket должен подключиться
    - если браузер категорически отказывается, добавить исключение для самоподписанного сертификата
* запросы к статическим файлам: отдает из /usr/share/nginx/html/static/
* запросы на /api/ (JSON, HTML, WebSocket) идут на Django (аутентификаци, получения/отправки данных)
  + `proxy_set_header ...` передает заголовки (Host, IP-адрес клиента, протокол, ...), чтобы бэкенд понимал, откуда запрос
  + **сохраняет заголовки ws**
* фронт самописный или с использованием минимальных библиотек (jQuery, Bootstrap, ...), не на SPA-фреймворке
* балансировщик нагрузки (если много сообщений, **что будет**?)
* в модели пользователя аватарки: **у фронтэнда есть требования к аватаркам или они могут сами отрисовывать?** вдруг пользователь сохранит свою фотографию 1000 х 1000 пикселей, на фронте вы сможете отрисовать аватарку ?
* четыре server{}-блока = один процесс
* proxy_http_version 1.1: ws‐подключения для апгрейда используют HTTP/1.1, если оставить по умолчанию HTTP/1.0, Nginx не передаcn заголовки Upgrade и Connection: upgrade
* locaiton /ws/
  proxy_http_version 1.1; позволяет Upgrade c WebSocket (HTTP/1.0 не умеет)
  proxy_set_header Upgrade $http_upgrade; Передаём клиентский заголовок Upgrade: websocket дальше на backend.
  proxy_set_header Connection "upgrade"; То же для Connection: upgrade. 
  proxy_set_header Host $host; Пробрасываем заголовок Host, чтобы backend знал исходный хост (например, для ALLOWED_HOSTS или логики).
  proxy_set_header X-Real-IP $remote_addr; Передаём реальный IP клиента.
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; Цепочка IP-адресов: чтобы приложение знало, кто настоящий клиент, если есть несколько прокси.
  proxy_set_header X-Forwarded-Proto $scheme; Показывает, что изначальный протокол был https (или http).
  proxy_read_timeout 3600s; proxy_send_timeout 3600s; Увеличиваем таймауты для WebSocket: чтобы не обрывать длинные соединения.
* location /
  proxy_http_version 1.1; позволяет использовать некоторые современные фичи: chunked transfer encoding, keep‐alive соединения, ..., рекомендуют, не мешает простым запросам, может улучшить поведение прокси (поддержка chunked‐ответов от бэкенда, ...)
  proxy_set_header Host $host; Сохраняем оригинальный хост в заголовке Host
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
  proxy_set_header X-Forwarded-Proto $scheme; 
  В блоках proxy_set_header X-... и Host нужно, чтобы Django видел правильные заголовки (и знал, что мы за SSL/TLS, реальный IP и т.д.).
* Amine: game front using js
* alexey: Layout on the pages – working on it
  + расположение и структура элементов пользовательского интерфейса на веб-страницах
  + Работа с CSS-фреймворками (например, Tailwind CSS или Bootstrap)
* SSL Labs проверить корректность настройки SSL
* **Карточка Bootstrap**  
* "Failed to load module script: Expected a JavaScript module script but the server responded with a MIME type of 'text/html'"
  + 1. Файл js не найден на сервере, сервер возвращает HTML-страницу с ошибкой (404, 500, ...)
  + 2. Network → Headers, Content-Type для скрипта должно быть `Content-Type: application/javascript`
    - `nginx.conf`: `types { application/javascript js; }`
  + 3. загружаешь `.js` как **ES-модуль (`type="module"`)**, но сервер не поддерживает это
    - Проверь `type="module"` в `<script>`
    - `<script src="/static/js/main.js"></script> <!-- вместо type="module" -->`
    - убедись, что сервер поддерживает корректный MIME-тип
* CORS (Cross-Origin Resource Sharing)
  + механизм, который позволяет веб-браузерам делать запросы к ресурсам на другом домене
  + по умолчанию браузеры запрещают такие запросы из соображений безопасности


### JAVASCRIPT
* a single-page application (subject)
  + один html, меняется с помощью js, js меняет параметры html
  + код внутри {} исполняется в django, он выполняет и заново отправляет html
  + не надо: в завимисости от какого-то условия показываем или нет какие-то части страницы
* js/app.js
  + центральный скрипт, точка входа в приложение, основной код, запускается при загрузке страницы
  + управление авторизацией
  + динамически обновляет DOM пользовательского интерфейса с использованием данных, полученных от бэка
  + '.nav-link' кроме 'loginLink' 'logoutLink'
    - обработка событий интерфейса, кликов на ссылки навигации
    - инициализация маршрутизации router 
    - `e.preventDefault()` запрещает браузеру перезагружать страницу 
    - `router.navigate()` для смены страницы
  + authService.logout()
    - запрос на разлогинивание (`POST /logout/`)
    - удаляет данные профиля из `sessionStorage`
    - перенаправляет на главную страницу (`/`)
  + `fetch('http://backend:8000/api/user-data/')`: HTTP-запрос к эндпоинту `/api/user-data/`, получить данные JSON 
  + `response => response.json()` конвертирует ответ JSON от сервера в объект js
  + после получения данных пользовательский интерфейс (UI) обновляется без перезагрузки страницы
  + `async/await` для упрощения чтения
* на место app подставляется div
  + class Component базовый, абстрактный
  + фиксированная часть страницы доступна в js
  + div = homapage, profile, ... (наследуют от Component)
  + позволяет добавлять аватары, имена пользователя, ...
  + state = переменные 
  + fetch = запрос к бэку
  + функция render своя в каждом компоненте
    - например нужен username - делаем запрос к бэку fetchuserprofile
  + событие DomContactLoaded = html полностью загрузился (у нас только 1 раз)
  + % body % уберем, т.к. у нас SPA
* class Navigation extends Component
  + компонент навигации, навигационного меню (navbar), управляет переходами между страницами
  + навигация внутри `<nav>`
  + `.querySelectorAll('.nav-link')` находит все ссылки
  + `addEventListener('click', (e) => {...})` предотвращает стандартный переход (`e.preventDefault()`).  
  + пользователь кликает на ссылку в `Navigation.js` "Profile" -> "/profile"`  
    - `Navigation.js` перехватывает клик
    -  `Navigation.js` вызывает `this.router.navigate("/profile")`  
    - this.router.navigate(e.target.getAttribute('href')); для изменения страницы без перезагрузки, без запроса к серверу
    - `Router.js` берет `"/profile"`
    - `Router.js` находит в `ROUTES` `"/profile": () => new ProfilePage().render()`
    - `Router.js` загружает `ProfilePage` в `document.getElementById("app")`
  + `this.element.innerHTML` создаётся `<nav>` с ссылками на главную страницу (`/`) и профиль (`/profile`)
  + добавление новых страниц = добавить ссылки `<a href="/stats" class="nav-link">Stats</a>`
  + `Router.js`: какую страницу загрузить при переходе по URL, использует `ROUTES` 
  + `Navigation.js`: отвечает за клики по меню и ссылки, вызывает `this.router.navigate()`
  + в шапке
* open chat
  + на странице login, profile, регистрация нету
  + во время игры - статистика, другой user
  + отправлять сообщение через js
* prepMsg забирает инпут и делает ws запрос
  + список пользователей из базы
* login = new ws connexion
* если логин - присылает **session id token**
* первый логин - header CSRF токен
  + в js функция post - в header токен CSRF
  + session id приязывает сессия + юзер в приложении
* обработка ошибок в js для устойчивости чата (попытки переподключения при разрыве соединения, ...)
* pop-up windows : login, chat, profile
* F12 concole
  + лучше всего в chrome
  + colsole.log отладка
  + кнопка квадратик со стрелкой слево вверх - код элемента html
* прямо в консоли писать js и пробовать
  + undetermined - что вернула функция
  + можно создать переменные (let)
    - они сохраняются в объекте window (window = браузер)
    - x или window.x досnуп к этой переменной
  + api браузера - геолокация, звук
* у нас 1 компонент = 1 станица (хотя обычно делают более гибко)
* vscode расширение Go Live - сразу смотреть, что получается на странице
* Router.js
  + определены фиксированные пути (/ → HomePage, /login → LoginPage, ...)
  + нет маршрутов /game/1 или /user/1 => они не обрабатываются Router.js => браузер отправляет запрос на сервер, запрос к localhost:4443/game/1, минуя Router.js
    - если django отдает HTML, то загружается шаблон с бэка
    - для /user/1 сервер отвечает 403 Forbidden, поэтому страница не рендерится
* {% load static %} загрузка стат файлов Django (table.css), нам не надо
* class = стиль
* ws объект js
* сначала выполняется header, потом подгружаются стили
* fetch() запрос обычный (не ws)
* **AbsTimeUser** (непралвьно написано) наш класс наследует
* johnResponse - временный
* view.py jsonResponse или httpResponse
* createuser встроенная, т.к. наследуем от ... 


### BACKEND DAPHNE 
* ASGI сервер 
  + asgi, wsgi - интерфейсы, стандарты для взаимодействия между веб-сервером и веб-приложениями/фреймворками
    - без них фреймворки, тулкиты, библиотеки питон не взаимодействуют между собой, у каждого свой метод установки, настройки
* хорошо интегрируется с Django Channels, поддерживает ws из коробки
* не настраивается под SSL
* слушает внутри контейнера на порту 8000 wss:// соединенияь API-запросы, HTTP-запросы фронтенда
  + страница загружается по https:// => браузер открывает wss://
* переменная `application` = точка входа для ASGI-сервера, ASGI-приложение, обрабатывает запросы
* `ASGI_APPLICATION = "myproject.asgi.application"`
  + `get_asgi_application()` initialize ASGI appli, exposes the ASGI callable as a module-level variable (загрузка приложений `INSTALLED_APPS`, конфигурация бд, среда AppRegistry is populated)
    - before importing ORM models, до использования ORM-моделей и др компонентов Django, конфигурации маршрутов ws
* AllowedHostsOriginValidator проверяет допустимые хосты для ws-соединений (**зачем дублировать с nginx**)
  + проверяет заголовок Origin против ALLOWED_HOSTS
* ws-маршруты Channels работают через ASGI, а не через файлы 
* альтернатива: WSGI, Web Server Gateway Interface
  + поддерживает только синхронные запросы HTTP, не подходит для реального времени или протокола ws
  + но бывает нужен: некоторые инструменты поддерживают только HTTP-запросы; Django-приложений без асинхронных функций стабильнее
  + ASGI, Asynchronous Server Gateway Interface - продолжение WSGI, когда есть асинхронные задачи
* альтернатива: `python manage.py runserver 0.0.0.0:8000`
  + запускает Django-приложение с использованием встроенного разработческого сервера
    - может работать как с WSGI, так и с ASGI, в зависимости от конфигурации проекта
    - не рекомендуется для продакшен, daphne: более высокая производительность и стабильность


### ЯДРО DJANGO 
* бэкенд-фреймворк
* диктует архитектуру (приложения, модели, views, urls, ...)
* DJANGO_SETTINGS_MODULE настройки
* `settings.py`
  + `ALLOWED_HOSTS` список доменов/IP, с которых разрешён доступ к Django-приложению (когда `DEBUG=False`)
  + `TEMPLATES` настройки шаблонов Django (HTML-шаблоны)
  + `REST_FRAMEWORK` **рендеры**, по умолчанию JSON и **Browsable API**
  + `AUTH_PASSWORD_VALIDATORS` валидаторы сложности паролей (минимальная длина, ...)
  + logging.basicConfig настройки логирования в Python, может конфликтовать с настройками логгеров Django и Channels
  + LOGGING настроить разные обработчики, уровни логирования и форматы для логгеров django, channels, django.db
* myproject/ конфигурация Django, корневой маршрутизатор, запуск, управление проектом
* manage.py
  + django-утилита, интерфейс для настройки, разработки, управлением проектом
  + точка запуска
  + myproject.settings обращается к `backend/myproject/settings.py`
  + `if __name__ == '__main__':` если файл запущен напрямую (не импортирован как модуль), выполняется `main()`
  + python manage.py runserver запуск сервера разработки, daphne обращается к проекту напрямую или через `mysite.asgi`
  + python manage.py createsuperuser
  + остальные команды: migrate, [auth]: changepassword, [contenttypes] remove_stale_contenttypes, [django] check compilemessages createcachetable dbshell diffsettings dumpdata flush inspectdb loaddata makemessages makemigrations optimizemigration sendtestemail shell showmigrations sqlflush sqlmigrate sqlsequencereset squashmigrations startapp startproject test testserverт, [rest_framework] generateschema, [sessions] clearsessions, [staticfiles] collectstatic findstatic, можно создавать собственные команды
* вcтроенные модульные приложения
  + 'django.contrib.messages'
  + 'django.contrib.admin',
    - /admin встроеный интерфейс для управления данными (пользователи, чаты, статистика, ...)
    - суперпользователь Django Admin: все права, полный доступ к системе управления данными (!= суперпользователь бд)
  + 'django.contrib.auth',
  + 'django.contrib.contenttypes',
  + 'django.contrib.sessions',
  + 'django.contrib.staticfiles'
  * **какие ещё**
* 'django.contrib.messages' API для работы с flash‐messages
  - уведомления пользователю **между запросами** (об регистрации, ошибка при неверном вводе данных, ... в верхней части страницы
  - исчезают через несколько секунд или после того, как пользователь обновит страницу или перейдет на другую
  - **сообщение добавляется во время перенаправлений (после POST-запросов, ...) или в шаблоны**
  - **передача статуса операций между страницами**: после сохранения формы пользователю показывается сообщение на новой странице
  - не используются для функций реального времени, не асинхронны
  - работают благодаря  `django.contrib.messages` + `MessageMiddleware`
  - `MessageMiddleware` **сохраняет между запросами** и извлекает из хранилища (cookie, сессии, ...)
  - сообщения остаются там только до следующего запроса, затем удаляются автоматически
  - без `MessageMiddleware` только для сообщений **внутри одного запроса**, сообщения не сохраняются между запросами
  - через ws соединения или HTTP-запросы
  - не имеет отношения к чату (чат требуют постоянного обновления страницы или использования ws для обмена сообщениями)
* middlware
  + набор классов
  + фильтры, обработчики, модифицируюют запросы/ответы
  + **для доступа к объекту пользователя (request.user) и другим данным авторизации**
* middlware на пути от клиента
  + SecurityMiddleware
    - добавляет заголовки безопасности (`Strict-Transport-Security` для HTTPS) **зачем**
    - перенаправляет HTTP-запросы **на HTTPS**
    - убирает опасные элементы из запросов (от уязвимости **XSS**, ...)
  + SessionMiddleware
    - загружает данные сессии из **хранилища**, управляет сессиями **через куки?**
    - если пользователь аутентифицирован, сессия **сохраняет идентификатор** в Redis/бд после выполнения запроса
  + CommonMiddleware
    - перенаправление при добавлении/удалении `/` в конце URL
    - возвращает стандартные заголовки HTTP (`Content-Type`, ...)
  + CsrfViewMiddleware
    - проверяет CSRF-токен
    - защита от **Cross-Site Request Forgery**
    - use csrf_protect on any views that use the csrf_token template tag, as well as those that accept the POST data
  + AuthenticationMiddleware 'django.contrib.auth.middleware.AuthenticationMiddleware'
    - to use Django’s default user authentication system
    - проверяет сессию **или заголовки авторизации**
    - checks if the user is authenticated in each request by looking at the session data, and it attaches that information to the request object
    - handles session-based user authentication
    - ensures session-based authentication: This is critical for apps that use session-based authentication (like logging in via the web interface)
    - добавляет **объект `request.user`**, чтобы представления могли определить, аутентифицирован ли пользователь
    - attaches the user object to the request in your views and templates
    - adds the `user` object to the request: Every time a request comes in, the AuthenticationMiddleware populates the request.user attribute with the currently authenticated user based on the session data
    - a part of **Django’s authentication framework** #see
    - 'django.contrib.auth' = the app config and not middleware that manages authentication logic
    - you Can’t Just Replace It with `'django.contrib.auth'
    - the request.user is automatically set => the app knows who the current user is, and is able to rely on request.user to check for authentication or permissions
    - is involved in maintaining and validating the session-based user login (users are authenticated in Django’s session-based system)
    - альтернатива: middleware for token authentication instead of session-based authentication, replace it with middleware like rest_framework.authentication.JWTAuthentication
    - альтернатива: Session Authentication = the built-in Django authentication system but with a different mechanism (like for non-session-based login), you could use Django’s Custom Authentication Backends along with custom middleware to modify how the user is authenticated
  + MessageMiddleware
    - обрабатывает flash messages
    - **обработка сообщений** и добавление **дополнительной информации или фильтрации на уровне WebSocket**
    - без `django.contrib.messages` невозможно использовать
  + XFrameOptionsMiddleware
    - добавляет заголовок `X-Frame-Options` для защиты **clickjacking** (запрещает отображение сайта в **iframe** на других доменах, ...)
* middleware на обратном пути
  + XFrameOptionsMiddleware обрабатывает ответ (добавляет заголовок X-Frame-Options)
    - добавление заголовков безопасности
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
* Django Middleware DRF Middleware — не одно и то же
  + оба работают с HTTP-запросами
  + Django Middleware  
    - Обрабатывает ВСЕ запросы в Django (и API, и HTML-страницы)
    - Работает на уровне всего приложения перед обработкой представлений (views)
    - Аутентификация  
    - Обработка CORS  
    - Сжатие запросов  
    - MIDDLEWARE = ['django.middleware.security.SecurityMiddleware',django.contrib.sessions.middleware.SessionMiddleware','django.middleware.common.CommonMiddleware',]
    - | **Функция** | **Django Middleware** | **DRF (Django REST Framework)** |
      |------------|--------------------|------------------|
      | Где работает? | ВО ВСЕМ Django | Только в API |
      | Уровень | Обработчик запросов (request/response) | DRF Views, Permissions, Authentication |
      | Используется для | Логирования, CORS, кеширования, аутентификации | API Permissions, Serializers, Views |
      | Определяется в | `MIDDLEWARE` в `settings.py` | `REST_FRAMEWORK` в `settings.py` |
* базовый serializers.ModelForm
* базовые механизмы аутентификации django.contrib.auth (**как связано с AuthenticationMiddleware**)
* регистрация и авторизация (в том числе OAuth/SSO)
* модель пользователей, проверка паролей, сессии
* модель групп и разрешений django.contrib.auth.models.Permission
  + разграничение доступа к страницам (приватные/публичные чаты, комнаты, ...)
* проверка **токена/сессии** при каждом запросе с фронтенда
* инструменты по защите от распространённых уязвимостей (**CSRF, XSS, SQL Injection**)
* DPR Data Processing and Rendering: данные (JSON, ...) обрабатываются на сервере с помощью Django, передаются в представление для рендеринга на клиентской стороне
* используем **стандартные структуры юзера для авторизации и для моделей данных**
* middleware работает **отдельно для каждого запроса**
  + создаётся заново ???
  + обычный middleware Django обрабатывает каждый HTTP-запрос
  + **DRF Middleware** (например, аутентификация) вызывается **отдельно для каждого запроса** от каждого клиента.
  + Middleware создаётся и выполняется каждый раз, когда клиент отправляет запрос
    - Когда первый клиент делает запрос → middleware создаётся и выполняется для него
    - Когда второй клиент делает запрос → создаётся новый экземпляр middleware, который обрабатывает этот запрос
  + В DRF аутентификация реализована через **Authentication Classes**. Они вызываются **отдельно для каждого запроса**:
    - Если клиент **отправляет запрос с токеном**, `TokenAuthentication` проверит его **только для этого запроса**.
    - Другой клиент отправляет запрос — DRF снова **запускает новую проверку** через authentication classes.
* среда `AppRegistry` управляет регистрацией приложений и моделей
* **типы вьюх**
  + `View` базовый класс для создания вьюх
    - можно переопределить методы `get()`, `post()`, `put()`, ...
  + `TemplateView для рендеринга шаблонов, в основном для статических страниц, когда не нужно взаимодействовать с базой данных
  + `RedirectView` для редиректов с одного URL на другой
  + `ListView`, `DetailView` классы для работы с базой данных, предоставляющие автоматическое отображение списка объектов или одного объекта
  + `CreateView`, `UpdateView`, `DeleteView` классы для создания, обновления и удаления объектов в базе данных
  + `APIView`, `ViewSet` (для DRF)
* Django Debug Toolbar `pip install django-debug-toolbar`, добавьте `'debug_toolbar'` в `INSTALLED_APPS` и настройте `MIDDLEWARE` и `INTERNAL_IPS`
  + альтернатива: LOGGING инфо о всех запросах и их обработке в файл `debug.log`
  + альтернатива: `django.db.connection.queries` SQL-запросы, выполненные Django
* SECRET_KEY
  + ключ безопасности Django-проекта
  + Шифрование и генерации хэшей: Django использует `SECRET_KEY` при создании хэшей паролей, CSRF-токенов, cookies и других криптографических операций
  + Защита сессий и безопасности запросов: Django использует его в механизмах безопасности, например, при подписи данных сессии
  + работа встроенных механизмов безопасности: `django.contrib.auth`, `django.middleware.csrf`  
  + не храните в репозитории #see  

  
### DJANGO REST FRAMEWORK DRF
* расширяет Django, строит над Django стек для работы с RESTful API (модели, виджеты, фильтры, классы)
* не имеет отдельного слоя middleware  
  + использует другие механизмы (APIView, Permissions, Authentication) вместо middleware
  + работает через APIView, Permissions, Throttling, Authentication
  + Можно использовать Django Middleware для API, но DRF обычно этого не требует.
  + можно использовать Django Middleware для обработки API-запросов
* аутентификация, пермиссии и сериализация — отдельные модули (слои), содержащие классы
  + в каждом из этих модулей находятся классы, которые можно переопределять
  + не только для middleware
  + аутентификация и разрешения подключаются как классы (Authentication Classes, Permission Classes) на уровне вью
  + сериализация — отдельный слой
  + бизнес-логика аутентификации/разрешений/сериализации чаще всего вне middleware
  + Middleware может использоваться для сессий или базовых вещей
  + `from rest_framework.authentication import TokenAuthentication` (`TokenAuthentication` класс)
  + экземпляры создаются заново для каждого запроса, DRF делает это для изоляции данных между запросами и безопасности  
  + DRF создает новый объект аутентификации на каждый запрос
  + DRF создает новый экземпляр класса разрешений и вызывает его метод `has_permission()`
  + Serializers создаются отдельно для каждого запроса, когда данные сериализуются или десериализуются, `serializer` будет уникальным объектом для текущего запроса
* модуль аутентификации #see
  + классы для проверки пользователя (`SessionAuthentication`, `TokenAuthentication`, ...)
  + проверка сеансовой куки, сессий, csrf, JWT (с библиотекой Simple JWT), Basic Authentication, авторизация по API-ключам
  + гибкие механизмы для работы с JWT
  + 1) во многом реализована через AuthenticationMiddleware
    - Может частично использовать стандартное Django middleware (`AuthenticationMiddleware`), когда речь о сессиях
    - привязать пользователя (request.user) до попадания запроса во вью
    - если JWT или токены, то не чистое middleware, а authentication classes
  + 2) на уровне Views / Viewsets  
    - определяете `authentication_classes = ` (`SessionAuthentication`, `JWTAuthentication` из внешних пакетов)  
    - эти классы вызываются при каждом запросе
    - технически не являются классическим Django middleware  
    - проверяют заголовки/токены/сессии
    - устанавливают `request.user`
    - тут логика
    - если вы говорите о токенах, JWT, Basic/Auth, обычно это **DRF Authentication Classes** на уровне вью.  
  + Считывает токен (Bearer, JWT), куку сессии или другие данные из запроса
  + Определяет, какой пользователь (или аноним) делает запрос, и выставляет `request.user`/`request.auth`
  + работает не напрямую с JSON, а с объектом запроса (request), где смотрят заголовки, куки, сессии, токены, ...
* Permissions
  + проверяют, имеет ли этот `request.user` право на доступ к данному ресурсу (вью), методу (GET, POST…), конкретным операциям (создание, редактирование, ...)   
  + если нет — возвращается `403 Forbidden`
  + работает не напрямую с JSON, а с объектом запроса (request), где смотрят заголовки, куки, сессии, токены, ...
* модуль сериализации данных для работы с JSON или API #see
  + django не имеет middleware для парсинга JSON
  + приходящий request.body в JSON парсится `JSONParser`, а ответ сериализуется `JSONRenderer`, подключены внутри DRF
  + применяется уже во вью для входных данных (`POST` JSON, ..) и выходных (формирование ответа).  
  + DRF вызывает `serializer.is_valid()` для проверки пришедших данных, затем, если всё верно, вычитанные поля доступны в `serializer.validated_data`.  
  + при возврате ответа вью может использовать сериализатор, чтобы преобразовать Python-объекты (модель, словарь) в JSON
  + классы для преобразования данных (например, `ModelSerializer`)
  + преобразование моделей Django и других объектов Python в JSON/XML и обратно
  + валидация данных
  + DRF Serializers (`rest_framework.serializers.Serializer` или `ModelSerializer`)  
    - отдельный слой, не связанный с middleware
    - преобразование (включая валидацию) Python-объектов (моделей, словарей) в JSON/др формат  
  + `JSONParser`, `JSONRenderer`
  + реализуется внутри вью/serializer классов DRF, встроена в вьюшки через классы сериализаторов (не middleware)  
* модуль прав доступа Permissions #see
  + классы для проверки прав доступа (например, `IsAuthenticated`, `IsAdminUser`)
  + контролировать доступ к API (`AllowAny`, `IsAuthenticated`, `IsAdminUser`, `IsAuthenticatedOrReadOnly`, кастомные)
  + проверка permissions обычно не как middleware
  + Permission classes
    - не middleware
    - `permissions.IsAuthenticated`, `permissions.IsAdminUser`, `permissions.DjangoModelPermissions`, кастомные классы
    - указываются либо глобально в `settings.py` (`REST_FRAMEWORK['DEFAULT_PERMISSION_CLASSES']`), либо на уровне вью, 
    - во вью
  + Перехват на уровне view  
    - DRF автоматически вызывает методы `has_permission()` и `has_object_permission()` для permission classes  
* ниже вспомогательные компоненты, работают в тандеме с тремя модулями
* APIView/ViewSet вспомогательные компонены
  + View из обычного Django не применяется для создания API
  + View не используется в DRF напрямую для обработки API-запросов, поскольку DRF специально предоставляет улучшенную функциональность с APIView и ViewSet для работы с RESTful API
  + в связке с сериализаторами для обработки данных запросов и формирования ответов
* APIView 
  + базовый класс представлений (views)
  + предоставляет функциональность для обработки запросов (например, `GET`, `POST`, `PUT`, ...)
  + класс для создания API, инструмент для создания API
  + позволит управлять данными через HTTP-запросы
  + можешь явно определять методы обработки HTTP-запросов, такие как get(), post(), put(), ...
  + встроенная поддержка систем аутентификации и контроля прав доступа (например, с использованием `IsAuthenticated`)
  + создавать собственные представления для обработки запросов
  + методы (`get`, `post`, `put`, `delete`, и т.д.), их можно переопределять для обработки HTTP-запросов
  + ограничения на количество запросов
  + вручную определять маршруты и методы для каждого запроса **против этого ViewSet или router?**
  + больше контроля, чем ViewSet, определяете логику запросов и маршрутизацию
  + подходит, если проект небольшой или требует точной настройки
  + class UserProfileView(APIView)
    - другие сервисы или фронтенд получают информацию о пользователе через API
    - `GET /user-profile/{id}/` вернет данные профиля в формате JSON
    - `get()` получать данные профиля пользователя по его `id`
    - API позволяет _дистанционно_ взаимодействовать с данными через HTTP-запросы, в отличие от Django Admin
    - Django Admin, интерфейс администрирования, не предназначен для работы с API, можно просматривать/редактировать данные профиля
  + class GameView(APIView)
  + class TournamentView(APIView)
* ViewSets вспомогательный компонент
  + обертка над APIView
  + более абстрактная форма
  + упрощает работу с CRUD, объединяют логику CRUD-операций в одном классе
  + автоматизирует маршрутизацию и типичные CRUD-операции с помощью Routers
  + генерирует действия для стандартных операций CRUD, таких как `list`, `create`, `retrieve`, `update`, `destroy`
  + полезен для упрощения кода и автоматизации создания стандартных операций CRUD
* routers вспомогательный компонент 
  + создаёт URL-маршруты (эндпоинты) для действий представления (viewset)
  + маршрутизация RESTful API
  + наследуются от базового класса **BaseRouter**
  + класс **`DefaultRouter`** (**`SimpleRouter`**) DRF: генерация путей (URL-паттернов) к методам ViewSet (`list`, `retrieve`, `create`, `update`, `destroy`)  
  + DefaultRouter один из предопределённых роутеров
    - расширяет функциональность SimpleRouter
    - поддержка автоматической маршрутизации API root (/) и обработку схемы API
    - для стандартных действий представления (viewset)
  + упрощает объявление маршрутов для ваших ViewSet’ов (CRUD-операций), улучшает читаемость кода, делает его единообразным
  + не нужно вручную добавлять каждый HTTP-метод в `urls.py`, писать `urlpatterns` для каждого метода (GET, POST, PATCH, DELETE, ...)
  + регистрируете ViewSet в `router.register()`
  + DRF автоматически формирует для него набор URL-маршрутов, настроит URL-адреса для всех действий из ViewSet
    - APIView для более кастомизированных вьюх
    - ViewSet для стандартных API-методов
  + лучше без DRF Router, если:
    - очень специфичные URLs или вы используете в основном обычные Django CBV/FBV (Class-Based Views/Function-Based Views) без API на базе DRF
    - всего несколько отдельных эндпоинтов и нет смысла создавать полноценные ViewSet’ы
    - REST Framework подключён точечно и используется лишь в небольшом модуле
  + у нас без router:
    - path(), re_path() указывает на APIView или ViewSet
    - django ищет, какая вьюха должна обработать запрос, основываясь на urls.py
    - django направляет запрос в `ItemListView`, запрос на `/users/1/` направляется в `ItemDetailView`
    - `path()`, `re_path()` связывают URL-адреса с определенными вьюхами
  + у нас дублирование `auth_app.urls`
  + у нас `re_path(r'^.*', TemplateView.as_view(template_name='index.html'))` перехватывает URL,не обработанные выше => несуществующие пути перенаправляются на `index.html`, что **неправильно для API**
    - надо ограничить этот `re_path` только на фронтенд-маршруты (например, `^(?!api/).*`)
    - либо убрать его из `urls.py`, а вместо этого обрабатывать фронтенд-роутинг на стороне фронта
  + у нас дубль path('', include('myapp.urls')), и path('', include('auth_app.urls')),
    - jango использует только первый, который он встретит
    - обычно в `''` подключается основной `myapp.urls`, а внутри `myapp.urls` уже могут быть другие вложенные маршруты
  + у нас API-роуты (`/users/`, `/user_42/`, `/game/<id>/` и т.д.) раскиданы по `urls.py` 
    - Группировать все API в `/api/` и перенести в `api/urls.py`
    - ```python
      urlpatterns = [
          path("api/users/", UserProfile_list, name="get_all_userprofiles"),
          path("api/user/<int:id>/", UserProfileView.as_view(), name="user-detail"),
          path("api/tournament/<int:id>/", TournamentView.as_view(), name="tournament-detail"),
          path("api/game/<int:id>/", GameView.as_view(), name="game-detail"),
      ]
      ```
    - `myproject/urls.py`: `path("api/", include("api.urls")),`
  + gpt предлагает
    ```
    backend/
    │── myproject/
    │   ├── urls.py         # Основные роуты (только include)
    │── auth_app/
    │   ├── urls.py         # Авторизация
    │── chat/
    │   ├── urls.py         # Чат
    │── myapp/
    │   ├── urls.py         # Главная страница (index)
    │── api/
    │   ├── urls.py         # Все API-роуты
    ```
  + gpt предлагает `backend/myproject/urls.py`
    ```python
    urlpatterns = [
        path("admin/", admin.site.urls),
        path("api/", include("api.urls")),  
        path("auth/", include("auth_app.urls")),  
        path("chat/", include("chat.urls")),
        path("", include("myapp.urls")),  # Главная страница (index)
        re_path(r'^(?!api/).*$', TemplateView.as_view(template_name="index.html")),
    ]
    ```
  + gpt предлагает `backend/api/urls.py`
    ```python
    urlpatterns = [
        path("users_42/", get_all_users_view, name="get_all_users_42"),
        path("user_42/<int:user_id>/", get_user_view, name="get_user_42"),
        path("users/", UserProfile_list, name="get_all_userprofiles"),
        path("user/<int:id>/", UserProfileView.as_view(), name="user-detail"),
        path("tournament/<int:id>/", TournamentView.as_view(), name="tournament-detail"),
        path("game/<int:id>/", GameView.as_view(), name="game-detail"),
    ]
    ```
  + альтернатива: DFR может **автоматически создавать маршруты** через `DefaultRouter`
    - router = DefaultRouter() # автоматически создаёт маршруты CRUD (`GET /user/1/`, `POST /user/`, `PUT /user/1/`, ...)
    - router.register(r'user', UserProfileView, basename="user")
    - urlpatterns += router.urls  # Добавляет автоматически сгенерированные URL
  + альтернатива: метод `get_absolute_url()` в модели `UserProfile` может **автоматически создавать URL для объекта**
  + альтернатива: декоратор `@action` может создаать кастомные URL
* Filters вспомогательный компонент
  + фильтрация данных, которые возвращает API, по параметрам запроса
* Throttles вспомогательный компонент
  + контролируют частоту запросов пользователей
  + защита от **DoS-атак** и чрезмерной нагрузки на сервер (**nginx недостаточно?**)
+ базовые подходы django
  + FBV function based views
    - представления, основанные на функциях
    - ручная обработкя запросов (`GET`, `POST`, ...)
    - проще
    - def my_view(request): return JsonResponse({"message": "Hello, world!"})
  + class based views CBV
    - представления как классы, ООП-подход 
    - встроенная логика обработки запросов (`get()`, `post()`, ...)
    - переопределять только нужные методы (`get`, `post`), а также использовать миксины
    - def get(self, request): return JsonResponse({"message": "Hello, world!"})
* 'DEFAULT_AUTHENTICATION_CLASSES': ['rest_framework.authentication.SessionAuthentication']
  + без этой настройки              ['rest_framework.authentication.SessionAuthentication', 'rest_framework.authentication.BasicAuthentication']
* 'DEFAULT_PERMISSION_CLASSES':     ['rest_framework.permissions.IsAuthenticated'] 
  + без этой настройки              ['rest_framework.permissions.AllowAny']
* с 'DEFAULT_RENDERER_CLASSES':     ['rest_framework.renderers.JSONRenderer', 'rest_framework.renderers.BrowsableAPIRenderer']
  + без этой настройки то же самое
* каждый запрос обрабатывается отдельно
* клиент получает ответ сразу
* рендеринг шаблонов (Server-Side Rendering) у нас нет


### DJANGO RESTFUL HTTP API 
* API Application Programming Interface = интерфейс = набор правил = WebSocket-путь = группа маршрутов (эндпоинтов)
* обрабатывает HTTP-запросы (GET, POST, PUT, DELETE) и отклики между клиентом и сервером
* REST Representational State Transfer стиль взаимодействия между клиентом и сервером
  + Request Body (json, xml, yaml, MessagePac, CSV)
  + Query Parameters `GET /users?name=Иван`
  + Path Parameters `GET /users/1/`
  + Headers `Content-Type: application/json`  
* API = 5–10 эндпоинтов (группа маршрутов, связанных с чатом)
* endpoint = URL-маршрут = конкретный маршрут, связанный с API
  + выполняет определённое действие
  + endpoints чата `GET /chat/rooms/`, `POST /chat/rooms/`, `GET /chat/rooms/<room_id>/messages/`, `POST /chat/rooms/<room_id>/messages/`
* APIView ViewSet специализированные CBV из DRF
  + обработчики HTTP-запросов для работы с API
  + определяются в View-классах или функциях
  + классы, наследующие от `APIView`, `GenericViewSet`, `ViewSet`
  + APIView (у нас)
    - класс, наследуется от класса View
    - CBV предназначенный для создания REST API
    - добавляет сериализацию, аутентификацию, работу с JSON, вспом. методы для обработки HTTP `GET` `POST` `PUT` `DELETE`, др API-функций
    - для простых API с базовой логикой
    - `views.py`: метод `game_detail` обрабатывает запрос
  + ViewSet
    - расширенный вариант APIView
    - использует методы по умолчанию, предоставляемые DRF (избежать написания методов для HTTP-запросов `GET`, `POST`, `PUT`, `DELETE`)
    - добавляет автом. обработку запросов для стандартных операций CRUD (Create, Read, Update, Delete) над данными, которые уже есть в базе
    - работает в связке с Router, что упрощает маршрутизацию API
    - генерирует маршруты для операций с ресурсами (создание нового объекта, получение списка объектов, обновление, удаление)
    - предоставляет стандартные действия для модели
    - подходит для ресурсо-ориентированных API, где операции с объектами ("пользователи", "продукты") 
  + | Вариант     | Для чего?                          | Маршрутизация |
    |-------------|------------------------------------|----------------|
    | **FBV**     | Обычные представления              | Ручная (`path()` или `re_path()`) |
    | **CBV**     | ООП-подход к представлениям        | Ручная (`as_view()`) |
    | **APIView** | API-представления                  | Ручная (`as_view()`) |
    | **ViewSet** | API + автоматическая маршрутизация | Через `Router` |
* позволяет **приложениям** взаимодействовать друг с другом
* сессии не нужны, авторизация через токены
* строит и отдаёт html
* реализуются
  + через **события**
  + через **статусы в ответах API**
  + не через механизм сообщений Django
* View (или APIView, если это Django REST Framework)
  + класс‐вью создаётся (инстанцируется) на каждый входящий запрос
  + при каждом запросе к этому урлу Django создаёт новый объект класса, вызывает соответствующие методы (get, post, ...) и затем уничтожает объект
  + хранить в себе контекст обработки (запрос, параметры, данные)
  + в объекте View можно было безопасно хранить данные, относящиеся к конкретному запросу, не опасаясь утечек между разными запросами
* один запрос — одна вьюшка
  + Маршрутизатор URLconf сопоставляет URL‑адрес вьюшке
  + исключения:  
    - middleware или декораторы вокруг вью, которые дополнительно модифицируют запрос/ответ
    - ручной вызов «другой» вью внутри кода (специфичный сценарий)
* AUTH_PASSWORD_VALIDATORS при регистрации нового пользователя или при смене пароля через форму
  + или в Django Admin (если используется для изменения пароля).
  + или в Custom views или views, работающие с DRF, если используется функциональность для смены пароля через API.
  + пользователь регистрируется/ли меняет пароль через Django Forms или DRF serializers => перед сохранением пароля Django проверяет его с помощью валидаторов
  + валидация происходит в момент вызова метода **set_password()**
* bakyt: API for Database - UserProfile works
* DEFAULT_AUTO_FIELD #see


### DJANGO CHANNELS
* расширение, framework, библиотека, надстройка для ws поверх стандартного стека django: привычные инструменты, структуры + асинхронное взаимодействие, **фоновые задачи**, обновление интерфейса в режиме реального времени
* слушает ws-запросы, обслуживание ws
  + непрерывное соединение, стрим
  + js инициирует соединение через **WebSocket API**: `new WebSocket`
  + daphne устанавливает ws-соединение
  + daphne передаёт ws-соединение в ASGI-приложение
  + создает consumer
  + связывает URL с обработчиками `consumers.ChatConsumer.as_asgi()`, URLRouter маршрутизирует запросы на consumers
  + **если сертификат отсутствует, браузер может блокировать подключение ws**
  + если страница загружена по https://, то используйте wss:// 
  + AuthenticationMiddlewareStack
    - соединение аутентифицировано (токены JWT, Cookies, ...)
    - связывает пользователя с запросом, с подключением
    - **оборачивает** маршруты ws
    - доб. объект user в scope["user"] => consumer self.scope["user"] user инфо, действия на основе его прав, ролей
  + SessionMiddleware
    - получение и сохранение id сессии и связанного с ним пользователя
    - обязателен, если используете сессии (хранение пользовательские данных между запросами)
    - обязателен, если есть `django.contrib.auth` (он опирается на сессии для хранения данных о пользователе)
    - обязателен, если взаимодействие с сессиями на сервере (сессионное хранилище в бд, Redis, файловой системе,...)
    - можно без него, если JWT токены или др безсессионные методы аутентификации 
    - можно без него, если данные о сессии хранятся в зашифрованных cookie (без серверного хранилища)
  + свои middleware: ограничение скорости, управление правами доступа, обработка ошибок
  + сессии передаются через cookie
* ws‑запрос (подключение) привязывается к одному consumer
  + Маршрутизация (routing): какой consumer обрабатывает точку подключения (по URL, пути, протоколу, ...)  
* назначение пользователей к группам каналов
* ws
  + как работает утсновка соединения:
    - js инициирует подключение к `wss://localhost:4443/ws/chat/room/`
    - F12 в списке сетевых запросов элемент, отражающий WebSocket (101 Switching Protocols), "Status" 101, network pending или active
    - nginx видит запрос с заголовками Upgrade: websocket
    - nginx Найдёт location /ws/ { ... }
    - nginx проксирует запрос на backend
    - daphne отвечает HTTP/1.1 101 Switching Protocols
    - при установке ws-соединения запрос пришёл с `Upgrade: websocket`
    - nginx вернёт 101
    - соединение переходит в ws‐режим
  + как работает обмен сообщениями:
    - `wss://<domain>/ws/...` -> `http://backend:8000/ws/...` - daphne принимает ws по пути `/ws/chat/<room>/` 
  * wss://localhost:4443/ws/chat/
  * ws://localhost:4443/ws/chat/room/ ws-запрос на /ws/chat/
  * wss://localhost:4443/ws/chat/room/
  + `F12` Network - ws - messages: входящие/исходящие сообщения, в Timing (Headers) статус 101 Switching Protocols
  + views.py обрабатвает HTTP Django URLs /chat/<room> 
  + channels обрабатывает /ws/chat/<room>
  + маршруты Channels: websocket_urlpatterns = [re_path(r'^ws/chat/(?P<room_name>\w+)/$', ChatConsumer.as_asgi()),]


### CHANNEL LAYERS
* абстракция, интерфейс/протокол, настройка, промежуточный уровень, канал-сервер, канальный слой для Django Channels
* публикует в канал, доставляет подписанным
* передача сообщений между частями приложения
  + ```
    [Consumers]
        ├──► [Django Models] ↔ [База данных]
        ├──► [Channel Layers (Redis)]
        │       └──► [Процессы]
        ├──► [Асинхронные задачи (Celery)]
        │       └──► [Фоновые Рабочие Процессы]
        └──► [Аутентификация и Авторизация]
      ```
  + инстансами приложения
  + клиент ↔ consumers: установление соединения
  + клиент ↔ consumers: обмен сообщениями
  + consumers ↔ channel layers: consumers присоединяются к группам (комнатам, игровым сессиям) через Channel Layers
  + consumers ↔ channel layers: отправка сообщений в группу
  + Consumers ↔ Django Models и База данных: чтение/Запись данных
  + Consumers ↔ Django Models и База данных: Транзакции: согласованности данных при одновременных изменениях
  + Consumers ↔ Асинхронные задачи (Celery): Запуск задач: При определённых событиях (окончание игровой сессии, ...) Consumers инициируют выполнение фоновых задач
  + Consumers ↔ Асинхронные задачи (Celery): Обработка результатов: Фоновые задачи могут обновлять данные или отправлять уведомления после выполнения
  + Consumers ↔ Аутентификация: проверяют аутентификацию пользователя через scope["user"]
  + Consumers ↔ Аутентификация: ограничение доступа к определённым действиям или комнатам на основе прав пользователя
  + Backend ↔ Внешние API и Сервисы (обработка платежей, отправка уведомлений)
  + Backend ↔ Внешние API и Сервисы: Маршрутизация: Обработка запросов от внешних источников и передача данных обратно клиентам через Consumers
  + Клиент ↔ REST API: клиент может использовать традиционные HTTP-запросы для получения или отправки данных
  + Клиент ↔ REST API: Синхронизация данных: Обеспечение **согласованности состояния** между частями приложения
  + Consumers ↔ Другие Consumers: Consumer может взаимодействовать с другим для координации действий или обмена данными
* **очереди сообщений**
  + операции в фоновом режиме (обработка запроса, отправка уведомлений)
  + избежать блокировки основного потока
* управляет **хранением состояния**
* масштабирование, легко работают в кластере
* альтернатива: асинхронный обмен сообщениями без Channel Layers
  + обрабатывать сообщения напрямую внутри процесса, используя функциональность Django Channels и ws-соединений
  + между клиентами в рамках одного процесса
  + для рассылки сообщений можно использовать **группы, которые предоставляет Django Channels**
  + отсутствие долговременного хранения сообщений (если не сохраняете их в базу данных)
  + channel layers проще, масштабируемо
* альтернатива: без внешнего channel layer
  + по умолчанию InMemoryChannelLayer
  + отправлять и получать сообщения в рамках одного процесса (одного экземпляра приложения)  
  + group_send или мульти‐user‐чат‐шаблон только в рамках **одного процесса** #question
  + реал-тайм обновления для одного пользователя (или небольшой группы в рамках одного процесса). Например, вы хотите уведомлять конкретного юзера о событиях (новые сообщения, новые заявки в друзья и т.п.), и ваш сервер – один.  
  + Мини-чат или прямое общение (point-to-point). Если все пользователи сидят в рамках одного сервера (и у вас не требуется распределённая горизонтальная архитектура), то и группы Channels будут работать внутри этого одного процесса 
  + Push-уведомления из кода в конкретный WebSocket‐консьюмер
* **ws один для всех пользователей для чата (лёша)** #question
* channels_redis поддержка групповой рассылки


### REDIS
* Remote Dictionary Server  
* **хранение данных**
* **маршрутизация данных**
* **подсчёт статистики, временных данных**
* Channel Layers и кэш — два независимых сервиса логически
  + могут использовать один и тот же Redis-сервер с разными бд или настройками, c разными ключами и настройками
  + 1. для проектов с ~100 пользователей и без высоких требований к масштабированию достаточно 1 Redis-инстанса и 1 базы, separating Redis instances can be overkill. The arguments (isolation, scalability, etc.) are still valid in theory, but one Redis instance is usually enough at this size. If you expect significant growth or want a clean separation for reliability and maintenance, you can still split them. 
  + 2. один Redis с двумя бд
    - для небольших проектов, если нет больших требований по нагрузке и безопасности
    - Redis поддерживает 16 баз данных (`db=0`, `db=1` ...)
    - память, процессы ввода-вывода, др. ресурсы общие, если одна база нагрузит Redis, другая пострадает
    - базы работают под общими параметрами (политика сброса ключей по LRU, максимальный объём памяти, ...)
    - ограниченные возможности мониторинга, метрики будут по одному инстансу в целом (использование памяти, CPU, ...)
  + 3. два Redis-инстанса или даже два сервера
    - Isolation: Flushing the cache won’t wipe out channel data, flush-операции, производительность
    - Scalability: Each instance can be tuned or scaled independently
    - Performance: Heavy traffic on one won’t harm the other
    - Security: Compartmentalizes sensitive channel data from the cache
    - Monitoring: Easier to track and troubleshoot each component’s load
* настройка
  + время жизни кэша (`TIMEOUT`), ...
  + `redis-cli info memory` **не перегружен?**
  + `redis-cli -n 1 flushdb` очистить кэш 
  + `FLUSHALL` (очищает весь инстанс)
  + `FLUSHDB` (очищает одну базу)
  +  **защищён паролем** #see
* `vm.overcommit_memory = 1` #see
  + параметр ядра (kernel)
  + как система распределяет виртуальную память
  + сохранять данные в условиях низкой памяти
  + избежать ошибки `OOM` при больших `maxmemory`
  + в `/etc/sysctl.conf` сохранить после перезагрузки
  + sysctls: vm.overcommit_memory: "1" в docker-compose.yml
    - Dockerfile: RUN echo "vm.overcommit_memory=1" >> /etc/sysctl.conf
    - если контейнер не в privileged-режиме, Docker не даёт ему доступ к глобальному kernel-параметру, кроме безопасных сетевых
    - запускать контейнер с `--privileged`
  + `cat /proc/sys/net/core/somaxconn` 4096 #see
    - макс длина очереди соединений (backlog) для ws **в режиме прослушивания**, сколько входящих соединений могут быть поставлены в очередь
    - внутри контейнера listen(fd, 10000) - очередь ограничена 4096 (если не переопределяется др. настройками)
    - значение берётся у хостового ядра, ведь контейнеры используют общее ядро с хостом
* очередь задач (отправки email, push-уведомления, логирование, обработка фоновых задач (обновление статистики матчей))
* для создания очередей задач (для обмена задачами между различными частями приложения, такими как обработка сообщений или фоновые задачи)
* как устроен технически
  + работает в оперативной памяти => высокая скорость
  + может сохранять данные на **диск**, чтобы избежать потери при сбоях #see
  + `channels_redis` библиотека для подключения и отправки сообщений
  + `django-redis` библиотека для сериализации, записи, извлечения
* Если используется Django Channels**, Redis выполняет сразу две роли
  + Backend channel layers (для WebSocket-обмена сообщениями)  
  + Хранилище сессий пользователей


### REDIS = BACKEND ДЛЯ CHANNEL LAYERS = реализация Channel Layer
* бэкэнд, брокер сообщений, посредник
  + Redis = брокер сообщений в режиме Pub/Sub
    - Подключение к Redis (`redis.createClient()` или аналог)
    - Вызовы методов `publish()` и `subscribe()`
* для передачи сообщений между потребителями ws
* интерфейс для Channels (методы `send()`, `receive()`, `group_add()`, ...)
* конкретный класс, реализует протокол Channel Layer
* для работы с асинхронными задачами через Django Channels (**какие ещё задачи**)
* распределять нагрузку **между несколькими контейнерами** (Горизонтальное Масштабирование)
+ **устойчивость к сбоям**
* долговременное хранение сообщений или очередей вне памяти приложения
* обработка большого количества одновременных WebSocket соединений и сообщений в реальном времени
* механизм Pub/Sub (Publish/Subscribe) для чата
  + Pub/Sub => идеальный для каналов WebSocket
  + рассылать сообщения всем пользователям чата
  + бекенд для Pub/Sub
  + механизм списков или Pub/Sub для реализации очередей сообщений
  + Поиск в коде `PubSub`, `publish`, `subscribe`, `event`, `broker`, `topic`
    - библиотеки  `Redis`, `RabbitMQ`, WebSocket-библиотеки с поддержкой Pub/Sub
  + Если чат реализован через WebSocket, Pub/Sub может быть встроен в логику
    . Например, отправка сообщения (`publish`) уведомляет подписчиков (`subscribe`)
    - Проверьте папку `services`, `chat`, или `events` на наличие Pub/Sub-логики
  + Найдите модули или файлы, связанные с чатом
    - если сообщения отправляются на брокер или WebSocket, это может быть признаком Pub/Sub
  + отправьте сообщение в чат и проверьте:
    - логирование на сервере
    - если сообщение доходит мгновенно, вероятно используется WebSocket с логикой Pub/Sub
* альтернатива: различные реализации того же абстрактного интерфейса, что и `channels_redis.core.RedisChannelLayer`
  + встроенный в Django Channels **InMemoryChannelLayer**
    - класс из пакета `channels.layers`  
    - хранит сообщения в памяти процесса Python  
    - не годится для продакшн: поддерживается единственным процессом, данные исчезнут при перезапуске
  + RabbitMQ брокер сообщений системных сообщений (уведомление о начале турнира, уведомление о добавлении в друзья)
  + PostgreSQL
  + IPC  
* bakyt: Redis is commonly used as a message broker and **in-memory store**
  + publish/subscribe capabilities, allowing messages to be instantly distributed to multiple subscribers
  + Channel Layer Backend
    - to handle real-time communication between different consumers
    - the backend for **Django’s ASGI channel layer**, enabling **inter-process communication**
    - Without Redis, different Django processes wouldn’t be able to share real-time events
  + Scalability: Redis allows your chat system to work across multiple Django instances. Even if your application is running on multiple servers, Redis ensures all instances remain synchronized.
  + Session and Message Storage
    - While messages are usually stored in a database, Redis can be used for temporary message caching before writing to the database.
    - It can also store user session data, improving chat performance.
  + Low Latency
    - Redis operates in-memory, making it extremely fast compared to traditional databases.
    - This ensures minimal delay in message delivery, improving the real-time experience.
* Протокол: Redis pub/sub
* если несколько серверов/контейнеров обрабатывают ws-запросы
  + daphne каждого инстанса/контейнера подключается к Redis-серверу
  + сервер получает сообщение от клиента, отправляет его в канал через Channel Layer
  + Redis доставляет соообщение на другой сервер/инстанс, подписаный на канал


### REDIS ДЛЯ ХРАНЕНЯ СЕССИЙ
* при подключении пользователей ws
* для управления **состоянием** ws-соединений
* управление пользовательскими сессиями, хранит **данные о подключениях**, о текущих авторизованных пользователях
* бекенд для **ws-сессий**
* функции Redis для управления WebSocket-сессиями и кэширование — это схожие, но разные задачи
  + Кэширование
    - Redis используется как высокоскоростное хранилище данных в памяти.
    - Обычно применяется для хранения результатов вычислений или запросов к базе данных, чтобы ускорить ответы сервера.
    - Данные в кэше имеют короткий срок жизни и могут быть сброшены.
  + Управление состоянием WebSocket-сессий
    - Redis может выступать как **хранилище данных о сессиях** и точка синхронизации для распределённых серверов.
    - Например, когда у вас несколько экземпляров backend-сервисов, Redis используется для:
      - Хранения информации о подключениях пользователей.
      - Сохранения авторизационных данных и текущих состояний пользователей.
      - Широковещательной передачи сообщений между backend-сервисами через Pub/Sub для синхронизации WebSocket-сообщений.
* В текущей конфигурации Redis не настроен для хранения пользовательских сессий
  + SESSION_ENGINE = 'django.contrib.sessions.backends.cache' указывает Django использовать кэш для хранения сессий
  + SESSION_CACHE_ALIAS = 'default' привязывает сессии к конфигурации Redis из вашего блока CACHES


### REDIS ДЛЯ КЭШИРОВАНИЯ
* django cash framework обращается к redis
  + инфраструктура для кэширования
  + Django API для кэширования данных
* что кэширует
  + данные, которые часто запрашиваются, но редко изменяются
  + запросы, объекты, шаблоны, ...
  + временне данные для ускорения работы приложений (имя, аватар, **временный статус пользователя: онлайн/офлайн**)
  + результатов сложных SQL-запросов, сложных вычислений
  + результаты REST API-запросов, чтобы не нагружать базу данных повторными запросами
  + API-ответов (данных профиля пользователя, данных статистики)
  + сеансы: **хранилище** сеансов для авторизованных пользователей
  + пользовательские сессии, что **ускоряет работу сессий в распределённых системах**
* **класс клиента определяет, как Django взаимодействует с Redis**
* `OPTIONS` = `CLIENT_CLASS` настраивает использование `DefaultClient` из библиотеки `django-redis` для взаимодействия с Redis
* django REST Framework обращается к redis
  + кэшировать ответы API, особенно одинаковые для множества пользователей
  + кэширование для представлений API: если данные уже есть в кэше, они будут возвращенbaы из Redis, если нет, будут получены из базы данных и затем кэшированы
  + кэширование на уровне вьюх или сериализаторов
* django channels обращается к redis
  + CHANNEL_LAYERS: позволяет Django Channels использовать Redis для обмена сообщениями через WebSocket или другие каналы
    - redis = очередь сообщений, которые могут быть отправлены клиентам
    - reids = механизм синхронизации для разных процессов или серверов
  + redis для кэширования промежуточных результатов в асинхронных операциях (промежуточные результаты обработки WebSocket-соединений)
* браузер кэширует картинки, CSS, JS), чтобы загружать их быстрее при повторных запросах
  + может кэшировать API-ответы, если сервер отправляет заголовки **`Cache-Control`**
* в DRF стоит кэшировать **запросы к базе данных**,
  + если данные часто запрашиваются, но редко изменяются
  + или если нужно снизить нагрузку на сервер
  + **можно** использовать Django Cache Framework и кэшировать API-ответы с `@cache_page()` или использовать `cache.get()`/`cache.set()`
  + django channels сам по себе не кэширует данные
    - redis используемый для Channel Layers может временно хранить сообщения до их доставки
    - если нужно кэшировать ws-сообщения** или результаты долгих вычислений, можно использовать Redis в качестве кэша.
* consumer может работать как напрямую с Redis, так и через Django Cache Framework (который **может** использовать Redis как бэкенд)
  + через Django Cache Framework (абстракция)
    - consumer не взаимодействует с Redis напрямую
    - `messages = cache.get(self.room_name, [])`  # Читаем из кеша (может быть Redis)
    - Django Cache сам выбирает бэкенд (можно легко переключиться с Redis на Memcached и т.д.)
    - код проще и унифицирован  
    - если нужен просто кеш (например, хранить сообщения), лучше использовать Django Cache Framework
    - если CACHES настроен, значит Django Cache Framework включен
  + напрямую с Redis (без Django Cache)
    - consumer сам подключается к Redis, управляет кэшем напрямую
    - `await self.redis.rpush(self.room_name, message)`
    - более гибкий контроль над Redis (можно использовать списки, pub/sub и т.д.)
    - можно хранить данные **без зависимости от Django**  
    - если ты используешь Redis как брокер сообщений **или** pub/sub, тогда лучше обращаться к нему напрямую
* сохранение сообщений в базу данных и кэширование могут быть организованы по разным принципам
  + 1) каждое входящее чат-сообщение записывается и в базу данных, и в кэш, сразу же после его получения
    - в бд сохраняются все детали сообщения: отправитель, получатель, дата, текст
    - кэш чтобы ускорить доступ к сообщениям в рамках текущей сессии чата, снижая нагрузку на базу данных при запросах
   - доступность сообщения в обоих хранилищах
   - нагрузка на серверы при большом объеме сообщений
   - более частым является первый вариант
  + 2) сохранение в базу данных только по достижении определенного порога (например, 10-12 сообщений)
    - сообщения сначала сохраняются в кэш
    - в базу данных они записываются только тогда, когда в чате накопится определенное количество сообщений или когда сессия завершена
    - минимизирует нагрузку на бд
    - эффективно управлять производительностью при больших объемах чатов
    - возможные **потери сообщений**, если что-то произойдет до того, как они будут сохранены в базу данных (если кэш будет очищен или сервер будет перезагружен)
  + 3) гибридный подход
    - система должна поддерживать высокую скорость работы
    - кэш для временного хранения
    - бд для долговременного
* альтернатива: по умолчанию django кэширует в памяти `django.core.cache.backends.locmem.LocMemCache` 
  + ограничено по производительности и функциональности
  + кэш в процессе работы сервера
  + если приложение работает в пределах одного сервера и не требует масштабируемости
  + нет сильной нагрузки или большого объёма данных
  + не требует настройки и внешнего сервиса
  + доступ к данным в памяти быстрее
  + в оперативной памяти локального сервера (`LocMemCache`)
    - очищается при перезапуске приложения
  + файловое кэширование (`FileBasedCache`)
    - медленнее, чем кэш в памяти или Redis
    - требует места на диске
  + в базе данных (`DatabaseCache`)
    - замедляет производительность
* возможности
  + сохранять кэшированные данные на диск (persistent storage) для предотвращения потери данных после перезапуска (например, для сессий)
  + сложные типы данных в кэше (списки, множества, хеши)
  + сложные стратегии кэширования: TTL (время жизни), евикция (удаление старых или неиспользуемых данных), ограничение количества запросов от одного пользователя за определённое время (rate limiting)
  + легко масштабируется
  + может использоваться в распределённых системах, в многосерверных / контейнеризованных средах
    - данные в процессе/сервисе Redis, а не в памяти отдельного экземпляра приложения
    - данные кэша разделяются между всеми процессами и серверами, что делает его идеальным для кластеров
    - несколько серверов обращаются к одному кэшированию
    - Redis общий слой
* тех подробности
  + протокол: Обычное key-value хранилище
  + `django_redis` берёт на себя работу с Redis, предоставляя **Django-friendly API**
  + не требует добавления в INSTALLED_APPS
* bakyt: только кэш сообщений, чат и **системные**


### DJANGO MODELS, DATABASE PostgreSQL
* Starting entrypoint script... : Apply all migrations: **admin, auth, contenttypes, myapp, sessions**
* ORM Object-Relational Mapping
  + `models.py`
  + описывают структуру данных (таблицы, поля, связи)  
  + вместо SQL-запросов, разработчик описывает модели (Python-классы), PUT = поменять поле в бд (вместо запроса sql)
  + Django преобразует модели в SQL-таблицы
  + легко менять базу с SQLite на PostgreSQL
  + ForeignKey, ManyToMany, OneToOne создают связи между таблицами
  + **защита от SQL-инъекций**
* `docker exec -it db psql -U myuser -d mydatabase`
  + then `\db`, `\dt`
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
* лучший вариант для Django – оставить `id`
  + django добавляет `id = AutoField(primary_key=True)`, если явный первичный ключ не указан
  + Минусы `game_id`:
    - Django уже создает `id`, так что `game_id` – это **избыточность**.  
    - В коде придется писать `game.game_id`, вместо простого `game.id`.  
    - Во всех внешних ключах (`ForeignKey`) тоже придется явно указывать `to_field='game_id'`, если хотят ссылаться на `game_id`.
* `AutoField` (по умолчанию) = 32-битный `int` (максимальное значение: ~2.1 млрд), чаще всего достаточно #see
  + `BigAutoField` = 64-битный `int` (максимальное значение: ~9 квинтиллионов)
* AUTH_USER_MODEL = 'myapp.UserProfile`,  кастомная модель `myapp.UserProfile` вместо стандартного `User`
  + class UserProfile(AbstractUser)
  + наследуется от `AbstractUser` => это полноценный пользователь Django
    - механизмы аутентификации **`login()`**, **`authenticate()`**, `request.user` работают
  + таблица пользователей `myapp_userprofile` (не `auth_user`)
    **public** | auth_group                         | table | myuser
    public | auth_group_permissions             | table | myuser
    public | auth_permission                    | table | myuser
    public | myapp_userprofile_user_permissions | table | myuser
  + со стандартной моделью django.contrib.auth.models.User невозможно напрямую добавить аватар, уникальный email, систему друзей, ..., нельзя изменить существующие поля (`username`, `email`)
  + `avatar` реализовать без расширения: Создать модель `UserProfile`, связанную с `User` через `OneToOneField`
  + `friends` реализовать без расширения: Создать модель **Friendship**, связанную `ManyToManyField(User)`
  + URL для фото picture_url реализовать без расширения: Добавить поле в `UserProfile`
  + когда дргуие модели ссылаются на `User`: **`get_user_model()`**
  + какую модель использует django: `docker exec -it backend python manage.py shell` -> `from django.contrib.auth import get_user_model` `print(get_user_model())`
* **с повторным емейлом не создаём?**
* генерировать граф: pip install django-extensions pygraphviz pydot
* migrations
  + `python manage.py makemigrations`
    - создаёт файлы миграций (инструкции по изменению базы)  
    - django выполняет изменения из файлов миграций
    - django обновляет структуру базы данных
  + python manage.py migrate применение миграций базы данных
    - создаёт таблицы для новых моделей
    - добавляет, изменяет, удаляет поля
    - обновляет связи между таблицами
  + docker exec -it backend python manage.py showmigrations
  + docker exec -it backend python manage.py inspectdb
  + docker exec -it backend python manage.py sqlmigrate myapp 0001


### GAME LOGIC
* a player should also be possible to propose a tournament (subject)
  + a tournament displaies who is playing against whom and the order of the players (subject)
• a matchmaking system: the tournament system organize the matchmaking of the participants, and announce the next fight
* Remote players (subject)
  + two distant players, each player is located on a separated computer, accessing the same website and playing the same Pong game
  + think about **network issues**, like unexpected disconnection or lag
  + you have to offer the best user experience possible
* Game Customization Options (subject) возможно будем
  + customization options for all available games on the platform
  + customization features, such as power-ups, attacks, or different maps, that enhance the gameplay experience
  + allow users to choose a default version of the game with basic features if they prefer a simpler experience
  + ensure that customization options are available and applicable to all games offered on the platform
  + implement user-friendly settings menus or interfaces for adjusting game parameters
  + maintain consistency in customization features across all games to provide a unified user experience
  + to give users the flexibility to tailor their gaming experience across all available games by providing a variety of customization options while also offering a default version for those who prefer a straightforward gameplay experience
* AI Opponent (subject)
  + to introduce data-driven elements to the project, with the major module introducing an AI opponent for enhanced gameplay, and the minor module focusing on user and game statistics dashboards, offering users a minimalistic yet insightful glimpse into their gaming experiences
  + an AI player into the game
  + the use of the A* algorithm is not permitted 
  + an AI opponent that provides a challenging and engaging gameplay experience for users
  + the AI replicates human behavior, meaning that in your AI implementation, you must simulate keyboard input
  + the AI refreshes its view of the game once per second, requiring it to anticipate bounces and other actions
  + the AI utilizes **power-ups** if you have chosen to implement the Game customization options module
  + AI logic and decision-making processes that enable the AI player to make intelligent and strategic moves
  + explore alternative algorithms and techniques to create an effective AI player without relying on A*
  + the AI adapts to different gameplay scenarios and user interactions
  + to explain in detail how your AI is working 
  + it must have the capability to win occasionally
  + an AI opponent that adds excitement and competitiveness without relying on the A* algorithm
* game logic, because we need it to do the multiplayer
* трекинг **когда пользователь был онлайн**
  + models.py: last online для индикатива
  + три решения 
    - через библиотеку channels - по вебсокетам следим пользователь онлайн или нет - НАМ ЭТОТ СПОСОБ
    - через Django sessions - как только юзер делает какое либо действие, Джанго сохраняет в бд дату действия 
    - через redis - не понял как работает
  + **"is_active": true** в http://localhost:8000/users/
* alexey: Tournaments – working 
* Амин 
  + whether we want to follow basic ping pong rules?
  + the ball should speed up when paddle hits the ball ?
  + https://stackoverflow.com/questions/54796089/python-ping-pong-game-speeding-up-the-ball-after-paddle-hit 
  + some sort of **anticheat** to be sure that the users mouvement are normal
    - my code outputs two players position and the ball and then you can render it however you want

### USER MANAGEMENT
* myapp: логика пользовательских профилей, турниров, историй игр
* user management, authentication, users across tournaments (subject)
  + Users can subscribe to the website in a secure way
  + Registered users can log in in a secure way
  + Users can select a unique display name to play the tournaments
  + Users can update their information
  + Users can upload an avatar, with a default option if none is provided
  + Users can add others as friends and view their online status
  + User profiles display stats, such as wins and losses
  + Each user has a Match History including 1v1 games, dates, and relevant details, accessible to logged-in users
  + the management of duplicate usernames/emails is at your discretion, you must provide a solution that makes sense
* user and Game Stats Dashboards (subject) может быть будем делать
  + dashboards that display statistics for individual users and game sessions
  + user-friendly dashboards that provide users with insights into their own gaming statistics
  + a separate dashboard for game sessions, showing detailed statistics, outcomes, historical data for each match
  + the dashboards offer an intuitive and informative user interface for tracking and analyzing data
  + data visualization techniques, such as charts and graphs, to present statistics in a clear and visually appealing manner
  + users access and explore their own gaming history and performance metrics conveniently
  + add any metrics you deem useful
  + users monitor their gaming statistics and game session details through user-friendly dashboards
  + a comprehensive view of their gaming experience
• a registration system
  + at the start of a tournament, each player must input their alias name
  + the aliases will be reset when a new tournament begins
  + this requirement can be modified using the Standard User Management module
* pass reset будет ли?
* change username, email будет ли?


### СТАТИЧЕСКИЕ ФАЙЛЫ html js CSS изображения шрифты
* чтобы все файлы находились/отдавались там, где ожидает браузер
* **Frontend: права только на чтение статических файлов Backend**
* python manage.py collectstatic собирает стат файлы DRF в STATIC_ROOT/
  + админские CSS, стат. файлы DRF для browsable API, ... - в `django.contrib.admin` 
* daphne не обслуживает статические файлы
  + может с библиотекой WhiteNoise
  + стили для jango приходят с фронтенда (собранный **бандл**)
* общий том, чтобы Nginx и backend контейнеры обменивались статическими файлами 
* nginx обслуживает ваши стат фалйы и стат фалйы Django
   + по пути STATIC_URL = '/static/ (url для nginx, не папка)
   + Django формирует ссылку <link rel="stylesheet" href="/static/admin/css/base.css">
    + браузер стучится по https://HOST/static/admin/css/base.css
    + nginx видит location /static/ { alias /usr/share/nginx/html/static/;} }
    + nginx берёт в /usr/share/nginx/html/static/backend/admin/css/base.css
* ошибка “MIME type ('text/html') is not a supported stylesheet” #see
  + лог: Refused to apply style from '/static/css/chat.css' because its MIME type ('text/html') is not a supported stylesheet MIME type
  + nginx отдает chat.css с заголовком Content-Type: text/html вместо text/css
  + убедитесь, что /static/css/chat.css (или /app/static/css/chat.css) существует
  + убедить, что настроены правильные заголовки MIME
  + **MIME-тип** автоматически определит, что *.css -> text/css
    location /static/ {
        alias /path/to/collected/static/;
        try_files $uri =404; 
    }
   + или включить **include /etc/nginx/mime.types;**
* django.contrib.staticfiles.testing.**StaticLiveServerTestCase**: to transparently serve all the assets during execution of these tests in a way very similar to what we get at development time with DEBUG = True, i.e. without having to collect them using collectstatic first
* CSS-OM = дереао как DOM
* bootstrap готовые стили, можно создавать кастомные на основе них
* .map (Source Map) 
  + при минификации или транспиляции CSS/JS (при сборке) код CSS превращается в сжатую скомпилированную версию
  + Source Map хранит сопоставление (mapping) между сжатым и исходным кодом%, чтобоы видеть исходный код для отладки кода в браузере
  + в продакшен отключают генерацию .map-файлов, чтобы уменьшить вес приложения и не раскрывать детали кода
* в разработке три других варианта
  + django.contrib.staticfiles
    - обслуживает **стат файлы для /admin**
    - позволяет добавлять версии, хеши в имена файлов на фронтенд, чтобы избежать кеширования старых версий
    - если CSS-файл кэшировался, браузер может залипать => добавить ?v=123 в конце ссылки или очистить кэш
  + manually serve user-uploaded media files from MEDIA_ROOT
    - use django.views.static.serve() view
    - don’t have django.contrib.staticfiles in INSTALLED_APPS
    - if STATIC_URL = static/, addi urlpatterns = [ ] to your urls.py `static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)`
      - works only if the prefix is local (static/) and not a URL (http://static.example.com/)
    - only serves STATIC_ROOT folder
    - doesn’t perform static files discovery like django.contrib.staticfiles
    - serves static files via a wrapper at the WSGI application layer => static files requests do not pass through the middleware chain
  + напрямую ссылаться на файлы через URL


### IDENTIFCATION = кто вы
+ определение личности пользователя (предоставление username, email)  
+ без проверки пароля
+ не вход в систему  


### AUTENTIFICATION = докажите, что это вы
* проверка личности, логин, система требует пароль, одноразовый код OTP из SMS, отпечаток пальца, лицо, вход через соцсети, ...
* базовая аутентификация (Basic Auth)  
  + логин/пароль передаются в заголовке при каждом запросе
  + простой
  + слабо защищён
* ТОКЕН АВТОРИЗАЦИИ JWT (основной вариант это JWT)  
  + jwt = формат токена
  + для Веб-приложения, API, ws, систем с аутентификацией без сохранения состояния (stateless authentication), SPA, мобильных приложений, микросервисов
  + сервер проверяет данные и создаёт JWT, содержащий информацию о пользователе (id, имя, ...)
  + клиент получает токен и отправляет его в заголовке `Authorization`, в заголовках ws-сообщений, в параметрах URL
  + **служба аутентификации** хранит закрытый ключ и может выдавать токены
  + другие микросервисы проверяют с помощью **открытого ключа**
  + самодостаточен, данные для проверки токена содержатся в нём
    - не требует хранения состояния на сервере (кроме валидации и возможной чёрной/белой списков)
    - подписан сервером, использует подпись, чтобы предотвратить изменения данных
  + может храниться в HTTP заголовках, cookie, localStorage, sessionStorage
    - хранится в браузере => безопасно **почему у нас не так?**
    - самый простой и безопасный — **cookie**, все микросервисы имеют одно и то же имя хоста => cookie отправляется браузером при каждом запросе
  + не нужен CSRF-токен
  + не нужны сессии
  + если используется только JWT, то можно **отключить SessionMiddleware и CsrfViewMiddleware**
  + если украдён, злоумышленник может использовать его до истечения срока действия: устанавливайте короткий срок действия токена
  + реализуйте механизмы обновления и отзыва токенов
    - отзыв сложен
  + обеспечивается через middleware Channels
  + обрабатывать его в consumer
  + https://django-rest-framework-simplejwt.readthedocs.io/en/latest/getting_started.html
  + https://dzone.com/articles/using-jwt-in-a-microservice-architecture
  + https://openclassrooms.com/fr/courses/7192416-mise-en-place-une-api-avec-django-rest-framework/7424720-give-access-with-the-tokens
* 42 oauth = разновидность токен-ориентированной аутентификации
  + 1) аутентификация через OAuth2
  + 2) данные пользователя (логин, email, имя, аватар) извлекаются и сохраняются в базе данных
  + 3) клиент получает токен доступа Bearer Token Authentication
  + 4) токен сохраняется в сессии пользователя `request.session.get('access_token')`
  + 5) для запроса к защищенному маршруту `/v2/me` токен передается в заголовке `Authorization: Bearer <access_token>`
  + токен имеет ограниченный срок действия
  + регулярно **обновляй токены через OAuth2 Refresh Token**
  + если хранится в Local Storage
    - простота реализации
    - данные остаются после перезагрузки страницы
    - злоумышленник может украсть токен через js-код (**XSS-атака**)
  + если хранитсяв Session Storage
    - простота реализации
    - данные удаляются при закрытии вкладки, недоступны между вкладками
    - уязвимо к XSS-атакам
  + если хранится в памяти приложения (In-Memory), в оперативной памяти клиента (в переменной в js, ...)
    - токен исчезает при обновлении страницы
    - требует реализации для повторного получения токена
    - защита от XSS, поскольку токен недоступен после перезагрузки страницы
  + если хранится на стороне сервера в базе данных (**Access Token, Refresh Token**)
    - безопасность: токен недоступен для клиента
    - удобное управление сроком действия и обновлением токенов
    - медленнее
    - дополнительная обработка
    - сохраняйте **Access Token** и **Refresh Token** в **защищённой таблице**, связанной с пользователем
    - шифруйте токены перед сохранением в базу данных
    - ограничить доступ к таблице с токенами необходимыми сервисами
  + если хранится на стороне сервера в кэше Redis 
    - быстрый доступ
    - легко управлять истечением токенов (**TTL**)
    - данные очищаются при сбое или перезагрузке
  + если хранится на стороне сервера в HTTP-only cookies
    - Токен (или сессионный идентификатор, связанный с токеном) передаётся клиенту в виде HTTP-only cookie
    - браузер отправляет cookies на каждый запрос
    - Требует настройки `SameSite`, `Secure`, HTTPS
    - защищён от XSS-атак (недоступен через js)
  + гибридный подход
    - Access Token в памяти клиента (`In-Memory`) для минимизации уязвимости к XSS
    - Refresh Token в HTTP-only cookie для долгосрочного хранения и обновления Access Token
  +
    | **Место хранения**        | **Безопасность**      | **Удобство** | **Долговечность** |
    |---------------------------|-----------------------|--------------|-------------------|
    | Local Storage             | Уязвимо к XSS         | Высокое      | Высокая           |
    | Session Storage           | Уязвимо к XSS         | Среднее      | Средняя           |
    | In-Memory                 | Защищено от XSS       | Низкое       | Низкая            |
    | База данных (сервер)      | Очень высокая         | Низкое       | Высокая           |
    | Redis (сервер)            | Высокая               | Высокое      | Средняя           |
    | HTTP-only cookies         | Защищено от XSS/CSRF  | Высокое      | Средняя/высокая   |
  + если важно удобство разработки: Local Storage или Session Storage
  + если требуется безопасность: храните Access и Refresh Tokens на сервере (в бд или Redis), используйте HTTP-only cookies для передачи сессионных данных
  + **Frontend не хранит токены**
    - frontend использует безопасные сессии через HTTP-only cookies для взаимодействия с Backend
  + Frontend отправляет запросы к Backend
    - backend использует токены для взаимодействия с внешними сервисами без передачи токенов на Frontend, токены не покидают Backend
  + получение Токенов
    - frontend перенаправляет пользователя на OAuth-провайдера **через Backend**
    - OAuth-провайдер перенаправляет пользователя обратно на Backend с кодом авторизации
    - backend **обменивает** **код авторизации** на Access Token и Refresh Token
  + кэшируйте Access Tokens в Redis для быстрого доступа, особенно если Backend часто обращается к внешним API
  + используйте Redis для хранения сессий пользователей, **связывая их с соответствующими OAuth-токенами**
  + как Backend взаимодействует с геолокационными сервисами
    - извлекает Access Token из базы данных или Redis
    - сервисы или модули на Backend, которые отвечают за взаимодействие с Google Maps API, используют Access Tokens для аутентификации запросов
    - Геолокационные компоненты на Backend используют сохранённые токены для взаимодействия с внешними API
  + автоматическое обновление Access Tokens с помощью **Refresh Tokens**
  + реализуйте механизмы для **аннулирования токенов** при необходимости
  + **Backend отвечает за OAuth-поток**, включая получение, хранение и обновление токенов
  + **Токены хранятся в защищённой базе данных** и кэшируются в Redis для быстрого доступа
  + настройте заголовки для безопасности (**`Strict-Transport-Security`**, ...)
  + **Обеспечивается безопасность токенов** через шифрование, ограничение доступа и использование защищённых методов передачи данных
* СЕССИИОННЫЙ КЛЮЧ = СЕССИОННЫЙ ИДЕНТИФИКАТОР = ФАЙЛ СЕССИИ
  + после входа сервер создает сессию, которая привязывается к пользователю
  + сервер хранит сессию **в памяти или базе**
  + в браузере появляется куки sessionId
    - `HttpOnly`, чтобы предотвратить доступ из js
    - `Secure`
    - настройте `SESSION_COOKIE_AGE`
  + при каждом запросе браузер отправляет `sessionid`
    - Django использует его для извлечения данных из хранилища сессий (база данных, кэш, файл)
  + подходит для stateful-приложений (например, с веб-интерфейсом)
* API-КЛЮЧИ
  + авторизация сторонних приложений
  + интеграция с внешними сервисами
  + предоставление публичного API
* DFR использует сессии или JWT для аутентификации пользователей
  + эти данные доступны в middleware или обработчиках запросов
  + Session storage: Если включён SessionMiddleware, то пользовательские данные сохраняюися в сессиях и доступны в обработчиках
  + JWT токен передаётся в заголовке авторизации (Authorization: Bearer <token>) и обрабатывается специальным классом аутентификации
* асимметричный алгоритм (например **RSA**)
* какой у нас тип - посмотреть auth, authentication, REST_FRAMEWORK, oauth, session, cookies, bearer, authorization, csrf, middleware, authentication classes   
    - F12 - network - headers, выполните login, посмотрите, что отправляется с клиентской стороны
    - Если есть заголовок `Authorization: Bearer <токен>` в последующих запросах, используется JWT (или др токен)
    - Если сервер выставляет куку (`Set-Cookie: sessionid=...`) => сессионная аутентификация (Django Sessions, например)
    - комбинированный вариант: JWT храним в куках, ...

| Метод           | Как работает                                       | Где хранится аутентификационная информация?   | Плюсы минусы
|-----------------|----------------------------------------------------|-----------------------------------------------|-------------------------------|
| логин + пароль  | user вводит логин/пароль, сервер создаёт сессию    | сервер проверяет данные в базе                | каждый запрос требует отправки пароля |
| JWT             | сервер выдаёт токен, клиент передаёт в заголовках  | `localStorage`/`sessionStorage`, куки         | проверка без БД, можно украсть токен |
| OAuth           | пользователь логинится через 42, получает токен    | Сторонний OAuth-сервер выдаёт токен           | не нужно хранить пароли |
| Сессионный ключ | сервер создаёт файл, выдаёт клиенту  `session_id`  | хранится в куках, проверяется на сервере      | авто управление сессиями, неудобно для API|
| API-ключ        | выдаётся уник. ключ (как токен), передаётся в загол| клиент хранит ключ, сервер проверяет его в бд | удобно для сервисов, API, можно украсть ключ|


### AUTORISATION = что вам разрешено делать?
+ что можете делать, к каким ресурсам имеете доступ 
* только залогиненные могут просматривать `StatsPage`
  + три инструмента работают на разных уровнях
  + 1) проверять аутентификацию перед рендерингом в `StatsPage`  
    - добавь `checkAuthentication()` в конструкторе, запрашивать данные текущего пользователя
      ```
      async checkAuthentication() {
          try {
              const response = await fetch('/auth/me', { method: 'GET' }); // Предполагается, что есть API /auth/me
              if (!response.ok) throw new Error('Not authenticated');
              const user = await response.json();
              this.setState({ currentUser: user });
              this.fetchGames();
          } catch (error) {
              console.error('Authentication error:', error);
              window.location.href = '/login'; // Перенаправление на страницу входа
          }
      }
      ```
  + 2) скрывать контент, если пользователь не залогинен
    ```javascript
    render() {
        if (!this.state.currentUser) {
            return '<p>Loading...</p>'; // Или ничего не рендерить
        }
        // код рендера
    }
    ```
  + 3) защитить API на сервере
    - если кто-то вручную запрашивает `/games/`, `/users/`, сервер не отдаст незалогиненному
* autorisation
  + При каждом запросе клиент отправляет токен в заголовке Authorization: Bearer <token>
  + Сервер декодирует токен, проверяет его подпись, использует данные для авторизации (например, проверяет роль пользователя)
  + виды авторизаций: Ecole 42, гугл, логин и пароль с базы данных
  + задача на бэкэнде
  + внедрить 2FA через электронную почту
    - Мы выполнили первую часть, генерируя код и все такое
    - были проблемы с последней частью, где мы на самом деле отправляем электронное письмо
      - попробовали sendgrid life, было сложно добиться успеха в отправке электронных писем
    - мы использовали Gmail, найти путь через пользовательский интерфейс Google для создания «mdp приложения» на стороне Django, функция send_mail
  + при проверке ролей и прав через сессию система также участвует в авторизации (например, проверяет, имеет ли пользователь доступ к определенным ресурсам).


### CSRF (CROSS-SITE REQUEST FORGERY) ключ, токен
* убедиться, что запрос исходит от легитимного пользователя
* защита формы на странице входа
* защищает формы и AJAX-запросы от подделки запросов
* защита от CSRF-атак
* для защиты UI-приложений
* в stateful-приложениях, где используется sessionid или cookie для отслеживания пользователя
  + в stateless-приложениях с JWT-авторизацией не используется
* django сохраняет в cookie `csrftoken`
  + доступно только для чтения с клиентской стороны (не `HttpOnly`)
  + js может получить доступ к токену (например, для AJAX-запросов)
  + cookie устанавливается с параметром `Secure`, если используется HTTPS
  + не должен быть доступен для сторонних доменов (используйте `SameSite=Lax` или `SameSite=Strict`)
* при отправке POST-запроса передаётся серверу в заголовке или скрытом поле формы
* django сравнивает токен из запроса с токеном из cookie, если совпадают, запрос легитимен
* токен **в settings.py**
* отдать клиенту при первом запросе
* клиент вставляет токен в заголовок запроса к **api** и при post/get запросах
  + **а при вебсокетах?**
* https://docs.djangoproject.com/en/5.1/howto/csrf/
* csrf MW проверяет токен **до этого не надо его проверять?**
* views.py **набор проверок** добавить при запросе к api
* CSRF_TRUSTED_ORIGINS¶
  + trusted origins for unsafe requests (e.g. POST)
  + Default: []
  + a request includes the Origin header => CSRF protection requires that header match the origin present in the **Host header**
  + a secure unsafe request that doesn’t include the Origin header => the request must have a **Referer header** that matches the origin present in the Host header
  + prevents a POST request from subdomain.example.com from succeeding against api.example.com


### ТОКЕНЫ
* WEBSOCKET TOKEN
  + для авторизации пользователя при установке ws-соединения
  + проверка прав доступа и идентификация пользователя
  + при установке ws-соединений клиент передаёт токен в URL или заголовках `wss://example.com/ws/chat/?token=<your-token>`
  + могут быть основаны на JWT или другом формате
  + передача должна быть защищена (через HTTPS/WSS, ...)
* комбинированные методы  
  + session + CSRF токен, или JWT в куке
  + часто в современных фреймворках для повышения безопасности
| токен/ключ    | Назначение                          | Где хранится                         | Особенности | пример |
|---------------|-------------------------------------|--------------------------------------|-------------|--------|
| CSRF          | от подделки запросов (CSRF-атак)    | cookie csrftoken                     | проверяется сервером при POST/PUT запросах, передаётся в скрытом поле формы или заголовке            | Защита формы входа; обеспечение легитимности запросов от пользователя; AJAX-запросы|
|ток авторизации| авторизация? аутентификация?        |HTTP-заг cookie locStorage sessStorage| Используется для REST API и WebSocket; может быть JWT | авторизация REST API (`Authorization: Bearer <token>`) или ws-соединения; REST API, GraphQL, WebSockets |
|сессионный ключ| управление пользов сессией          | cookie sessionid                     | долговрем. связь пользователя с сервером; подходит для **веб-интерфейсов**; сохраняется на сервере | отслеживание состояния авторизации в веб-приложении; данные польз. (корзина); Веб-приложения с авторизацией|
| ws-ток        | авторизация при установке ws-соедине| URL параметр / заголовок ws-запроса  | передаётся при установке соединения; проверка прав доступа | авторизация чата, потоковой системы `wss://example.com/ws/chat/?token=<...>`|
| API-ключи     |идентификация, авторизация стор прилож| HTTP заголовок запроса              | передаётся с каждым запросом; ограниченный доступ к API | интеграция с внеш сервисом, предост. публ. API `Authorization: Api-Key <>`|
| щифров. ключи | шифрование сообщений                | На сервере в хранилище               | не передаётся клиенту; для шифрования | шифрование сообщений чата на стороне сервера|
| OAuth-токен   | аутентиф и авторизация через Google |  cookie или localStorage             | для OAuth 2.0. | авторизация Google OAuth API; доступ к данным польз. через API (email, ...)|

* js обращается к rest api (post) endpoints /history, /users/, /send
  + login pages.js - запрос post к бэку
  + в заголовке запроса каждый раз CSRF токен, чтобы знать, что это не юзер с третьего сайта
  + F12 network выбрать сокет: accept-encoding:
    - cookie: csrftoken=KRaxdt052lYdhYUoahiFkhwGgB00H4jg
    - sec-websocket-key: BNMxSoHDbO66J+298LsweQ==
* cookies sessionId 
  + идентификатор сессии
  + отслеживание состояния пользователя на сервере
    - например, для аутентификации или сохранения данных между запросами
* **Stateful/Stateless**
* где backend хранит сессии?
  + настройки бекенда
* хранятся SESSION_ENGINE
  + База данных в таблице django_session (по умолчанию)
    - `session_key`: Уникальный идентификатор сессии.
    - `session_data`: Зашифрованные данные сессии.
    - `expire_date`: Дата истечения срока действия сессии.
  + Файловая система (в виде файлов)
  + в кеше, настроенном в `CACHES`
  + кеш с fallback на базу данных: сначала пишутся в кеш, а при его недоступности — в базу данных
  + на стороне клиента в зашифрованных cookie
* **http2**
* `SECRET_KEY = 'django-insecure-5equ&8dpiahftu52n^a5aq=52s8a_d8ty6(smkzjpsmvyk8q(#'`
  + для шифрования сессионных данных (cookies) и других временных данных, создаваемых Django
  + для генерации подписи токенов CSRF и других значений, требующих криптографической защиты
  + для генерации случайных токенов, например, в механизмах сброса пароля
  + Если злоумышленники узнают ваш `SECRET_KEY`, они могут:
    - Создавать фальшивые cookies, сессии или токены.
    - Подделывать запросы или манипулировать приложением.
* можно без Session MiddleWareStack, если JWT токены или др безсессионные методы аутентификации (например, в DRF сессии не нужны, авторизация через токены)
* можно без Session MiddleWareStack, если данные о сессии хранятся полностью в зашифрованных cookie (без серверного хранилища)
* DRF: проверка пользователя выполняется при каждом запросе через аутентификационные классы (TokenAuthentication, SessionAuthentication, JWTAuthentication)
  + каждый запрос должен содержать соответствующий токен или cookie для подтверждения личности пользователя
  + Контроль доступа Обрабатывается через permissions (например, `IsAuthenticated`)
* Django Channels: Аутентификация один раз при установлении соединения
  + Использует middleware (`AuthMiddlewareStack`) для сопоставления пользователя.
  + Поддерживает передачу аутентификационных данных через Cookie (например, `sessionid`) или токены
* если хотите объединить аутентификацию для DRF и Django Channels, рекомендуется использовать JWT или SessionAuthentication и настроить  middleware
* any password stored in your database, if applicable, must be hashed; a strong password hashing algorithm (subject)
* your website must be protected against **SQL injections/XSS** (subject)
* if you have a backend or any other features, it is mandatory to enable an HTTPS connection for all aspects (Utilize wss instead of ws...)
* some form of validation for forms and any user input, either within the base page if no backend is used or on the server side if a backend is employed (subject)
* the security of your website (subject)
  + regardless of whether you choose to implement the JWT Security module with 2FA
  + even if you decide not to use JWT tokensp
  + for instance, if you opt to create an API, ensure your routes are protected
* views.py проверяет токен/подпись
* alexey: Authorization and authorization logic on frontend — almost works
* Implementing a remote authentication OAuth 2.0 authentication with 42 (subject)
  + obtain the credentials and permissions from the authority to enable a secure login
  + user-friendly login and authorization flows that adhere to best practices and security standards
  + the secure exchange of authentication tokens and user information between the web application and the authentication provider
* аутентификация реализуется через middleware, через стандартный механизм HTTP-запросов, например, с использованием **JWT или сессий**
* сессии управляются с помощью cookie-сессионных данных или токенов для аутентификации API
* из гугл док:
  + Any password stored in your database, if applicable, must be hashed. (DONE)
  + Your website must be protected against SQL injections/XSS. If you have a backend or any other features, it is mandatory to enable an HTTPS connection for all aspects (Utilize wss instead of ws...) - (DONE excl livechat)
  + You must implement some form of validation for forms and any user input, either within the base page if no backend is used or on the server side if a backend is employed. (Validation by Front-end)
  + Regardless of whether you choose to implement the JWT Security module with 2FA, it’s crucial to prioritize the security of your website. (OK)
  + For instance, if you opt to create an API, ensure your routes are protected. Remember, even if you decide not to use JWT tokens, securing the site remains essential.
* **Forbidden (403). CSRF verification failed. Request aborted. когда создала суперпользлвателя и вхожу в джанго.**
  + Origin checking failed - https://localhost:4443 does not match any trusted origins
  + a **genuine Cross Site Request Forgery**, or when Django’s CSRF mechanism has not been used correctly
  + for POST forms, ensure:
    - your browser is accepting cookies
    - the view function passes a request to the template’s render method
    - in the template, there is a {% csrf_token %} template tag inside each POST form that targets an internal URL
    - the form has a valid CSRF token. After logging in in another browser tab or hitting the back button after a login, you may need to reload the page with the form, because the token is rotated after a login


### COOKIE LOCALSTORAGE SESSIONSTORAGE
| **Свойство**        | **Cookie**                            | **LocalStorage**      | **SessionStorage**             |
|---------------------|----------------------------------------|----------------------|-------------------------------|
| **Хранение данных** | В браузере, отправляются серверу      | Только в браузере     | Только в браузере             |
| **Срок действия**   | До истечения срока или закрытия браузера | Постоянное         | До закрытия вкладки/окна      |
| **Объём**           | ~4 KB                                 | 5-10 MB               | 5-10 MB                       |
| **Доступ к данным** | js и сервер (если `HttpOnly` нет)     | Только js             | Только js             |
| **Область действия**| Общая для всего домена                | Общая для домена      | Изолирована для вкладки       |

* Cookie
  + автоматически отправляются серверу при каждом HTTP-запросе на соответствующий домен
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
  + доступны только через js
  + использование: 
    - хранение пользовательских предпочтений (тема сайта)
    - кеширование данных (JSON-ответы от API)
    - для долговременного хранения данных, доступных только клиенту (например, настройки интерфейса)
* SessionStorage
  + Похож на localStorage
  + данные хранятся только на время текущей сессии браузера
  + изолированно для текущей вкладки или окна
  + данные удаляются, как только вкладка или окно браузера закрываются
  + доступны только через js
  + доступны только для вкладки или окна, где они были созданы
  + использование
    - хранение временных данных для одной сессии пользователя
    - **статус пользователя** на текущей странице (выбранные фильтры)
    - для временных данных, которые нужны только на время одной сессии пользователя
* валидация данных
  + **js на фронте читает инпут, проверяет с помощью regex**
  + функция make password django шифрует на сервере ?
  + бэк ещё раз валидирует (проверяет пароль и почту, ...) **зачем два раза** 


### ПРОТОКОЛЫ
* **прикладные** протоколы: как данные кодируются и интерпретируются
  + HTTP (HyperText Transfer Protocol) для передачи данных между браузером и сервером в открытом виде
  + HTTPS (Secure), шифрование через протокол SSL/TLS
  + Redis
  + PostgreSQL
* **транспортные**
  + TCP (Transmission Control Protocol)
    - сетевой протокол для передачи данных на порту
    - надежность соединения
    - без потерь, в нужном порядке, без дублирования
    - управление потоком: отправка данных регулируется, чтобы не перегружать получателя
    - контроль ошибок: если пакет теряется или повреждается, он будет переотправлен
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
  + **certBox** сгенерировать свой сертификат, который браузер будет принимать локально без ошибок
  + SSL-сертификат решает другие задачи, связанные с безопасностью
    - SSL-сертификаты шифруют соединение, защищая передаваемые токены (CSRF, авторизации, WebSocket)
    - токены (JWT, sessionid) передаются внутри защищённого соединения, чтобы их не могли перехватить злоумышленники
  + SSL session ID
    - низкоуровневая часть протокола SSL/TLS
    - для ускорения установления безопасного соединения
    - сохраняется на сервере
    - может быть сохранена в cookie, но не обязательно


### DOCKER
* **docker volume ls** лишний том
* Чтобы файлы нового приложения, созданные внутри контейнера, появились на хосте
  + директория проекта внутри контейнера и на хосте связаны с помощью volumes (точек монтирования) `./backend:/app `
* `context ./backend`
  + файлы в `./backend` доступны для процесса сборки
  + только файлы COPY, ADD, ... скопируются в контейнер
  + `./backend` рабочая область команд в Dockerfile
* `WORKDIR /app`
  + используйте, даже если копируете все файлы в корень (`/`)
  + файлы будут копироваться `/app`, а не в `/`
  + все команды после этой строки будут выполняться в этой директории
    - без неё `CMD ["python", "/app/manage.py", "runserver", "0.0.0.0:8000"]`
* `volumes` is applied at runtime, does not affect the build stage
* ```dockerfile
  COPY requirements.txt .
  RUN pip install --no-cache-dir -r requirements.txt
  COPY . .
  ```
  + если файл `requirements.txt` не менялся, то `RUN pip install ...` будет  использовать кэш
    - Docker кэширует каждый слой (команду)
* ```dockerfile
  COPY . .
  RUN pip install --no-cache-dir -r requirements.txt
  ```
  + если `requirements.txt` не менялся, то любые изменения (python, HTML, CSS, документации) => повторня установка зависимостей
* **`PYTHONPATH`** переменная окружения - удалили
  + где Python ищет директории с модулями, кастомные библиотеки, пакеты при импорте
  + `PYTHONPATH` `export PYTHONPATH="/mnt/md0/42/14_ft_transendence/backend:$PYTHONPATH"`
* PYTHONUNBUFFERED нужна
* `COPY . .` + `volumes` ?
  + `COPY . .` включение файлов в образ на этапе сборки
  + `volumes` доступ к файлам хоста внутри контейнера
    - маскирует содержимое /app, которое было скопировано в образе
  + в разработке
    - frontend монтирует volume для статических файлов Frontend
    - используйте volume и не полагайтесь на файлы, скопированные в образе
    - `COPY . .` не нужно
    - `volumes` позволяет вносить изменения и видеть их в работающем контейнере
  + в производстве
    - backend (django + Daphne) Копирует все файлы приложения в образ Docker, собирает статические файлы и запускает Daphne
    - frrontend обслуживает статические файлы Frontend из `/static/` и статические файлы Backend из `/staticfiles/` через общий volume
    - не монтируйте volume, используйте только скопированные файлы в образе
    - монтируйте `static_backend` только для Статических Файлов 
    - `COPY . .` нужно
    - `volumes` Не требуется (код уже включен в образ с пломощью `COPY . .`)
* почему в производстве не монтировать код через volume и использовать скопированные файлы в образе Docker
  + в производственных средах придерживаются принципа **immutable infrastructure**
    - Образы являются неизменяемыми: После сборки образа он остаётся постоянным и не изменяется
    - Контейнеры запускаются на основе этих образов: Каждый запуск контейнера гарантирует, что он идентичен предыдущим
  + контейнеры, запущенные из одного образа, имеют одинаковый код, что упрощает отладку и снижает вероятность появления неожиданных ошибок
  + код внутри контейнера не зависит от файлов на хост-машине, что предотвращает несоответствия между средами разработки и производства
  + код, включённый в образ, защищён от несанкционированного доступа извне
  + меньше точек входа для потенциальных злоумышленников, так как код не доступен через volume
  + образы, содержащие весь код, самодостаточны и могут быть развернуты в любой среде без дополнительной настройки
  + Упрощённые пайплайны CI/CD: сборка, тестирование и развертываник более прямолинейны, так как весь код включён в образ
  + Docker может эффективно кэшировать слои, что ускоряет сборку и развертывание
  + монтирование volume может вносить дополнительные накладные расходы на файловую систему
  + облегчает управление версиями и откат к предыдущим стабильным версиям при необходимости
  + самодостаточные образы, не зависящие от внешних volume, развертываются быстрее и надёжнее, так как не требуют дополнительных настроек
* ./static — это папка на хосте, относительно того места, где находится Dockerfile.
* /usr/share/nginx/html/static — путь внутри контейнера
* volume staticfiles
  + Docker Volume
  + именованный том
  + могут быть использованы всеми контейнерами, которые подключены к этому тому
  + содержимое не связано с файловой системой хоста
  + volumes: - staticfiles:/app/staticfiles
    - Docker подключает том `staticfiles` к папке /app/staticfilesтом внутри контейнера
    - `/app/staticfiles` связан с томом `staticfiles
    - когда Django выполняет `collectstatic`, файлы помещаются в эту папку внутри контейнера, изменения синхронизируются с томом
  + volumes: - staticfiles:/usr/share/nginx/html/static
    - в контейнере frontend путь `/usr/share/nginx/html/static` также связан с тем же томом `staticfiles`
    - файлы, помещенные в том в одном контейнере, доступны для использования в другом 
  + `docker volume inspect staticfiles`
  + если вы хотите, чтобы том был доступен и через файловую систему хоста, вы можете использовать bind mount, а не именованный том
* Docker runtime files must be located in /goinfre or /sgoinfre (subject)
* You can’t use so called “bind-mount volumes” between the host and the container if non-root UIDs are used in the container (subject), but several fallbacks exist:
  + Docker in a VM
  + rebuild you container after your changes
  + craft **your own docker image** with root as unique UID
* virtual environment не нужно, потому что у нас докер
* настроить `sysctl`-параметры для контейнера
  + минимальные образы не содержат `/etc/sysctl.conf`, системные параметры (sysctl) ориентированы на хостовый kernel, контейнеры используют ядро хоста, не своё собственное
  + `docker run --name redis --sysctl net.core.somaxconn=1024 -d your_image`
  + или docker-compose
    ```yaml
    services:
      redis:
        image: ...
        sysctls:
          net.core.somaxconn: 1024
          net.ipv4.tcp_syncookies: 1
    ```
  + проверить значения sysctl в рантайме `cat /proc/sys/net/core/somaxconn`


### ЗАПУСК
* ASGI_APPLICATION = "myproject.asgi.application" есть, зачем CMD ["daphne", "-b", "0.0.0.0", "-p", "8000", "**myproject.asgi:application**"]
* For Ecole42 computers, I've updated settings of docker file in DEV branch 
  + порт, который нужен для django, занят
  + поменять номера портов в docker и в nginx
  + 6800  port for redis 
  + 4444 port frontend HTTP connections
  + 4443 port frontend for SSL connections over HTTPS
  + for local Ecole 42's network `ALLOWED_HOSTS = ['*']` in settings.py
* **Секретный ключ от api раз в две недели примерно обновляется**
* 'docker compose up --build'
  + все скачается и распакуется
  + потом в терминале можно Ctrl + C (условное клавиатурное прерывание)
  + пересобрать образы -> Django подхватывает изменения (если настроен **hot-reload**), фронтенд тоже
* `sudo ufw status` 80, 443, 8000 открыты
* через 42 на посте кластера, отличном от того, на котором он размещен
  + 1 способ: каждый раз менять URL-адрес перенаправления, чтобы он соответствовал IP-адресу хост-компьютера 
  + 2 саособ лучше:
    - каждый компьютер использует VM, в ней изменяете файл хостов, чтобы связать IP-адрес исходной станции с URL-адресами
    - подключиться к другому компьютеру в кластере через его внутреннее доменное имя школы, добавить это в nginx.conf, чтобы веб-сервер разрешал связываться с вами в этом домене, а также на стороне django
* from another computer (so local network is working) 
  + When Ivan tried to login with 42Auth from another computer (not server) - he got error 400; however basic sign up with email is working 
  + My login with 42Auth from server computer worked


### РАЗНОЕ
* CSR client-side rendering / SSR server-side rendering, данные загружаются на клиентскую сторону, HTML генерируется динамически с помощью js
  + SSR сервер генерирует и отправляет готовый HTML на клиентскую сторону, каждый запрос требует пересоздания всей страницы на сервере, может быть медленным, сервер должен выполнить обработку данных и сгенерировать страницу каждый раз, когда поступает запрос 
  + сервер отправляет данные (JSON, ...)
  + клиент с помощью js генерирует HTML на основе полученных данных
  + не требуется повторная генерация страниц на сервере для каждого запроса
  + it is almost front-end
  + if we do module - power-ups for AI are obligatory (so it is back end part): для модуля AI требуется бэкенд (сложная обработка данных, вычисления, доступ к серверным ресурсам)
* rendering
  + фронтенд: рендеринг HTML-шаблонов или динамически обновляемых данных через js, процесс генерации и отображения контента на веб-странице
  + Views: обработка запросов и отправка ответов в разных форматах (JSON, ...) через API
  + DRF: процесс обработки запросов и создания ответов в виде JSON, XML или других форматов
  + Django Channels: передача данных пользователю в реальном времени через протокол WebSocket
* the use of libraries or tools that provide an immediate and complete solution for a global feature or a module is rohibited (subject)
• the use of a small library or tool that solves a simple and unique task, representing a subcomponent of a global feature or module, is allowed (subject)
* список эндпоинтов
  - `python manage.py show_urls` 
  -  `grep -r "path(" backend/`, `grep -r "re_path(" backend/`
  - `curl -X GET http://localhost:8000/api/endpoint/`
  - `curl -X POST http://localhost:8000/api/endpoint/ -H "Content-Type: application/json" -d '{"key": "value"}'`
  - Postman
  - Chrome + расширения
  - Python + библиотека `websockets`
  - http://localhost:8000/api/, http://localhost:8000/ автоматичесая документация эндпоинтов **Browsable API** 
  - http://localhost:8000/swagger/, http://localhost:8000/redoc/, если настроена автоматическая документация
* pathname = часть адреса после корня
* `Uncaught SyntaxError: Unexpected token '<'`
  + `theme.js` и `nav_sidebar.js` отдаются со статусом 200, но на деле возвращают HTML
  + вместо js браузер получил от сервера HTML (страницу с ошибкой, редирект на логин, ...)
* Для 100 пользователей Transcendence можно считать небольшим или средним, в зависимости от нагрузки.  
  + **размер проекта зависит не только от количества пользователей, но и от**:
    - Частоты запросов (активные ли пользователи или заходят раз в неделю?)  
    - Типа данных (много ли храним? используем ли WebSockets?)  
    - Распределения нагрузки (одновременные соединения или распределённые по времени?)  
    - 100 пользователей играют одновременно в реальном времени — это средний уровень нагрузки  
    - 100 пользователей, но только 10-20 активны одновременно → небольшой проект  
    - 100 пользователей, но они все активно играют в реальном времени → средний проект  
    - 100 пользователей, но каждые 2 секунды генерируются сотни запросов → может быть нагруженным  
    - Если игра в реальном времени (ws + Redis Channels), Нагрузка выше, особенно если много обновлений состояния
    - Если используется кэширование (Redis как кэш), Нагрузка ниже, так как база нагружается меньше.  
    - Polling создаёт больше нагрузки, чем WebSockets.  
    - Если база данных нагружена (часто читаются и записываются данные, ...), **оптимизировать SQL-запросы и индексы**
  + минимизация нагрузки: кэширование, Channel Layers, daphne + Channels для асинхронной обработки, Nginx балансировка нагрузки
  + если код написан грамотно, то Redis, Django и PostgreSQL справятся с 100 пользователями
  + WebSockets (чат, ракетки) – Redis как Channel Layers поможет масштабировать и оптимизировать передачу данных. При 100 одновременных игроках трафик не критичен для Redis
  + PostgreSQL (профили, истории игр, турниры, сообщения) – Если запросы оптимизированы (**индексы на часто используемые поля, правильные связи**), выдержит
  + Сервер в одном инстансе – ок, нагрузка небольшая
* https://localhost:4443/ нормально открывается, а по https://localhost:4443/index.html 404
  + внутри SPA нет маршрута для пути "/index.html" — поэтому «клиентская» часть выводит надпись «404»
  + в SPA (React Router, Vue Router, ваш собственный роутер и т.д.) маршруты /, /login, /profile и т.п. могут быть прописаны в коде, но "/index.html" обычно не предусмотрен как валидный маршрут и воспринимается как «неизвестный адрес». В результате фронтенд при переходе на "/index.html" показывает экран «404 – Not Found» (или любое действие по умолчанию для неизвестных путей).
  + добавить соответствующее правило в вашем клиентском роутере (чтобы "/index.html" перенаправлялся на "/" или обрабатывался так же, как "/")
  + Сообщение "404 – Page Not Found" исходит из логики клиентского кода (SPA), потому что /index.html там не прописан
  + где-то в коде «клиента» (вашего фронтенда) прописать логику, которая:
    - Отслеживает текущий URL (например, через window.location.pathname или события popstate / hashchange).  
    - Если путь — `"/index.html"`, то вы либо  Перенаправляете браузер на "/", или Точно так же обрабатываете его, как если бы это был путь "/"
  + Nginx не должен выдавать реальную 404 на /index.html. Сейчас у вас уже настроено, что index.html физически отдается и статус запроса — 200.  
  + Если внутри app.js при загрузке вы показываете «404», значит это ваш JS решает, что «/index.html» — невалидный маршрут. Нужно добавить логику (как в примерах), чтобы считать его эквивалентным "/".  
  + При желании вы можете вообще редиректить браузер на "/", чтобы пользователь видел именно "/" в адресной строке. Либо просто обрабатывать "/index.html" так же, как "/", без изменения URL.
  + аким образом, добавить правило для "/index.html" можно прямо в ваш «клиентский роутер» (даже если он «кустарный»), переписывая путь на "/" или вручную вызывая код для главной страницы.
  + Пример 1. Моментальный редирект при загрузке
    - как только скрипт загружается, он проверяет, не находитесь ли вы на "/index.html". Если да, тут же подменяет URL на "/" (через history.replaceState) и продолжает работу.  
    - Визуально пользователь сразу же увидит "/" в адресной строке.  
    - Рендериться будет основной код, как будто пользователь зашёл на "/".
    - ```
      html
      <!-- index.html -->
      <!DOCTYPE html>
      <html>
      <head>
        <meta charset="utf-8">
        <title>My SPA</title>
      </head>
      <body>
        <div id="app"></div>
        <script src="app.js"></script>
      </body>
      </html>
      
      // app.js
      window.addEventListener("DOMContentLoaded", () => {
        // Если зашли на "/index.html", то заменяем путь на "/"
        if (window.location.pathname === "/index.html") {
          // Меняем URL (без перезагрузки) на "/"
          window.history.replaceState({}, "", "/");
        }
        // Далее ваш остальной код, который строит страницы в зависимости от location.pathname
        startSPA();
      });
      function startSPA() {
        // Простейший «роутер»:
        const path = window.location.pathname;
        if (path === "/") {
          showHomePage();
        } else if (path === "/login") {
          showLoginPage();
        } else {
          showNotFoundPage();
        }
      }
      ```
  + Пример 2. Обработка в роутере
    - принудительно считать "/index.html" за "/":
    - вы не меняете адресную строку, но показываете тот же контент, что и на "/"
    - ```
      js
      function startSPA() {
        let path = window.location.pathname;
        // Если путь /index.html, воспринимаем его как /
        if (path === "/index.html") {
          path = "/";
        }
        switch (path) {
          case "/":
            showHomePage();
            break;
          case "/login":
            showLoginPage();
            break;
          default:
            showNotFoundPage();
            break;
        }
      }
      ```
  + Пример 3. «Общий» роутер с несколькими маршрутами
    - объект routes, где ключи — это пути, а значения — функции для отрисовки
    - если человек набирает https://example.com/index.html в адресной строке, ваш JS увидит это, приравняет к "/", и покажет нужную страницу
    - ```
      js
      const routes = {
        '/': showHomePage,
        '/login': showLoginPage,
        // можете добавить и другие
      };
      // Функция, запускающая роутер
      function router() {
        let path = window.location.pathname;
        // Приравниваем /index.html к /
        if (path === '/index.html') {
          path = '/';
          // Дополнительно можно сделать history.replaceState({}, '', '/');
        }
        // Ищем обработчик, если нет — 404
        const routeHandler = routes[path] || showNotFoundPage;
        routeHandler();
      }
      // Запускаем роутер при загрузке страницы
      window.addEventListener('DOMContentLoaded', router);
      // Перехватываем «назад»/«вперёд» в браузере
      window.addEventListener('popstate', router);
      ```
* в REDIRECT_URI_42=http://127.0.0.1:8000/auth/callback **убрать 127.0.0.1:8000**
* oauth
  + без - ./.env:/app/.env
    ```
    backend  | 2025-02-08 11:13:06,348 INFO     Listening on TCP address 0.0.0.0:8000
    backend  | 2025-02-08 11:07:13,433 ERROR    code: b2576fcadb28f3cfc4ca237f216c604723996d05bf6637d4ce1308105a56d2b5
    backend  | 2025-02-08 11:07:13,433 ERROR    CLIENT_ID_42: u-s4t2ud-0f34f9a5a8eab694436a048c258fc509b67c21763329d02ade82ff3a77005eb8
    backend  | 2025-02-08 11:07:13,433 ERROR    CLIENT_SECRET_42: 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
    backend  | 2025-02-08 11:07:13,433 ERROR    REDIRECT_URI_42: http://127.0.0.1:8000/auth/callback
    backend  | 2025-02-08 11:07:13,581 ERROR    response: {'access_token': '7354455b03c5a130ac8e00e40511bd9bd820377c251f5aee91f4577c009dd733', 'token_type': 'bearer', 'expires_in': 6923, 'refresh_token': '7b8569e4268c2a9d83eb719302632647cf97eed1492eecbfd1e165a0eae748f7', 'scope': 'public', 'created_at': 1739012556, 'secret_valid_until': 1740819629}
    backend  | New user created with username: akostrik
    backend  | 172.22.0.1:49912 - - [08/Feb/2025:11:07:14] "GET /auth/callback?code=b2576fcadb28f3cfc4ca237f216c604723996d05bf6637d4ce1308105a56d2b5" 302 -
    ```
  + с - ./.env:/app/.env
    ```
    backend  | 2025-02-08 11:13:06,348 INFO     Listening on TCP address 0.0.0.0:8000
    backend  | 172.23.0.5:59248 - - [08/Feb/2025:11:23:03] "GET /api/auth/me" 200 54
    backend  | 172.23.0.5:59262 - - [08/Feb/2025:11:23:03] "GET /api/csrf-token/" 200 81
    ```

### ORGANISATION
* искать по истории коммитов и разным веткам в VS Code, установите GitLens
* https://github.com/bakyt92/14_ft_transendence
* https://docs.google.com/document/d/14zC4f2D8vdh9cYKosDQxsjWYc9aax2hPGuh8Y7CoENI/edit?tab=t.0
* https://docs.google.com/document/d/1O1r9jEdxISjMV29lZgLXWNh-bgPzSlnZ6Nr8QuyP_Jc/edit?pli=1
* Application - хранилище:
  + куки
  + local storage
  + ... storage
* basic requirements 20.01.2025 
  + All pong game part will be done by Amine? Do you need help with front-end (table, paddles, ball, some activity of JS or someone will do it?
  + Tournament, registration and matchmaking system by Alexey
  + Basic front-end will be done by Alexey? (profile page, other pages)
  + Security - probably we meet requirements by we need to validate input and follow some basic security rules on the front-end part.
* Modules (only that needs some response / comment)
  + User management - I did back-end (almost); but we need profile page with history of games, possibility to change profile data; see statistics of wins and loses - who will be responsible for this part? I can do it but I need template of js page (single page structure should be already applied) 
  + OAuth42 - almost finished by Alexey, please check that all requirements of this module are met/
  + Remote Players - will do Amine
  + LiveChat - is the proccess by me and Anna 
  + AI Opponent - will we do this module? Amine, can you implement it or you need help?
  + Game Customization Option - do we need this module? Who will do it? 
  + Multiple language supports - who will implement it? 
  + Server-side pong  - do we need this module? Is it implemented by Amine? For this module we need API for paddle, ball and other features
  + User and Game Stats Dashboards - do we need this module? Who will do it?
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
* tutorials:
  + чат tuto https://channels.readthedocs.io/en/latest/index.html
  + бэк, фронт, база данных, API https://www.youtube.com/watch?v=XBu54nfzxAQ
  + REST API на DRF в Pycharm https://blog.jetbrains.com/pycharm/2023/09/building-apis-with-django-rest-framework/
  + https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/WebSocket
  + https://docs.djangoproject.com/en/5.1/ref/contrib/auth/
  + Django Tutorial https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website
  + https://www.djangoproject.com/start/
  + https://www.django-rest-framework.org/tutorial/quickstart/
  + django https://youtube.com/playlist?list=PLA0M1Bcd0w8xZA3Kl1fYmOH_MfLpiYMRs&si=mza4MvfFeRMqgU_B
  + django https://youtube.com/playlist?list=PLA0M1Bcd0w8yU5h2vwZ4LO7h1xt8COUXl&si=ddHMMDnBVPiUuEXy
  + The Browsable API - Django REST frameworkhttps://www.django-rest-framework.org/topics/browsable-api/
  + design https://www.figma.com/design/aWDJYfDmaeCv2NKJ8bJ15n/FT_Transcendence?node-id=109-32&p=f
  + // https://miro.com/app/board/uXjVLAphyh8=/
* **DO NOT FORGET**
  + remove убрать settings.py SECRET_KEY, frontend/etc/private.key
  + close ports 8000 and 6800 
  + localhost:8000 убрать
  + `http://backend:8000` хранить в перменной окружения
  + We set DJANGO_SETTINGS_MODULE in the .env, docker-compose and Dockerfile. Then, we set it again: os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings'). This appears to protect us from forgetting to set this variable in the .env, but it seems redundant in our case. May I remove os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings')?
  + DEBUG = False
  + в продакшен отключают генерацию .map-файлов
  + 42_REDIRECT_URI=http://127.0.0.1:8000/auth/callback убртаь
  + to justify your choices during the evaluation
