# Документ по инфраструктуре Redis

Документ описывает реализацию инфраструктуры Redis Streams в awada-server для проверки и использования инженерами.

## Структура файлов

```
src/
├── index.ts                           # Главная точка входа
├── infrastructure/redis/
│   ├── types.ts                       # Определения типов (протокол событий, конфигурация и т.д.)
│   ├── connection.ts                  # Управление подключением Redis (одиночный экземпляр, пул подключений)
│   ├── producer.ts                    # EventProducer (запись XADD)
│   ├── consumer.ts                    # EventConsumer (чтение XREADGROUP)
│   ├── idempotency.ts                 # Управление уникальностью/избежанием повторов
│   ├── session.ts                     # Управление блокировкой сеанса и номерами последовательностей
│   ├── conversation.ts                # Управление отображением Conversation ID
│   └── index.ts                       # Единый экспорт
└── examples/
    ├── server-example.ts              # Пример использования на стороне сервера
    └── bot-example.ts                 # Пример использования на стороне бота
```

## Основные модули

| Модуль | Файл | Функциональность |
|------|------|------|
| **EventProducer** | `producer.ts` | Запись в Inbound/Outbound Stream через `XADD`, автоматическое управление `session_seq` |
| **EventConsumer** | `consumer.ts` | Потребление через `XREADGROUP`, автоматическое подтверждение (ACK), восстановление pending, обработка DLQ |
| **IdempotencyManager** | `idempotency.ts` | Проверка уникальности через `SETNX`, предотвращение повторной обработки |
| **SessionManager** | `session.ts` | Распределенная блокировка + контроль номеров последовательностей для последовательной обработки одного сеанса |
| **ConversationManager** | `conversation.ts` | Поддержание отображения (platform, user, channel) -> conversation_id |
| **RedisConnection** | `connection.ts` | Управление одиночным подключением, поддержка нескольких клиентов |

## Установка зависимостей

```bash
# Продакшн-зависимости
npm install ioredis uuid

# Зависимости разработки
npm install -D typescript @types/node @types/uuid tsx
```

## Спецификация формата Payload

### Структура Payload

`payload` — это массив, каждый элемент представляет одну часть сообщения. Элементы в массиве отправляются последовательно.

```json
[{
  "type": "text",
  "text": "Привет"
},
{
  "type": "image",
  "file_url": "https://example.com/image.png"
},
{
  "type": "audio",
  "file_path": "/path/to/audio.mp3"
},
{
  "type": "file",
  "file_id": "dddddxxxxxxxxx"
}]
```

### Определение типов сообщений

| type | Поля | Описание |
|------|------|------|
| `text` | `text` | Текстовое содержимое (строка), допускаются emoji (заключаются в `[]`), допускаются URL |
| `image` | `file_url` или `file_path` или `file_id` | Изображение (одно из трех) |
| `audio` | `file_url` или `file_path` или `file_id` | Аудио (одно из трех) |
| `file` | `file_url` или `file_path` или `file_id` | Файл (одно из трех) |

**Описание полей:**
- `file_url`: Доступный URL
- `file_path`: Абсолютный путь на локальном диске
- `file_id`: ID файла, полученный после загрузки

### Правила ограничений

1. `type` допускается только `text`, `image`, `audio`, `file`
2. В одном массиве payload может быть максимум **1** элемент типа `text`, но может быть несколько элементов `file`, `image`, `audio`
3. Если массив payload содержит тип `text`, должен присутствовать хотя бы 1 элемент `file` или `image`
4. Чистое текстовое сообщение можно отправить с одним элементом `text`, например: `[{"type": "text", "text": "Привет"}]`
5. Поддерживается отправка чистого изображения или чистого файла, однако перед или после такого сообщения должна идти строка `text`, обеспечивающая контекст пользовательского запроса

**Важно:** При записи в Redis весь события сериализуется в JSON; при чтении достаточно одного `json.loads()` / `JSON.parse()`

