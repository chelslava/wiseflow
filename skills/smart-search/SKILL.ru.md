---
name: smart-search
description: Создание оптимизированных URL-поиска для основных платформ и переход к результатам с помощью браузера. Заменяет встроенный инструмент web_search для целевых, специфичных для платформ поисков.
metadata:
  {
    "openclaw":
      {
        "emoji": "🔍",
        "always": true,
      }
  }
---

# Руководство по умному поиску

Используйте этот навык всякий раз, когда пользователь просит вас найти информацию в в Интернете или на конкретной платформе. **Создайте URL-поиска напрямую и перейдите по нему**, вместо использования встроенного инструмента `web_search`.

## Кодирование ключевых слов

- **Пробелы**: используйте `+` для Bing, GitHub, Bilibili; используйте `%20` для Douyin, Twitter, Facebook, Zhihu; для Baidu, Quark, YouTube подойдет любой вариант
- **Специальные символы**: URL-кодируйте их (например, `#` → `%23`, `&` → `%26`, `?` → `%3F`)
- **Китайские символы**: URL-кодируйте (браузеры обрабатывают это автоматически при переходе)

---

## Прогрев cookie — КРИТИЧЕСКИ важный этап для аутентифицированных платформ

Многие платформы вернут пустые результаты или перенаправят на страницу входа, если вы переходите **непосредственно** к URL-поиска, не посетив сначала главную страницу. Всегда проводите прогрев сессии в два этапа:

| Платформа | Шаг 1 (прогрев) | Шаг 2 (поиск) |
|----------|-----------------|-----------------|
| 知乎 | Перейти на `https://www.zhihu.com` | Перейти на URL-поиска |
| Reddit | Перейти на `https://www.reddit.com` | Перейти на URL-поиска |
| 微博 | Перейти на `https://weibo.com` | Перейти на URL-поиска |
| YouTube | Перейти на `https://www.youtube.com` | Перейти на URL-поиска |
| 雪球 | Перейти на `https://xueqiu.com` | Перейти на URL-поиска |
| 路透社 | Перейти на `https://www.reuters.com` | Перейти на URL-поиска |
| Bilibili | Перейти на `https://www.bilibili.com` | Перейти на URL-поиска |
| 小红书 | Перейти на `https://www.xiaohongshu.com` | Перейти на URL-поиска |
| TikTok | Перейти на `https://www.tiktok.com` | Перейти на URL-поиска |

**Платформы, которым НЕ требуется прогрев** (публичные API / без аутентификации):
- Google, Bing, Baidu, Quark, GitHub, arXiv, Wikipedia, BBC, HackerNews, V2EX, Tieba, Amazon

---

## Общий веб-поиск

### Bing (рекомендуется)

```
https://www.bing.com/search?q={keyword}
```

Фильтры по времени (добавьте к URL):
- Последние 24 часа: `&filters=ex1:"ez1"`
- Последняя неделя: `&filters=ex1:"ez2"`
- Последний месяц: `&filters=ex1:"ez3"`

Пагинация: `&first={offset}`, где offset = (page − 1) × 10 + 1

### Bing News

```
https://www.bing.com/news/search?q={keyword}
```

### Bing Images

```
https://www.bing.com/images/search?q={keyword}
```

Фильтры по времени для изображений: `&qft=filterui:age-lt{minutes}`, где minutes = 1440 (день) / 10080 (неделя) / 44640 (месяц) / 525600 (год)

### Baidu (резервный вариант)

Общий веб-поиск:
```
https://www.baidu.com/s?wd={keyword}
```

Изображения Baidu:
```
https://image.baidu.com/search/index?tn=baiduimage&fm=result&ie=utf-8&word={keyword}
```

### Quark / 夸克 (резервный вариант)

```
https://quark.sm.cn/s?q={keyword}
```

---

## Академические и справочные ресурсы

### arXiv (предпечатные статьи: CS, Physics, Math, Biology, Economics и др.)

Поиск в браузере:
```
https://arxiv.org/search/?searchtype=all&query={keyword}
```

Поиск по конкретному полю:
- Заголовок: `?searchtype=ti&query={keyword}`
- Автор: `?searchtype=au&query={keyword}`
- Аннотация: `?searchtype=abs&query={keyword}`
- Категория (например, cs.AI): `?searchtype=all&query={keyword}&searchtype=all&start=0`

Сортировка по свежести: добавьте `&order=-announced_date_first`

