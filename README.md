# Web Application на Go с полной DevOps инфраструктурой

Полнофункциональное веб-приложение на Go с использованием Gin фреймворка, включающее полную DevOps инфраструктуру с GitLab CI/CD, мониторингом, логированием и автоматическим развертыванием.

## 🏗️ Архитектура проекта

```
-test-task-web-app/
├── main.go                 # Основное приложение
├── main_test.go           # Тесты приложения
├── go.mod                 # Зависимости Go
├── go.sum                 # Хеши зависимостей
├── Dockerfile             # Multi-stage Docker образ
├── docker-compose.yml     # Полная инфраструктура
├── .gitlab-ci.yml         # CI/CD пайплайн
├── env.example            # Пример переменных окружения
├── scripts/
│   └── deploy.sh          # Скрипт развертывания
├── monitoring/
│   ├── prometheus.yml     # Конфигурация Prometheus
│   ├── loki-config.yml    # Конфигурация Loki
│   ├── promtail-config.yml # Конфигурация Promtail
│   └── grafana/
│       └── provisioning/  # Дашборды Grafana
├── infrastructure/
│   ├── nginx/             # Конфигурация Nginx
│   └── registry/          # Container Registry
└── docs/                  # Документация
```

## 🚀 Быстрый запуск

### Требования
- Docker 20.10+
- Docker Compose 2.0+
- Минимум 8GB RAM
- 20GB свободного места

### 1. Клонирование и запуск
```bash
git clone <repository-url>
cd -test-task-web-app
./scripts/deploy.sh
```

### 2. Доступ к сервисам
После запуска будут доступны:
- **Веб-приложение**: http://localhost
- **GitLab** (локальная разработка): http://localhost:8081 (admin/admin123)
- **Grafana**: http://localhost:3000 (admin/admin123)
- **Prometheus**: http://localhost:9090
- **Container Registry** (локальный): http://localhost:5000
- **GitHub Container Registry**: ghcr.io/[username]/[repository]

## 📋 Функционал приложения

### REST API Endpoints
- `GET /health` - Health check приложения
- `GET /metrics` - Метрики Prometheus
- `GET /api/users` - Получение списка пользователей
- `POST /api/users` - Создание нового пользователя

### База данных
- **PostgreSQL** с автоматической инициализацией
- Автоматическое создание таблиц
- Тестовые данные при первом запуске

### Логирование
- **Структурированное логирование** с logrus
- **Уровни**: INFO, DEBUG, ERROR
- **Сбор логов** через Loki + Promtail
- **Визуализация** в Grafana

## 🔧 Технические особенности

### Docker оптимизации
- **Multi-stage build** для минимизации размера образа
- **Non-root пользователь** для безопасности
- **Health checks** для всех сервисов
- **Оптимизация RAM** с лимитами и резервированием

### Сети и безопасность
- **Кастомные подсети** для разных компонентов
- **Изоляция сетей** между приложением, мониторингом и GitLab
- **Rate limiting** в Nginx
- **SSL готовность** (конфигурация для HTTPS)

### Мониторинг и метрики
- **Prometheus** для сбора метрик
- **Grafana** для визуализации
- **Loki** для централизованного логирования
- **Node Exporter** для системных метрик
- **cAdvisor** для метрик контейнеров

## 🏭 CI/CD Pipeline (GitHub Actions)

### Стадии пайплайна
1. **validate** - Валидация кода (go vet, go fmt)
2. **test** - Запуск тестов с покрытием
3. **build** - Сборка Docker образа в GitHub Container Registry
4. **security** - Сканирование уязвимостей (Trivy + CodeQL)
5. **deploy** - Развертывание (staging/production)
6. **monitor** - Мониторинг развертывания

### Особенности
- **Автоматическая сборка** при push в main
- **GitHub Container Registry** для хранения образов
- **CodeQL** для статического анализа кода
- **Dependabot** для автоматического обновления зависимостей
- **Сканирование безопасности** образов и кода
- **Кэширование** для ускорения сборки
- **Артефакты** тестового покрытия

## 📊 Мониторинг и логирование

### Дашборды Grafana
- **Метрики приложения**: HTTP запросы, длительность, ошибки
- **Системные метрики**: CPU, RAM, диск, сеть
- **Метрики контейнеров**: использование ресурсов
- **Логи приложения**: уровни DEBUG, WARN, ERROR

### Prometheus метрики
- `http_requests_total` - Общее количество запросов
- `http_request_duration_seconds` - Длительность запросов
- `go_goroutines` - Количество горутин
- `go_memstats_*` - Статистика памяти

### Loki + LogQL
```logql
# Логи уровня ERROR
{container="webapp"} |= "level=error"

# Запросы к API
{container="webapp"} |= "path=/api/"

# Медленные запросы (>1s)
{container="webapp"} | json | latency > 1s
```

