# Патчи Wiseflow

Патчи и перекрытия зависимостей, предоставляемые wiseflow для стандартного openclaw, являются базовыми универсальными возможностями wiseflow и автоматически применяются `apply-addons.sh`.

### 1. Кодовые патчи (*.patch)

Git-патчи для OpenClaw, именованные по номерам и применяемые в указанном порядке:

| Патч | Назначение |
|------|------------|
| `002-disable-web-search-env-var.patch` | Добавляет переменную окружения `OPENCLAW_DISABLE_WEB_SEARCH` для отключения встроенного web_search по требованию |
| `003-act-field-validation.patch` | Усиливает проверку полей act для инструмента браузера, предотвращая действия-иллюзии |
| `005-browser-timeout-env-var.patch` | Добавляет переменную окружения `OPENCLAW_BROWSER_TIMEOUT` для настройки тайм-аута браузера |

> **Примечание**: Исходный патч `001-suppress-stale-reply-context.patch` был удалён 25 апреля 2026 года. Канал awada теперь использует per-peer inbound debouncer на входном уровне для объединения последовательных сообщений, что позволяет решить проблему загрязнения истории без использования ядерного патча openclaw.

### 2. Перекрытия зависимостей (overrides.sh)

`overrides.sh` выполняется первым после восстановления `openclaw/` в чистое состояние и используется для внедрения перекрытий зависимостей pnpm (например, замена playwright на patchright).

### Вспомогательные инструменты

- `generate-patch.sh`: Вспомогательный скрипт для генерации файлов патча из текущей рабочей области openclaw. Запускайте в корневом каталоге проекта.

---

# Wiseflow Patches (Original in English)

Wiseflow provides non-intrusive patches and dependency overrides for the original openclaw. These serve as common foundational capabilities for wiseflow and are automatically applied by `apply-addons.sh`.

### 1. Code Patches (*.patch)

Git patches applied to OpenClaw, named sequentially and applied in order:

| Patch | Function |
|-------|----------|
| `002-disable-web-search-env-var.patch` | Adds `OPENCLAW_DISABLE_WEB_SEARCH` environment variable to optionally disable built-in web_search |
| `003-act-field-validation.patch` | Strengthens act field validation for browser tool to prevent hallucinated actions |
| `005-browser-timeout-env-var.patch` | Adds `OPENCLAW_BROWSER_TIMEOUT` environment variable to customize browser timeout |

> **Note**: Original `001-suppress-stale-reply-context.patch` removed on 2026-04-25. awada channel now uses per-peer inbound debouncer at the entry layer to merge consecutive messages, solving historical pollution without requiring openclaw core patches.

### 2. Dependency Overrides (overrides.sh)

`overrides.sh` runs first after `openclaw/` is restored to a clean state, used to inject pnpm dependency overrides (e.g., replacing playwright with patchright).

### Auxiliary Tools

- `generate-patch.sh`: Helper script to generate patch files from the current openclaw workspace. Run in project root directory.