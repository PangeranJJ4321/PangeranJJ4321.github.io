---
title: "📊 Dasar-Dasar Data Scraping dengan Python"
date: 2025-02-25T00:00:00.000+00:00
categories: [Data Science, Web Scraping, Python, Data Meaning]
layout: post
author: Pangeran Juhrifar Jafar
permalink: "/daily/3"
tags:
- Data Science
---

## 🧐 Apa Itu Data Scraping?

Data Scraping adalah proses otomatis untuk mengambil data dari sebuah website. Teknik ini sangat berguna ketika data yang kita butuhkan tidak tersedia dalam format yang bisa langsung diunduh, seperti CSV atau API.

## 🔄 Alur Dasar Data Scraping

1. **🎯 Memilih Website Target** – Tentukan website yang ingin diambil datanya.
2. **🔍 Memahami Struktur HTML** – Gunakan *Inspect Element* di browser untuk melihat struktur HTML.
3. **🛠️ Menggunakan Pustaka Scraping** – Biasanya menggunakan Python dengan pustaka seperti:
   - 🏗️ `BeautifulSoup` untuk parsing HTML
   - 🌐 `Requests` untuk mengunduh halaman website
   - 🕷️ `Scrapy` untuk scraping dalam skala besar
4. **💾 Menyimpan Data yang Didapat** – Data yang diambil biasanya disimpan dalam format seperti CSV, JSON, atau database.
5. **🛡️ Mengatasi Kendala Scraping** – Beberapa website melarang scraping, jadi butuh strategi seperti rotasi User-Agent, penggunaan proxy, atau Selenium untuk menangani JavaScript.

## 📜 Logbook Proses Scraping

### **1️⃣ Deskripsi Pekerjaan**
📝 Melakukan web scraping untuk mengambil daftar hari Ramadhan 2025 dari situs Detik.

### **2️⃣ Langkah-langkah yang Dilakukan**

#### **🔗 Menentukan Sumber Data**
- 🌍 **URL sumber data:** [Detik](https://www.detik.com/jateng/berita/d-7793120/bulan-ramadhan-2025-berapa-hijriah-cek-kalender-dan-tanggalan-islam-tahun-ini)
- 🏷️ **Elemen yang akan di-scrape:** Daftar hari Ramadhan (ditemukan dalam tag `<li>`).

#### **📩 Melakukan Request ke Halaman Web**
- 📡 Menggunakan `requests` untuk mengambil halaman HTML.
- 🛡️ Menambahkan User-Agent agar tidak diblokir oleh situs.

#### **🧩 Parsing HTML dengan BeautifulSoup**
- 🏗️ Menggunakan `BeautifulSoup` untuk memproses HTML.
- 🔎 Mencari semua elemen `<li>` dalam halaman.

#### **🧹 Filter Data**
- ✨ Membersihkan teks dari `<li>` yang ditemukan.
- 🔎 Memfilter hanya teks yang mengandung kata "Ramadhan" agar hanya data yang relevan yang diambil.

#### **📌 Menampilkan Hasil**
- 📜 Mencetak daftar hari Ramadhan yang telah diproses.

### **3️⃣ Kode yang Digunakan**
```python
import requests
from bs4 import BeautifulSoup

# Step 1: Request halaman web
url = "https://www.detik.com/jateng/berita/d-7793120/bulan-ramadhan-2025-berapa-hijriah-cek-kalender-dan-tanggalan-islam-tahun-ini"
headers = {"User-Agent": "Mozilla/5.0"}
response = requests.get(url, headers=headers)

# Step 2: Parsing HTML
soup = BeautifulSoup(response.text, "html.parser")

# Step 3: Ambil data yang diinginkan
dates = soup.find_all("li")  # Menyesuaikan dengan struktur HTML

# Step 4: Filter dan cetak hasil
for date in dates:
    if "Ramadhan" in date.text:
        print(date.text.strip())
```
💡 **Kode ini akan mengambil semua elemen `<li>` dari halaman web dan hanya menampilkan yang mengandung kata "Ramadhan".**  

### **4️⃣ Hasil yang Diperoleh**
✅ **Berhasil** mengambil daftar hari Ramadhan dari halaman web.  
✅ Hanya **data yang relevan** yang diambil dan ditampilkan.  
✅ **Hasil sesuai** dengan ekspektasi.  

<h3>⚠️ Kendala dan Solusi</h3>
<table border="1">
    <tr>
        <th>Kendala</th>
        <th>Solusi</th>
    </tr>
    <tr>
        <td>🌐 Website menggunakan JavaScript</td>
        <td>🖥️ Gunakan Selenium untuk mengeksekusi JavaScript sebelum scraping.</td>
    </tr>
    <tr>
        <td>🚫 Pemblokiran oleh Website</td>
        <td>🔄 Gunakan User-Agent yang berbeda atau rotasi proxy.</td>
    </tr>
    <tr>
        <td>📊 Data Tidak Terstruktur</td>
        <td>🛠️ Gunakan regex atau metode parsing tambahan untuk membersihkan data.</td>
    </tr>
</table>


## 🔜 Rencana Selanjutnya
- ✅ Mengembangkan kode agar bisa menyimpan hasil scraping dalam file CSV.
- ✅ Menggunakan Selenium jika data ternyata dimuat secara dinamis.
- ✅ Mengotomatiskan proses scraping dan logging.

---

📌 **Catatan:** Logbook ini dapat diperbarui setiap kali ada perkembangan dalam tugas scraping. 🚀

