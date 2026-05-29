# scripts/ Описание скриптов

## Обзор скриптов

| Скрипт | Назначение | Pull кода | apply addons | pnpm build | Установка daemon | Перезапуск gateway |
|--------|-----------|:---------:|:------------:|:----------:|:---------------:|:-----------------:|
| `install.sh` | Новая установка / Обновление | ✅ | ✅ | ✅ | ✅ | ✅ |
| `apply-addons.sh` | Локальное тестирование изменений | — | ✅ | ✅ | — | ✅ |
| `dev.sh` | Режим разработки (foreground) | — | ✅ | — | — | — |
| `setup-crew.sh` | Только синхронизация crew markdown | — | — | — | — | — |

---

## install.sh

**Единая установка / обновление**. Используется для первой установки новыми пользователями и для обновления опытными пользователями.

```bash
./scripts/install.sh              # Стандартная установка / обновление
./scripts/install.sh --skip-crew  # Пропустить синхронизацию workspace crew
./scripts/install.sh --force      # Принудительно перезаписать существующие файлы workspace (включая MEMORY.md)
```

Процесс выполнения:
1. git fetch + reset (получение последней версии кода wiseflow)
2. checkout openclaw до версии, указанной в `openclaw.version`
3. `pnpm install` (установка / обновление зависимостей)
4. `apply-addons.sh --no-build --no-restart` (применение патчей, навыков и шаблонов crew)
5. `pnpm build` + `pnpm ui:build` (компиляция dist и Control UI)
6. Запись `daemon.env` (сбор переменных окружения, ссылаемых в openclaw.json)
7. `daemon uninstall` + `daemon install` (переустановка systemd service)
8. systemd drop-in + restart (внедрение daemon.env и перезапуск gateway)

> ⚠️ Перед обновлением убедитесь, что система свободна (нет активных сессий агентов).

---

## apply-addons.sh

**Применение изменений addon одним действием**. Используется для локального тестирования после добавления или изменения патчей, навыков или шаблонов crew. Не извлекает удалённый код и не обновляет версию openclaw — использует только локальные исходные коды.

```bash
./scripts/apply-addons.sh              # Применить addons + build + перезапустить gateway
./scripts/apply-addons.sh --skip-crew  # Пропустить синхронизацию workspace crew
./scripts/apply-addons.sh --no-build   # Не выполнять pnpm build (вызывающая сторона должна обработать самостоятельно)
./scripts/apply-addons.sh --no-restart # Не перезапускать gateway service
./scripts/apply-addons.sh --force      # Принудительно перезаписать существующие файлы workspace
```

Процесс выполнения:
1. Восстановление `openclaw/` в исходное состояние (`git reset --hard`)
2. Синхронизация элементов конфигурации из `config-templates/` в рабочий `openclaw.json`
3. Установка глобальных навыков (`skills/` → `openclaw/skills/`)
4. Последовательная загрузка каждого addon: overrides → patches → skills → шаблоны crew
5. `pnpm install` (только при наличии overrides/patches)
6. `setup-crew.sh` (синхронизация workspace crew, можно пропустить с `--skip-crew`)
7. `pnpm build` (компиляция dist, можно пропустить с `--no-build`)
8. `systemctl restart` (перезапуск gateway, можно пропустить с `--no-restart`)

---

## dev.sh

**Запуск в режиме разработки (foreground)**. Автоматически применяет addons, но **не производит build** — пользователь должен выполнить `cd openclaw && pnpm build` вручную.

```bash
cd openclaw && pnpm build && cd ..   # Вручную выполнить build при первом запуске или после изменения исходного кода
./scripts/dev.sh gateway             # Запустить gateway в foreground
./scripts/dev.sh cli config set ...  # Выполнить команду CLI openclaw
```

---

## setup-crew.sh

**Синхронизация только markdown-файлов workspace crew**. Не затрагивает исходный код, не выполняет build и не перезапускает. Подходит для случаев обновления только содержимого шаблона crew (SOUL.md, AGENTS.md и т.д.).

```bash
./scripts/setup-crew.sh          # Идемпотентная синхронизация (не перезаписывает существующие файлы)
./scripts/setup-crew.sh --force  # Принудительная перезапись (включая персонализированные файлы, такие как MEMORY.md)
```

---

## Устаревшие скрипты

Следующие скрипты по-прежнему присутствуют в репозитории, но больше не рекомендуются к использованию:

| Скрипт | Альтернатива |
|--------|-------------|
| `upgrade.sh` | Используйте `install.sh` |
| `reinstall-daemon.sh` | Используйте `install.sh --skip-crew` (только переустановка daemon) или `apply-addons.sh` (перезапуск после применения изменений) |

---

## Быстрый поиск типичных сценариев

