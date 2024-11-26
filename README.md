# Web-Scrapping-tool
Directory Structure:
bash
Copy code
web_scraper/
│
├── config/
│   └── config.yaml              # Configuration file for scraping parameters
├── logs/
│   └── scraper.log              # Logs for scraper runs
├── outputs/
│   └── output.csv               # Scraped data output
├── scrapers/
│   ├── static_scraper.py        # Static scraper logic
│   └── dynamic_scraper.py       # Dynamic scraper logic
├── main.py                      # Main entry point for the scraper
├── requirements.txt             # Python dependencies
└── README.md                    # Documentation

# Code Implementation
1. config/config.yaml
yaml
Copy code
default_headers:
  User-Agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36"

timeout: 10  # Request timeout in seconds

# scrapers/static_scraper.py

import requests
from bs4 import BeautifulSoup
import logging

def static_scraper(url, selectors, headers):
    try:
        response = requests.get(url, headers=headers, timeout=10)
        response.raise_for_status()

        soup = BeautifulSoup(response.text, 'html.parser')
        data = {key: soup.select_one(selector).get_text(strip=True) if soup.select_one(selector) else None
                for key, selector in selectors.items()}
        
        logging.info(f"Successfully scraped {url}")
        return data
    except Exception as e:
        logging.error(f"Error in static_scraper: {e}")
        return None
# scrapers/dynamic_scraper.py

from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
import logging

def dynamic_scraper(url, selectors):
    options = Options()
    options.add_argument('--headless')
    options.add_argument('--disable-gpu')
    
    driver = webdriver.Chrome(service=Service(), options=options)
    data = {}
    try:
        driver.get(url)
        for key, selector in selectors.items():
            try:
                element = driver.find_element(By.CSS_SELECTOR, selector)
                data[key] = element.text.strip()
            except Exception:
                data[key] = None

        logging.info(f"Successfully scraped dynamic content from {url}")
    except Exception as e:
        logging.error(f"Error in dynamic_scraper: {e}")
    finally:
        driver.quit()
    return data
main.py
python
Copy code
import argparse
import yaml
import logging
from scrapers.static_scraper import static_scraper
from scrapers.dynamic_scraper import dynamic_scraper
import pandas as pd

# Configure logging
logging.basicConfig(filename="logs/scraper.log", level=logging.INFO, 
                    format="%(asctime)s - %(levelname)s - %(message)s")

# Load config
with open("config/config.yaml", "r") as f:
    config = yaml.safe_load(f)

def main():
    parser = argparse.ArgumentParser(description="Web Scraper Tool")
    parser.add_argument("--url", required=True, help="Target URL to scrape")
    parser.add_argument("--type", choices=["static", "dynamic"], required=True, help="Scraping type")
    parser.add_argument("--output", default="outputs/output.csv", help="Output file path")
    args = parser.parse_args()

    # Example selectors for demo
    selectors = {
        "title": "title",
        "header": "h1",
        "description": "meta[name='description']::attr(content)"
    }

    # Scraping logic
    if args.type == "static":
        data = static_scraper(args.url, selectors, headers=config["default_headers"])
    else:
        data = dynamic_scraper(args.url, selectors)

    if data:
        # Save to CSV
        pd.DataFrame([data]).to_csv(args.output, index=False)
        logging.info(f"Data saved to {args.output}")
    else:
        logging.warning("No data scraped.")

if __name__ == "__main__":
    main()
