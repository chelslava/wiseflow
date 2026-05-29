# Поиск на платформах SOSMedia

## bilibili (Bilibili, сокр. B-站):

https://search.bilibili.com/{channel}?keyword={keyword}

Если ключевых слов несколько, соединяйте их через `+`

Допустимые `channel`:
- Комплексный: all
- Видео: video
- Фанби (аниме/манга): bangumi
- Фильмы и сериалы: pgc
- Прямые трансляции: live
- Статьи: article
- Пользователи (UP): upuser

Для каждого `channel` можно указать дополнительные правила поиска (если не указано, используется значение по умолчанию):

### all

Поддерживаемые правила:
- Больше всего просмотров: &order=click
- Самые новые: &order=pubdate
- Больше всего комментариев: &order=dm
- Больше всего избранного: &order=stow

Примеры:
- Поиск по умолчанию: https://search.bilibili.com/all?keyword=%E8%8D%AF%E5%B1%8B%E5%B0%91%E5%A5%B3%E7%9A%84%E5%91%A2%E5%96%83
- Сортировка по просмотрам: https://search.bilibili.com/all?keyword=%E8%8D%AF%E5%B1%8B%E5%B0%91%E5%A5%B3%E7%9A%84%E5%91%A2%E5%96%83&order=click

### video

Поддерживаемые правила:
- Больше всего просмотров: &order=click
- Самые новые: &order=pubdate
- Больше всего комментариев: &order=dm
- Больше всего избранного: &order=stow

Примеры:
- Поиск по умолчанию: https://search.bilibili.com/video?keyword=%E8%8D%AF%E5%B1%8B%E5%B0%91%E5%A5%B3%E7%9A%84%E5%91%A2%E5%96%83
- Сортировка по newest: https://search.bilibili.com/video?keyword=%E8%8D%AF%E5%B1%8B%E5%B0%91%E5%A5%B3%E7%9A%84%E5%91%A2%E5%96%83&order=pubdate

### bangumi

Этот канал не поддерживает дополнительные правила поиска, работает только поиск по умолчанию

### pgc

Этот канал не поддерживает дополнительные правила поиска, работает только поиск по умолчанию

### live

Поддерживаемые правила (по умолчанию ищет все):
- Поиск主播: &search_type=live_user
- Поиск комнаты: &search_type=live_room
- Поиск комнат по времени: &search_type=live_room&order=live_time

Примеры:
- Поиск по умолчанию (всё): https://search.bilibili.com/live?keyword=%E8%8D%AF%E5%B1%8B%E5%B0%91%E5%A5%B3%E7%9A%84%E5%91%A2%E5%96%83
- По комнатам по времени: https://search.bilibili.com/live?keyword=%E8%8D%AF%E5%B1%8B%E5%B0%91%E5%A5%B3%E7%9A%84%E5%91%A2%E5%96%83&search_type=live_room&order=live_time

### article

Поддерживаемые правила:
- Самые новые: &order=pubdate
- Больше всего просмотров: &order=click
- Самые популярные: &order=attention
- Больше всего комментариев: &order=scores

Примеры:
- Поиск по умолчанию: https://search.bilibili.com/article?keyword=%E8%8D%AF%E5%B1%8B%E5%B0%91%E5%A5%B3%E7%9A%84%E5%91%A2%E5%96%83
- Сортировка по комментариям: https://search.bilibili.com/article?keyword=%E8%8D%AF%E5%B1%8B%E5%B0%91%E5%A5%B3%E7%9A%84%E5%91%A2%E5%96%83&order=scores

### upuser

Поддерживаемые правила:
- By подписчиков (по убыванию): &order=fans
- By подписчиков (по возрастанию): &order=fans&order_sort=1
- По уровню会员 (по убыванию): &order=level
- По уровню会员 (по возрастанию): &order=level&order_sort=1

Примеры:
- Поиск по умолчанию: https://search.bilibili.com/upuser?keyword=%E8%8D%AF%E5%B1%8B%E5%B0%91%E5%A5%B3%E7%9A%84%E5%91%A2%E5%96%83
- По подписчикам (по убыванию): https://search.bilibili.com/upuser?keyword=%E8%8D%AF%E5%B1%8B%E5%B0%91%E5%A5%B3%E7%9A%84%E5%91%A2%E5%96%83&order=fans

## douyin (TikTok, сокр. dy):

- Комплексный поиск: https://www.douyin.com/search/{keyword}?type=general
- Поиск видео: https://www.douyin.com/search/{keyword}?type=video
- Поиск пользователей: https://www.douyin.com/search/{keyword}?type=user
- Поиск прямых трансляций: https://www.douyin.com/search/{keyword}?type=live

Если несколько ключевых слов, используйте %20 между ними, например: https://www.douyin.com/search/wiseflow%20%E8%B4%9F%E9%9D%A2

По умолчанию `type` — комплексный поиск

При необходимости сортировки результатов или фильтрации необходимо использовать интерактивные действия в браузере

## weibo (Weibo, сокр. wb):

- Комплексный поиск: https://s.weibo.com/weibo/{keyword}
- Поиск в реальном времени (новейшие): https://s.weibo.com/realtime?q={keyword}
- Поиск пользователей: https://s.weibo.com/user?q={keyword}
- Поиск тем: https://s.weibo.com/topic?q={keyword}

