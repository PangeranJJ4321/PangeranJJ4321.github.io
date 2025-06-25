---
title: "🛒 Association Rules Apriori Market Basket Analysis"
date: 2025-06-24T00:00:00.000+00:00
categories: [Data Science, Web Scraping, Python, Data Meaning]
layout: post
author: Pangeran Juhrifar Jafar
permalink: "/post/4"
image: "https://annalyzin.files.wordpress.com/2016/04/association-rule-support-table.png?w=503&h=447"
tags:
- Data Science
---

## Pengantar

Analisis Keranjang Belanja adalah teknik yang digunakan untuk mengungkap asosiasi antar item. Ini banyak digunakan dalam ritel untuk mengidentifikasi hubungan antara produk-produk yang sering dibeli pelanggan secara bersamaan. Dengan menemukan hubungan ini, bisnis dapat memperoleh wawasan tentang kebiasaan pembelian pelanggan, yang dapat digunakan untuk berbagai tujuan seperti penjualan silang (cross-selling), penempatan produk, dan pemasaran bertarget. 

Algoritma Apriori adalah algoritma klasik yang digunakan untuk penemuan itemset yang sering muncul (frequent itemset mining) dan pembelajaran aturan asosiasi (association rule learning) di atas database transaksional.

## Key Concepts

### Support

Support adalah indikasi seberapa sering itemset muncul dalam dataset. Dihitung sebagai jumlah transaksi yang mengandung itemset dibagi dengan total jumlah transaksi.

```
Support(X) = Number of transactions containing X / Total number of transactions
```

Nilai support yang lebih tinggi menunjukkan bahwa itemset tersebut lebih umum dalam dataset.

### Confidence

Confidence adalah ukuran seberapa sering item Y muncul dalam transaksi yang mengandung X. Dihitung sebagai jumlah transaksi yang mengandung X dan Y dibagi dengan jumlah transaksi yang mengandung X.

```
Confidence(X => Y) = Support(X ∪ Y) / Support(X)
```

Confidence menunjukkan keandalan aturan. Nilai confidence yang tinggi menunjukkan bahwa jika pelanggan membeli X, mereka kemungkinan juga akan membeli Y.

### Lift

Lift adalah metrik yang mengukur seberapa besar kemungkinan item Y dibeli ketika item X dibeli, dibandingkan dengan probabilitas membeli Y secara independen.

```
Lift(X => Y) = Confidence(X => Y) / Support(Y)
```

- Jika Lift = 1, menunjukkan tidak ada asosiasi antara X dan Y
- Jika Lift > 1, menunjukkan asosiasi positif
- Jika Lift < 1, menunjukkan asosiasi negatif

## Dataset

Dataset yang digunakan untuk analisis market basket ini bersumber dari Kaggle dengan judul "[Coffee Sales](https://www.kaggle.com/datasets/ihelon/coffee-sales)". Dataset ini berisi data transaksional terkait penjualan kopi dengan kolom-kolom utama:

- `datetime`: Timestamp transaksi
- `card`: Identifier unik untuk pelanggan atau metode transaksi
- `coffee_name`: Nama produk kopi yang dibeli

Dataset ini menyediakan catatan pembelian kopi individual yang penting untuk mengidentifikasi item kopi yang sering muncul bersamaan dalam transaksi pelanggan.

## Implementation

### Data Loading

Langkah pertama adalah memuat data transaksional dari dua file CSV yang disediakan ke dalam pandas DataFrames dan menggabungkannya.

```python
import pandas as pd
import os

# Konstruksi path file untuk kedua CSV
csv_file_path_1 = os.path.join(path, "index_1.csv")
csv_file_path_2 = os.path.join(path, "index_2.csv")

# Load kedua file CSV ke pandas DataFrames
df1 = pd.read_csv(csv_file_path_1)
df2 = pd.read_csv(csv_file_path_2)

# Gabungkan kedua DataFrames
df_combined = pd.concat([df1, df2], ignore_index=True)

# Tampilkan head dan info dari DataFrame gabungan
display(df_combined.head())
display(df_combined.info())
```

### Preprocessing

Sebelum menerapkan algoritma Apriori, data perlu ditransformasi ke format yang sesuai:

#### 1. Transaction Formatting (Grouping by Card)

Setiap `card` unik diperlakukan sebagai satu transaksi, menggabungkan semua item `coffee_name` yang terkait dengan card tersebut ke dalam list.

```python
# Group by 'card' untuk memperlakukan semua pembelian oleh card sebagai satu transaksi
# Drop rows dengan 'card' NaN karena tidak dapat dikelompokkan
transactions_by_card = df_combined.dropna(subset=['card']).groupby('card')['coffee_name'].apply(list).reset_index(name='items')

# Tampilkan head dari DataFrame transaksi baru
display(transactions_by_card.head())
```

#### 2. One-Hot Encoding

List item untuk setiap transaksi dikonversi ke format one-hot encoded menggunakan `TransactionEncoder` dari library `mlxtend`.

```python
from mlxtend.preprocessing import TransactionEncoder

# Konversi kolom 'items' menjadi list of lists
list_of_items_by_card = transactions_by_card['items'].tolist()

# Inisialisasi TransactionEncoder
te_card = TransactionEncoder()

# Fit dan transform list of lists menjadi one-hot encoded DataFrame
te_ary_card = te_card.fit(list_of_items_by_card).transform(list_of_items_by_card)
transactions_encoded_by_card = pd.DataFrame(te_ary_card, columns=te_card.columns_)

# Tampilkan head dari one-hot encoded DataFrame
display(transactions_encoded_by_card.head(10))
```

