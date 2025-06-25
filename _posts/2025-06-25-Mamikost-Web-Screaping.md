---
layout: post
title: "Web Scraping Mamikos: Mengumpulkan Data Kost dengan Selenium dan Lazy Loading"
date: 2025-06-25
categories: [web-scraping, python, real-estate]
tags: [selenium, mamikos, lazy-loading, dynamic-content, indonesia]
author: "Pangeran Juhrifar Jafar"
permalink: "/daily/5"
image: "assets/images/mamikost.png"
excerpt: "Tutorial lengkap scraping data kost dari Mamikos.com menggunakan Selenium untuk menangani dynamic loading, infinite scroll, dan lazy loading images."
---


## 🏠 Tentang Mamikos

Mamikos adalah sebuah perusahaan rintisan (startup) asal Indonesia yang bergerak di bidang penyediaan layanan pencarian dan penyewaan kost atau hunian sewa. Didirikan pada tahun 2015, Mamikos telah memperluas jangkauannya ke berbagai kota besar di Indonesia, termasuk Jakarta, Bogor, Depok, Tangerang, Bekasi, Yogyakarta, Medan, Bandung, dan Surabaya.

Platform ini memungkinkan pengguna untuk mencari pilihan kost terlengkap di kota tujuan dengan berbagai potongan serta cashback. Mamikos menjadi aplikasi anak kos No.1 se-Indonesia selama 9 tahun dan sangat populer di kalangan mahasiswa dan pekerja muda.

## 🎯 Tujuan Scraping

Dalam project ini, kita akan melakukan scraping data kost di **Depok** dari Mamikos.com untuk mengumpulkan informasi seperti:
- Nama kost
- Tipe kost (Putra/Putri/Campur)
- Lokasi
- Fasilitas
- Rating
- Harga (sebelum dan sesudah diskon)
- Gambar kost

## 🔴 Tantangan Teknis

Website Mamikos menggunakan beberapa teknologi modern yang membuat scraping menjadi challenging:

### 1. **Dynamic Content Loading**
- Data kost tidak langsung tersedia dalam HTML awal
- Perlu klik tombol "Lihat lebih banyak lagi" untuk memuat data tambahan

### 2. **Lazy Loading Images**
- Gambar hanya dimuat ketika pengguna scroll ke bawah
- Awalnya hanya placeholder yang ditampilkan

### 3. **Anti-Bot Detection**
- Website bisa mendeteksi automated browsing
- Perlu user-agent yang proper dan delay yang wajar

## 🚀 Implementasi dengan Selenium

### Setup WebDriver

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.action_chains import ActionChains
from bs4 import BeautifulSoup
import pandas as pd
import time

# Setup Chrome options untuk menghindari deteksi bot
options = webdriver.ChromeOptions()
options.add_argument("--headless") 
options.add_argument("--disable-gpu") 
options.add_argument("--no-sandbox") 
options.add_argument("--disable-blink-features=AutomationControlled")
options.add_argument("user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36")

driver = webdriver.Chrome(options=options)
```

### Handling Dynamic Content

```python
# URL target untuk kost di Depok
URL = "https://mamikos.com/cari/depok-kota-depok-jawa-barat-indonesia/all/bulanan/0-15000000/124?keyword=Depok&suggestion_type=search&rent=2&sort=price,-&price=10000-20000000&singgahsini=0"
driver.get(URL)
time.sleep(3)

# Loop untuk klik tombol "Lihat lebih banyak lagi"
click_count = 0
while True:
    try:
        button = WebDriverWait(driver, 5).until(
            EC.element_to_be_clickable((By.CLASS_NAME, "nominatim-list__see-more"))
        )
        ActionChains(driver).move_to_element(button).click().perform()
        time.sleep(3)  # Tunggu data baru termuat
        click_count += 1
        
        if click_count > 2:  # Batasi jumlah klik untuk demo
            break
    except:
        print("❌ Tidak ada tombol lagi, lanjut scraping...")
        break
```

### Mengatasi Lazy Loading Images

Salah satu tantangan unik adalah **lazy loading** pada gambar. Gambar hanya dimuat ketika pengguna scroll ke area tersebut:

```python
# Scroll perlahan untuk memuat semua gambar
print("🐌 Scroll perlahan untuk memuat semua gambar...")
scroll_pause_time = 2
screen_height = driver.execute_script("return window.innerHeight;")
scroll_height = driver.execute_script("return document.body.scrollHeight")

for i in range(0, scroll_height, screen_height):
    driver.execute_script(f"window.scrollTo(0, {i});")
    time.sleep(scroll_pause_time)

# Tunggu agar semua gambar termuat
time.sleep(5)
```

### Data Extraction

```python
# Parse dengan BeautifulSoup setelah semua data termuat
html_source = driver.page_source
soup = BeautifulSoup(html_source, "html.parser")
kost_list = soup.find_all("div", class_="kost-rc__inner")

all_data_kost = []

