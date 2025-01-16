* https://github.com/bakyt92/14_ft_transendence
* https://tr.naurzalinov.me/users/
* http://95.217.129.132:8000/
* http://95.217.129.132:8000/chat/1
* https://localhost
* http://127.0.0.1:8000/admin/login/?next=/admin/
* https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/WebSocket
* бэк, фронт, база данных, API https://www.youtube.com/watch?v=XBu54nfzxAQ
* REST API на DRF в Pycharm https://blog.jetbrains.com/pycharm/2023/09/building-apis-with-django-rest-framework/
* бэкенд
  + Django Tutorial https://developer.mozilla.org/en-US/docs/Learn/Server-side/Django/Tutorial_local_library_website
  + https://www.djangoproject.com/start/
  + https://www.django-rest-framework.org/tutorial/quickstart/
  + django https://youtube.com/playlist?list=PLA0M1Bcd0w8xZA3Kl1fYmOH_MfLpiYMRs&si=mza4MvfFeRMqgU_B
  + django https://youtube.com/playlist?list=PLA0M1Bcd0w8yU5h2vwZ4LO7h1xt8COUXl&si=ddHMMDnBVPiUuEXy
  + The Browsable API - Django REST frameworkhttps://www.django-rest-framework.org/topics/browsable-api/
* https://docs.djangoproject.com/en/5.1/ref/contrib/auth/

```
     Browser/User HTTPS               GitHub Webhook (HTTPS) 
               v                                 v
        +--------------+                 +-----------+
        | Nginx        |                 |  Nginx    |
        +------+-------+                 +-------+---+
               |                                 |
       location /static           location /webhook proxy_pass
               |      v                  v
               |  /usr/share/nginx/html/static/      
               |                                      
               |    ( location / ) proxy_pass http://backend:8000
               v
        +------+------------------------+
        | Django backend runserver 8000 |
        +------+--------+---------------+
Запросы к API/логике |        |  Веб-хуки ( /webhook ), чаты, API и т.д.
                     v        v
               +-----+--+   +-------+
               | Postgres|  |Redis  |
               +---------+  +-------+
```

+ Условное деление директорий  
 - `frontend/`: сервировка фронта: статика, конфиг Nginx
 - `backend/`: код Django, его настройки, requirements, миграции, ...
 - Разделение frontend /backend — логическая структура, не привязка к тому, что фронтенд работает в браузере, а конфиг = сервер

### frontend
* контейнер frontend
* Nginx + собранный фронт
* точка входа для пользователей
* **SSL**, статика, проксирование
* Пользователь открывает https://tr.naurzalinov.me/
  + приходит запрос к http://95.217.129.132 на контейнер Nginx, порт 443 (набираем http://95.217.129.132:8000/ **?**)
  + защищённое соединение с Nginx
  + если запрос к статическому файлу (CSS/JS/картинк/HTML), nginx отдает его из /usr/share/nginx/html/static/
  + если /, /api/…, nginx проксирует запрос на backend:8000
  + Django-представление (views.py) проверяет токен/подпись, делает git pull, docker-compose up --build -d, и т.д
  + Django обрабатывает API
    - если это данные из Django (JSON, HTML, WebSocket), проходят через **прокси** на порт 8000
  + Фронтенд при необходимости вызывает методы API (для аутентификации, получения/отправки данных)
    - эти запросы идут через Nginx на Django
  + Ответ возвращается по цепочке Django → Nginx → Пользователь/Сервис
* второй вариант: GitHub посылает webhook на https://tr.naurzalinov.me/webhook
  + nginx проксирует запрос на backend:8000/webhook
  + Django обрабатывает пуши GitHub, может сделать git pull, пересобрать образы Docker и т.д