### Applying the Apriori Algorithm

Dengan data yang sudah dipreproses, algoritma Apriori diterapkan untuk mengidentifikasi frequent itemsets dan menghasilkan association rules.

```python
from mlxtend.frequent_patterns import apriori, association_rules

# Temukan frequent itemsets menggunakan algoritma Apriori
frequent_itemsets_by_card = apriori(transactions_encoded_by_card, min_support=0.01, use_colnames=True)

# Generate association rules
rules_by_card = association_rules(frequent_itemsets_by_card, metric="confidence", min_threshold=0.05)

# Tampilkan frequent itemsets
print("Frequent Itemsets (Grouped by Card):")
display(frequent_itemsets_by_card)

# Tampilkan association rules
print("\nAssociation Rules (Grouped by Card):")
display(rules_by_card)
```

**Parameter yang digunakan:**
- `min_support=0.01`: Itemset harus muncul di setidaknya 1% dari transaksi
- `min_threshold=0.05`: Rules harus memiliki confidence minimal 5%

## Results and Interpretation

### Frequent Itemsets

DataFrame `frequent_itemsets_by_card` berisi itemsets yang sering muncul dalam transaksi beserta nilai support mereka:

- `support`: Proporsi transaksi yang mengandung itemset
- `itemsets`: Kombinasi actual dari nama-nama kopi

### Association Rules

DataFrame `rules_by_card` berisi aturan asosiasi yang menggambarkan hubungan antara frequent itemsets:

- `antecedents`: Itemset pada sisi kiri aturan ("jika" bagian)
- `consequents`: Itemset pada sisi kanan aturan ("maka" bagian)
- `support`: Support dari keseluruhan aturan
- `confidence`: Probabilitas pelanggan akan membeli consequent ketika mereka membeli antecedent
- `lift`: Rasio observed support terhadap expected support jika independen
- Dan metrik-metrik lainnya (leverage, conviction, jaccard, dll.)

### Business Strategies

Hasil analisis dapat menginformasikan berbagai strategi bisnis:

- **Product Placement**: Menempatkan item dengan asosiasi tinggi berdekatan
- **Cross-Selling**: Merekomendasikan produk terkait berdasarkan association rules
- **Targeted Promotions**: Menargetkan pelanggan dengan promosi berdasarkan pola pembelian
- **Inventory Management**: Mengoptimalkan stok berdasarkan kombinasi populer
- **Bundle Offers**: Menciptakan paket untuk item yang sering dibeli bersamaan

## Visualization

### Frequent Itemset Visualization

#### Support Distribution (Line Plot)

```python
from matplotlib import pyplot as plt

frequent_itemsets_by_card['support'].plot(kind='line', figsize=(8, 4),title='Support of Frequent Itemsets')
plt.ylabel('Support')
plt.xlabel('Itemset Index')
plt.gca().spines[['top', 'right']].set_visible(False)
plt.show()
```

#### Support Distribution (Histogram)

```python
frequent_itemsets_by_card['support'].plot(kind='hist', bins=20,title='Distribution of Frequent Itemset Support')
plt.xlabel('Support')
plt.ylabel('Frequency')
plt.gca().spines[['top', 'right']].set_visible(False)
plt.show()
```

#### Itemsets vs. Support (Violin Plot)

```python
import seaborn as sns

figsize = (12, 1.2 * len(frequent_itemsets_by_card['itemsets'].unique()))
plt.figure(figsize=figsize)
sns.violinplot(data=frequent_itemsets_by_card, x='support', y='itemsets', inner='stick', palette='Dark2')
plt.title('Support of Frequent Itemsets')
plt.xlabel('Support')
plt.ylabel('Itemsets')
sns.despine(top=True, right=True, bottom=True, left=True)
plt.show()
```

## Application

### Recommendation Systems

Insights dari frequent itemsets dan association rules sangat berharga untuk membangun sistem rekomendasi. Contohnya, jika pelanggan menambahkan 'Espresso' ke keranjang, aturan asosiasi seperti `{Espresso} => {Americano}` dengan confidence dan lift tinggi dapat memicu rekomendasi untuk 'Americano'.

### Business Strategies Lainnya

- **Targeted Marketing Campaigns**: Mengidentifikasi segmen pelanggan berdasarkan pola pembelian
- **Optimizing Product Placement**: Menempatkan item dengan asosiasi kuat berdekatan
- **Improving Inventory Management**: Forecasting demand untuk kombinasi produk
- **Bundle Offers and Promotions**: Menciptakan penawaran khusus untuk item yang sering dibeli bersamaan
- **Cross-Selling and Upselling**: Memberikan saran produk tambahan berdasarkan pembelian saat ini

## Libraries Used

```python
import pandas as pd              # Data manipulation dan preprocessing
import os                        # Interaksi dengan sistem operasi
from mlxtend.preprocessing import TransactionEncoder  # One-hot encoding
from mlxtend.frequent_patterns import apriori, association_rules  # Algoritma Apriori
import matplotlib.pyplot as plt  # Visualisasi data
import seaborn as sns           # Enhanced data visualization
```

## Installation

Untuk menjalankan project ini, install dependencies berikut:

```bash
pip install pandas mlxtend matplotlib seaborn
```

---

**Note**: Project ini menggunakan algoritma Apriori klasik untuk market basket analysis. Untuk dataset yang lebih besar, pertimbangkan menggunakan algoritma yang lebih efisien seperti FP-Growth.