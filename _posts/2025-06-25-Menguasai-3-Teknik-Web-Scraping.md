---
layout: post
title: "Menguasai 3 Teknik Web Scraping: Forms, AJAX, dan iFrames"
date: 2025-06-25T00:00:00.000+00:00
categories: [web-scraping, python, tutorial]
tags: [beautifulsoup, selenium, requests, pagination, ajax, iframe]
author: "Pangeran Juhrifar Jafar"
image: "assets/images/scrape-site.png"
permalink: "/post/6"
excerpt: "Pelajari cara mengatasi tantangan web scraping modern dengan tiga teknik berbeda: menangani forms dengan pagination, scraping data AJAX/JavaScript, dan mengekstrak data dari iFrames."
---


Web scraping adalah salah satu skill penting dalam data science dan web development. Namun, website modern sering menggunakan teknologi yang membuat scraping menjadi lebih menantang. Dalam artikel ini, saya akan membahas tiga teknik web scraping untuk mengatasi tantangan yang berbeda.

## 🏒 Teknik 1: Scraping dengan Forms dan Pagination

### Tantangan
Banyak website menggunakan form pencarian dan membagi data dalam beberapa halaman (pagination). Contohnya adalah data tim hockey yang tersebar di 25 halaman.

### Solusi: Requests + BeautifulSoup

```python
from bs4 import BeautifulSoup
import requests
import pandas as pd

BASE_URL = "https://www.scrapethissite.com/pages/forms/"
SEARCH_URL = "https://www.scrapethissite.com/pages/forms/?page={}"

def scrape_hockey_data(max_page=25):
    """Scraping seluruh data dari halaman dengan pagination."""
    data = []
    page = 1

    while page <= max_page:
        print(f"📄 Scraping halaman {page}...")
        
        response = requests.get(SEARCH_URL.format(page))
        
        if response.status_code != 200:
            print(f"❌ Gagal mengambil halaman {page}")
            break

        soup = BeautifulSoup(response.text, "html.parser")
        teams = soup.find_all("tr", class_="team")

        if not teams:
            print("✅ Tidak ada halaman berikutnya")
            break

        for team in teams:
            team_data = extract_team_data(team)
            data.append(team_data)

        page += 1

    return data
```

### Key Points:
- **Loop pagination**: Gunakan parameter `page` dalam URL
- **Error handling**: Check status code dan keberadaan data
- **Data extraction**: Parse HTML dengan BeautifulSoup untuk setiap tim

## 🎬 Teknik 2: Scraping Data AJAX/JavaScript

### Tantangan
Website modern sering memuat data secara dinamis menggunakan JavaScript dan AJAX. Data tidak tersedia langsung dalam HTML awal.

### Solusi: Selenium WebDriver

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import time

# Setup WebDriver
options = webdriver.ChromeOptions()
options.add_argument("--headless")
driver = webdriver.Chrome(options=options)

url = "https://www.scrapethissite.com/pages/ajax-javascript/"
driver.get(url)

years = [2010, 2011, 2012, 2013, 2014, 2015]
all_data = []

for year in years:
    print(f"🔍 Scraping data untuk tahun {year}...")
    
    # Klik tombol tahun
    year_button = WebDriverWait(driver, 10).until(
        EC.element_to_be_clickable((By.LINK_TEXT, str(year)))
    )
    year_button.click()
    
    # Tunggu data dimuat
    time.sleep(3)
    
    # Parse data yang sudah dimuat
    soup = BeautifulSoup(driver.page_source, "html.parser")
    movies = soup.find_all("tr", class_="film")
    
    for movie in movies:
        title = movie.find("td", class_="film-title").text.strip()
        nominations = int(movie.find("td", class_="film-nominations").text.strip())
        awards = int(movie.find("td", class_="film-awards").text.strip())
        best_picture = movie.find("td", class_="film-best-picture").text.strip() == "Yes"
        
        all_data.append([title, nominations, awards, best_picture])

