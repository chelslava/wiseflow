# Бюллетень по исследованию решений для обхода обнаружения браузера

> Дата исследования: 2026-02-20
> Цель: Оценка двух решений, rebrowser-patches и patchright, для интеграции способности обхода обнаружения в OpenClaw

---

## I. Контекст проблемы

### 1.1 Почему локальным AI-ассистентам также требуется обход обнаружения

OpenClaw как локальный AI-ассистент выполняет пользовательские инструкции через браузер (сравнение цен, заполнение форм, чтение информации, поиск и т.д.). Целевые веб-сайты не различают «локального ассистента» и «злонамеренного скрейпера» — при обнаружении признаков автоматизации срабатывают механизмы защиты:

- Сравнение цен на电商 → триггер капчи или блокировка IP
- Отправка формы → отказ
- Банки/почтовые службы → требование повторной верификации из-за риска
- Поиск Google → CAPTCHA
- Управление бэкендом → блокировка WAF

### 1.2 Основные источники утечки данных Playwright

| Источник утечки | Принцип обнаружения | Скрытность |
|----------------|-------------------|-----------|
| CDP вызов `Runtime.enable` | Активация области Runtime вызывает изменения во внутреннем поведении браузера, скрипты антиспама могут обнаруживать их через побочные каналы | **Критически** |
| CDP вызов `Console.enable` | Аналогично утечке Runtime.enable через побочные каналы | Высокая |
| Аргумент запуска `--enable-automation` | Устанавливает `navigator.webdriver = true` | Высокая |
| `//# sourceURL=pptr:evaluate` | Инжект скрипта имеет характерную комментарий sourceURL | Средняя |
| `__playwright_utility_world__` | Имя utility world может быть обнаружено | Средняя |
| Init script через CDP | `Page.addScriptToEvaluateOnNewDocument` имеет метод обнаружения | Средняя |
| Собственный Chromium Playwright | Пользовательская версия браузера имеет отличия от оригинального Chrome по отпечатку | Средняя (не существует при connectOverCDP) |
| Множественные параметры запуска автоматизации | Комбинация параметров сама по себе является отпечатком | Низкая-Средняя |

### 1.3 Текущее состояние OpenClaw

- Используется `playwright-core@1.58.2`
- Уже используется `chromium.connectOverCDP()` для подключения к браузеру (не launch)
- В режиме Extension relay подключается к настоящему браузеру пользователя, без параметров запуска
- Однако **утечка данных от драйвера Playwright через CDP не обработана** (Runtime.enable, Console.enable и т.д.)

---

## II. Решение A: rebrowser-patches

### 2.1 Обзор проекта

- GitHub: `rebrowser/rebrowser-patches` (1,2k звёзд)
- Экосистема: rebrowser-playwright-core (drop-in замена), rebrowser-puppeteer и т.д.
- Метод: патчинг существующего исходного кода playwright-core (команда Unix `patch`)
- Поддержка: Puppeteer + Playwright

### 2.2 Патчи

#### Патч Runtime.enable — 3 режима

**Режим 1: `addBinding` (по умолчанию, рекомендуется)**

```
Принцип:
1. Генерация случайного имени binding (например, "x7k2m9q")
2. Регистрация через Runtime.addBinding (не требует Runtime.enable)
3. В изолированной области вызывание пользовательского события, инициирующего binding
4. Получение реального executionContextId из обратного вызова Runtime.bindingCalled
5. Использование этого contextId для последующего Runtime.evaluate

Преимущества:
- Полностью без вызова Runtime.enable
- Сохранение полного доступа к main world (чтение переменных страницы)
- Поддержка web workers и iframes
```

**Режим 2: `alwaysIsolated`**

```
Принцип: Все выполнения скриптов происходят в изолированном контексте, созданном Page.createIsolatedWorld

Преимущества: Полная изоляция, предотвращает обнаружение через MutationObserver
Недостатки: Нет доступа к переменным main world, не поддерживает web workers
```

**Режим 3: `enableDisable`**

```
Принцип: Быстрое enable → захват ID контекста → немедленное disable

Преимущества: Полный доступ к main world
Недостатки: Есть короткий временной окно, в течение которого может быть обнаружено
```

#### Другие патчи

