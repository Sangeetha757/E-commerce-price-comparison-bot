import requests
from bs4 import BeautifulSoup

headers = {"User-Agent": "Mozilla/5.0"}

def clean_price(price):
    return float(price.replace("₹","").replace(",","").strip())

def amazon_scrape(product):
    url = f"https://www.amazon.in/s?k={product.replace(' ','+')}"
    res = requests.get(url, headers=headers)
    soup = BeautifulSoup(res.text, "html.parser")

    item = soup.select_one("div.s-result-item")

    if not item:
        return None

    title = item.h2.text.strip()
    price = item.select_one(".a-price-whole")
    if not price:
        return None

    return {
        "website": "Amazon",
        "title": title,
        "price": clean_price(price.text),
        "link": "https://www.amazon.in" + item.h2.a["href"]
    }

def flipkart_scrape(product):
    url = f"https://www.flipkart.com/search?q={product.replace(' ','+')}"
    res = requests.get(url, headers=headers)
    soup = BeautifulSoup(res.text, "html.parser")

    title = soup.select_one("div._4rR01T")
    price = soup.select_one("div._30jeq3")

    if not (title and price):
        return None

    return {
        "website": "Flipkart",
        "title": title.text.strip(),
        "price": clean_price(price.text),
        "link": url
    }
