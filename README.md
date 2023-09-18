# Kittygram
***
## Kittygram проект, созданный для того чтобы вы могли делиться фотографиями котиков) - https://paulkittigram.ddnsking.com
***
## Технологии:
* Python 3.10
* Django3
* Nginx
* Gunicorn
* React
* djangorestframework
* Certbot
***
## установка на удаленный сервер
1. Клонируйте репозитарий:
```git clone git@github.com:Paulman132/infra_sprint1.git```
2. Создайте виртуальное окружение: ```python3 -m venv venv```
3. Установите зависимости: ```pip install -r requirements.txt```
4. Создайте файл ***.env***  в корневой директории , добавьте  SECRET_KEY
***
# Задеплойте проект на удаленный сервер
***
 ### Подключитесь к GitHub
Установите Git: 

```sudo apt install git```

Сгенерируйте SSH ключ: 

```ssh-keygen```

Сохраните публичный ключ на свой аккаунт GitHub: 

```cat .ssh/id_rsa.pub```

Клонируйте проект на удаленный сервер: 

```git clone git@github.com:Your_account/Project_name.git```  
***
### Start the backend
Установите пакетный менеджер и виртуальное окружение:  
```sudo apt install python3-pip python3-venv -y```  
Создайте виртуальное окружение:  
```python3 -m venv venv ```  
Активируйте виртуальное окружение:  
```source venv/bin/activate```
Установите зависимости:  
```pip install -r requirements.txt```  
Совершите миграции:   
```python manage.py migrate```  
Создайте суперпользователя:  
```python manage.py createsuperuser```
В settings.py файле в ALLOWED_HOSTS, добавьте IP адрес, также как и '127.0.0.1', 'localhost'.  
***
### Start the frontend
Установите the npm package manager на сервер:  
```curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash - &&\```  
```sudo apt-get install -y nodejs```
Установите frontend зависимости из ***infra_sprint1/frontend*** директории:  
``npm i``  
***
### Install and run Gunicorn
```pip install gunicorn==20.1.0```
Создайте файл конфигурации юнита systemd для Gunicorn:  
```sudo nano /etc/systemd/system/gunicorn.service ```  
In the gunicorn.service file, describe the process configuration:
```html
[Unit]
Description=gunicorn daemon 
After=network.target 

[Service]
User=yc-user 
WorkingDirectory=/home/yc-user/taski/backend/
ExecStart=/home/yc-user/taski/backend/venv/bin/gunicorn --bind 0.0.0.0.0:8000 backend.wsgi

[Install]
WantedBy=multi-user.target  
```

Запустите gunicorn.service :  
```sudo systemctl start gunicorn```  
Добавьте процесс в автозапуск:  
```sudo systemctl enable gunicorn```  
***
### Install Nginx
```sudo apt install nginx -y```  
Start Nginx:  
```sudo systemctl start nginx```  
Tell the firewall which ports should remain open:  
```sudo ufw allow 'Nginx Full'```  
```sudo ufw allow OpenSSH```  
Enable firewall:  
```sudo ufw enable```
***

### Build the frontend application statics
Из директории ***infra_sprint1/frontend*** запустите:  
```npm run build```
```sudo cp -r /home/yc-user/infra_sprint1/frontend/build/. /var/www/kittygram/```  
Опишите настройки для frontend application static:   
``` sudo nano /etc/nginx/sites-enabled/default```  
Уберите из файла все настройки и добавьте:  
```html
server {
    server_name YOUR_IP YOUR DOMAIN;
    client_max_body_size 32m;
    server_tokens off;

    location /api/ {
        proxy_pass http://127.0.0.1:8080;
    }
    location /admin/ {
        proxy_pass http://127.0.0.1:8080;
    }

    location /media/ {
        alias /var/www/kittygram/media/;
    }
    location / {
        root /var/www/kittygram;
        index index.html index.html index.htm;
        try_files $uri /index.html;
    }
```
***
### Describe the settings for working with the backend application
В файле ***settings.py*** добавьте или измените:  
```STATIC_URL = 'static_backend'```
```STATIC_ROOT = BASE_DIR / 'static_backend'```  
При включенном виртуальном окружении запустите:  
```python manage.py collectstatic```
В корневой директории запустите:  
```sudo cp -r infra_sprint1/backend/static_backend/ /var/www/kittygram/```  
***
### Configure encryption
Скачайте ***certbot***:  
```sudo apt install snapd```

```sudo snap install core; sudo snap refresh core```  

```sudo snap install --classic certbot```

```sudo ln -s /snap/bin/certbot /usr/bin/certbot```

Запустите certbot:  

```sudo certbot --nginx```

Перезапустите конфигурацию Nginx:

```sudo systemctl reload nginx```  

Настройте автоматическое продление сертификата SSL:  

```sudo certbot certificates```  

```sudo certbot renew --dry-run``` 