| Патч | Эффект | Конфигурация |
|------|--------|--------------|
| Спуфинг sourceURL | `pptr:evaluate` → `app.js` (настраивается) | `REBROWSER_PATCHES_SOURCE_URL=jquery.min.js` |
| Имя utility world | `__puppeteer_utility_world__` → `util` (настраивается) | `REBROWSER_PATCHES_UTILITY_WORLD_NAME=util` |

### 2.3 Что не патчится

- Console.enable — не обрабатывается (также не отключается, сохранены полные функции)
- Способ инъекции init script — по-прежнему через CDP
- CSP — не обрабатывается
- Closed Shadow Root — не обрабатывается
- Параметры запуска — не изменяются (необходимо настраивать вручную)
- Имитация отпечатка — не входит в рамки

### 2.4 Способ конфигурации

```bash
# Переменные окружения (можно переключать во время выполнения)
REBROWSER_PATCHES_RUNTIME_FIX_MODE=addBinding    # По умолчанию
REBROWSER_PATCHES_RUNTIME_FIX_MODE=alwaysIsolated
REBROWSER_PATCHES_RUNTIME_FIX_MODE=enableDisable
REBROWSER_PATCHES_RUNTIME_FIX_MODE=0              # Отключить

REBROWSER_PATCHES_SOURCE_URL=app.js
REBROWSER_PATCHES_UTILITY_WORLD_NAME=util
REBROWSER_PATCHES_DEBUG=1
```

### 2.5 Способ интеграции

```bash
# Способ 1: Патч (после npm install требуется повторное выполнение)
npx rebrowser-patches@latest patch --packageName playwright-core

# Способ 2: Замена пакета (рекомендуется, навсегда)
# package.json:
#   "playwright-core": "1.58.2" → "rebrowser-playwright-core": "1.58.2"
# Не требует изменения путей импорта
```

Решение A показало определенные проблемы совместимости с openclaw.

OpenClaw использует playwright 1.58.2, но rebrowser-patches наиболее полно тестируется на playwright 1.52.0.

OpenClaw сильно зависит от приватных API playwright, например, `_snapshotForAI()`. После применения rebrowser-patches этот интерфейс не работает.

---

## III. Решение B: Patchright

### 3.1 Обзор проекта

- GitHub: `Kaliiiiiiiiii-Vinyzu/patchright` + `patchright-python` (1,1k звёзд)
- Метод: форк исходного кода Playwright, переписывание 22 ключевых модулей через ts-morph AST, компиляция в отдельный пакет
- Поддержка: только Playwright
- Автоматизация: ежечасная проверка новых версий Playwright, автоматический патчинг и публикация

### 3.2 Патчи

| Патч | Подробности |
|------|-------------|
| **Удаление Runtime.enable** | Прямое удаление вызовов из crPage, crDevTools, crServiceWorker |
| **Отключение Console.enable** | Полное удаление области Console |
| **Очистка параметров запуска** | Удаление 6 параметров, включая `--enable-automation`, добавление `--disable-blink-features=AutomationControlled` |
| **Инъекция init script** | Замена на HTTP route interception → вставка `<script>` в `<head>` HTML |
| **Обход CSP** | Автоматическое изменение Content-Security-Policy, добавление nonce/unsafe-inline |
| **Удаление sourceURL** | Удаление всех комментариев `//# sourceURL` |
| **Service Worker** | Тихое блокирование регистрации (удаление Console.warn для раскрытия информации) |
| **Closed Shadow Root** | Поддержка проникновения mode:'closed' Shadow DOM |
| **Модификация evaluate()** | Добавление параметра `isolated_context` (по умолчанию true) |

### 3.3 Стоимость

| Утраченные возможности | Влияние |
|-----------------------|---------|
| **Complete禁用 Console API** | `page.on("console")` никогда не срабатывает |
| **Отладка page.pause()** | Неизвестно, затронуто ли |
| **Сбор логов Console** | Требуется альтернативное решение (инъекция JS) |

### 3.4 Способ интеграции

```bash
# Требуется изменить пути импорта
# package.json:
#   "playwright-core": "1.58.2" → "patchright-core": "1.57.0"
# Все исходные коды:
#   import { chromium } from "playwright-core" → import { chromium } from "patchright-core"
```

Уже внедрено в этом репозитории (2026-02-22):