arXiv API (возвращает структурированный XML — полезно для программного доступа):
```
https://export.arxiv.org/api/query?search_query=all:{keyword}&max_results=10
```

### 百度学术 (Baidu Scholar)

```
https://xueshu.baidu.com/s?wd={keyword}
```

Множественные ключевые слова: объедините с помощью `+`. Прогрев не требуется.

### 万方数据 (Wanfang Data)

```
https://s.wanfangdata.com.cn/paper?q={keyword}
```

Китайские академические статьи, диссертации и материалы конференций. Прогрев не требуется.

### Wikipedia

Английская:
```
https://en.wikipedia.org/w/index.php?search={keyword}
```

Китайская (中文):
```
https://zh.wikipedia.org/w/index.php?search={keyword}
```

Другие языки: замените языковой код (например, `de`, `fr`, `ja`, `ko`, `es`).

---

## Видеоплатформы

### YouTube

```
https://www.youtube.com/results?search_query={keyword}
```

Фильтры по времени (добавьте к URL):
- Последний час: `&sp=EgIIAQ%3D%3D`
- Сегодня: `&sp=EgIIAg%3D%3D`
- Эта неделя: `&sp=EgIIAw%3D%3D`
- Этот месяц: `&sp=EgIIBA%3D%3D`
- Этот год: `&sp=EgIIBQ%3D%3D`

Фильтры по типу (добавьте к URL, нельзя комбинировать с фильтрами времени/сортировки):
- Только видео: `&sp=EgIQAQ%3D%3D`
- Только Shorts: `&sp=EgIQCQ%3D%3D`
- Только каналы: `&sp=EgIQAg%3D%3D`
- Только плейлисты: `&sp=EgIQAw%3D%3D`

Опции сортировки (добавьте к URL, нельзя комбинировать с фильтрами типа):
- По дате загрузки: `&sp=CAI%3D`
- По количеству просмотров: `&sp=CAM%3D`
- По рейтингу: `&sp=CAE%3D`

> **Примечание**: `sp=` принимает только одно значение — фильтры типа, времени и сортировки взаимно исключают друг друга. Используйте наиболее релевантный.

Множественные ключевые слова: объедините с помощью `+` (например, `wiseflow+AI+поиск`)

---

## Китайские социальные сети

### 哔哩哔哩 (Bilibili / B站)

```
https://search.bilibili.com/{channel}?keyword={keyword}
```

Каналы: `all` (综合) | `video` (视频) | `bangumi` (番剧) | `pgc` (影视) | `live` (直播) | `article` (专栏) | `upuser` (UP主)

Опции сортировки для `all` и `video`:
- Больше всего просмотров: `&order=click`
- Новейшие: `&order=pubdate`
- Больше всего danmaku: `&order=dm`
- Больше всего в избранном: `&order=stow`

Опции сортировки для `live`:
- Только стримеры: `&search_type=live_user`
- Только live комнаты: `&search_type=live_room`
- Live комнаты по времени начала: `&search_type=live_room&order=live_time`

Опции сортировки для `upuser`:
- Больше всего подписчиков (убывание): `&order=fans`
- Меньше всего подписчиков (возрастание): `&order=fans&order_sort=1`
- Самый высокий уровень: `&order=level`

Опции сортировки для `article`:
- Новейшие: `&order=pubdate`
- Больше всего кликов: `&order=click`
- Самые популярные: `&order=attention`
- Больше всего комментариев: `&order=scores`

Множественные ключевые слова: объедините с помощью `+`

### 抖音 (Douyin / TikTok China)

```
https://www.douyin.com/search/{keyword}?type={type}
```

Типы: `general` (综合, по умолчанию) | `video` (视频) | `user` (用户) | `live` (直播)

Множественные ключевые слова: объедините с помощью `%20` (например, `wiseflow%20AI`)

Для опций сортировки и фильтрации: взаимодействуйте с пользовательским интерфейсом страницы после перехода.

### 微博 (Weibo)

- Общий: `https://s.weibo.com/weibo/{keyword}`
- В реальном времени / Последние: `https://s.weibo.com/realtime?q={keyword}`
- Пользователи: `https://s.weibo.com/user?q={keyword}`
- Темы: `https://s.weibo.com/topic?q={keyword}`

### 小红书 (Xiaohongshu / RED / 红薯)

```
https://www.xiaohongshu.com/search_result?keyword={keyword}&source=web_search_result_notes
```

