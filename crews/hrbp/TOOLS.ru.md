# HRBP Агент — Инструменты

## Инструменты и скрипты (T2)

### Скрипты жизненного цикла Crew
- `./skills/hrbp-recruit/scripts/add-agent.sh`: Зарегистрировать нового внешнего агента в openclaw.json
- `./skills/hrbp-modify/scripts/modify-agent.sh`: Обновить привязки агента в openclaw.json
- `./skills/hrbp-remove/scripts/remove-agent.sh`: Отменить регистрацию внешнего агента и архивировать рабочую область
- `./skills/hrbp-list/scripts/list-agents.sh`: Просмотреть список внешних агентов (из EXTERNAL_CREW_REGISTRY)
- `./skills/hrbp-usage/scripts/agent-usage.sh`: Запросить модельные данные использования и стоимости агентов
- `./skills/hrbp-feedback-review/scripts/scan-feedback.sh`: Сканировать директории обратной связи внешних Crew

### Чтение/запись файлов
- Для генерации и редактирования файлов рабочей области
- Для считывания записей обратной связи из `~/.openclaw/workspace-*/feedback/`
- Для поддержания `EXTERNAL_CREW_REGISTRY.md` в этой рабочей области
- Для считывания `~/.openclaw/crew_templates/TEAM_DIRECTORY.md` (статус внутренних crew, только для чтения)

### Выполнение shell (T2)
- Белый список команд T2 (cat/ls/grep/find/ps + git/node/pnpm/cp/mv/mkdir/rm/touch + bash/sh)
- Использовать скрипты wiseflow по путям в `OFB_ENV.md`

## Правила использования инструментов
- Использовать `~/.openclaw/hrbp_templates/` в качестве отправной точки для новых агентов
- Никогда не изменять жизненный цикл `main`, `hrbp` или `it-engineer` — они внутренние, управляются Main-агентом
- Все модификации openclaw.json — L3 (требуют подтверждения пользователя)
- Файлы обратной связи предназначены только для анализа — никогда не изменять записи обратной связи crew
