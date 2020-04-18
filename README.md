# Aegea docker, настройка и запуск

> В данном репозитории находится Aegea Версия 2.8, сборка 3386

## Цели

- [ ] Клонирование проекта и настройка прав
- [ ] Подготовка файла конфигурации **nginx**
- [ ] Редактирование файла конфигурации **certbot**
- [ ] Установка пароля **mysql**
- [ ] Генерирование **ssl** сертификата
- [ ] Настройка автоматического запуска контейнеров

## Подготовительный этап

- [ ] Сервер `Ubuntu 18.04` или `Debian 9, 10`
- [ ] Открытые порты `80` и `443`
- [ ] Установленный `docker` и `docker-compose`
- [ ] Зарегистрированные доменные имена

## Настройка на сервере

### Этап 1. Клонирование проекта и настройка прав

Вы можете клонировать проект в любое удобное место, рекомендую клонировать в папку `/opt` 

```bash
git clone https://github.com/smart-leo/aegea.git
```

> Если у вас возникла ошибка с доступом, то запустите команду клонирования из под `sudo`, после понадобиться заменить пользователя следующей командой:
>
> ```bash
> sudo chown -R ${USER}:${USER} /opt/aegea
> ```

Так же необходимо дать правильную группу и пользователя для папки с кодом блога, для этого введите в терминал:

```bash
sudo chown -R www-data:www-data /opt/aegea/blog
sudo chmod -R 755 /opt/aegea/blog
```

#### Этап 1.1. Настройка сети

Из-за того что на некоторых серверах могут быть конфликты с подсетью сервера и docker. Необходимо создать отдельную сеть для docker. Название сети `br0`

```bash
docker network create \
--driver=bridge \
--subnet=10.10.0.0/16 \
--gateway=10.10.0.1 \
--ip-range=10.10.88.0/24 \
br0
```



### Этап 2. Подготовка файла конфигурации nginx

Перейдите в папку с конфигурации сайта:

```bash
cd /opt/aegea/nginx/conf.d
```

После этого необходимо заменить в нескольких местах имя домена на ваш:

```nginx
...
4:  server_name smartleo.ru www.smartleo.ru;
...
14: ssl_certificate /etc/letsencrypt/live/smartleo.ru/fullchain.pem; # managed by Certbot
15: ssl_certificate_key /etc/letsencrypt/live/smartleo.ru/privkey.pem; # managed by Certbot
...
53: server_name smartleo.ru www.smartleo.ru;
...
```

### Этап 3. Редактирование файла конфигурации certbot

> Для запуска **certbot** в режиме тестирования вы можете установить параметр `staging` в значение `1`

Так же необходимо поправить `init-letsencrypt.sh` тк в нем указываются домены и почтовый адрес для получения `ssl` сертификата, но для начала необходимо разрешить файл на исполнение:

```bash
chmod +x init-letsencrypt.sh
```

После этого необходимо отредактировать этот файл и исправить несколько строк, заменив их на свои:

```bash
...
8:  domains=(smartleo.ru www.smartleo.ru)
...
11: email="iskander.kamaliev@gmail.com" # Adding a valid address is strongly recommended
...
```

### Этап 4. Установка пароля mysql

В файле `docker-compose.yml` стоит очень простой пароль, рекомендутеся заменить его на свой. Для этого достаточно установить его для переменной `MYSQL_ROOT_PASSWORD`. Можно заменить имя базы данных с помощью переменной `MYSQL_DATABASE`.

### Этап 5. Генерирование **ssl** сертификата

Запустите в командной строке:

```bash
sudo ./init-letsencrypt.sh
```

Ответье на все вопросы, которые выдаст скрипт, дождитесь его завершения, если все прошло без ошибок скрипт сообщит вам об этом.

Остановите запущенные контейнеры:

```bash
docker-compose down
```

### Этап 6. Настройка автоматического запуска контейнеров

Вы можете теперь запустить все контейнеры в фоновом режиме, если все прошло отлично вы должны увидеть страницу настройки блога:

```bash
docker-compose up -d
```

Для автоматического запуска контейнеров после перезапуска сервера необходимо настроить сервис, для этого создайте файл `docker-compose-app.service` в папке `/etc/systemd/system/`:

```nginx
# /etc/systemd/system/docker-compose-app.service
 
[Unit]
Description=Docker Compose Application Service
Requires=docker.service
After=docker.service
 
[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/aegea
ExecStart=/usr/bin/docker-compose up -d
ExecStop=/usr/bin/docker-compose down
TimeoutStartSec=0
 
[Install]
WantedBy=multi-user.target
```

После этого выполните команду в терминале:

```bash
sudo systemctl enable docker-compose-app
```

