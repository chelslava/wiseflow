# Агент IT-инженера — Инструменты

## Доступные инструменты

### Общие инструменты
- Чтение и запись файлов: чтение журналов, конфигурационных файлов, изменение файлов workspace
- Выполнение shell-команд: запуск системных команд, проверка состояния, просмотр журналов

### Встроенные в wiseflow сценарии (необходимо сначала перейти в каталог проекта перед выполнением)

> Путь к проекту wiseflow см. в файле `OFB_ENV.md` в том же каталоге (обновляется каждый раз при запуске `setup-crew.sh`, содержит полную команду).

```bash
# Запуск в режиме разработки на переднем плане (с выводом журналов)
cd <PROJECT_ROOT> && ./scripts/dev.sh gateway

# Переустановка фоновой службы в режиме производства
cd <PROJECT_ROOT> && ./scripts/reinstall-daemon.sh

# Повторная синхронизация конфигурации crew (идемпотентно, безопасно к выполнению)
cd <PROJECT_ROOT> && ./scripts/setup-crew.sh

# Повторное применение addon
cd <PROJECT_ROOT> && ./scripts/apply-addons.sh

# Обновление системы wiseflow (перед выполнением обязательно убедитесь, что система свободна)
cd <PROJECT_ROOT> && ./scripts/upgrade.sh
```

> ⚠️ **Запрещено прямое выполнение команды `openclaw`** (`openclaw` отсутствует в системном PATH).
> При необходимости прямого вызова CLI upstream необходимо выполнить в подкаталоге `openclaw/` через `pnpm openclaw`:
> ```bash
> cd <PROJECT_ROOT>/openclaw && pnpm openclaw <subcommand>
> ```

### Проверка состояния работы системы

```bash
# Проверка активности процесса openclaw
ps aux | grep openclaw.mjs | grep -v grep

# Просмотр процессов, управляемых pm2 (режим производства)
pm2 list
pm2 logs openclaw --lines 50

# Проверка целостности файла конфигурации
node -e "require('fs').readFileSync(process.env.HOME + '/.openclaw/openclaw.json', 'utf8'); console.log('✅ Config OK')"
```

### GitHub / Кодовые инструменты (требуется включенный скилл github, gh-issues, coding-agent)
- `github`: чтение последней информации из репозиториев wiseflow и OpenClaw (commits, releases, README)
- `gh-issues`: просмотр issuewiseflow и OpenClaw,了解 известных проблем и статус их решения
- `coding-agent`: аналитика проблем с кодом, генерация конфигурационных файлов, интерпретация сообщений об ошибках

### Чтение истории сессий других агентов

> ⚠️ **Запрещено использование команд скиллов `sessions_send`/`sessions_list`/`sessions_history`/`sessions_status` для запроса сессий других агентов** — эти команды применимы только к текущему собственному агенту.

Если требуется чтение истории диалога другого агента (например, для анализа последовательного улучшения), выполните прямое чтение локальных файлов:

```bash
# Просмотр индекса сессии агента (с метаданными всех сессий)
cat ~/.openclaw/agents/<agentId>/sessions/sessions.json

# Просмотр полной истории диалога по сессии (формат JSONL, одна строка — одно сообщение)
cat ~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl
```

- `sessions.json`: JSON-объект, ключ = session key (например, `agent:cs-001:awada:direct:user123`), значение = метаданные сессии
- `<sessionId>.jsonl`: Полный контент диалога, построчно JSON, содержит поля role/content/timestamp и т.д.
- Архивы транскриптов сессий: каталог `.archived/` в `~/.openclaw/agents/<agentId>/sessions/`

## Правила использования инструментов

1. **Резервное копирование важных файлов**: перед изменением `~/.openclaw/openclaw.json` сначала создайте резервную копию
2. **Приоритет сценариев**: используйте встроенные сценарии wiseflow, не редактируйте код в каталоге `openclaw/` напрямую
3. **Журналы как первая подсказка**: при возникновении проблем сначала проверьте журналы, затем выдвигайте предположения
4. **Проверка результатов**: после каждой операции подтвердите эффект (например, после перезапуска проверьте, работает ли сервис нормально)

### Технические инструменты SEO

```bash
# Оценка производительности/SEO с помощью Lighthouse (требуется Chrome)
npx lighthouse https://yoursite.com --only-categories=performance,seo --output json

# Валидация sitemap (проверка формата и доступности)
curl -sf https://yoursite.com/sitemap.xml | python3 -c "import sys; import xml.etree.ElementTree as ET; ET.parse(sys.stdin); print('✅ sitemap valid')"

# Проверка robots.txt
curl -sf https://yoursite.com/robots.txt

# Обнаружение состояния внутренних/внешних ссылок (используйте xurl или пакетную проверку через curl)
curl -o /dev/null -s -w "%{http_code}" https://yoursite.com/some-page

# Google Search Console (доступ через браузер или API GSC)
# Документация API: https://developers.google.com/webmaster-tools/v1/api_reference_index
```

| Инструмент | Назначение |
|------|------|
| `smart-search` | Поиск лучших практик SEO, поиск технических решений конкурентов |
| `xurl` | Прямой доступ к API GSC, API PageSpeed, API тестирования структурированных данных |
| `coding-agent` | Генерация sitemap.xml, JSON-LD Schema, содержимое robots.txt |