- `openclaw/package.json` переключено на `patchright-core@1.57.0` (самая новая доступная версия на npm)
- Все импорты `playwright-core` в `openclaw/src/browser/*` заменены на `patchright-core`
- `scripts/dev.sh` / `scripts/apply-patches.sh` удалены автоматические процессы патчинга rebrowser
- Изменения верхнего уровня сгенерированы в виде отдельного патча: `patches/001-switch-playwright-to-patchright-core.patch`

### 3.5 Известные проблемы (из GitHub Issues)

- `#94` Новые версии наоборот обнаруживаются (dist-info открыт)
- `#100` Ошибка Cloudflare 403
- `#101` Триггер Google Anti-Bot
- `#170` Sannysoft обнаруживает patchright

---

## IV. Сравнение решений

### 4.1 Ключевые различия

| Параметр | rebrowser-patches | Patchright |
|----------|-------------------|-----------|
| **Способ патчинга** | Патчинг во время выполнения / drop-in пакет | Компиляция на основе форка с переписыванием |
| **Глубина изменений** | Низкая — можно отменить одной командой | Высокая — требуется изменение всех импортов |
| **Runtime.enable** | 3 режима на выбор, по умолчанию addBinding | Единое решение — изолированный контекст |
| **Console API** | **Сохранено** | **Отключено** |
| **Доступ к main world** | ✅ addBinding полностью сохраняет | ⚠️ isolated context имеет ограничения |
| **Инъекция init script** | Не изменено (по-прежнему через CDP) | ✅ Изменено на HTML инъекцию |
| **Обход CSP** | Не обрабатывается | ✅ Автоматически |
| ** penetrability Closed Shadow Root** | Не обрабатывается | ✅ Поддерживается |
| **Очистка параметров запуска** | Не обрабатывается (необходима самостоятельная настройка) | ✅ Автоматически |
| **Гибкость конфигурации** | Переключение во время выполнения через переменные окружения | Зафиксировано на этапе компиляции |
| **Совместимость с `_snapshotForAI`** | Вероятно, совместимо (малая область изменений) | Высокий риск (широкая переписка) |
| **Сбор консольных данных OpenClaw** | ✅ Не затрагивается | ❌ Требуется модификация |
| **Зависимости GitHub dependents** | Есть экосистема drop-in замены | 0 известных проектов-зависимостей |
| **Проход率 обхода обнаружения** | Не опубликованы полные тесты | Гарантирует проход率 Cloudflare/Kasada/Datadome (но есть отзывы о неудачах в issues) |

### 4.2 Оценка применимости

| Сценарий | Рекомендуемое решение | Причины |
|----------|-----------------------|---------|
| **Интеграция OpenClaw (приоритет)** | rebrowser-patches | Сохранение Console API, малые изменения, низкий риск, возможность отмены |
| **Максимально агрессивный обход обнаружения** | Patchright | HTML инъекция init script + обход CSP + penetrability closed shadow root |
| **Быстрая проверка жизнеспособности** | rebrowser-patches | Однократная команда для патчинга, без изменения кода |
| **Долгосрочное сопровождение** | Оба решения | rebrowser имеет drop-in пакет; patchright автоматически отслеживает верхний уровень |

---

## V. План интеграции OpenClaw

### 5.1 Phase 1: Минимальные изменения для проверки (rebrowser-patches)

**Цель**: Нулевое изменение кода, проверка базовой совместимости

```bash
cd openclaw
# Патчинг
npx rebrowser-patches@latest patch --packageName playwright-core

# Установка переменных окружения
export REBROWSER_PATCHES_RUNTIME_FIX_MODE=addBinding
export REBROWSER_PATCHES_SOURCE_URL=app.js
export REBROWSER_PATCHES_UTILITY_WORLD_NAME=util

# Запуск OpenClaw и тестирование
```

**Чек-лист проверки**:
- [ ] OpenClaw нормально запускается
- [ ] `_snapshotForAI()` работает нормально
- [ ] Событие `page.on("console")` нормально срабатывает
- [ ] `page.evaluate()` нормально выполняется
- [ ] Навигация страницы и взаимодействие с элементами нормально
- [ ] Режим Extension relay работает нормально
- [ ] Режим без расширения нормально работает

### 5.2 Phase 2: Квантовая оценка эффективности обхода обнаружения