## 🔄 Управление развертыванием

### Команды скрипта deploy.sh
```bash
./scripts/deploy.sh deploy    # Развертывание
./scripts/deploy.sh stop      # Остановка
./scripts/deploy.sh restart   # Перезапуск
./scripts/deploy.sh logs      # Просмотр логов
./scripts/deploy.sh status    # Статус сервисов
./scripts/deploy.sh clean     # Полная очистка
```

### Переменные окружения
Создайте файл `.env` на основе `env.example`:
```bash
# Приложение
PORT=8080
LOG_LEVEL=INFO

# База данных
DB_HOST=postgres
DB_PASSWORD=your_secure_password

# GitLab
GITLAB_ROOT_PASSWORD=your_gitlab_password

# Grafana
GRAFANA_ADMIN_PASSWORD=your_grafana_password
```

## 🧪 Тестирование

### Запуск тестов
```bash
# Локально
go test -v -race -coverprofile=coverage.out ./...

# В Docker
docker-compose run --rm webapp go test -v ./...
```

### Покрытие кода
```bash
go tool cover -html=coverage.out -o coverage.html
```

## 🔍 Troubleshooting

### Проблемы с запуском
1. **Недостаточно памяти**: Увеличьте RAM до 8GB+
2. **Порт занят**: Остановите другие сервисы на портах 80, 3000, 8081
3. **GitLab не запускается**: Подождите 5-10 минут для инициализации

### Просмотр логов
```bash
# Все сервисы
docker-compose logs -f

# Конкретный сервис
docker-compose logs -f webapp
docker-compose logs -f gitlab
```

### Проверка здоровья сервисов
```bash
# Приложение
curl http://localhost/health

# Метрики
curl http://localhost/metrics

# GitLab
curl http://localhost:8081/-/health
```

## 📚 API Документация

### Health Check
```http
GET /health
```
**Ответ:**
```json
{
  "status": "healthy",
  "timestamp": "2024-01-01T12:00:00Z",
  "service": "web-app"
}
```

### Получение пользователей
```http
GET /api/users
```
**Ответ:**
```json
{
  "users": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "john@example.com",
      "created_at": "2024-01-01T12:00:00Z"
    }
  ]
}
```

### Создание пользователя
```http
POST /api/users
Content-Type: application/json

{
  "name": "New User",
  "email": "newuser@example.com"
}
```
**Ответ:**
```json
{
  "user": {
    "id": 4,
    "name": "New User",
    "email": "newuser@example.com",
    "created_at": "2024-01-01T12:00:00Z"
  }
}
```

### Метрики Prometheus
```http
GET /metrics
```
Возвращает метрики в формате Prometheus.

## 🔐 Безопасность

### Рекомендации для production
1. **Измените пароли** в `.env` файле
2. **Настройте SSL/TLS** сертификаты
3. **Ограничьте доступ** к GitLab и Grafana
4. **Настройте firewall** правила
5. **Регулярно обновляйте** образы
6. **Настройте GitHub Secrets** для CI/CD

### Переменные безопасности
```bash
# Обязательно измените в production
GITLAB_ROOT_PASSWORD=very_secure_password
GRAFANA_ADMIN_PASSWORD=very_secure_password
DB_PASSWORD=very_secure_password
GITHUB_TOKEN=your_github_token
```

### GitHub Secrets
Настройте следующие secrets в репозитории:
- `GITHUB_TOKEN` - автоматически предоставляется GitHub
- `STAGING_SSH_PRIVATE_KEY` - SSH ключ для staging сервера
- `PRODUCTION_SSH_PRIVATE_KEY` - SSH ключ для production сервера

## 🚀 Развертывание в облаке

### Подготовка к production
1. **Настройте домен** и SSL сертификаты
2. **Настройте backup** базы данных
3. **Настройте мониторинг** и алерты
4. **Настройте CI/CD** для production

### Terraform (опционально)
Создайте `infrastructure/terraform/` для автоматизации развертывания в облаке.

## 📞 Поддержка

### Полезные команды
```bash
# Мониторинг ресурсов
docker stats

# Очистка неиспользуемых ресурсов
docker system prune -a

# Просмотр сетей
docker network ls

# Просмотр volumes
docker volume ls
```

### Логи и отладка
- **Логи приложения**: `docker-compose logs webapp`
- **Логи GitLab**: `docker-compose logs gitlab`
- **Логи Grafana**: `docker-compose logs grafana`
- **Метрики**: http://localhost:9090 (Prometheus)

## 📄 Лицензия

MIT License - см. файл LICENSE для деталей.

---

**Автор**: DevOps Team  
**Версия**: 1.0.0  
**Дата**: 2024