| Сценарий | Команда |
|----------|---------|
| Первая установка для новых пользователей | `./scripts/install.sh` |
| Обновление для опытных пользователей до последней версии | `./scripts/install.sh` |
| Тестирование после изменения patch | `./scripts/apply-addons.sh` |
| Синхронизация после изменения crew markdown | `./scripts/setup-crew.sh` |
| Разработка и отладка (foreground запуск) | `cd openclaw && pnpm build && cd .. && ./scripts/dev.sh gateway` |

---

# scripts/ Описание скриптов (Оригинал на английском)

## Scripts Overview

| Script | Purpose | Pull code | apply addons | pnpm build | Install daemon | Restart gateway |
|--------|---------|:---------:|:------------:|:----------:|:---------------:|:-----------------:|
| `install.sh` | New install / Upgrade | ✅ | ✅ | ✅ | ✅ | ✅ |
| `apply-addons.sh` | Local test of changes | — | ✅ | ✅ | — | ✅ |
| `dev.sh` | Development mode (foreground) | — | ✅ | — | — | — |
| `setup-crew.sh` | Sync crew markdown only | — | — | — | — | — |

---

## install.sh

**One-click install / upgrade**. Used for first-time deployment by new users and upgrades by existing users.

```bash
./scripts/install.sh              # Standard install/upgrade
./scripts/install.sh --skip-crew  # Skip crew workspace sync
./scripts/install.sh --force      # Force overwrite existing workspace files (including MEMORY.md)
```

Execution process:
1. git fetch + reset (fetch latest wiseflow code)
2. checkout openclaw to version pinned in `openclaw.version`
3. `pnpm install` (install/update dependencies)
4. `apply-addons.sh --no-build --no-restart` (apply patches + skills + crew templates)
5. `pnpm build` + `pnpm ui:build` (compile dist + Control UI)
6. Write `daemon.env` (collect environment variables referenced in openclaw.json)
7. `daemon uninstall` + `daemon install` (reinstall systemd service)
8. systemd drop-in + restart (inject daemon.env and restart gateway)

> ⚠️ Ensure the system is idle (no active agent sessions) before upgrading.

---

## apply-addons.sh

**Apply addon changes in one step**. Used for local testing after adding or modifying patches, skills, or crew templates. Does not fetch remote code or upgrade openclaw — uses local source code only.

```bash
./scripts/apply-addons.sh              # Apply addons + build + restart gateway
./scripts/apply-addons.sh --skip-crew  # Skip crew workspace sync
./scripts/apply-addons.sh --no-build   # Skip pnpm build (handled by caller)
./scripts/apply-addons.sh --no-restart # Do not restart gateway service
./scripts/apply-addons.sh --force      # Force overwrite existing workspace files
```

Execution process:
1. Restore `openclaw/` to clean state (`git reset --hard`)
2. Sync configuration items from `config-templates/` to runtime `openclaw.json`
3. Install global skills (`skills/` → `openclaw/skills/`)
4. Load each addon sequentially: overrides → patches → skills → crew templates
5. `pnpm install` (only when overrides/patches present)
6. `setup-crew.sh` (sync crew workspace; can skip with `--skip-crew`)
7. `pnpm build` (compile dist; can skip with `--no-build`)
8. `systemctl restart` (restart gateway; can skip with `--no-restart`)

---

## dev.sh

**Run in development mode (foreground)**. Automatically applies addons but **does not build** — user must manually `cd openclaw && pnpm build`.

```bash
cd openclaw && pnpm build && cd ..   # Manually build on first run or after modifying source code
./scripts/dev.sh gateway             # Start gateway in foreground
./scripts/dev.sh cli config set ...  # Run openclaw CLI command
```

---

## setup-crew.sh

**Sync only crew workspace markdown files**. Does not touch source code, build, or restart. Suitable for scenarios where only crew template content (e.g., SOUL.md, AGENTS.md) has been updated.

```bash
./scripts/setup-crew.sh          # Idempotent sync (does not overwrite existing files)
./scripts/setup-crew.sh --force  # Force overwrite (including personalized files like MEMORY.md)
```

---

## Deprecated Scripts

The following scripts remain in the repository but are no longer recommended:

| Script | Alternative |
|--------|-------------|
| `upgrade.sh` | Use `install.sh` |
| `reinstall-daemon.sh` | Use `install.sh --skip-crew` (daemon reinstall only) or `apply-addons.sh` (restart after applying changes) |

---

## Common Scenarios Quick Reference

| Scenario | Command |
|----------|---------|
| New user first install | `./scripts/install.sh` |
| Existing user upgrade to latest | `./scripts/install.sh` |
| Test after modifying patch | `./scripts/apply-addons.sh` |
| Sync after modifying crew markdown | `./scripts/setup-crew.sh` |
| Development调试 (foreground run) | `cd openclaw && pnpm build && cd .. && ./scripts/dev.sh gateway` |
