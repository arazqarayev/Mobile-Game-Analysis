# Mobile-Game-Analysis
Python, Data cleaning, Data Modelling- Günlük/Aylık Kullanıcı Aktivitesi (DAU / MAU)


import numpy as np     

import pandas as pd

import seaborn as sns

import matplotlib.pyplot as plt

<img width="1221" height="447" alt="image" src="https://github.com/user-attachments/assets/e2ac139f-a319-4c66-8aa5-75f20824dad7" />


# -----------------------------
# 0) VERİYİ YÜKLE
# -----------------------------
PATH = r"C:\Users\User\OneDrive\Desktop\data\AA.csv"
df = pd.read_csv(PATH)

print("Raw dtypes:\n", df.dtypes, "\n")
print("Raw shape:", len(df))
print("Missing counts (raw):\n", df.isna().sum(), "\n")

# -----------------------------
# 1) TEMİZLİK
# -----------------------------
# Analize kritik 3 metrikte (CashedOut, Bonus, Profit) NaN olan satırları at
df = df.dropna(subset=['CashedOut', 'Bonus', 'Profit'])

# Boş/Geçersiz metin değer kontrolü (isteğe bağlı rapor)
bad_vals = {"", " ", "NA", "N/A", "na", "Na", "NULL", "Null", "null", "None", "-", "—", "?"}
bad_mask = df.map(lambda x: str(x).strip() in bad_vals)
if bad_mask.any().any():
    print("⚠️ bad_vals içinde olan hücreler var. Gözden geçirin.")
else:
    print("✅ bad_vals içinde değer yok.")
    
<img width="855" height="495" alt="image" src="https://github.com/user-attachments/assets/d1a5d0b7-6de6-43ad-884d-18dce3a86f37" />

# -----------------------------
# 2) TARİH/SAAT AYIRIMI
# -----------------------------
df['PlayDate'] = pd.to_datetime(df['PlayDate'], errors='coerce')
# Kayıp tarihleri atmak istersen (opsiyonel):
# df = df.dropna(subset=['PlayDate'])

df['Date'] = df['PlayDate'].dt.date
df['Time'] = df['PlayDate'].dt.time
df = df.drop(columns=['PlayDate'])

# Görselleştirme uyumu için Date'i datetime64'e çevir
df['Date'] = pd.to_datetime(df['Date'])

print(df[['Username', 'Date', 'Time']].head())

# -----------------------------
# 3) DAU / WAU / MAU
# -----------------------------
# DAU (Daily Active Users): Günlük benzersiz kullanıcı
dau = (df.groupby('Date')['Username']
         .nunique()
         .rename('DAU')
         .reset_index())

# WAU (Weekly Active Users): Haftalık benzersiz kullanıcı
df['Week'] = df['Date'].dt.to_period('W')
wau = (df.groupby('Week')['Username']
         .nunique()
         .rename('WAU')
         .reset_index())

# MAU (Monthly Active Users): Aylık benzersiz kullanıcı
df['Month'] = df['Date'].dt.to_period('M')
mau = (df.groupby('Month')['Username']
         .nunique()
         .rename('MAU')
         .reset_index())

# -----------------------------
# 4) DAU/MAU ORANI (Sadakat)
# -----------------------------
dau['Month'] = dau['Date'].dt.to_period('M')
ratio = dau.merge(mau, on='Month', how='left')
ratio['DAU_MAU_Ratio'] = ratio['DAU'] / ratio['MAU']

# -----------------------------
# 5) KULLANICI FUNNEL / ÖZET METRİKLER
# -----------------------------
users = (df.groupby('Username')
           .agg(n_bets=('Bet', 'count'),
                total_bet=('Bet', 'sum'),
                total_cashed=('CashedOut', 'sum'),
                total_profit=('Profit', 'sum'))
           .reset_index())

share_cashed = (users['total_cashed'] > 0).mean()  # 0..1
print(f"\n🔎 Cashout yapan kullanıcı oranı: {share_cashed:.2%}")

# -----------------------------
# 6) RETENTION / COHORT ANALİZ
# -----------------------------
# Her kullanıcının ilk oynadığı günü bul
first_date = (df.groupby('Username')['Date']
                .min()
                .rename('first_date')
                .reset_index())

df2 = df.merge(first_date, on='Username')
df2['days_since_first'] = (df2['Date'] - df2['first_date']).dt.days

# Cohort pivot: (cohort=first_date (ay), gün farkına göre benzersiz kullanıcı sayısı)
df2['cohort'] = df2['first_date'].dt.to_period('M')
cohort = (df2.groupby(['cohort', 'days_since_first'])['Username']
            .nunique()
            .unstack(fill_value=0))

# İsteğe bağlı: cohort'u yüzdeye çevir (ilk güne göre normalizasyon)
cohort_pct = cohort.divide(cohort.iloc[:,0], axis=0).round(3)

# -----------------------------
# 7) GRAFİKLER
# -----------------------------
plt.figure(figsize=(9,4))
plt.plot(dau['Date'], dau['DAU'])
plt.title('Daily Active Users (DAU)')
plt.xlabel('Date')
plt.ylabel('Users')
plt.tight_layout()
plt.show()

plt.figure(figsize=(9,4))
plt.plot(wau['Week'].astype(str), wau['WAU'])
plt.title('Weekly Active Users (WAU)')
plt.xlabel('Week')
plt.ylabel('Users')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()

plt.figure(figsize=(9,4))
plt.bar(mau['Month'].astype(str), mau['MAU'])
plt.title('Monthly Active Users (MAU)')
plt.xlabel('Month')
plt.ylabel('Users')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()

plt.figure(figsize=(9,4))
plt.plot(ratio['Date'], ratio['DAU_MAU_Ratio'])
plt.title('DAU/MAU Ratio (Retention Proxy)')
plt.xlabel('Date')
plt.ylabel('Ratio')
plt.tight_layout()
plt.show()

# Cohort (Count) ısı haritası
plt.figure(figsize=(10,6))
sns.heatmap(cohort, cmap='viridis', cbar=True)
plt.title('Cohort – Unique Users by Days Since First Play')
plt.xlabel('Days Since First')
plt.ylabel('Cohort (Month)')
plt.tight_layout()
plt.show()

# Cohort (Percent) ısı haritası
plt.figure(figsize=(10,6))
sns.heatmap(cohort_pct, cmap='viridis', cbar=True)
plt.title('Cohort – Retention Ratio (Normalized to Day 0)')
plt.xlabel('Days Since First')
plt.ylabel('Cohort (Month)')
plt.tight_layout()
plt.show()

print("\nusers (sample):")
print(users.head(10))
print("\ncohort (counts) head:")
print(cohort.iloc[:5, :10])
print("\ncohort (ratios) head:")
print(cohort_pct.iloc[:5, :10])