driver.quit()
```

### Key Points:
- **WebDriverWait**: Tunggu elemen muncul sebelum interaksi
- **Click interactions**: Simulate klik user untuk trigger AJAX
- **Dynamic content**: Parse HTML setelah JavaScript selesai execute

## 🐢 Teknik 3: Scraping iFrames

### Tantangan
Beberapa website menggunakan iFrames untuk menampilkan konten. Data yang diinginkan berada dalam frame terpisah.

### Solusi: Selenium dengan Frame Switching

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
import time
from bs4 import BeautifulSoup

# Setup WebDriver
driver = webdriver.Chrome()
URL = "https://www.scrapethissite.com/pages/frames/"
driver.get(URL)
time.sleep(3)

# 1. Temukan iframe
iframe = driver.find_element(By.TAG_NAME, "iframe")

# 2. Pindah ke dalam iframe
driver.switch_to.frame(iframe)
time.sleep(2)

# 3. Scrape data dalam iframe
soup = BeautifulSoup(driver.page_source, "html.parser")
turtles = soup.find_all("div", class_="turtle-family-card")

all_data = []
for turtle in turtles:
    title = turtle.find("h3", class_="family-name")
    title_text = title.get_text(strip=True) if title else "Nama tidak ditemukan"
    
    # Ambil link detail
    learn_more = turtle.find("a", class_="btn btn-default btn-xs")
    if learn_more:
        detail_link = "https://www.scrapethissite.com" + learn_more["href"]
        
        # Navigate ke halaman detail
        driver.get(detail_link)
        time.sleep(3)
        
        detail_soup = BeautifulSoup(driver.page_source, "html.parser")
        desc = detail_soup.find("p", class_="lead")
        desc_text = desc.get_text(strip=True) if desc else "Deskripsi Tidak Ada"
    else:
        desc_text = "Deskripsi Tidak ada"
    
    all_data.append([title_text, desc_text])

# Kembali ke konteks utama
driver.switch_to.default_content()
driver.quit()
```

### Key Points:
- **Frame switching**: Gunakan `switch_to.frame()` untuk masuk ke iframe
- **Navigation**: Bisa navigate ke halaman lain dari dalam iframe
- **Context switching**: Jangan lupa kembali ke default content

## 🛠️ Tools dan Library yang Digunakan

### 1. **Requests + BeautifulSoup**
- **Kapan digunakan**: Website statis, form sederhana
- **Kelebihan**: Cepat, lightweight, mudah debug
- **Kekurangan**: Tidak bisa handle JavaScript

### 2. **Selenium WebDriver**
- **Kapan digunakan**: AJAX, JavaScript, complex interactions
- **Kelebihan**: Bisa handle dynamic content, simulasi user behavior
- **Kekurangan**: Lambat, resource intensive

### 3. **Pandas**
- **Fungsi**: Data manipulation dan export ke CSV
- **Kelebihan**: Easy data handling, berbagai format export

## 💡 Best Practices

### 1. **Error Handling**
```python
if response.status_code != 200:
    print(f"❌ Gagal mengambil halaman, status: {response.status_code}")
    break
```

### 2. **Respectful Scraping**
```python
time.sleep(3)  # Jangan terlalu cepat request
```

### 3. **Data Validation**
```python
# Handle empty values
ot_losses = int(ot_losses_text) if ot_losses_text else 0
```

### 4. **Proper Resource Management**
```python
driver.quit()  # Selalu tutup browser
```

## 📊 Output dan Penyimpanan Data

Semua data disimpan dalam format CSV dengan struktur yang jelas:

```python
def save_to_csv(data, filename="output.csv"):
    columns = ["Column1", "Column2", "Column3"]
    df = pd.DataFrame(data, columns=columns)
    df.to_csv(filename, index=False, encoding="utf-8")
    print(f"📂 Data berhasil disimpan ke {filename}!")
```

## 🎯 Kesimpulan

Web scraping modern memerlukan pendekatan yang berbeda tergantung pada arsitektur website:

1. **Forms + Pagination** → Requests + BeautifulSoup
2. **AJAX/JavaScript** → Selenium WebDriver
3. **iFrames** → Selenium dengan frame switching

Kunci sukses web scraping adalah:
- Analisis struktur website terlebih dahulu
- Pilih tools yang tepat untuk tantangan spesifik
- Implementasikan error handling yang baik
- Respect website dengan delay yang wajar

Dengan menguasai ketiga teknik ini, Anda bisa handle mayoritas tantangan web scraping di dunia nyata!

---

*Happy scraping! 🚀*