for kost in kost_list:
    # Ambil gambar utama - handle lazy loading
    image_tag = kost.find("img", {"data-testid": "roomCardCover-photo"})
    image_url = image_tag.get("src") or image_tag.get("data-src") or "Tidak ada gambar"

    # Ambil tipe kost (Putra/Putri/Campur)
    kost_type_tag = kost.find('div', class_='rc-overview__label')
    kost_type = kost_type_tag.text.strip() if kost_type_tag else 'Tidak diketahui'
    
    # Ambil nama kost
    name_tag = kost.find('span', class_='rc-info__name')
    kost_name = name_tag.text.strip() if name_tag else 'Tidak ada nama'
    
    # Ambil lokasi
    location_tag = kost.find('span', class_='rc-info__location')
    location = location_tag.text.strip() if location_tag else 'Tidak ada lokasi'
    
    # Ambil fasilitas
    facilities_tags = kost.find_all('span', {'data-testid': 'roomCardFacilities-facility'})
    facilities = [facility.text.strip() for facility in facilities_tags]
    
    # Ambil rating
    rating_tag = kost.find('span', class_='rc-overview__rating-text')
    rating = rating_tag.text.strip() if rating_tag else 'Tidak ada rating'
    
    # Ambil harga sebelum dan sesudah diskon
    original_price_tag = kost.find('span', class_='rc-price__additional-discount-price')
    original_price = original_price_tag.text.strip() if original_price_tag else 'Tidak ada harga sebelum diskon'
    
    discounted_price_tag = kost.find('span', class_='rc-price__text')
    discounted_price = discounted_price_tag.text.strip() if discounted_price_tag else 'Tidak ada harga setelah diskon'
    
    all_data_kost.append([
        kost_name, kost_type, location, facilities, 
        rating, original_price, discounted_price, image_url
    ])

driver.quit()
```

## 🛠️ Teknik Anti-Detection

### 1. **User Agent Spoofing**
```python
options.add_argument("user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36")
```

### 2. **Disable Automation Flags**
```python
options.add_argument("--disable-blink-features=AutomationControlled")
```

### 3. **Natural Delays**
```python
time.sleep(3)  # Jeda yang wajar antar aksi
```

### 4. **ActionChains untuk Natural Clicking**
```python
ActionChains(driver).move_to_element(button).click().perform()
```

## 📊 Data Output

Data yang berhasil dikumpulkan disimpan dalam format CSV dengan struktur:

```python
df = pd.DataFrame(all_data_kost, columns=[
    "Nama kost",
    "Tipe kost (Putra/Putri/Campur)",
    "Lokasi", 
    "Fasilitas", 
    "Rating", 
    "Harga Sebelum Diskon", 
    "Harga Setelah Diskon", 
    "Image"
])
```

## 💡 Key Insights

### 1. **Handling Lazy Loading**
Teknik lazy loading pada gambar memerlukan scrolling bertahap untuk memuat semua konten visual.

### 2. **Dynamic Button Clicking**
Tombol "Lihat lebih banyak lagi" perlu diklik berulang kali untuk memuat semua data yang tersedia.

### 3. **Robust Error Handling**
Website real-world sering memiliki elemen yang tidak konsisten, perlu penanganan `None` values.

### 4. **Performance Optimization**
Batasi jumlah klik dan scroll untuk menghindari timeout dan overload.

## ⚠️ Considerations

### 1. **Terms of Service**
Selalu periksa robots.txt dan terms of service website sebelum scraping.

### 2. **Rate Limiting**
Gunakan delay yang wajar untuk menghormati server dan menghindari IP blocking.

### 3. **Data Accuracy**
Verifikasi data hasil scraping dengan sample manual untuk memastikan akurasi.

### 4. **Legal Compliance**
Pastikan penggunaan data sesuai dengan hukum yang berlaku dan tidak melanggar privacy.

## 🎯 Hasil dan Manfaat

Dengan teknik ini, kita berhasil mengumpulkan data kost komprehensif yang bisa digunakan untuk:
- **Analisis pasar** properti sewa di Depok
- **Perbandingan harga** antar kost
- **Mapping fasilitas** yang paling dicari
- **Trend rating** dan kepuasan penghuni

## 🔍 Kesimpulan

Scraping website modern seperti Mamikos memerlukan pendekatan yang lebih sophisticated dibanding website statis. Kunci sukses meliputi:

1. **Selenium untuk dynamic content** - Handle JavaScript dan AJAX
2. **Proper scrolling strategy** - Atasi lazy loading
3. **Anti-detection techniques** - Hindari bot detection
4. **Robust error handling** - Handle inconsistent elements
5. **Respectful scraping** - Rate limiting dan compliance

Website real estate seperti Mamikos menyediakan data berharga untuk analisis pasar, namun memerlukan teknik scraping yang canggih dan etis. Dengan implementasi yang tepat, kita bisa mendapatkan insights yang valuable untuk berbagai keperluan bisnis dan penelitian.

---

*Happy scraping responsibly! 🏠✨*