**Цель**: Количественная оценка улучшения обхода обнаружения

Направление тестирования по сайтам выявления:

| Сайт выявления | URL | Тестовые пункты |
|---------------|-----|----------------|
| CreepJS | `https://nicepkg.github.io/nicepkg-test/` | Комбинированный отпечаток |
| Sannysoft | `https://bot.sannysoft.com/` | navigator.webdriver и т.д. |
| Incolumitas | `https://bot.incolumitas.com/` | Расширенные методы обнаружения |
| Browserscan | `https://browserscan.net/` | Отпечаток браузера |
| Pixelscan | `https://pixelscan.net/` | Согласованность отпечатка |

**Матрица тестирования** (6 комбинаций):

```
                           Без патча    rebrowser-patches
Режим без расширения:           A1              A2
Расширение + настоящий Chrome:   B1              B2
```

**Сравнительные показатели**:
- Оценки/проход率各выявляя sites
- Значение `navigator.webdriver`
- Утечка Runtime.enable
- Trust Score CreepJS

### 5.3 Phase 3: Тестирование Patchright

**Цель**: Оценка дополнительных преимуществ Patchright и соответствующей стоимости

```bash
cd openclaw
# Замена пакета
# package.json: "playwright-core" → "patchright-core"
# Массовое изменение импорта (8 файлов)
# Модификация логики сбора консоли

# Повторение матрицы теста из Phase 2
```

**Дополнительная проверка**:
- [ ] Совместимость `_snapshotForAI()` (самый критичный)
- [ ] Надежность альтернативной схемы сбора консоли
- [ ] Дополнительная проход率 благодаря HTML инъекции init script

### 5.4 Phase 4: Подготовка к продакшену (на основе результатов Phase 2/3)

**Если выбран rebrowser-patches**:
```json
// package.json
{
  "dependencies": {
    "rebrowser-playwright-core": "1.58.2"  // замена playwright-core
  }
}
```

**Дополнительные укрепления** (независимо от выбора):
- [ ] Режим без расширения OpenClaw: добавление `--disable-blink-features=AutomationControlled` в аргументы запуска
- [ ] Режим без расширения OpenClaw: удаление `--enable-automation` и других параметров автоматизации
- [ ] Анализ прямого вызова `Runtime.enable` в `cdp.ts`, оценка возможности удаления
- [ ] Режим расширения: рассмотреть добавление `chrome.runtime.onStartup` для авто-переподключения

---

## VI. Очистка слоя raw CDP OpenClaw

Собственный `cdp.ts` OpenClaw напрямую отправляет CDP команды через WebSocket, обходя драйвер Playwright:

```typescript
// Эти вызовы не проходят через Playwright, не затрагиваются rebrowser-patches / patchright
send("Runtime.enable")
send("Runtime.evaluate", { expression, awaitPromise })
send("Runtime.terminateExecution")
```

**Требуется оценка**:
1. Можно ли удалить `Runtime.enable`? В сценариях только `Runtime.evaluate` некоторые версии Chrome не требуют сначала enable
2. Требует ли `Runtime.terminateExecution` предварительного enable? Требуется тестирование
3. Если обязательное enable необходимо, можно использовать режим enableDisable rebrowser-patches (быстрое enable → получение contextId → немедленное disable)

---

## VII. Карта областей воздействия

После завершения модификации, области воздействия двух режимов:

### Режим без расширения (rebrowser-patches + очистка параметров)

```
✅ Устранено:
  - Утечка Runtime.enable (режим addBinding rebrowser-patches)
  - Функция sourceURL (rebrowser-patches)
  - Имя utility world (rebrowser-patches)
  - navigator.webdriver (--disable-blink-features=AutomationControlled)
  - Параметр --enable-automation

⚠️ Все еще присутствует:
  - Параметр --remote-debugging-port (обязательный)
  - Console.enable (не обрабатывается rebrowser-patches)
  - Отпечаток Chromium, поставляемого с Playwright (если не используется connectOverCDP)
  - Runtime.enable в cdp.ts OpenClaw (требуется отдельная очистка)
  - Init script через CDP инъекцию (не изменено rebrowser-patches)
```

### Режим расширения + настоящий браузер (rebrowser-patches + очистка параметров)

