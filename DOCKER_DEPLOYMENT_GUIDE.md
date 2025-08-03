# Руководство по развертыванию Flask приложения с Docker

## Различия между Docker и Docker Compose

### Docker
**Docker** - это платформа для разработки, доставки и запуска приложений в контейнерах.

**Основные характеристики:**
- **Назначение**: Создание и управление отдельными контейнерами
- **Файл конфигурации**: `Dockerfile`
- **Команды**: `docker build`, `docker run`, `docker ps`, `docker stop`
- **Преимущества**:
  - Легковесность (контейнеры меньше виртуальных машин)
  - Портативность (работает везде, где есть Docker)
  - Изоляция (каждый контейнер изолирован)
  - Быстрое развертывание
- **Недостатки**:
  - Нет встроенной оркестрации нескольких сервисов
  - Ручное управление сетями между контейнерами
  - Сложность управления зависимостями между сервисами

**Пример использования:**
```bash
# Сборка образа
docker build -t my-app .

# Запуск контейнера
docker run -p 8080:5000 my-app

# Управление контейнером
docker ps
docker stop <container_id>
```

### Docker Compose
**Docker Compose** - это инструмент для определения и запуска многоконтейнерных приложений.

**Основные характеристики:**
- **Назначение**: Оркестрация нескольких связанных контейнеров
- **Файл конфигурации**: `docker-compose.yml`
- **Команды**: `docker-compose up`, `docker-compose down`, `docker-compose logs`
- **Преимущества**:
  - Автоматическая оркестрация сервисов
  - Встроенные сети между контейнерами
  - Автоматическое управление томами
  - Простое управление переменными окружения
  - Единая команда для запуска всего приложения
- **Недостатки**:
  - Дополнительный уровень сложности
  - Требует установки Docker Compose
  - Может быть избыточным для простых приложений

**Пример использования:**
```bash
# Запуск всех сервисов
docker-compose up -d

# Просмотр логов
docker-compose logs

# Остановка всех сервисов
docker-compose down
```

## Пошаговое руководство по развертыванию

### Этап 1: Подготовка сервера

#### 1.1 Установка Docker на сервер

**Ubuntu/Debian:**
```bash
# Обновление пакетов
sudo apt update

# Установка необходимых пакетов
sudo apt install apt-transport-https ca-certificates curl software-properties-common

# Добавление GPG ключа Docker
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Добавление репозитория Docker
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# Обновление списка пакетов
sudo apt update

# Установка Docker
sudo apt install docker-ce docker-ce-cli containerd.io

# Добавление пользователя в группу docker
sudo usermod -aG docker $USER

# Запуск Docker
sudo systemctl start docker
sudo systemctl enable docker
```

**CentOS/RHEL:**
```bash
# Установка необходимых пакетов
sudo yum install -y yum-utils

# Добавление репозитория Docker
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Установка Docker
sudo yum install docker-ce docker-ce-cli containerd.io

# Запуск Docker
sudo systemctl start docker
sudo systemctl enable docker

# Добавление пользователя в группу docker
sudo usermod -aG docker $USER
```

#### 1.2 Установка Docker Compose

```bash
# Скачивание Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.20.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

# Установка прав на выполнение
sudo chmod +x /usr/local/bin/docker-compose

# Проверка установки
docker-compose --version
```

#### 1.3 Настройка firewall (опционально)

```bash
# Открытие портов для приложения
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 5000/tcp

# Включение firewall
sudo ufw enable
```

### Этап 2: Подготовка приложения

#### 2.1 Загрузка кода на сервер

**Вариант A: Клонирование из Git**
```bash
# Клонирование репозитория
git clone https://github.com/your-username/flask-landing.git
cd flask-landing
```

**Вариант B: Загрузка файлов через SCP**
```bash
# С локальной машины
scp -r ./flask-landing user@your-server:/home/user/
```

**Вариант C: Создание архива и загрузка**
```bash
# Создание архива локально
tar -czf flask-landing.tar.gz flask-landing/

# Загрузка на сервер
scp flask-landing.tar.gz user@your-server:/home/user/

# Распаковка на сервере
tar -xzf flask-landing.tar.gz
cd flask-landing
```

#### 2.2 Проверка структуры проекта

```bash
# Проверка наличия всех необходимых файлов
ls -la

# Ожидаемая структура:
# ├── app.py
# ├── requirements.txt
# ├── Dockerfile
# ├── docker-compose.yml
# ├── nginx.conf
# ├── templates/
# └── static/
```

### Этап 3: Развертывание с Docker Compose (рекомендуемый способ)

#### 3.1 Запуск приложения

```bash
# Сборка и запуск всех сервисов в фоновом режиме
docker-compose up -d

# Проверка статуса контейнеров
docker-compose ps

# Просмотр логов
docker-compose logs
```

#### 3.2 Проверка работоспособности

```bash
# Проверка доступности приложения
curl http://localhost

# Проверка health check
curl http://localhost/health

# Проверка статических файлов
curl http://localhost/static/css/style.css
```

#### 3.3 Настройка автозапуска

```bash
# Создание systemd сервиса для автозапуска
sudo nano /etc/systemd/system/flask-landing.service
```

**Содержимое файла `/etc/systemd/system/flask-landing.service`:**
```ini
[Unit]
Description=Flask Landing Application
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/home/user/flask-landing
ExecStart=/usr/local/bin/docker-compose up -d
ExecStop=/usr/local/bin/docker-compose down
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
```

```bash
# Перезагрузка systemd
sudo systemctl daemon-reload

# Включение автозапуска
sudo systemctl enable flask-landing

# Запуск сервиса
sudo systemctl start flask-landing

# Проверка статуса
sudo systemctl status flask-landing
```

