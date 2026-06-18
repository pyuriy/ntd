## Як парсити RSS-фід

**✅ Як парсити RSS-фід (покроково)**

Ось найпрактичніші способи парсингу RSS-стрічок. Найзручніший і найпоширеніший варіант — **Python + бібліотека `feedparser`**.

### 1. Найпростіший спосіб: Python + feedparser

```bash
pip install feedparser
```

#### Готовий код для парсингу `https://ntdtv.com.ua/feed`

```python
import feedparser
from datetime import datetime, timedelta, timezone
import json

# === Налаштування ===
FEED_URL = "https://ntdtv.com.ua/feed"
DAYS_BACK = 2                    # скільки днів назад брати
OUTPUT_FILE = "ntd_recent.json"  # куди зберегти результат

def parse_rss_feed(url):
    feed = feedparser.parse(url)
    
    if feed.bozo:  # помилка парсингу
        print("Помилка при парсингу фіду")
        return []
    
    articles = []
    now = datetime.now(timezone.utc)
    cutoff_date = now - timedelta(days=DAYS_BACK)
    
    for entry in feed.entries:
        # Дата публікації
        pub_date = None
        if hasattr(entry, 'published_parsed') and entry.published_parsed:
            pub_date = datetime(*entry.published_parsed[:6], tzinfo=timezone.utc)
        elif hasattr(entry, 'updated_parsed') and entry.updated_parsed:
            pub_date = datetime(*entry.updated_parsed[:6], tzinfo=timezone.utc)
        
        # Фільтруємо за датою
        if pub_date and pub_date >= cutoff_date:
            article = {
                "title": entry.get("title", ""),
                "link": entry.get("link", ""),
                "published": entry.get("published", ""),
                "pub_date_iso": pub_date.isoformat() if pub_date else None,
                "category": entry.get("category", ""),
                "summary": entry.get("summary", "")[:300] + "..." if entry.get("summary") else "",
                "author": entry.get("author", "")
            }
            articles.append(article)
    
    return articles

# === Запуск ===
if __name__ == "__main__":
    articles = parse_rss_feed(FEED_URL)
    
    print(f"Знайдено {len(articles)} статей за останні {DAYS_BACK} дні:\n")
    
    for i, art in enumerate(articles, 1):
        print(f"{i}. {art['title']}")
        print(f"   📅 {art['published']}")
        print(f"   🔗 {art['link']}\n")
    
    # Зберігаємо в JSON
    with open(OUTPUT_FILE, "w", encoding="utf-8") as f:
        json.dump(articles, f, ensure_ascii=False, indent=2)
    
    print(f"✅ Результати збережено у файл: {OUTPUT_FILE}")
```

### 2. Що робить цей код

- Парсить фід
- Фільтрує статті **тільки за останні N днів**
- Витягує: заголовок, посилання, дату, категорію, короткий опис
- Зберігає результат у `ntd_recent.json`

### 3. Альтернативні способи

| Спосіб                    | Коли використовувати                  | Складність | Бібліотека          |
|---------------------------|---------------------------------------|------------|---------------------|
| **Python + feedparser**   | Найкращий варіант для більшості     | Легка      | `feedparser`        |
| **Python без бібліотек**  | Не хочеш встановлювати пакети       | Середня    | `xml.etree.ElementTree` |
| **JavaScript / Node.js**  | Робота у веб-проєктах               | Легка      | `rss-parser`        |
| **Командний рядок**       | Швидка перевірка                    | Дуже легка | `curl` + `python`   |
| **No-code**               | Zapier, Make.com, n8n               | Дуже легка | —                   |

### 4. Швидкий варіант без встановлення бібліотек (тільки стандартна бібліотека)

```python
import urllib.request
import xml.etree.ElementTree as ET
from datetime import datetime, timedelta, timezone

url = "https://ntdtv.com.ua/feed"
req = urllib.request.Request(url, headers={'User-Agent': 'Mozilla/5.0'})
with urllib.request.urlopen(req) as response:
    xml_data = response.read()

root = ET.fromstring(xml_data)
# ... парсинг item-ів (трохи складніше)
```

### 5. Корисні поради

- **Дати** в RSS майже завжди в UTC. Використовуй `timezone.utc`.
- Деякі фіди не мають `published_parsed` — тоді бери `updated_parsed`.
- Якщо фід великий — додай `limit` (наприклад, брати тільки перші 20 записів).
- Для регулярного запуску використовуй **cron** (Linux/Mac) або **Task Scheduler** (Windows).
- Якщо сайт блокує запити — додай `User-Agent` в заголовки.

---
4. Показав, як парсити фід **без зовнішніх бібліотек**?

Напиши, який варіант тобі потрібен найбільше.