```
✅ Устранено:
  - Утечка Runtime.enable
  - Функция sourceURL
  - Имя utility world
  - navigator.webdriver
  - Параметр --remote-debugging-port (расширение не требует)
  - Параметр --enable-automation (расширение не требует)
  - Различие отпечатка браузера (настоящий Chrome)
  - Пустой Profile (настоящий пользовательский Profile)
  - Все параметры автоматизации (нулевые параметры)

⚠️ Все еще присутствует:
  - Console.enable
  - Runtime.enable в cdp.ts OpenClaw (требуется отдельная очистка)
  - Init script через CDP инъекцию
  - Расширение Chrome может быть обнаружено (расширение можно видеть в chrome://extensions)
  - Лента отладки chrome.debugger (недоступна для JS страницы, но видима для пользователя)
```

Если заменить rebrowser-patches на Patchright, можно также устранить Console.enable и инъекцию init script, но с ценой потери Console API и более высокого риска совместимости.

---

## VIII. Индекс справочных файлов

### Исходный код проекта

| Проект | Ключевые файлы | Назначение |
|--------|--------------|-----------|
| OpenClaw | `src/browser/pw-session.ts:335` | Точка входа в `chromium.connectOverCDP()` |
| OpenClaw | `src/browser/pw-session.ts:217-283` | Слушатели событий страницы (включая console) |
| OpenClaw | `src/browser/pw-tools-core.snapshot.ts:54-62` | Вызов `_snapshotForAI()` |
| OpenClaw | `src/browser/extension-relay.ts` | Сервер relay расширения |
| OpenClaw | `assets/chrome-extension/background.js` | Основная логика расширения |
| OpenClaw | `src/browser/cdp.ts` | Слой raw CDP (содержит Runtime.enable) |
| rebrowser-patches | `patches/playwright-core/src.patch` | Патчи ядра Playwright |
| rebrowser-patches | `scripts/patcher.js` | Скрипт применения патчей |
| Patchright | `patchright_driver_patch.js` | Главный сценарий компиляции |
| Patchright | `driver_patches/crPagePatch.js` | Наибольший патч (~470 строк) |
| Patchright | `driver_patches/crNetworkManagerPatch.js` | Патч HTML-инъекции (~465 строк) |

### Ссылки по принципам обнаружения

| Вектор обнаружения | Описание |
|------------------|----------|
| Утечка Runtime.enable | Изменения внутреннего поведения браузера после активации области CDP, могут быть обнаружены JS.page по побочному каналу |
| `navigator.webdriver` | `--enable-automation` устанавливает это свойство в true |
| Отпечаток sourceURL | Комментарий `//# sourceURL=pptr:evaluate` в инжектном скрипте раскрывает фреймворк автоматизации |
| Обнаружение utility world | Имя исполняемого контекста `__playwright_utility_world__` может быть перечислено |
| Отпечаток бинарного файла Chrome | UA/WebGL/внутренние API браузера, поставляемого с Playwright, отличаются от оригинального Chrome |
| Отпечаток аргументов запуска | Комбинация большого количества `--disable-*` аргументов является признаком автоматизации |
| Пустой Profile | Отсутствие истории, Cookie, пустой localStorage — сильный сигнал автоматизации |

---

## IX. План экспериментов на завтра

### Приоритетное упорядочение

1. **Базовая проверка rebrowser-patches** (Phase 1) — 30 минут
   - Патчинг → Запуск OpenClaw → Базовые тесты функций

2. **Квантовая оценка эффективности обхода обнаружения** (Phase 2) — 1 час
   - 6 комбинаций × 5 сайтов выявления = 30 тестов

3. **Проверка Patchright** (Phase 3, если Phase 2 недостаточна) — 1-2 часа
   - Замена пакета → Тесты совместимости → Тесты обхода обнаружения

4. **Оценка очистки cdp.ts** (Phase 4) — в зависимости от результатов первых шагов

### Ожидаемые выводы

- Если режим addBinding rebrowser-patches сможет пройти основные сайты выявления и функции OpenClaw работают нормально → выбор rebrowser-patches
- Еслиrebrowser-patches недостаточен, а Patchright пройдет ключевые выявления → оценить затраты совместимости Patchright
- Если оба недостаточны → рассмотреть комбинированный подход (rebrowser-patches + дополнительные JS инъекции для допо)