1. Selenium Script with ProxyMesh and MongoDB Integration
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from pymongo import MongoClient
import uuid
from datetime import datetime
import requests

# MongoDB configuration
MONGO_URI = "mongodb://localhost:27017/"
DB_NAME = "trending_topics_db"
COLLECTION_NAME = "topics"

client = MongoClient(MONGO_URI)
db = client[DB_NAME]
collection = db[COLLECTION_NAME]

# Selenium configuration
def get_driver_with_proxy():
    options = Options()
    options.add_argument("--headless")
    options.add_argument("--disable-gpu")
    options.add_argument("--no-sandbox")
    options.add_argument('--proxy-server=http://your-proxymesh-username:password@us.proxymesh.com:31280')

    service = Service("path/to/chromedriver")  # Replace with the path to your ChromeDriver
    driver = webdriver.Chrome(service=service, options=options)
    return driver

def scrape_trending_topics():
    driver = get_driver_with_proxy()
    driver.get("https://twitter.com/login")

    # Log in to Twitter
    username = driver.find_element(By.NAME, "session[username_or_email]")
    password = driver.find_element(By.NAME, "session[password]")
    username.send_keys("your-twitter-username")  # Replace with your Twitter username
    password.send_keys("your-twitter-password")  # Replace with your Twitter password
    password.submit()

    driver.implicitly_wait(5)
    trending_section = driver.find_element(By.XPATH, '//section[contains(@aria-labelledby, "accessible-list")]')
    trends = trending_section.find_elements(By.XPATH, './/span')[0:5]

    trends_list = [trend.text for trend in trends]
    ip_address = requests.get("https://api.ipify.org").text

    result = {
        "_id": str(uuid.uuid4()),
        "trend1": trends_list[0],
        "trend2": trends_list[1],
        "trend3": trends_list[2],
        "trend4": trends_list[3],
        "trend5": trends_list[4],
        "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        "ip_address": ip_address
    }

    # Save to MongoDB
    collection.insert_one(result)
    driver.quit()

    return result