> **Примечание**: Используйте `source=web_search_result_notes` (не `web_explore_feed`), чтобы получить результаты поиска, а не ленту рекомендаций.
> После перехода дождитесь ~3 секунды и прокрутите вниз дважды — результаты загружаются лениво.

Для выбора канала, фильтрации и сортировки: взаимодействуйте с пользовательским интерфейсом страницы после перехода.

### 知乎 (Zhihu)

```
https://www.zhihu.com/search?type=content&q={keyword}
```

Типы контента: `content` (综合) | `people` (用户) | `scholar` (论文) | `column` (专栏) | `publication` (电子书) | `ring` (圈子) | `topic` (话题) | `zvideo` (视频)

Фильтры для комплексного поиска (`type=content`):
- Только ответы: `&vertical=answer`
- Только статьи: `&vertical=article`
- Только видео: `&vertical=zvideo`

Сортировка:
- Больше всего голосов "за": `&sort=upvoted_count`
- Новейшие: `&sort=created_time`

Диапазон времени:
- Последний день: `&time_interval=a_day`
- Последняя неделя: `&time_interval=a_week`
- Последний месяц: `&time_interval=a_month`
- Последние 3 месяца: `&time_interval=three_months`
- Последние 6 месяцев: `&time_interval=half_a_year`
- Последний год: `&time_interval=a_year`

Пример — новейшие статьи за последний месяц:
```
https://www.zhihu.com/search?type=content&q={keyword}&vertical=article&sort=created_time&time_interval=a_month
```

Множественные ключевые слова: объедините с помощью `%20`

---

### 百度贴吧 (Tieba)

```
https://tieba.baidu.com/f/search/res?qw={keyword}&ie=utf-8
```

Поиск в конкретном форуме (吧):
```
https://tieba.baidu.com/f/search/res?qw={keyword}&kw={forum_name}&ie=utf-8
```

> **Примечание**: Публичный контент, прогрев не требуется. Надежно доступна только первая страница результатов. Множественные ключевые слова: URL-кодируйте пробелы как `%20`.

---

## Международные социальные сети

### Twitter / X

- Топ результаты: `https://x.com/search?q={keyword}`
- Последние: `https://x.com/search?q={keyword}&f=live`
- Люди: `https://x.com/search?q={keyword}&f=user`
- Медиа: `https://x.com/search?q={keyword}&f=media`
- Списки: `https://x.com/search?q={keyword}&f=list`

Добавьте фильтр "Рядом с вами": добавьте `&lf=on`

> **Примечание**: Twitter/X использует интенсивную клиентскуюside-рендеринг. После перехода подождите не менее **5 секунд** перед снимком экрана, чтобы убедиться, что контент твитов загружен.

Множественные ключевые слова: объедините с помощью `%20`

### Facebook

- Все: `https://www.facebook.com/search/top/?q={keyword}`
- Люди: `https://www.facebook.com/search/people/?q={keyword}`
- Страницы: `https://www.facebook.com/search/pages?q={keyword}`
- Группы: `https://www.facebook.com/search/groups?q={keyword}`
- События: `https://www.facebook.com/search/events?q={keyword}`

Для опций фильтрации и сортировки: взаимодействуйте с пользовательским интерфейсом страницы после перехода.

Множественные ключевые слова: объедините с помощью `%20`

### Reddit

```
https://www.reddit.com/search/?q={keyword}
```

Опции сортировки: `&sort=relevance` | `hot` | `top` | `new` | `comments`

Фильтр по времени (для `sort=top`): `&t=hour` | `day` | `week` | `month` | `year` | `all`

Поиск в конкретном сабреддите:
```
https://www.reddit.com/r/{subreddit}/search/?q={keyword}&restrict_sr=on&sort=relevance&t=all
```

Множественные ключевые слова: объедините с помощью `+`

### TikTok (международная версия)

```
https://www.tiktok.com/search?q={keyword}
```

> **Примечание**: Требуется прогрев cookie — сначала перейдите на `https://www.tiktok.com`. Подождите ~3 секунды после перехода к результатам поиска для загрузки контента.

Множественные ключевые слова: объедините с помощью `%20`

---

## Платформы для разработчиков

### GitHub

```
https://github.com/search?q={keyword}&type={type}
```

Типы: `repositories` | `users` | `code` | `issues` | `pullrequests` | `discussions` | `topics` | `wikis`

