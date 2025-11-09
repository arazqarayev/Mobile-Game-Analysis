# Mobile-Game-Analysis
Python, Data cleaning, Data Modelling- Günlük/Aylık Kullanıcı Aktivitesi (DAU / MAU)


import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

df = pd.read_csv(r'C:\Users\User\OneDrive\Desktop\data\AA.csv')
df.dtypes
df.head()
#Id             int64
#GameID         int64
#Username      object
#Bet            int64
#CashedOut    float64
#Bonus        float64
#Profit       float64
#BustedAt     float64
#PlayDate      object
len(df)
df.isna().sum()
df = df.dropna(subset=['CashedOut', 'Bonus', 'Profit'])
df.isna().sum()

# Hatalı değer listesi
bad_vals = ["", " ", "NA", "N/A", "na", "Na", "NULL", "Null", "null", "None", "-", "—", "?"]
df.map(lambda x: str(x).strip() in bad_vals) ## bu degerleri gosterir  bad vals icerisidne olan. yoxdur.(tek na degil bosluk eh seyi check ettik)


df['PlayDate'] = pd.to_datetime(df['PlayDate'], errors='coerce') # PlayDate sütununu datetime formatına çevirdik:

df['Date'] = df['PlayDate'].dt.date  # Tarih ve saat olarak iki ayrı sütun oluşturduk
df['Time'] = df['PlayDate'].dt.time

# Sonucu göruyoruz
print(df[['Username', 'Date', 'Time']].head()) ## ayrica ekranda gosterir

df = df.drop(columns=['PlayDate']) # play date sutunu silirik, lazim degil

# 1)Günlük aktif kullanıcı sayısını (DAU) hesaplayalim
dau = df.groupby('Date')['Username'].nunique().rename('DAU').reset_index()
dau.plot(x='Date', y='DAU', kind='line', figsize=(8,4), title='Daily Active Users (DAU)')
<img width="935" height="477" alt="image" src="https://github.com/user-attachments/assets/9bd12108-8820-4a11-8fdf-373ad36971b6" />

# 1. Date sütununu datetime64'e çevirek, Datetime olmasi gerekli
df['Date'] = pd.to_datetime(df['Date'])

# 2) Aylık bazda gruplamak için yeni sütun oluşturalim simdi
df['Month'] = df['Date'].dt.to_period('M')

# 3. MAU (Monthly Active Users) hesapla
mau = df.groupby('Month')['Username'].nunique().rename('MAU').reset_index()

# 4. Grafiği çizdik
mau.plot(x='Month', y='MAU', kind='bar', figsize=(8,4), title='Monthly Active Users (MAU)')



<img width="1012" height="537" alt="image" src="https://github.com/user-attachments/assets/96f4e361-8c60-4b3f-ad9a-5a8f7c6e6c01" />



