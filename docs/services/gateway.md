# Gateway API

**Порт:** `8000`

Единая точка входа в систему. Принимает все внешние запросы и маршрутизирует их к соответствующим микросервисам.

## Назначение

Gateway API выполняет следующие функции:

- 🔀 Маршрутизация запросов к микросервисам
- 🔒 Централизованная аутентификация
- 📊 Сбор метрик и мониторинг
- 🚦 Балансировка нагрузки
- 📝 Логирование всех запросов

## API Эндпоинты

### Корневой эндпоинт

```http
GET /
```

**Ответ:**
```json
{
    "service": "ИИ Бармен Gateway API",
    "version": "1.0.0",
    "status": "active"
}
```

### Проверка здоровья

```http
GET /health
```

**Ответ:**
```json
{
    "status": "healthy",
    "services": [
        {"name": "telegram", "status": "healthy"},
        {"name": "rag", "status": "healthy"},
        {"name": "validation", "status": "healthy"}
    ]
}
```

### Основной эндпоинт для общения с ботом

```http
POST /bartender/ask
Content-Type: application/json

{
    "query": "Как приготовить Мохито?",
    "user_id": "123456",
    "k": 3,
    "with_moderation": true
}
```

**Параметры запроса:**

| Поле | Тип | Обязательный | Описание |
|:---|:---|:---:|:---|
| `query` | string | ✅ | Запрос пользователя |
| `user_id` | string | ❌ | ID пользователя |
| `k` | integer | ❌ | Количество результатов |
| `with_moderation` | boolean | ❌ | Включить модерацию |

**Пример ответа:**

!!! success "Успешный ответ"
    ```json
    {
        "answer": "Для приготовления Мохито вам понадобится:...",
        "blocked": false,
        "retrieved_count": 3,
        "processing_time": 1.234
    }
    ```

!!! warning "Заблокированный ответ"
    ```json
    {
        "answer": "Извините, ваш запрос не прошел модерацию",
        "blocked": true,
        "reason": "Нарушение правил",
        "retrieved_count": 0
    }
    ```

## Примеры использования

=== "Python"
    ```python
    import httpx
    import asyncio
    
    async def ask_bartender():
        async with httpx.AsyncClient() as client:
            response = await client.post(
                "http://localhost:8000/bartender/ask",
                json={
                    "query": "Как приготовить Маргариту?",
                    "user_id": "123456",
                    "k": 3
                }
            )
            return response.json()
    
    # Запуск
    result = asyncio.run(ask_bartender())
    print(result["answer"])
    ```

=== "curl"
    ```bash
    curl -X POST http://localhost:8000/bartender/ask \
      -H "Content-Type: application/json" \
      -d '{
        "query": "Как приготовить Маргариту?",
        "user_id": "123456",
        "k": 3
      }'
    ```

=== "JavaScript"
    ```javascript
    fetch('http://localhost:8000/bartender/ask', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        query: 'Как приготовить Маргариту?',
        user_id: '123456',
        k: 3
      })
    })
    .then(response => response.json())
    .then(data => console.log(data.answer));
    ```

## Проксирование запросов

Gateway автоматически проксирует запросы к другим микросервисам:

| Путь | Целевой сервис |
|:---|:---|
| `/telegram/*` | Telegram Bot Service |
| `/rag/*` | RAG Service |
| `/validation/*` | Validation Service |
| `/yandex/*` | Yandex API Service |
| `/logging/*` | Logging Service |

## Коды ответов

| Код | Описание |
|:---:|:---|
| 200 | Успешный запрос |
| 400 | Неверный формат запроса |
| 403 | Доступ запрещен |
| 404 | Ресурс не найден |
| 500 | Внутренняя ошибка |
| 503 | Сервис недоступен |
| 504 | Таймаут сервиса |

## Конфигурация

```yaml
# Переменные окружения для Gateway
GATEWAY_HOST: "0.0.0.0"
GATEWAY_PORT: 8000
TELEGRAM_SERVICE_URL: "http://telegram-service:8001"
RAG_SERVICE_URL: "http://rag-service:8002"
VALIDATION_SERVICE_URL: "http://validation-service:8003"
YANDEX_SERVICE_URL: "http://yandex-service:8004"
LOGGING_SERVICE_URL: "http://logging-service:8005"
```

## Мониторинг

Для мониторинга доступны следующие метрики:

- Количество запросов в секунду
- Время ответа (p95, p99)
- Количество ошибок по типам
- Статус подключенных сервисов