### Этап 4: Развертывание только с Docker (альтернативный способ)

#### 4.1 Сборка образа

```bash
# Сборка Docker образа
docker build -t flask-landing .

# Проверка созданного образа
docker images
```

#### 4.2 Запуск контейнера

```bash
# Запуск контейнера в фоновом режиме
docker run -d \
  --name flask-landing \
  -p 80:5000 \
  --restart unless-stopped \
  flask-landing

# Проверка статуса контейнера
docker ps

# Просмотр логов
docker logs flask-landing
```

#### 4.3 Управление контейнером

```bash
# Остановка контейнера
docker stop flask-landing

# Запуск контейнера
docker start flask-landing

# Перезапуск контейнера
docker restart flask-landing

# Удаление контейнера
docker rm flask-landing
```

### Этап 5: Развертывание с Docker Hub

#### 5.1 Подготовка образа для Docker Hub

```bash
# Войдите в Docker Hub
docker login

# Сборка образа с тегом для Docker Hub
docker build -t your-username/flask-landing:latest .

# Загрузка образа на Docker Hub
docker push your-username/flask-landing:latest
```

#### 5.2 Развертывание на сервере

```bash
# Загрузка образа с Docker Hub
docker pull your-username/flask-landing:latest

# Запуск контейнера
docker run -d \
  --name flask-landing \
  -p 80:5000 \
  --restart unless-stopped \
  your-username/flask-landing:latest
```

### Этап 6: Настройка домена и SSL

#### 6.1 Настройка домена

```bash
# Редактирование nginx.conf для вашего домена
sudo nano nginx.conf
```

**Изменения в nginx.conf:**
```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    
    # ... остальная конфигурация
}
```

#### 6.2 Настройка SSL с Let's Encrypt

```bash
# Установка Certbot
sudo apt install certbot python3-certbot-nginx

# Получение SSL сертификата
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Автоматическое обновление сертификатов
sudo crontab -e
# Добавить строку:
# 0 12 * * * /usr/bin/certbot renew --quiet
```

### Этап 7: Мониторинг и обслуживание

#### 7.1 Просмотр логов

```bash
# Логи всех сервисов
docker-compose logs

# Логи конкретного сервиса
docker-compose logs web
docker-compose logs nginx

# Логи в реальном времени
docker-compose logs -f

# Логи только Docker контейнера
docker logs flask-landing -f
```

#### 7.2 Обновление приложения

```bash
# Остановка сервисов
docker-compose down

# Получение обновлений (если используете Git)
git pull origin main

# Пересборка и запуск
docker-compose up -d --build
```

#### 7.3 Резервное копирование

```bash
# Создание резервной копии данных
docker run --rm -v flask-landing_redis_data:/data -v $(pwd):/backup alpine tar czf /backup/redis-backup.tar.gz -C /data .

# Восстановление данных
docker run --rm -v flask-landing_redis_data:/data -v $(pwd):/backup alpine tar xzf /backup/redis-backup.tar.gz -C /data
```

### Этап 8: Масштабирование

#### 8.1 Горизонтальное масштабирование

```bash
# Масштабирование веб-сервиса
docker-compose up -d --scale web=3

# Проверка количества контейнеров
docker-compose ps
```

#### 8.2 Настройка Load Balancer

```bash
# Установка HAProxy
sudo apt install haproxy

# Конфигурация HAProxy
sudo nano /etc/haproxy/haproxy.cfg
```

**Пример конфигурации HAProxy:**
```conf
global
    daemon

defaults
    mode http
    timeout connect 5000ms
    timeout client 50000ms
    timeout server 50000ms

frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    server web1 localhost:5001 check
    server web2 localhost:5002 check
    server web3 localhost:5003 check
```

### Этап 9: Безопасность

#### 9.1 Настройка firewall

```bash
# Установка UFW
sudo apt install ufw

# Настройка правил
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Включение firewall
sudo ufw enable
```

#### 9.2 Обновление базовых образов

```bash
# Проверка устаревших образов
docker images

# Обновление образов
docker-compose pull
docker-compose up -d
```

#### 9.3 Сканирование уязвимостей

```bash
# Установка Trivy
sudo apt install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt update
sudo apt install trivy

# Сканирование образа
trivy image your-username/flask-landing:latest
```

### Этап 10: Устранение неполадок

#### 10.1 Частые проблемы

**Проблема: Контейнер не запускается**
```bash
# Проверка логов
docker-compose logs web

# Проверка портов
netstat -tulpn | grep :5000

# Проверка ресурсов
docker stats
```

**Проблема: Nginx не может подключиться к Flask**
```bash
# Проверка сети
docker network ls
docker network inspect flask-landing_app-network

# Проверка доступности Flask из Nginx
docker exec flask-landing-nginx-1 curl http://web:5000
```

**Проблема: Статические файлы не загружаются**
```bash
# Проверка монтирования томов
docker-compose exec nginx ls -la /var/www/static/

# Проверка прав доступа
docker-compose exec nginx chown -R nginx:nginx /var/www/static/
```

#### 10.2 Полезные команды для диагностики

```bash
# Информация о системе
docker system df
docker system info

# Проверка использования ресурсов
docker stats

# Очистка неиспользуемых ресурсов
docker system prune -a

# Проверка конфигурации
docker-compose config
```

## Заключение

Этот гайд покрывает все основные аспекты развертывания Flask приложения с использованием Docker. Выберите подходящий метод в зависимости от ваших потребностей:

- **Docker Compose** - для сложных приложений с несколькими сервисами
- **Docker** - для простых приложений
- **Docker Hub** - для распространения и развертывания в разных средах

Не забудьте настроить мониторинг, резервное копирование и безопасность для продакшн среды. 