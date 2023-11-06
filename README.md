# KITTYGRAM

 Kittygram — социальная сеть для обмена фотографиями любимых питомцев. Это полностью рабочий проект, который состоит из бэкенд-приложения на Django и фронтенд-приложения на React.

https://yandexkitty.ddns.net

# Техногогии 
-Python 3.10.12

-Django3

-Nginx

-Gunicorn

-React

-djangorestframework

-Certbot

## Клонирование проекта с GitHub на сервер

```bash
git clone git@github.com:ваш_аккаунт/название_репозитория.git
```

## Настройка бэкенд-приложения

1. Установите зависимости из файла requirements.txt:
```bash
cd название_проекта/backend/

python3 -m venv venv

source venv/bin/activate

pip install -r requirements.txt
```
2. Выполните миграции и создайте суперюзера из директории с файлом manage.py:
```bash
# Примените миграции.
python3 manage.py migrate
# Создайте суперпользователя.
python3 manage.py createsuperuser
```
3. Добавьте в список ALLOWED_HOSTS внешний IP сервера, 127.0.0.1, localhost и домен:
```bash
ALLOWED_HOSTS = ['xxx.xxx.xxx.xxx', '127.0.0.1', 'localhost', 'ваш_домен']
```
4. Откройте файл settings.py и отключите режим дебага для бэкенд-приложения:
```bash
from pathlib import Path
...
# Вот тут нужно поменять значение с True на False.
DEBUG = False
...
```
5. Подготовьте бэкенд-приложение для сбора статики. В файле settings.py укажите директорию, куда эту статику нужно сложить. Через редактор Nano откройте файл settings.py,
укажите новое значение для константы STATIC_URL, добавьте константу STATIC_ROOT и
сохраните изменения в файле:
```bash
# Замените стандартное значение 'static' на 'static_backend'.
STATIC_URL = 'static_backend'
# Укажите директорию, куда бэкенд-приложение должно сложить статику.
STATIC_ROOT = BASE_DIR / 'static_backend'
```
6. Соберите статику бэкенд-приложения:
```bash
python3 manage.py collectstatic
```
7. Скопируйте директорию static_backend/ в директорию /var/www/название_проекта/:
```bash
sudo cp -r путь_к_директории_с_бэкендом/static_backend /var/www/название_проекта
```
## Настройка фронтенд-приложения

Находясь в директории с фронтенд-приложением:
```bash
npm i
npm run build
sudo cp -r путь_к_директории_с_фронтенд-приложением/build/. /var/www/имя_проекта/
```
## Установка и настройка WSGI-сервера Gunicorn
1. Подключитесь к удалённому серверу, активируйте виртуальное окружение
бэкенд-приложения и установите пакет gunicorn:
```bash
pip install gunicorn==20.1.0
```
2. Перейдите в директорию с файлом manage.py, и запустите Gunicorn:
```bash
gunicorn --bind 0.0.0.0:8000 backend.wsgi
```
3. Создайте файл конфигурации юнита systemd для Gunicorn в директории
/etc/systemd/system/. Назовите его по шаблону gunicorn_название_проекта.service:
```bash
sudo nano /etc/systemd/system/gunicorn_название_проекта.service
```
4. Подставьте в код из листинга свои данные, добавьте этот код без комментариев в файл
конфигурации Gunicorn и сохраните изменения:
```bash[Unit]
# Это текстовое описание юнита, пояснение для разработчика.
Description=gunicorn daemon
# Условие: при старте операционной системы запускать процесс только после того,
# как операционная система загрузится и настроит подключение к сети.
After=network.target
[Service]
# От чьего имени будет происходить запуск.
User=имя_пользователя_в_системе
# Путь к директории проекта.
WorkingDirectory=/home/имя_пользователя/папка_с_проектом/папка_с_файлом_manage.py/
# Команду, которую вы запускали руками, теперь будет запускать systemd.
# Чтобы узнать путь до Gunicorn, воспользуйтесь командой which gunicorn.
ExecStart=/.../venv/bin/gunicorn --bind 0.0.0.0:8000 backend.wsgi:application
[Install]
# В этом параметре указывается вариант запуска процесса.
# Значение <multi-user.target> указывает, чтобы systemd запустил процесс
# доступный всем пользователям и без графического интерфейса.
WantedBy=multi-user.target
```
Команда sudo systemctl с параметрами start, stop или restart запустит, остановит
или перезапустит Gunicorn. Например, вот команда запуска:
```bash
sudo systemctl start gunicorn_название_проекта
```
Чтобы systemd следил за работой демона Gunicorn, запускал его при старте системы
и при необходимости перезапускал, используйте команду:
```bash
sudo systemctl enable gunicorn_название_проекта
```
## Установка и настройка веб- и прокси-сервера Nginx
1. Установите, запустите и обновите настройки nginx:
```bash
sudo apt install nginx -y
sudo systemctl start nginx
sudo nano /etc/nginx/sites-enabled/default
```
…очистите содержимое файла и запишите новые настройки:
```bash
server {
 listen 80;
 server_name ваш_домен;
 location /api/ {
 proxy_pass http://127.0.0.1:8000;
 }

 location /admin/ {
 proxy_pass http://127.0.0.1:8000;
 }
 location / {
 root /var/www/имя_проекта;
 index index.html index.htm;
 try_files $uri /index.html;
 }
}
```
2.Сохраните изменения в файле, закройте его и проверьте на корректность:
```bash
sudo nano /etc/nginx/sites-enabled/default
```
3.Перезагрузите конфигурацию Nginx:
```bash
sudo systemctl reload nginx
```
## Настройка файрвола ufw
```bash
# 80, 443: с ними будут работать пользователи, делая запросы к приложению.
sudo ufw allow 'Nginx Full'
# 22: нужен, чтобы вы могли подключаться к серверу по SSH.
sudo ufw allow OpenSSH
sudo ufw enable
```
## Получение и настройка SSL-сертификата
1. Находясь на сервере, установите certbot, если он ещё не установлен:
```bash
# Установка пакетного менеджера snap.
sudo apt install snapd
# Установка и обновление зависимостей для пакетного менеджера snap.
sudo snap install core; sudo snap refresh core
# Установка пакета certbot.
sudo snap install --classic certbot
# Обеспечение доступа к пакету для пользователя с правами администратора.
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```
2. Запустите certbot и получите SSL-сертификат. Для этого выполните команду:
```bash
sudo certbot --nginx
```
Далее система попросит вас указать электронную почту и ответить на несколько вопросов. Сделайте это.
Следующим шагом укажите имена, для которых вы хотели бы активировать HTTPS:
```bash
Account registered.
Which names would you like to activate HTTPS for?
We recommend selecting either all domains, or all domains
in a VirtualHost/server block.
1: <доменное_имя_вашего_проекта_1>
2: <доменное_имя_вашего_проекта_2>
3: <доменное_имя_вашего_проекта_3>
...
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel):
```
Введите номер нужного доменного имени и нажмите Enter.

5. Проверьте конфигурацию Nginx, и если всё в порядке, перезагрузите её
