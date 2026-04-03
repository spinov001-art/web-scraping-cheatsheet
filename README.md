# Web Scraping Cheatsheet 2026 🕷️

The ultimate reference for web scraping — techniques, tools, code snippets, and anti-detection strategies. Updated for 2026.

## Table of Contents

- [Quick Start](#quick-start)
- [Python](#python-beautifulsoup--requests)
- [JavaScript](#javascript-puppeteer--playwright)
- [Headless Browsers](#headless-browser-comparison)
- [Selectors](#css-selectors--xpath-cheatsheet)
- [Anti-Detection](#anti-detection-techniques)
- [APIs vs Scraping](#when-to-use-apis-instead)
- [Legal](#legal-considerations)
- [Tools](#tools--platforms)

---

## Quick Start

### Python (requests + BeautifulSoup)
```python
import requests
from bs4 import BeautifulSoup

response = requests.get("https://example.com", headers={
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/122.0.0.0"
})
soup = BeautifulSoup(response.text, "html.parser")

# Get all links
links = [a["href"] for a in soup.select("a[href]")]

# Get text from specific element
title = soup.select_one("h1").get_text(strip=True)

# Get table data
rows = []
for tr in soup.select("table tr"):
    cells = [td.get_text(strip=True) for td in tr.select("td, th")]
    rows.append(cells)
```

### JavaScript (Playwright)
```javascript
import { chromium } from 'playwright';

const browser = await chromium.launch({ headless: true });
const page = await browser.newPage();
await page.goto('https://example.com');

// Wait for dynamic content
await page.waitForSelector('.product-card');

// Extract data
const items = await page.$$eval('.product-card', cards =>
    cards.map(card => ({
        title: card.querySelector('h2')?.textContent?.trim(),
        price: card.querySelector('.price')?.textContent?.trim(),
        link: card.querySelector('a')?.href,
    }))
);

await browser.close();
```

### cURL (quick & dirty)
```bash
# Basic GET
curl -s "https://api.example.com/data" | jq '.results[]'

# With headers
curl -s -H "User-Agent: Mozilla/5.0" -H "Accept: application/json" \
  "https://example.com/api/products?page=1"

# POST with form data
curl -s -X POST -d "query=web+scraping" "https://example.com/search"
```

---

## Python (BeautifulSoup + Requests)

### Install
```bash
pip install requests beautifulsoup4 lxml
```

### Common Patterns

| Task | Code |
|------|------|
| Get page | `requests.get(url, headers=headers)` |
| Parse HTML | `BeautifulSoup(html, "lxml")` |
| Find by class | `soup.select(".class-name")` |
| Find by ID | `soup.select_one("#element-id")` |
| Get attribute | `element["href"]` or `element.get("href")` |
| Get text | `element.get_text(strip=True)` |
| Find by tag + class | `soup.select("div.product")` |
| Nested selector | `soup.select("div.list > ul > li")` |
| Regex match | `soup.find_all("a", href=re.compile(r"/product/\d+"))` |
| Next sibling | `element.find_next_sibling("p")` |
| Parent | `element.parent` |

### Session with Cookies
```python
session = requests.Session()
session.headers.update({"User-Agent": "Mozilla/5.0 ..."})

# Login
session.post("https://example.com/login", data={
    "username": "user", "password": "pass"
})

# Authenticated request
response = session.get("https://example.com/dashboard")
```

### Pagination
```python
all_items = []
page = 1

while True:
    response = requests.get(f"https://example.com/items?page={page}")
    soup = BeautifulSoup(response.text, "lxml")
    items = soup.select(".item")

    if not items:
        break

    all_items.extend(items)
    page += 1
    time.sleep(1)  # Be respectful
```

---

## JavaScript (Puppeteer + Playwright)

### Playwright vs Puppeteer (2026)

| Feature | Playwright | Puppeteer |
|---------|-----------|-----------|
| Multi-browser | ✅ Chrome, Firefox, WebKit | Chrome only |
| Auto-wait | ✅ Built-in | Manual |
| Speed | Faster | Slower |
| API | Cleaner | Older |
| Stealth | Better | Needs plugin |
| **Verdict** | **Use this** | Legacy |

### Playwright Patterns
```javascript
// Infinite scroll
while (true) {
    const prevHeight = await page.evaluate(() => document.body.scrollHeight);
    await page.evaluate(() => window.scrollTo(0, document.body.scrollHeight));
    await page.waitForTimeout(2000);
    const newHeight = await page.evaluate(() => document.body.scrollHeight);
    if (newHeight === prevHeight) break;
}

// Handle popups/modals
page.on('dialog', dialog => dialog.dismiss());

// Screenshot for debugging
await page.screenshot({ path: 'debug.png', fullPage: true });

// Intercept network requests
await page.route('**/*.{png,jpg,gif}', route => route.abort()); // Block images
```

---

## CSS Selectors + XPath Cheatsheet

### CSS Selectors (most common)

| Selector | Meaning | Example |
|----------|---------|---------|
| `tag` | Element type | `div`, `p`, `a` |
| `.class` | Class name | `.product-card` |
| `#id` | Element ID | `#main-content` |
| `tag.class` | Tag with class | `div.item` |
| `parent > child` | Direct child | `ul > li` |
| `ancestor descendant` | Any descendant | `div p` |
| `[attr]` | Has attribute | `[data-id]` |
| `[attr="val"]` | Attribute equals | `[href="/about"]` |
| `[attr*="val"]` | Contains | `[class*="product"]` |
| `[attr^="val"]` | Starts with | `[href^="https"]` |
| `:nth-child(n)` | Nth child | `li:nth-child(2)` |
| `:first-child` | First child | `li:first-child` |
| `:not(.x)` | Negation | `div:not(.hidden)` |

### XPath (when CSS isn't enough)

| XPath | Meaning |
|-------|---------|
| `//div[@class="item"]` | Div with exact class |
| `//a[contains(@href, "product")]` | Link containing text |
| `//h2[text()="Title"]` | Element with exact text |
| `//div[contains(text(), "Price")]` | Contains text |
| `//table/tbody/tr[position()>1]` | Skip header row |
| `//div[@class="price"]/following-sibling::span` | Sibling |
| `//a[starts-with(@href, "/product")]` | Starts with |

---

## Anti-Detection Techniques

### Headers (minimum set)
```python
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",
    "Accept-Language": "en-US,en;q=0.5",
    "Accept-Encoding": "gzip, deflate, br",
    "DNT": "1",
    "Connection": "keep-alive",
    "Upgrade-Insecure-Requests": "1",
}
```

### Proxy Rotation
```python
proxies = [
    "http://user:pass@proxy1:8080",
    "http://user:pass@proxy2:8080",
]
proxy = {"http": random.choice(proxies), "https": random.choice(proxies)}
response = requests.get(url, proxies=proxy)
```

### Rate Limiting
```python
import time
import random

for url in urls:
    response = requests.get(url)
    time.sleep(random.uniform(1, 3))  # Random delay 1-3 seconds
```

### Fingerprint Checklist

- [x] Rotate User-Agent strings
- [x] Random delays between requests (1-5 sec)
- [x] Use residential proxies for sensitive sites
- [x] Set realistic Accept/Accept-Language headers
- [x] Handle cookies properly (use Session)
- [x] Rotate IP addresses
- [x] Disable WebDriver flag in headless browsers
- [x] Set realistic viewport size (1920x1080)

---

## When to Use APIs Instead

**Always prefer APIs over scraping when available.** They're faster, more reliable, and legal.

### Free APIs for Common Data

| Data Type | Free API | Limit |
|-----------|----------|-------|
| Reddit posts | `reddit.com/r/{sub}.json` | ~60 req/min |
| GitHub repos | `api.github.com` | 60 req/hr (unauth) |
| Wikipedia | `en.wikipedia.org/api/rest_v1/` | Unlimited |
| Weather | `open-meteo.com/api` | Unlimited |
| Stock prices | `finnhub.io/api` | 60 req/min |
| News | `newsapi.org` | 100 req/day |
| Academic papers | `api.semanticscholar.org` | 100 req/5min |
| Patents | `api.patentsview.org` | Unlimited |
| Clinical trials | `clinicaltrials.gov/api/v2` | Unlimited |
| IP geolocation | `ip-api.com/json/` | 45 req/min |

**📚 Full collection:** [16 Free API Toolkits](https://github.com/spinov001-art) — ready-to-use code for all APIs above.

---

## Data Export

### Save to CSV
```python
import csv

with open("output.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["title", "price", "url"])
    writer.writeheader()
    writer.writerows(data)
```

### Save to JSON
```python
import json

with open("output.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=2, ensure_ascii=False)
```

### Save to SQLite
```python
import sqlite3

conn = sqlite3.connect("scraped.db")
conn.execute("CREATE TABLE IF NOT EXISTS items (title TEXT, price REAL, url TEXT)")
conn.executemany("INSERT INTO items VALUES (?, ?, ?)",
    [(d["title"], d["price"], d["url"]) for d in data])
conn.commit()
```

---

## Tools & Platforms

| Tool | Best For | Pricing |
|------|----------|---------|
| [Apify](https://apify.com) | Production scraping, scheduling | Free tier + usage |
| [Scrapy](https://scrapy.org) | Large-scale Python scraping | Free (open source) |
| [Playwright](https://playwright.dev) | JavaScript-heavy sites | Free (open source) |
| [Selenium](https://selenium.dev) | Legacy, browser automation | Free (open source) |
| [Crawlee](https://crawlee.dev) | Node.js scraping framework | Free (open source) |

### Pre-Built Scrapers (Ready to Use)

- [Reddit Scraper Pro](https://apify.com/knotless_cadence/reddit-scraper-pro) — Reddit posts + comments via JSON API
- [60+ Free Scrapers](https://github.com/spinov001-art/awesome-web-scraping-2026) — curated list
- [15 MCP Servers](https://github.com/spinov001-art/mcp-servers-collection) — AI agent tools

---

## Legal Considerations

- ✅ **Public data** — generally OK to scrape
- ✅ **APIs** — always preferred, follow rate limits
- ⚠️ **Terms of Service** — read them, some sites explicitly prohibit scraping
- ⚠️ **robots.txt** — respect it (not legally binding but good practice)
- ❌ **Copyrighted content** — don't redistribute scraped copyrighted material
- ❌ **Personal data** — be careful with GDPR/CCPA compliance
- ❌ **Login-required content** — scraping behind authentication is risky

---

## Contributing

Found a better technique? Submit a PR!

## License

MIT — use freely in your projects.

---

**Built by [spinov001-art](https://github.com/spinov001-art)** — 180+ open source repos for developers.

**Need custom scraping?** [View all tools](https://github.com/spinov001-art) | [Email me](mailto:spinov001@gmail.com)
