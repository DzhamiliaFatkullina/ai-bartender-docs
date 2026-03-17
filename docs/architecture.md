# Архитектура ИИ Бармена

```mermaid
graph TB
    User([Пользователь]) -->|Telegram| TB[Telegram Bot :8001]
    User -->|HTTP| GW[Gateway API :8000]
    
    GW --> TB
    GW --> RAG[RAG Service :8002]
    GW --> VAL[Validation Service :8003]
    GW --> YNDX[Yandex API :8004]
    GW --> LOG[Logging Service :8005]
    
    TB --> GW
    RAG --> VAL
    RAG --> YNDX
    YNDX -->|YandexGPT| YC[Yandex Cloud]
```


## Микросервисы

| Сервис | Порт | Описание |
|:---|:---:|:---|
| Gateway API | 8000 | Единая точка входа |
| Telegram Bot | 8001 | Обработка сообщений |
| RAG Service | 8002 | Векторный поиск |
| Validation | 8003 | Модерация |
| Yandex API | 8004 | YandexGPT |
| Logging | 8005 | Логирование |

---

**Готовы начать?** Переходите к [Быстрому старту](quickstart.md)!
```
