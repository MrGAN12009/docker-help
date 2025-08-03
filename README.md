# Flask Landing Page

Современный лендинг-сайт, созданный с использованием Flask, с тремя основными страницами: О нас, Услуги и Контакты.

## Особенности

- 🎨 Современный и отзывчивый дизайн
- 📱 Адаптивная верстка для всех устройств
- ⚡ Быстрая загрузка и оптимизация
- 🔧 Легкое развертывание с Docker
- 📊 Аналитика и отслеживание

## Структура проекта

```
├── app.py                 # Основное Flask приложение
├── requirements.txt       # Python зависимости
├── Dockerfile            # Docker образ
├── docker-compose.yml    # Docker Compose конфигурация
├── nginx.conf           # Nginx конфигурация
├── templates/           # HTML шаблоны
│   ├── base.html       # Базовый шаблон
│   ├── index.html      # Главная страница
│   ├── about.html      # Страница "О нас"
│   ├── services.html   # Страница "Услуги"
│   └── contacts.html   # Страница "Контакты"
└── static/             # Статические файлы
    ├── css/
    │   └── style.css   # Пользовательские стили
    └── js/
        └── script.js   # JavaScript функциональность
```

## Быстрый старт

### Локальная разработка

1. Клонируйте репозиторий:
```bash
git clone <repository-url>
cd flask-landing
```

2. Создайте виртуальное окружение:
```bash
python -m venv venv
source venv/bin/activate  # Linux/Mac
# или
venv\Scripts\activate     # Windows
```

3. Установите зависимости:
```bash
pip install -r requirements.txt
```

4. Запустите приложение:
```bash
python app.py
```

5. Откройте браузер и перейдите по адресу: `http://localhost:5000`

### Развертывание с Docker

#### Простой запуск с Docker Compose

1. Убедитесь, что у вас установлен Docker и Docker Compose

2. Запустите приложение:
```bash
docker-compose up -d
```

3. Откройте браузер и перейдите по адресу: `http://localhost`

#### Сборка и запуск только Flask приложения

1. Соберите Docker образ:
```bash
docker build -t flask-landing .
```

2. Запустите контейнер:
```bash
docker run -p 5000:5000 flask-landing
```

## Развертывание на Docker Hub

### 1. Подготовка образа

1. Соберите образ с тегом:
```bash
docker build -t your-username/flask-landing:latest .
```

2. Протестируйте образ локально:
```bash
docker run -p 5000:5000 your-username/flask-landing:latest
```

### 2. Загрузка на Docker Hub

1. Войдите в Docker Hub:
```bash
docker login
```

2. Загрузите образ:
```bash
docker push your-username/flask-landing:latest
```

### 3. Развертывание на сервере

1. На сервере с Docker выполните:
```bash
docker pull your-username/flask-landing:latest
docker run -d -p 80:5000 --name flask-landing your-username/flask-landing:latest
```

## Различия между Dockerfile и docker-compose.yml

### Dockerfile
- **Назначение**: Инструкция для создания Docker образа
- **Содержит**: Пошаговые инструкции для сборки образа
- **Использование**: `docker build` и `docker run`
- **Преимущества**: 
  - Легковесность
  - Портативность
  - Простота развертывания
- **Недостатки**: 
  - Нет оркестрации нескольких сервисов
  - Ручное управление сетями и томами

### docker-compose.yml
- **Назначение**: Оркестрация нескольких контейнеров
- **Содержит**: Конфигурацию для нескольких сервисов
- **Использование**: `docker-compose up`
- **Преимущества**:
  - Автоматическая оркестрация сервисов
  - Встроенные сети и тома
  - Простое управление зависимостями
  - Переменные окружения
- **Недостатки**:
  - Больше сложности
  - Требует Docker Compose

## Конфигурация

### Переменные окружения

- `FLASK_APP`: Имя основного файла приложения
- `FLASK_ENV`: Окружение (development/production)
- `PYTHONUNBUFFERED`: Отключение буферизации Python

### Nginx конфигурация

Файл `nginx.conf` содержит:
- Обратный прокси к Flask приложению
- Обслуживание статических файлов
- Gzip сжатие
- SSL конфигурация (закомментирована)

## Мониторинг и логи

### Просмотр логов

```bash
# Логи всех сервисов
docker-compose logs

# Логи конкретного сервиса
docker-compose logs web
docker-compose logs nginx

# Логи в реальном времени
docker-compose logs -f
```

### Health Check

Приложение доступно по адресу: `http://localhost/health`

## Масштабирование

### Горизонтальное масштабирование

```bash
# Масштабирование веб-сервиса
docker-compose up -d --scale web=3
```

### Load Balancer

Для продакшена рекомендуется использовать внешний load balancer (например, AWS ALB, Nginx Plus).

## Безопасность

### Рекомендации для продакшена

1. Используйте HTTPS (раскомментируйте SSL секцию в nginx.conf)
2. Настройте firewall
3. Регулярно обновляйте базовые образы
4. Используйте secrets для чувствительных данных
5. Настройте мониторинг и логирование

## Разработка

### Добавление новых страниц

1. Создайте новый маршрут в `app.py`
2. Создайте HTML шаблон в `templates/`
3. Добавьте ссылку в навигацию (`templates/base.html`)

### Изменение стилей

Редактируйте файл `static/css/style.css` для изменения внешнего вида.

## Лицензия

MIT License

## Поддержка

Для вопросов и предложений создайте issue в репозитории. 