Опции сортировки для **репозиториев**:
- Больше всего звезд: `&s=stars&o=desc`
- Меньше всего звезд: `&s=stars&o=asc`
- Больше всего форков: `&s=forks&o=desc`
- Недавно обновленные: `&s=updated&o=desc`

Опции сортировки для **пользователей**:
- Больше всего подписчиков: `&s=followers&o=desc`
- Больше всего репозиториев: `&s=repositories&o=desc`
- Недавно присоединившиеся: `&s=joined&o=desc`

Фильтр по языку (для `repositories` и `users`): `&l={language}` (например, `&l=Python`, `&l=TypeScript`, `&l=Go`)

Множественные ключевые слова: объедините с помощью `+`

Пример:
```
https://github.com/search?q=wiseflow+addon&type=repositories&s=stars&o=desc&l=Python
```

### LinkedIn

Поиск工作 (требуется прогрев cookie — сначала перейдите на `https://www.linkedin.com`):
```
https://www.linkedin.com/jobs/search/?keywords={keyword}&location={location}
```

Поиск пользователей:
```
https://www.linkedin.com/search/results/people/?keywords={keyword}
```

Поиск компаний:
```
https://www.linkedin.com/search/results/companies/?keywords={keyword}
```

Множественные ключевые слова: объедините с помощью `%20`

---

## После перехода

1. Сделайте снимок, чтобы убедиться, что результаты загружены.
2. Если появляется CAPTCHA, стена входа или проверка подлинности, следуйте навыку **browser-guide**.
3. Извлеките соответствующую информацию из видимых результатов.
4. Если нужны дополнительные результаты, перейдите на следующую страницу, используя:
   - Изменение параметра пагинации в URL, или
   - Нажатие кнопки "Следующая страница" на странице.
5. Закройте вкладку сразу после извлечения всей необходимой информации.

---

## Правительственные ресурсы и политика

### 国务院政策搜索 (Поиск политики Госсовета Китая)

```
https://sousuo.www.gov.cn/sousuo/search.shtml?code=17da70961a7&dataTypeId=107&searchWord={keyword}
```

Ищет официальные политические документы, опубликованные на www.gov.cn. Прогрев не требуется.

---

## Финансовые платформы

### 雪球 (Xueqiu) — Акции и финансы

Поиск акций/символов (требуется прогрев cookie — сначала перейдите на `https://xueqiu.com`):
```
https://xueqiu.com/search?q={keyword}
```

Примеры запросов: `茅台`, `AAPL`, `腾讯`, `SH600519`

Страница деталей акции: `https://xueqiu.com/S/{symbol}` (например, `/S/SH600519`)

---

## Технические сообщества

### Hacker News (публичный, вход не требуется)

```
https://news.ycombinator.com/
```

Поиск через Algolia (неофициальный, но надежный):
```
https://hn.algolia.com/?q={keyword}
```

Сортировка по дате: `&dateRange=pastWeek` | `pastMonth` | `pastYear`

### V2EX (публичный, вход не требуется)

Поиск (подход с поиском по сайту Google, самый надежный):
```
https://www.google.com/search?q=site:v2ex.com+{keyword}
```

Или перейдите напрямую на V2EX и используйте встроенный поиск:
```
https://www.v2ex.com/?q={keyword}
```

---

## Новости

### Reuters

Поиск новостей (требуется прогрев cookie — сначала перейдите на `https://www.reuters.com`):
```
https://www.reuters.com/search/news?blob={keyword}
```

Множественные ключевые слова: объедините с помощью `+`

---

## Покупки

### Amazon

```
https://www.amazon.com/s?k={keyword}
```

Фильтр отдела (добавьте к URL): `&i={department}` — распространенные значения: `electronics`, `books`, `clothing-shoes-jewelry`, `grocery`, `toys-and-games`

Опции сортировки (добавьте к URL):
- Релевантность (по умолчанию): `&s=relevance-rank`
- Цена от низкой к высокой: `&s=price-asc-rank`
- Цена от высокой к низкой: `&s=price-desc-rank`
- Средний рейтинг покупателей: `&s=review-rank`
- Новейшие поступления: `&s=date-desc-rank`

> **Защита от ботов**: Перейдите и подождите не менее 2-3 секунд перед снимком экрана. Если вы столкнетесь со страницей верификации робота, не пытайтесь снова сразу — следуйте навыку **browser-guide**.

Множественные ключевые слова: объедините с помощью `+`