## Спецификация именования ключей Redis

Определено в `types.ts` как `STREAM_KEYS`:

| Шаблон ключа | Назначение |
|----------|------|
| `awada:events:inbound:{lane}` | Поток inbound-событий (Server -> Bot) |
| `awada:events:outbound:{lane}` | Поток outbound-событий (Bot -> Server) |
| `awada:events:inbound:dlq` | DLQ (Dead Letter Queue) inbound |
| `awada:events:outbound:dlq` | DLQ outbound |
| `awada:session_seq:{sessionId}` | Счетчик последовательности сеанса |
| `awada:session_next_seq:{sessionId}` | Ожидаемый следующий номер последовательности |
| `awada:lock:session:{sessionId}` | Блокировка сеанса |
| `awada:processed:{eventId}` | Метка уникальности |
| `awada:conversation:{platform}:{userId}:{channelId}` | Отображение Conversation |

## Спецификация именования групп потребителей (Consumer Group)

| Шаблон группы | Назначение |
|------------|------|
| `bot_workers_{lane}` | Потребление Bot из Inbound |
| `server_dispatchers_{lane}` | Потребление Server из Outbound |

## Механизмы надежности

### 1. Доставка по принципу "at-least-once"

- Redis Streams Consumer Group обеспечивает семантику "at-least-once"
- Подтверждение (ACK) выполняется только после успешной обработки сообщения
- Необработанные сообщения остаются в pending и ожидают повторной попытки

### 2. Гарантия уникальности

- `IdempotencyManager` выполняет дедупликацию по `event_id`
- Атомарная операция `SETNX` + TTL
- При ошибке обработки метка уникальности удаляется, разрешая повторную попытку

### 3. Гарантия последовательности

- `session_seq`: Server генерирует увеличивающийся номер для каждого сеанса
- `session_next_seq`: Bot поддерживает ожидаемый следующий номер последовательности
- Сообщения в неправильном порядке не обрабатываются, ожидают повторной попытки

### 4. Контроль параллелизма

- `SessionManager` использует распределенную блокировку для обеспечения последовательной обработки одного сеанса
- Блокировка обеспечивает автоматическое продление для предотвращения истечения срока во время длительной обработки

### 5. Обработка DLQ

- Сообщения, превысившие `maxRetries` повторных попыток, автоматически перемещаются в DLQ
- DLQ-сообщения содержат исходное событие, информацию об ошибке, количество повторных попыток

## Параметры конфигурации

### StreamConfig (конфигурация потребителя)

```typescript
interface StreamConfig {
  consumerGroup: string;        // Имя группы потребителей
  consumerName: string;         // Имя потребителя (рекомендуется включать PID)
  maxRetries: number;           // Максимальное число повторных попыток, по умолчанию 5
  minIdleTimeMs: number;        // Тайм-аут простоя сообщений pending (мс), по умолчанию 30000
  blockTimeMs: number;          // Время ожидания BLOCK для XREADGROUP (мс), по умолчанию 5000
  batchSize: number;            // Количество извлекаемых сообщений за раз, по умолчанию 10
  idempotencyTtlSeconds: number; // Время жизни ключа уникальности (сек), по умолчанию 86400
}
```

### SessionLockOptions (конфигурация блокировки сеанса)

```typescript
interface SessionLockOptions {
  lockTimeoutMs: number;    // Время истечения блокировки (мс), по умолчанию 60000
  renewIntervalMs: number;  // Интервал продления (мс), по умолчанию 20000
}
```

## Справочная документация

- [awada_top_architecture.md](../references/awada_top_architecture.md) — Проектная документация архитектуры верхнего уровня
- [PYTHON_INTEGRATION.md](./PYTHON_INTEGRATION.md) — Руководство по интеграции Python
- [README.md](../../README.md) — Описание проекта

(Конец файла - всего 176 строк)