* http://95.217.129.132 и http://95.217.129.132:8000/
* приём HTTPS/HTTP-запросы извне на порты 80/443
* отвечает на порты 80/443
* проверяет сертификаты SSL
* обеспечивает HTTPS (SSL/TLS)
* SSL/HTTPS, проксирование, статика, редиректы, вебхуки
* выступает как обратный прокси (reverse proxy) для бэкенда
* проксировать запросы на бэкенд (Django) и/или выдавать статический фронтенд
* Запросы на /api/ уходят на Django-контейнер
* остальные запросы отдаются статическими файлами фронтенда
* перенаправляет запросы к backend, проксирует API-запросы (например, http://localhost/api/...) на бэкенд
* Проксирует основные запросы к бэкенду (Django) по внутреннему адресу http://backend:8000
* Раздаёт статические файлы (CSS, JS, картинки) мз /usr/share/nginx/html/static
* клиентский код (CSS, JS, либо отдельный React/ Vue-сборщик)
* certif/ и etc/ для сертификатов, дополнительных конфигураций, скриптов, вспомогательных файлов
* static/ статические файлы фронтенда (CSS, JS, картинки и т.п.)
  + Иногда при сборке фронтенда (React/Vue/Angular) готовый билд кладётся сюда или в подпапку build/ (в зависимости от настроек).
* nginx.conf называется default.conf внутри контейнера
  + переопределяет стандартные настройки Nginx
* Копируем статические файлы (из локальной ./static) в /usr/share/nginx/html/static внутри контейнера
  + так Nginx может их раздавать напрямую по пути /static/
* копируем SSL-сертификаты в нужные места
* внутри контейнера Nginx слушает 80 и 443, а снаружи к frontend привязаны "80:80" и "443:443", это позволяет доступ извне по HTTP(80) и HTTPS(443)
* отдельный endpoint для вебхука
* строит или берёт ваши статические файлы фронтенда
* на хост-машине порт 80 (HTTP) и порт 443 (HTTPS) проброшены в контейнер
* слушает и проксирует или статику фронта (SPA), или перенаправляет /api запросы на backend:8000
* отдаёт фронтенд на http://localhost/
* в роли прокси/сервера статики
* Nginx осуществляет proxy_pass на бэкенд, т.е становится точкой входа для запросов (HTTPS/HTTP)
* четыре server{}-блока = один процесс

#### frontend/nginx.conf
* в папке `frontend` по следующим причинам:
  + рядом с Dockerfile, чтобы во время сборки он копировался внутрь контейнера
  + Обслуживание статических файлов  
   - Файлы, относящиеся к клиентской части (CSS, JS, изображения), лежат в `frontend/static/`, Nginx в контейнере фронтенда их раздаёт
   - `location /static/ { alias /usr/share/nginx/html/static/; }` логика фронта, конфигурация там же
* Сервер для tr.naurzalinov.me
  + прослушивает HTTPS-трафик на порту 443
  + Запросы к /static/ перенаправляются на локальную папку /usr/share/nginx/html/static/
  + остальные запросы проксируются на бэкенд http://backend:8000
  + proxy_set_header передаёт заголовки (IP клиента, протоколь Host, X-Forwarded-For, ...) для Django
* Прослушивает HTTP-запросы на 80 и перенаправляет на HTTPS с использованием кода 301 (постоянное перенаправление)
* If you are encountering errors after merging the two server blocks
  + server_name cannot match raw IP addresses: The server_name directive is designed for matching domain names. While you can technically include an IP address, it won't work as expected because NGINX does not use server_name for requests with an IP address; it relies on the listen directive instead
  + Conflicting rules for IP-based requests: When a request is made to http://95.217.129.132, NGINX does not use the server_name directive for matching. Instead, it matches the listen directive and uses the first server block that matches the port
  + By merging the two server_name values, requests to http://95.217.129.132 may not behave as intended because NGINX will not consider the IP address part of server_name
  + you should keep separate server blocks
* работает через HTTP/WS-протоколы
* использует эндпоинты Django для обмена данными и взаимодействия (чат, игра, профили пользователей)
* отправляет запросы к бэкенду (GET/POST, ...)
* подписывается на WebSocket-каналы для чата

### backend
* we implement game logic in backed, backend because we need it to do the multiplayer
* runserver 0.0.0.0:8000 => запустили Django-приложение
* слушает внутри контейнера на порту 8000 внутри сети Docker
* снаружи его можно вызвать на localhost:8000
* получает запросы фронтенда
* обрабатывает их: аутентификация, чат (через Channels), API для фронтенда, авторизация, API, игровая логика, аутентификация
* возвращает ответы
  + рендеринг **шаблонов** (Server-Side Rendering)
  + отдаёт JSON-данных (**REST API**)
  + отдаёт JSON/HTML
  + отдаёт websocket (Channels)
* JSON API (используя **Django REST Framework** или WebSocket-соединения через Channels)
* обращается к Postgres для чтения/записи данных (пользователи, чат, профили)
* общается с Redis для кэша, сессий, Pub/Sub (чат, обновления статусов игроков в реальном времени)
* Django Channels обрабатывает событийную логику для чатов/онлайн-игр
* myproject/
  + конфигурация всего проекта Django, настройки, корневой маршрутизатор, запуск, управление проектом
  + urls.py роутинг (ссылки) для всего проекта
  + `__init__.py`, `asgi.py`, `wsgi.py` **точки входа** Django
  + `settings.py` 
    - конфиг проекта Django на высоком уровне (middleware, установленными приложения, роутинг, шаблоны, локаль, приложения, статика, ключи, переменные окружения, параметры безопасности, подключение к БД,  ...)
    - использует среды (.env) и пакеты `decouple`, `dotenv`, чтобы пароли, **ключи OAuth**, ... не хранились в коде
    - `BASE_DIR` корневая папка проекта  
    - `STATIC_ROOT = staticfiles` куда Django или система сборки складывает статические файлы при выполнении `collectstatic` (staticfiles обычно не трекается гитом)
    - `SECRET_KEY` уникальная строка для криптографических подписей (в продакшене не выкладывают в открытый доступ) 
    - `DEBUG = True` режим отладки (отображает детальные ошибки), в продакшене `False`  
    - `ALLOWED_HOSTS` список доменов/IP, с которых разрешён доступ к этому Django-приложению (когда `DEBUG=False`)
    - `ASGI_APPLICATION` = "myproject.asgi.application" Django + Channels используют **ASGI** для работы с WebSocket и др асинхронными протоколами
    - `CHANNEL_LAYERS` для Django Channels
    - `CACHES` Django по умолчанию может кэшировать в памяти, но вы используете Redis
    - `INSTALLED_APPS` список приложений активных в проекте: стандартные (admin, auth, sessions), кастомные: `myapp`, `chat`, `api_42`, ...
    - `MIDDLEWARE` Django-посредники, обрабатывают запросы/ответы (сессии, безопасность, авторизация и т. д.)
    - `ROOT_URLCONF` главный файл роутов `myproject/urls.py`
    - `TEMPLATES` настройки для системы шаблонов Django (где HTML-шаблоны, какие контекстные процессоры использовать)
    - `WSGI_APPLICATION` точка входа для **WSGI**-серверов (Gunicorn, uWSGI, ...) если запускаете проект в синхронном режиме (или для совместимости), в сочетании с `ASGI_APPLICATION` показывает, что проект может работать и под WSGI, и под ASGI (Channels)
    - `DATABASES` используете PostgreSQL и берёте параметры (имя пользователя, ...) из `.env` (`decouple.config`), бд на сервисе `db`
    - `REST_FRAMEWORK` **рендеры**, по умолчанию JSON и Browsable API
    - `AUTH_PASSWORD_VALIDATORS` включают валидаторы сложности паролей (минимальная длина, отсутствие совпадений с логином и т.п.)
    - `LANGUAGE_CODE`, `TIME_ZONE` языка, часового пояса, включения интернационализации/локализации
    - `STATIC_URL = '/static/'` статич файлы, в связке с `STATIC_ROOT` это позволяет управлять раздачей CSS, JS, изображений
  + `urls.py` (корневой маршрут)
  + `wsgi.py` и `asgi.py` (точки входа для запуска приложения)
  + `__init__.py` (делает папку пакетом Python)
* сервис backend получает имя backend (proxy_pass http://backend:8000)
* ./backend:/app локальная backend примонтирована в контейнер как /app
  + изменения в коде сразу видны внутри контейнера (при корректной настройке)
* **`daphne`** ASGI-сервер для **Django Channels**
* manage.py django-утилита для миграций, запуска сервера, создания суперпользователя, ...
* requirements.txt: Python-библиотеки для работы бэкенда
* трекинг когда пользователь был онлайн
  + в models.py добавлю last online для индикатива
  + есть три ключевых решения 
    - через Django sessions - как только юзер делает какое либо действие, Джанго сохраняет в базу данных дату этого действия 
    - через библиотеку channels - по вебсокетам следим пользователь онлайн или нет 
    - через redis - но не понял как это работает. 
* бэкенд подключается к Redis по адресу redis:6379

### django в целом
* Бэкенд-фреймворк
* с ключевыми механиками (аутентификация, управление базой данных, админка, API), готовые функции и практики из коробки
* диктует архитектуру (приложения, модели, views, urls, ...)
* `models.py` структура данных, связь между ними (пользователи, профили, чаты, сообщения, статистика игры, ...)
* `url.py` какие функции обрабатывают какие пути, какие запросы обрабатывает по адресу endpoint
* `views.py` функции при обращении под адресу = эндпоинт, обрабатывают логику запросов (создание комнаты, отправка сообщения, обновление профиля
  + определяет функцию для обработки endpoint
* `admin.py`
* `apps.py` (регистрация приложения в Django)
* `migrations/` (миграции для моделей)
* `tests.py` (тесты для этого приложения)
* Управление пользователями и аутентификацией
  + встроенная система аутентификации и модель пользователей
  + Регистрацию и авторизацию (в том числе OAuth/SSO)
  + Разграничение доступа к страницам (приватные/публичные чаты, комнаты и т.д.)
  + Проверку токена/сессии при каждом запросе с фронтенда
* Администрирование
  + Django Admin позволяет «набросать» интерфейс для управления данными (пользователи, чаты, статистика, ...)
  + /admin встроено в django
* Безопасность и валидация
  + инструменты по защите от распространённых уязвимостей (CSRF, XSS, SQL Injection)
  + благодаря джанговским формам и сериализаторам (в связке с Django REST Framework, если вы его используете), упрощается валидация данных, приходящих от фронтенда
* на базе Python

### приложения Django app - отдельные модульные приложения внутри проекта
* api_42 авторизация через intra 42 и т.д.
* module_auth логика авторизации
* auth_app логика аутентификации
* chat (модели для сообщений, WebSocket-каналы, Django Channels)
* myapp
  + бизнес-логика пользовательских профилей, турниров, историй игр
  + модель `UserProfile`: инфо о пользователе, друзьях, аватаре, дате последней активности, ...
  + модель `Tournament`: список участников (many-to-many к `UserProfile`)
    - даёт возможность собирать игры в рамках конкретного соревнования, управлять списком участников
  + модель `Game` история матчей, кто участвовал, какой турнир, счёт, победитель
  + `friends = models.ManyToManyField("self")` пользователи могут быть друзьями 
  + расширить профили (добавить рейтинг, биографию, статистику), турнирную логику (сетка турнира, раунды), механику игр (разные типы игр):
     - добавлять поля
     - добавтиь **миграции** в соответствующие модели
* встроенное прилождение `'django.contrib.messages'`
  + встроенное приложение Django
  + API для работы с сообщениями
  + добавить в `INSTALLED_APPS`
  + добавить в соответствующий в `MIDDLEWARE`
  + интеграция уведомлений в веб-приложения, чтобы взаимодействовать с пользователем
  + для временных сообщений (flash messages)
  + сообщения пользователю **между запросами** (об успешной регистрации, показать ошибку при неверном вводе данных, о сохранении изменений)
  + по умолчанию сообщения хранятся в сессии
    - можете изменить бэкэнд для хранения сообщений
    - например, использовать куки: `settings.py`: `MESSAGE_STORAGE = 'django.contrib.messages.storage.cookie.CookieStorage'`
  + сообщения автоматически удаляются после отображения
  + уровни важности сообщений: `messages.debug`, `messages.info`, `messages.success`, `messages.warning`, `messages.error`
  + Отправка сообщений в представлении:
    ```python
    def my_view(request):
        messages.success(request, 'Your action was successful!')
        messages.error(request, 'Something went wrong.')
        return redirect('home')
    ```
  + в шаблонах сообщения можно выводить с помощью цикла
    ```html
    {% if messages %}
        <ul>
            {% for message in messages %}
                <li class="{{ message.tags }}">{{ message }}</li>
            {% endfor %}
        </ul>
    {% endif %}
    ```

### django app chat
* приложение чата с реальным временем на WebSocket
* Настройка Channels в `settings.py`:
  ```python
  INSTALLED_APPS = [
      # другие приложения
      'channels',
      'chat',  # новое приложение чата
  ]
  ASGI_APPLICATION = 'transcendence.asgi.application'
  # Настройте канал слоя (например, для разработки используется in-memory)
  CHANNEL_LAYERS = {
      'default': {
          'BACKEND': 'channels.layers.InMemoryChannelLayer',
      },
  }
  ```
* Создайте файл `asgi.py` в корне проекта
* создайте приложение "chat": `python manage.py startapp chat`
* добавьте `chat` в `INSTALLED_APPS`
* в `chat/models.py` создайте модели сообщений и комнат
  ```python
  class ChatRoom(models.Model):
      name = models.CharField(max_length=100, unique=True)
      def __str__(self):
          return self.name
  class Message(models.Model):
      user = models.ForeignKey(User, on_delete=models.CASCADE)
      room = models.ForeignKey(ChatRoom, on_delete=models.CASCADE, related_name='messages')
      content = models.TextField()
      timestamp = models.DateTimeField(auto_now_add=True)
      def __str__(self):
          return f'{self.user.username}: {self.content[:20]}'
  ```
* создайте WebSocket consumer
  + обрабатывает соединения WebSocket
  + `chat/consumers.py`:
    ```python
    class ChatConsumer(AsyncWebsocketConsumer):
        async def connect(self):
            self.room_name = self.scope['url_route']['kwargs']['room_name']
            self.room_group_name = f'chat_{self.room_name}'
            await self.channel_layer.group_add(             # Присоединиться к группе комнаты
                self.room_group_name,
                self.channel_name
            )
            await self.accept()
        async def disconnect(self, close_code):
            await self.channel_layer.group_discard(                  # Отключиться от группы комнаты
                self.room_group_name,
                self.channel_name
            )
        async def receive(self, text_data):            # Получение сообщения от WebSocket
            text_data_json = json.loads(text_data)
            message = text_data_json['message']
            await self.channel_layer.group_send(             # Отправить сообщение группе
                self.room_group_name, {
                    'type': 'chat_message',
                    'message': message
                }
            )
        async def chat_message(self, event):       # Получение сообщения от группы
            message = event['message']
            await self.send(text_data=json.dumps({            # Отправить сообщение обратно в WebSocket
                'message': message
            }))
    ```
* Настройте маршруты WebSocket
  + создайте `chat/routing.py`:
    ```python
    websocket_urlpatterns = [
        path('ws/chat/<str:room_name>/', consumers.ChatConsumer.as_asgi()),
    ]
```
* Добавьте маршруты WebSocket в `asgi.py`
  ```python
  application = ProtocolTypeRouter({
      'http': get_asgi_application(),
      'websocket': AuthMiddlewareStack(
          URLRouter(
              websocket_urlpatterns
          )
      ),
  })
  ```
* в `chat/urls.py` настройте маршруты для комнаты чата
  ```python
  urlpatterns = [
      path('<str:room_name>/', views.chat_room, name='chat_room'),
  ]
  ```
* В `chat/views.py` создайте представление
  ```python
  def chat_room(request, room_name):
      return render(request, 'chat/room.html', {'room_name': room_name})
  ```
* создайте шаблоны для чата - создайте файл `chat/templates/chat/room.html`
  ```html
  <!DOCTYPE html>
  <html>
  <head>
      <title>Chat Room</title>
  </head>
  <body>
      <h1>Room: {{ room_name }}</h1>
      <div id="chat-log"></div>
      <input id="chat-message-input" type="text" size="100">
      <button id="chat-message-submit">Send</button>
      <script>
          const roomName = "{{ room_name }}";
          const chatSocket = new WebSocket(
              'ws://' + window.location.host + '/ws/chat/' + roomName + '/'
          );
          chatSocket.onmessage = function(e) {
              const data = JSON.parse(e.data);
              document.querySelector('#chat-log').innerHTML += '<br>' + data.message;
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
      </script>
  </body>
  </html>
  ```
* `python manage.py makemigrations`, `python manage.py migrate` создайте и примените миграции для моделей 
* `python manage.py runserver` запустите сервер разработки 
* убедитесь, что сервер ASGI корректно работает
* можете расширить функциональность, добавив авторизацию пользователей, отображение истории сообщений, обработку ошибок и уведомления

### Chat
* можно ли делать live chat с библиотекой channels или надо целиком писать
  + Какие библиотеки можно использовать?
* Django Channels
  + библиотека для вебсокетов
  + надстройка для Django
  + добавить поддержку протоколов, отличных от HTTP (WebSocket, ...)
  + для приложений с функциями реального времени, выходящих за рамки стандартного HTTP-протокола: онлайн-игра, онлайн-статус пользователей
  + WebSocket - передавать данные между сервером и клиентом в режиме реального времени
  + WebSocket обеспечивает постоянное соединение между клиентом и сервером
  + подходит для чата
* **RabbitMQ**
  + брокер сообщений
  + для системных сообщений (уведомление о начале турнира, уведомление о добавлении в друзья)_
  + **нужны ли оба?** Channels library / RabbitMQ 
* чат в формате поп-ап окна сбоку
* to set up the database **для чего?**
  + `python manage.py makemigrations`, `python manage.py migrate`
  + create a superuser `python manage.py createsuperuser`
  + reload the server in Docker

### chat
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
* ws объект js
* send.msg
* receive = render msg
* ws: кому пишем сообщение? в запросе
* prepMsg забирает инпут и делает ws запрос
  + либо взять список друзей из базы и сделать выпадающие список
  + либоа поле инпут - кому сообщение
* @login_required только если залогинен, если нет токена или он неправильный - не пропустит
* отдельный endpoint проверить, существует ли такой юзер (не нужен)
* fetch() запрос обычный (не ws)
* ws один для всех пользователей для чата
* **AbsTimeUser** (непралвьно написано) наш класс наследует
* johnResponse - временный
* view.py не html
  + jsonResponse или httpResponse
* createuser встроенная, т.к. наследуем от ... 
* **[2302]** в контейнере бэк: python manage.py make migrations
* если логин - присылает session id token
* первый логин - header SCRF токен
  + посмотреть это: F12 Applocation storage cookies
  + в js фенкция post - в header токен CSRF
  + session id приязывает сессия + юзер в приложении
* сначала выполняется header, потом подгружаются стили
* CSS-OM = дереао как DOM
* incognito
* в django новая функция: python manage.py statup, migrations, INSTALLED_APS ... 'chat'
* 1 эндпоинт слушает чат
  + в заголовке запроса - кому сообщение
* если сообщение не дошло, то такого пользователя нет
* в контейнере бэк: python manage.py superuser то ли python manage.py createsuperuser, с именем админ пародлем админ
  + после этог модет быть migrations
* pathname = часть адреса после корня

### F12 concole
* лучше всего в chrome
* colsole.log для отладки
* bootstrap готовые стили
  - можно сосдавать кастомные на основе н их
* кнопка квадратик со стрелкой слево вверх - смотреть код элемента html
* js менять параметры html, class = стиль
* open chat
  + на странице login, profile, решгистрация нету
  + во время игры - статистика, другой user
  + приложение, отправлятть сообщение через js !!
  + django channel = **fr.w.** для сокетов
* front: django channel создает WS websocket, слушает запросы
  + отличия от html: непрерывное соединение, стрим
  + new Websocket(127.0.0.1:8000/game, wss)
    - wss протокол
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

### SSL (HTTPS)
* **где SSL, CSRF**
* шифровать соединение между клиентом (браузером) и сервером
* `listen 443 ssl;` сервер слушает и обрабатывает SSL/HTTPS-соединения  
* `ssl_certificate /etc/nginx/ssl/certificate.crt;`, `ssl_certificate_key /etc/nginx/ssl/private.key;` сертификат и приватный ключ для домена
  + Когда клиент (браузер) устанавливает HTTPS-соединение, Nginx использует этот сертификат и ключ, чтобы:  
    - Подтвердить подлинность **сервера** (браузер проверяет, подписан ли сертификат доверенным центром)
    - Установить канал связи (SSL/TLS) между сервером и браузером, чтобы шифровать трафик
* `ssl_protocols TLSv1.2 TLSv1.3;` версии протокола шифрования
  + старые протоколы TLS 1.1, SSLv3, ... не поддерживаются из соображений безопасности  
* `ssl_ciphers 'HIGH:!aNULL:!MD5';` использовать надежные шифры, `!aNULL` и `!MD5` исключают небезопасные варианты  
* `ssl_prefer_server_ciphers on;` браузер отдаёт предпочтение **списку шифров** сервера, а не клиента
* `proxy_set_header ...` передает заголовки (Host, IP-адрес клиента, протокол и т.д.), чтобы бэкенд понимал, откуда пришел запрос
* два SSL-контура, два набора сертификатов:
  + явно разделять несколько доменных имен/служб или когда webhook обслуживается по отдельному каналу.  
  + `etc/nginx/ssl/` для главного домена Transcendence (отдача фронта, проксирование бэкенда), для сайта (домена, где крутится Transcendence), для основного сайта/приложения
    -`etc/nginx/ssl/` Основные сертификаты для сайта (или сайта и API), для основного домена/сайта, который отрабатывает в большинстве ваших `server { listen 443 ssl; }` блоков 
    - certificate.crt, private.key, certificate.csr SSL-сертификат и приватный ключ
    - `.crt`, `.key`, `.csr` необходимы для **обеспечения HTTPS** и **шифрования соединения**
    - `nginx.conf: ssl_certificate /etc/nginx/ssl/certificate.crt; ssl_certificate_key /etc/nginx/ssl/private.key;`
    - обеспечивает HTTPS 
  + `certif/` для webhook-сервера
    - другие сертификаты для конкретного server-блока  
    - запросы от GitHub на `https://…/webhook`  
    - `webhook.csr`, `webhook.crt`, `webhook.key`
    - `openssl.cnf` конфиг OpenSSL для генерации этих файлов (**для первого сертификата такого нет**), при создании или подписывании сертификатов (например, когда генерируют CSR — Certificate Signing Request)
  + `.crt` сертификат, выданный CA (Certificate Authority) или сгенерированный самостоятельно (self-signed)  
  + `.key` приватный ключ, хранится на сервере, не разглашается, нужен для расшифровки соединения и подтверждения подлинности  
  + `.csr` запрос на подпись сертификата (Certificate Signing Request)
    - генерируете `.csr` с приватным ключом локально, отправляете `.csr` в удостоверяющий центр (Let’s Encrypt, ZeroSSL, GeoTrust, ...), получаете `.crt`  
* **файл сессии, сертификат CSRF, токен авторизации**

### authentification
* Аутентификация:
  + Пользователь вводит логин и пароль
  + Сервер проверяет данные и создаёт JWT, содержащий информацию о пользователе
  + Токен отправляется клиенту
* Для аутентификации асимметричный алгоритм (например **RSA**)
  + служба аутентификации — единственная, которая хранит закрытый ключ и может выдавать токены
  + Токены содержат имя пользователя, которого они идентифицируют
  + другие микросервисы могут проверять их с помощью открытого ключа и могут доверять данному имени пользователя, если подпись проверена
  + Самый простой и самый безопасный — **cookie**
    - Поскольку все микросервисы имеют одно и то же имя хоста, файл cookie будет автоматически отправляться браузером при каждом запросе
    - Со стороны DRF у вас есть расширение для JWT, но мне не удалось заставить его работать с RSA, было проще создать собственное промежуточное программное обеспечение для аутентификации
  + https://dzone.com/articles/using-jwt-in-a-microservice-architecture

### autorisation
* При каждом запросе клиент отправляет токен в заголовке Authorization: Bearer <token>
* Сервер декодирует токен, проверяет его подпись, использует данные для авторизации (например, проверяет роль пользователя)
* виды авторизаций: Ecole 42, гугл, логин и пароль с базы данных
* задача на бэкэнде
* разбираться с нюансами **DRF** и с фронтом
* внедрить 2FA через электронную почту
  + Мы выполнили первую часть, генерируя код и все такое
  + были проблемы с последней частью, где мы на самом деле отправляем электронное письмо
    - попробовали sendgrid life, но, не тратя никаких денег, было сложно добиться успеха в отправке электронных писем
  + мы использовали Gmail, самое сложное — найти путь через пользовательский интерфейс Google для создания «mdp приложения» на стороне Django, все легко делается с помощью функции send_mail
    - сначала пришло как спам, а потом появилось само по себе после нескольких писем
    - со стороны Django это происходит, Gmail изменил условия две недели назад, с тех пор это стало проблемой
* CSRF (Cross-Site Request Forgery) token
  + отдать токен клиенту при первом запросе
  + потом клиент будет слать запросы к api и вставлять этот токен в заголовок запроса 
  + в рамках авторизации делать, и видимо при post / get запросах?
  + токен прописан в settings.py
  + views.py: набор проверок которые нужно добавить при запросе к api
  + https://docs.djangoproject.com/en/5.1/howto/csrf/ использование CSRF с Django
  + лучше использовать **JWT** и тогда не будет необходимости в CSRF-токене
* JWT (JSON Web Token)
  + формат токена для передачи данных между сторонами в безопасном и компактном виде
  + Base64-строка => лёгкий для передачи через URL, заголовки, ...
  + для аутентификации, авторизации, обмена информацией в веб-приложениях
  + 1 часть Header: метаинформация о токене (тип токена,алгоритм подписи, ...) `{ "alg": "HS256", "typ": "JWT" }` 
  + 2 часть Payload = полезная нагрузка: данные, которые нужно передать (user ID, роль пользователя (admin, user), срок действия токена, ...): `{ "sub": "1234567890", "name": "John Doe", "admin": true, "exp": 1672531200 }`
  + 3 часть Signature: подписывается с использованием алгоритма из заголовка (например, HS256) и секретного ключа => целостность и защита от изменения токена: `HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret )`
  + Пример
    ```
    eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
    .eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
    .SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
    ```
  + Безопасность: Использует подпись, чтобы предотвратить изменения данных
  + Самодостаточность: Все данные для проверки токена уже содержатся в нём, нет необходимости хранить их на сервере
  + Масштабируемость: Подходит для микросервисов, так как не требует централизованного хранилища сессий
  + Использование
    - Веб-приложения: Для аутентификации пользователей
    - API: Для обеспечения безопасного доступа к API-ресурсам
    - Мобильные приложения: Вместо сессий или cookie
  + Недостатки
    - Если токен украдён, злоумышленник может использовать его до истечения срока действия.
    - Отзыв токенов сложен, так как они самодостаточны (нет централизованного хранилища).
  + Для снижения рисков:
    - Используйте HTTPS
    - Устанавливайте короткий срок действия токена (exp)
    - Реализуйте механизмы обновления и отзыва токенов
* **JWT** в серверной службе DRF
  + удалось добавить несколько токенов, следуя официальной документации
  + возникли трудности с вложением их в nginx сервер, который служит **API-шлюзом/обратным прокси-сервером**
  + документация по реализации службы аутентификации как части микросервисной архитектуры?
    - https://django-rest-framework-simplejwt.readthedocs.io/en/latest/getting_started.html
    - https://openclassrooms.com/fr/courses/7192416-mise-en-place-une-api-avec-django-rest-framework/7424720-give-access-with-the-tokens
  + nginx должен быть прозрачным в управлении jtwt
    - он ничего не делает, кроме пересылки запросов
    - хранить jwt: наиболее безопасным способом, не сохраняя его в cookie, а в **локальном хранилище** (только в переменной в js, и мы обновляем как только он истечет)

### db (PostgreSQL)
* СУБД (база данных) для хранения пользователей, сообщений, данных о матчах в Pong, статистики и т.д.
* Создаёт базу данных mydatabase с логином/паролем myuser / mypassword
* том db_data, чтобы не терять БД при перезапуске
* доступен внутри сети Docker по адресу db:5432

### контейнер Redis (Remote Dictionary Server)
* инструмент управления данными
* может использоваться:
  + Кэширование данных: Хранение временных данных для ускорения работы приложений, хранение кэш, кэширование с гибким управлением временем жизни данных (TTL) (Django-кэш / Celery кэш)
  + Репликация и кластеризация для масштабирования и высокой доступности
  + Сессии: Управление пользовательскими сессиями в веб-приложениях
  + Очереди сообщений: Использование механизма списков или Pub/Sub для реализации очередей
  + Лидеры и рейтинги: Упорядоченные множества используются для хранения рейтингов в реальном времени
  + Анализ данных: Быстрый подсчёт статистики, временных данных
  + для хранения часто запрашиваемых данных, чтобы не обращаться каждый раз к основной базе (Хранение информации о пользователях (имя, аватар, статус) для быстрого доступа. Кэширование результатов сложных SQL-запросов.)
  + Реализация чата (Совместно с WebSocket для быстрой обработки сообщений. В Pub/Sub позволяет рассылать сообщения всем пользователям чата.)
  + для хранения данных сессий, потому что он работает в оперативной памяти и обеспечивает быструю запись и чтение. (Хранение данных о текущих авторизованных пользователях. Управление сессиями при подключении пользователей к WebSocket.)
  + может работать как очередь задач (Очередь отправки email, push-уведомления. Логирование или обработка фоновых задач (например, обновление статистики матчей))
  + позволяет задавать время жизни (TTL) для хранимых данных, что делает его полезным для временных данных. (Ограничение количества запросов от одного пользователя за определённое время (rate limiting). Хранение временного статуса пользователя: онлайн/офлайн.)
  + для матчей в реальном времени (Хранение текущего состояния игры. Обмен данными между игроками. Хранение лидербордов с помощью упорядоченных множеств (sorted sets))
  + база данных
  + брокер сообщений
  + быстрый обмен сообщениями
  + очереди
  + хранение Pub-Sub
  + Pub/Sub (для реального времени)
  + Pub/Sub для чата (если используете Django Channels, Redis часто выступает как канал-сервер)
  + Поддержка Pub/Sub (механизм публикации/подписки) для обмена сообщениями
  + Pub/Sub в реальном времени (чат, игра Pong)
  + Хранение сессий (если Django настроен использовать Redis для session storage)
  + бекенд для Pub/Sub и WebSocket-сессий (чат, игра в реальном времени, ...)
  + Pub/Sub для чата (если используете Django Channels, Redis = канал-сервер)
  + для реализации временных и динамических данных
* у нас для поддержки системы каналов через Django Channels
  + реализация реального времени через Django Channels (WebSocket)
  + канальный слой
  + промежуточный уровень (channel layer) для обработки событий в реальном времени
  + Коммуникации между разными инстансами приложения
  + инструмент маршрутизации между подключёнными клиентами
  + хранилище сообщений
  + Обработка событий WebSocket
  + Django Channels позволяет приложению обрабатывать протоколы реального времени (WebSocket, ...)
  + Когда клиент отправляет сообщение через WebSocket, оно сначала сохраняется в Redis, после чего перенаправляется другим клиентам или серверу в зависимости от логики
  -
  ```python
  CHANNEL_LAYERS = {
      "default": {
          "BACKEND": "channels_redis.core.RedisChannelLayer",
          "CONFIG": {
              "hosts": [('redis', 6379)],
          },
      },
  }
  ```
* Доступен по адресу redis:6379 в сети
* in-memory key-value store
* значения могут быть: string, list, hash, set, sorted sets, bitmap, HyperLogLogs, геопространственные индексы и др  
* простой интерфейс
* работает в оперативной памяти => высокая скорость
  + может сохранять данные на диск, чтобы избежать потери при сбоях
* поддерживает множество языков программирования
* Транзакции и скрипты на языке Lua
* у нас тиспользуется для кэширования данных  
  + чтобы ускорить доступ к часто используемым данным и снизить нагрузку на базу данных
  + хранит кэшированные данные в оперативной памяти
  + Django использует этот кэш для данных, которые часто запрашиваются, но редко изменяются
  +
  ```python
  CACHES = {
      'default': {
          'BACKEND': 'django_redis.cache.RedisCache',
          'LOCATION': 'redis://redis:6379/1',  # Используется хост redis, Указывает, где Redis запущен (хост `redis`, порт `6379`)
          'OPTIONS': {
              'CLIENT_CLASS': 'django_redis.client.DefaultClient', # класс клиента, который используется для взаимодействия с Redis
          },
      }
  }
  ```
  + это полезно для
    - ускорения запросов: вместо того чтобы запрашивать данные из базы каждый раз, результаты сложных вычислений можно кэшировать
    - кэширование сеансов: хранилище сеансов для авторизованных пользователей
    - кэширование часто используемых API-ответов: например, кэширование данных профиля пользователя или данных статистики
* почему Redis
  + высокая производительность: данные хранятся в оперативной памяти
  + пддержка Pub/Sub (публикации/подписки) => идеальный для каналов WebSocket
  + простоте настройки: легко интегрируется с Django через библиотеки, такие как `channels_redis` и `django-redis`

### подключить css
* статические файлы Django (table.css) должны быть настроены для загрузки через тег {% static %}
* в начале шаблона (base.html) {% load static %}
  + Без этой строки Django не сможет корректно обработать ссылку {% static ... %}
* backend/settings.py
  ```
  STATIC_URL = '/static/'
  STATICFILES_DIRS = [
      os.path.join(BASE_DIR, '../frontend/static'),  # Путь к статическим файлам
  ]
  ```
* `python manage.py collectstatic` файлы будут скопированы в backend/staticfiles
* `python manage.py findstatic css/popUpChat.css`
* `python manage.py runserver`
* В urls.py добавьте:
  ```
  from django.conf import settings
  from django.conf.urls.static import static
  urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
  ```
* Убедитесь, что сервер Django может обслуживать статические файлы. Для локальной разработки добавьте в urls.py обработку статических файлов:
   ```
  from django.conf import settings
  from django.conf.urls.static import static
  if settings.DEBUG:
      urlpatterns += static(settings.STATIC_URL, document_root=settings.STATICFILES_DIRS[0])
  ```
* Если вы используете Docker, убедитесь, что папка frontend/static доступна контейнеру Django. Для этого в docker-compose.yml добавьте frontend/static как volume:
  ```
  backend:
    volumes:
      - ./backend:/app
      - ./frontend/static:/app/frontend/static
  ```
* to activate a virtual environment
* http://localhost:8000/static/css/table.css
* настройки Django
  ```
  INSTALLED_APPS = [ ...  'django.contrib.staticfiles', ... ]
  STATIC_URL = '/static/'
  
  # Если вы используете нестандартную структуру (например, папка статических файлов во фронтенде), пропишите её здесь. Например:
  import os
  BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
  STATICFILES_DIRS = [
      os.path.join(BASE_DIR, '..', 'frontend', 'static'),  # путь до папки frontend/static
  ]
  # STATICFILES_DIRS = родительская директория, которая содержит статические файлы
  ```
* В режиме разработки (DEBUG = True), статика обслуживается самим Django
  + обычно достаточно просто запустить python manage.py runserver
* В продакшене (DEBUG = False)
  + настроить раздачу статики через collectstatic и веб-сервер Nginx
  + сделать collectstatic
  + настроить сервер для раздачи статических файлов
* Инструменты разработчика (DevTools) → вкладка Network
  + какой путь к CSS-файлу пытается загрузить браузер
  + какой код ответа (200 или 404)
    - если 404, то Django не может найти нужный файл
    - Убедитесь, что конечный URL, по которому файл запрашивается, совпадает с расположением файла
* Иногда, если CSS-файл уже кэшировался, браузер может «залипать» на устаревшей версии
  + проверьте, обновляется ли версия CSS (можно добавить некий ?v=123 в конце ссылки или очистить кэш браузера)
* бэкенд или фронтенд, зависит от архитектуры вашего проекта и того, как именно вы собираетесь отдавать статику (CSS, JS, картинки). нередко делают так:
  + Отдельный фронтенд (React/Vue/Angular) со своей сборкой:
    - логика, стили и ресурсы (в том числе CSS) хранятся и собираются во фронтенде
    - результат сборки (папка build или dist) может отдаёся отдельным сервером (Nginx), либо «прокидываться» через бэкенд, если вы используете Django для статики (но чаще это делают через Nginx)
    - фронтенд — отдельное SPA (React/Vue)
    - в бэкенде только делаете API (Django REST или Channels)
    - CSS лежат в папке фронта
    - после сборки CSS окажется в итоговых бандлах (или отдельных .css файлах)
    - Django будет их отдавать или (чаще) их будет отдавать Nginx
    - бэкенд отдает данные (API)
    - в корне репозитория есть frontend/ (с package.json, src/, node_modules/ и т.п.)
    - отдельное фронтенд-приложение (React, Vue и т.д.)
    - CSS будет находиться внутри этой папки
    - а Django лишь отдаёт или проксирует статику (или вы используете Nginx)
    - settings.py: STATIC_URL = '/static/', STATICFILES_DIRS = [ # ... ]: статика берётся из папки frontend/dist или build
    - фронтенд собирается отдельным билдом, Django подхватывает статические файлы оттуда
    - в Django-шаблоне либо нет упоминания о стилях, либо там просто <div id="root"></div> (для React)
    - стили «приходят» с фронтенда (собранный бандл)
    - Запустите приложение, откройте DevTools в браузере, Network, запрос к файлу .css: http://localhost:3000/... = отдельный дев-сервер фронтенда  
  + Monolith (Django + шаблоны):
    - не используете отдельный фреймворк (React/Vue и т.п.)
    - шаблоны на Django
    - CSS-файлы, картинки, .... в Django-приложении в папке static/
    - Django настроен, чтобы обслуживать файлы из директории static/ (или директорий, указанных в STATICFILES_DIRS)
    - в шаблонах подключаете статику через {% load static %} и href="{% static 'css/myfile.css' %}"
    - Django рендерит HTML-шаблоны
    - нет отдельного SPA
    - CSS проще хранить в backend/static/css/, backend/webapp/static/webapp/css/, ... и подключать через {% static %}
    - не делаете отдельный фронтенд-фреймворк
    - храните CSS в static/ внутри Django-приложения
    - вся верстка и стили лежат прямо в Django-приложении
    - только папка backend/, внутри которой есть папка templates/ и static/
    - settings.py: пути типа os.path.join(BASE_DIR, 'static')
    - статика, включая CSS, лежит на стороне Django
    - в backend/templates/ есть файлы с конструкциями вроде {% load static %} и <link rel="stylesheet" href="{% static 'css/style.css' %}">
    - стили находятся в Django-статике (в папке static/)
    - Запустите приложение, откройте DevTools в браузере, Network, запрос к файлу .css: http://localhost:8000/static/css/... =  раздачей занимается Django
* CSS-файлы и другая статика могут располагаться в разных местах
  + некоторые стили автоматически «приносят» сторонние пакеты (например, Django Admin, Django REST Framework)
  + некоторые — это ваши собственные файлы (например, bootstrap.min.css во фронтенде)
  + когда вы ставите Django, вместе с ним из коробки идёт Django Admin и стандартные CSS-файлы для админки
    - Django складывает их в staticfiles/admin/… (или admin/…)
    - при выполнении collectstatic они копируются туда автоматически
  + Django REST Framework (DRF) имеет свои статические файлы для browsable API или других админ-панелей
    - при установке и использовании DRF стили (CSS, JS, изображения) могут оказаться в rest_framework/...
    - это автоматически подтягивается по `python manage.py collectstatic`
  + Ваши собственные стили (frontend)
    - frontend/static/css
    - например, bootstrap.min.css или ваши собственные файлы
    - Если используете React/Vue/Angular — тогда вообще всё может лежать в папке фронтенда, и при сборке результирующие файлы уходят в папку билда (например, build/static/css/...)
  + `python manage.py collectstatic` Django собирает все статические файлы, включая админку и DRF, в общую директорию сборки
    - по умолчанию STATIC_ROOT
    - Эта директория может называться backend/staticfiles/ (в зависимости от настроек settings.py)
    - внутри этой директории подпапки для каждого приложения (admin, rest_framework, ...) и для ваших собственных стилей, если вы тоже настроили STATICFILES_DIRS
  + Django складывает их либо в подпапках staticfiles/…, либо хранит их отдельно, пока вы не соберёте их в одну директорию
  + разные пакеты = разные CSS
    - каждый пакет мог содержать свои статические ресурсы, не мешаясь с чужими
  + Автоматическая сборка — Django умеет автоматически находить все static/ директории в установленных приложениях и «складывать» их в финальную папку (часто называемую staticfiles или то, что указано в STATIC_ROOT)
  + Архитектурное разделение — если у вас, к примеру, есть ещё и отдельный фронтенд (React), вы можете хранить файлы фронтенда отдельно и вообще отдавать их Nginx-ом без участия Django
  + когда приложение запускается, самое главное — чтобы все нужные CSS-файлы находились/отдавались там, где ожидает браузер
  + физически эти файлы могут лежать хоть в десяти разных местах — при правильных настройках статики Django/Nginx это не проблема
  + Django/collectstatic работает так, чтобы все статические файлы из разных приложений можно было собрать в одном месте или отдать по правильным путям
  + настроить раздачу этих файлов (через Django в DEV, через Nginx или другую службу в PRODUCTION)

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

### index.html
* `{% extends "base.html" %}` расширение базового шаблона
  + Шаблон наследует структуру из базового файла base.html
  + всё из ({% block body %}) будет вставлено в base.html на место тега `{% block body %}`
*  ```
   <div class="container mt-5">                              // контейнер с отступом сверху
    <h1 class="text-center">Привет из Django!</h1>
    <p class="text-center">База данных: {{ db_version }}</p> // Шаблонная переменная {{ db_version }}, значение передаётся в шаблон из представления Django (view)
   </div>
   ```
* `<a href="#" class="btn btn-primary">Нажми меня</a>`  Класс btn btn-primary задаёт стиль кнопки
* ```
  {% if user.is_authenticated %}                             // Шаблонное условие
    <p>Пользователь: <strong>{{ user.username }}</strong> (авторизован)</p> // user - Стандартная переменная, доступная в шаблоне, содержащая информацию о текущем пользователе
  {% else %}
    <p>Вы не авторизованы. Пожалуйста, <a href="/login/">войдите</a> в систему.</p>
  {% endif %}
  ```
* **Карточка Bootstrap**  

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

### Organisation
* https://miro.com/app/board/uXjVLAphyh8=/
* https://docs.google.com/document/d/1O1r9jEdxISjMV29lZgLXWNh-bgPzSlnZ6Nr8QuyP_Jc/edit?pli=1
* У нас на всю линейку продуктов от JetBrains бесплатная студенческая лицензия
  + для фронта WebStorm
* git
  + создать dev ветку и работать с ней, потом смерджить в main
  + в dev-ветке создавать суб-ветки под каждую фичу или фикс
    - git checkput -b feature/backend/add_user_model
    - в этой ветке работать именно над конкретной задачей и оформлять в пул реквест.
  + не пушить код большими и хаотичными кусками
  + we do not "git push" directly to main branch - only push to your own branch and after check (we have no tests so it is just double check from your side) - merge your branch with main branch
  + https://befitting-chili-056.notion.site/Branch-management-171a80978370805f8faedeadcb86e35d?pvs=4 
* git
  + клонировать dev
  + `git checkout dev` переключитесь в dev
  + `git checkout -b an` создайте новую ветку для ваших изменений
  + Теперь вы можете работать над своим кодом, добавлять, изменять и удалять файлы в ветке an
  + `git add .`
  + `git commit -m "Добавляет функционал ..."`
  + `git push origin an`
  + Перейдите на GitHub и найдите вашу ветку an
  + Создайте Pull Request для слияния ветки an с dev
    - Добавьте описание изменений в PR, чтобы команда могла понять, над чем вы работали
    - Дождитесь проверки и одобрения от других участников команды
  + `git branch -d an` можете удалить локальную ветку
* the password for the django admin panel ...
* the docker extension in vscode
  + you can launch bash inside a container from gui
* web sockets
  + Django Channels стандартное средство реализации веб-сокетов на базе Django
  + не запрещено в сабжекте
  + реализация собственного асинхронного слоя может быть проблемой
* frontend:
  + Bootstrap toolkit
  + игра сама, картинка
  + assessibility
  + multiple language

### запуск проекта
* Секретный ключ от api раз в две недели примерно обновляется
* 'docker compose up --build'
  - все скачается и распакуется
  - потом в терминале можно Ctrl + C (условное клавиатурное прерывание)
* `sudo ufw status` убедитесь, что 80, 443, и 8000 открыты
* https://localhost:8000 проверка бэкенда
  + 502 Bad Gateway = Nginx не смог подключиться к бэкенду или получил от него некорректный ответ
* `curl http://localhost:8000`
* `docker exec -it git-backend-1 ss -tuln` проверить порты бэкенда
  + `tcp LISTEN 0 511 0.0.0.0:8000 0.0.0.0:*`
* `docker exec -it git-frontend-1 curl http://git-backend-1:8000` подключиться к бэкенду из контейнера
* `docker logs git-backend-1`
* `docker logs git-frontend-1`
* конфигурация Nginx
  + proxy_pass указывает на git-backend-1 и 8000
* заставить работать соединение через 42 на посте кластера, отличном от того, на котором он размещен
  + один способ: каждый раз менять URL-адрес перенаправления, чтобы он соответствовал IP-адресу хост-компьютера 
  + лучшее решение, позволяющее избежать хлопот, особенно с учетом ограничений школьных компьютеров: пройти через виртуальную машину и каждый компьютер использует виртуальную машину, и в ней вы изменяете файл хостов, чтобы связать IP-адрес исходной станции с URL-адресами в бизнесе/проде у вас есть доменные имена, так или иначе связанные, так что это не проблема в вашей голове но я думаю, что вы можете подключиться к другому компьютеру в кластере через его внутреннее доменное имя школы соответствующее тому же названию должности, которую вы используете в штатном расписании но вам придется добавить это в свою конфигурацию nginx, чтобы ваш веб-сервер разрешал связываться с вами в этом домене, а также на стороне django
* в школе
  + порт, который нужен для django, занят
  + поменять номера портов в docker и в nginx

### разное
* Vanilla JS (без фреймворков)
  + фронт реализован самописно или с использованием минимальных библиотек (jQuery, Bootstrap, ...), но не на SPA-фреймворке
* DPR, Data Processing and Rendering (Обработка и отображение данных) - данные (JSON, ...) обрабатываются на серверной стороне с помощью Django и затем передаются в представление для рендеринга на клиентской стороне с использованием шаблонов и JavaScript **у нас нет?**
* нет ясности, как между собой взаимодействовать частям приложения
  + нет четкого контракта между фронт/бэк частями
  + отсюда неясности и с моделями в джанге: какие данные нужны, как их обрабатывать и отдавать
* в модели пользователя сохраняю аватарки
  + у фронт энда есть требования какие нибудь к аватаркам или они могут сами отрисовывать?
  + вдруг пользователь сохранит свою фотографию 1000 х 1000 пикселей, на фронте вы сможете сами отрисовать аватарку ?
* с точки зрения реализации api есть class based views (не сильно сложнее) / functions based views (проще)
* используем ли мы стандартную модель пользователя с кастомизацией или используем свою модель
  + от этого зависит будущая авторизация и как авторизация / permissions будут работать
  + Если стандартную модель - надо перезагрузить все примененные миграции и таблички
* Используем стандартные структуры юзера для авторизации и для моделей данных
* Шаблон = супер базовый html файл с полями кнопками, без разметки
* вы должны предупредить клиента, что его ждет игра, даже если он в чате
* валидация данных
  + js на фронте читает инпут, проверяет с помощью regex
  + функция make password django шифрует на сервере ?
  + бэк ещё раз валидирует (например, проверяет пароль и почту) 
* update_docker.sh автоматизирует обновление и пересборку Docker-контейнеров (например, docker-compose pull, docker-compose build, docker-compose up -d)
* docker-compose up --build (или update_docker.sh)
  + чтобы пересобрать образы -> Django подхватывает изменения (если настроен hot-reload), фронтенд тоже
* change the post requests to a websocket. The idea was to make the game and chat using websockets (native browser api), it’s beneficial in terms of continuous data streaming
* `frontend/static/app.js`
  + вызывает fetch к бэкенду и затем динамически обновляет DOM, это не делает приложение SPA-фреймворком — это обычная логика на чистом JS
  + связь между бекендом и фронтендом
  + получать информацию о пользователе (имя, статус, аватар, результаты игр)
  + динамическое обновления пользовательского интерфейса с использованием данных, полученных от бэка 
  + `fetch('http://backend:8000/api/user-data/')`: HTTP-запрос к эндпоинту бекенда `/api/user-data/`, получить данные JSON о пользователе
  + `response => response.json()` конвертирует ответ JSON от сервера в объект JavaScript
  + после получения данных пользовательский интерфейс (UI) (отображение имени пользователя, аватарка, настройки) обновляется без перезагрузки страницы
  + `http://backend:8000` лушче хранить в перменной окружения
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
* вы должны предупредить клиента о том, что его ждет игра, даже если он в чате     ```
* single-page (SPA), сайт одностраничный, один html файл
  + этот html меняется с помощью js
  + не надо: server side rendering подход
  + не надо: django html - server-side code
  + не надо: в завимисости от какого-то условия, показываем или нет какие-то части страницы
  + надо: js
  + надо: код внутри {} исполняется в django, если условие выполняется (например, есть есть юзер), он выполняет и заново отправляет html

### to do
* pop-up windows : login, chat, profile
* страница comptetition, profile, настройки
* написовать стол, чтобы реагировал на клавиатуру и мышку
  + потом туда подсоединим вебсокеты, базовый дизайн
* Б
  + структуры данных
  + Django REST Framework
  + модуль с сокетами и чатом (live chat, уведомления о турнирах, и проч)
  + API, эндпоинты и методы для API
  + live chat
  + **rabbitMQ модуль сообщение от сервера**
* Л
  + авторизация
  + фронт-энд
  + шаблон для фронт энда
* Ан
  + фронт-энд
  + накидать в Figma шаблоны страничек (часть есть в Миро) страницы: страница с логином, с самой игрой (пока без игры), профиль пользователя, страница с турниром
  + писать код, делать шаблончики  
  + обсудить с Л. структуры страничек, API с бэкэнда
* Амин
  + вебсокеты для модуля remote players
    - обмен информацией между игроками и сервером о локации ракетки и мяча
  + логика игры 

### My questions
* **убрать** settings.py SECRET_KEY, frontend/etc/private.key
* backend/webhook/migrations/__init__.py пустой