## xiaohongshu (Xiaohongshu, сокр. xhs, также известна как "红薯"):

https://www.xiaohongshu.com/search_result?keyword={keyword}&source=web_explore_feed

Конкретные каналы, фильтры и параметры сортировки требуют интерактивных действий в браузере

## zhihu:

- Комплексный: https://www.zhihu.com/search?type=content&q={keyword}
- Пользователи (поиск человека): https://www.zhihu.com/search?q={keyword}&type=people
- Статьи: https://www.zhihu.com/search?q={keyword}&type=scholar
- Колонки: https://www.zhihu.com/search?q={keyword}&type=column
- Электронные книги: https://www.zhihu.com/search?q={keyword}&type=publication
- Круги: https://www.zhihu.com/search?q={keyword}&type=ring
- Темы: https://www.zhihu.com/search?q={keyword}&type=topic
- Видео: https://www.zhihu.com/search?q={keyword}&type=zvideo

Если несколько ключевых слов, используйте %20 между ними, например: https://www.zhihu.com/search?type=zvideo&q=wiseflow%20%E4%BB%98%E8%B4%B9

Следующие параметры можно формировать через конструирование URL:

### Комплексный

Базовый: https://www.zhihu.com/search?type=content&q={keyword}

- Фильтры:
  - Только ответы: &type=content&vertical=answer
  - Только статьи: &type=content&vertical=article
  - Только видео: &type=content&vertical=zvideo

- Сортировка:
  - By количеству одобрений: &sort=upvoted_count
  - By новизне: &sort=created_time

- Ограничение по времени:
  - За день: &time_interval=a_day
  - За неделю: &time_interval=a_week
  - За месяц: &time_interval=a_month
  - За три месяца: &time_interval=three_months
  - За полгода: &time_interval=half_a_year
  - За год: &time_interval=a_year

Можно комбинировать, например: https://www.zhihu.com/search?q=wiseflow%20%E4%BB%98%E8%B4%B9&sort=created_time&time_interval=a_month&type=content&vertical=article

## twitter (X, Twitter):

- TOP (рейтинг): https://x.com/search?q={keyword}
- Latest (новейшие): https://x.com/search?q={keyword}&f=live
- People (поиск человека): https://x.com/search?q={keyword}&f=user
- Media (медиа): https://x.com/search?q={keyword}&f=media
- Lists (списки): https://x.com/search?q={keyword}&f=list

Если несколько ключевых слов, используйте %20 между ними, например: https://x.com/search?q=wiseflow%20%E8%BD%AF%E4%BB%B6&src=typed_query&f=list

Можно добавить опцию "Near You" (рядом с вами): &lf=on, например: https://x.com/search?q=wiseflow&f=live&lf=on

## facebook (FB, Facebook):

- ALL (все): https://www.facebook.com/search/top/?q={keyword}
- People (поиск человека): https://www.facebook.com/search/people/?q={keyword}
- Pages (страницы): https://www.facebook.com/search/pages?q={keyword}
- Groups (группы): https://www.facebook.com/search/groups?q={keyword}
- Events (мероприятия): https://www.facebook.com/search/events?q={keyword}

Если несколько ключевых слов, используйте %20 между ними, например: https://www.facebook.com/search/top/?q=jinchen%20%E4%BD%8F%E5%8F%8B

Параметры поиска и фильтрация требуют интерактивных действий в браузере

## github

- Репозитории: https://github.com/search?q={keyword}&type=repositories
- Пользователи: https://github.com/search?q={keyword}&type=users
- Issues: https://github.com/search?q={keyword}&type=issues
- Pull Requests: https://github.com/search?q={keyword}&type=pullrequests
- Code: https://github.com/search?q={keyword}&type=code
- Discussions: https://github.com/search?q={keyword}&type=discussions
- Wikis: https://github.com/search?q={keyword}&type=wikis
- Topics: https://github.com/search?q={keyword}&type=topics

Если несколько ключевых слов, используйте + между ними, например: https://github.com/search?q=wiseflow+addon&type=topics

### Репозитории — поддерживаемые условия поиска:

- Больше всего звёзд: &s=stars&o=desc
- Меньше всего звёзд: &s=stars&o=asc
- Больше всего фORKов: &s=forks&o=desc
- Меньше всего фORKов: &s=forks&o=asc
- Недавно обновлённые: &s=updated&o=desc
- Недавно обновлённые (самые новые): &s=updated&o=asc

### Пользователи — поддерживаемые условия поиска:

- Больше всего подписчиков: &s=followers&o=desc
- Меньше всего подписчиков: &s=followers&o=asc
- Присоединившиеся недавно: &s=joined&o=desc
- Не присоединившиеся: &s=joined&o=asc
- Больше всего репозиториев: &s=repositories&o=desc
- Меньше всего репозиториев: &s=repositories&o=asc

Репозитории и поиск пользователей также поддерживают фильтрацию по языку: &l=HTML

Например: https://github.com/search?q=wiseflow+language%3AHTML&type=users&s=repositories&o=desc&l=HTML

Поддерживаемые языки для фильтрации: HTML, CSS, JavaScript, Python, Ruby, Java, C++, PHP, Swift, Go, Kotlin, TypeScript, Rust, Scala, Haskell, Lua, Shell, Dockerfile, JSON, YAML, Markdown, SVG