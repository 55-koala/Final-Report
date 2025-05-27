# Final Report

## 個人記帳管理系統


## 摘要

這次的主題是基於使用者每日支出的記錄資料，透過探索性資料分析與視覺化，探討不同支出類別的消費分布情況，並進行每月統計與每日預算上限檢查，以協助使用者掌握消費習慣並及早預警過度消費風險。報告同時將資料匯出為 Excel 檔案以利保存與進一步分析。圖表呈現支出結構與時間變化趨勢，可做為個人財務管理、自動記帳系統設計與理財教育範例的基礎。

## 引言

- 背景 : 
隨著行動支付與線上消費普及，個人理財需求日益增加，簡易可用的記帳與支出分析工具逐漸成為現代人管理財務的重要手段，這個記帳系統建置於 Google Colab 平台，結合 Python 資料處理與視覺化工具，能快速輸入、整理與分析消費紀錄。

- 目的 : 
透過每日記帳資料，建立一套基礎分析流程，內容包含：指定月份或全部資料的查詢、每月總支出與每日平均花費的統計、每日是否超過預算上限的警示、分類圓餅圖與時間折線圖的視覺化分析、資料自動匯出為 Excel 檔案等，透過這些功能，使用者能快速掌握財務狀況、找出主要花費類別並即時調整支出策略。

## 方法

- 分析工具 : 
  - Python：資料處理與分析的主要語言
    
  - Pandas：資料收集、清理、統計運算
  
  - Matplotlib：支出視覺化分析（長條圖、折線圖、圓餅圖）
  
  - OpenPyXL：將資料匯出為 Excel 檔案
  
  - Google Colab：程式執行與結果展示環境
  
  - Google Drive/Files：資料下載與儲存

- 數據處理與分析步驟 :
  - 1.接收使用者輸入的每日記帳資訊，包括：日期、類別、金額與備註
    
  - 2.將輸入資料存入 DataFrame 並依照日期排序
    
  - 3.使用者可選擇查詢特定月份或全部資料
    
  - 4.統計查詢資料的總花費、每日平均花費
    
  - 5.比對每日總支出是否超出設定上限（如每日上限為 500 元），並提供警示
    
  - 6.將完整資料輸出為 Excel 檔，供使用者下載備份
    
  - 7.使用圖表呈現「每日支出折線圖」與「類別支出比例圓餅圖」，輔助使用者理解消費習慣與分布情形

## 程式碼

```python
!pip install openpyxl matplotlib

import pandas as pd
import datetime
import matplotlib.pyplot as plt
from IPython.display import display
from google.colab import files

if 'records' not in globals():
    records = []

print("輸入一筆記帳資料：")
date_str = input("日期（YYYY-MM-DD，預設今天）: ") or datetime.date.today().strftime('%Y-%m-%d')
category = input("類別（如：餐飲、交通、娛樂）: ")
amount = float(input("金額: "))
note = input("備註（可空）: ")


records.append({
    "日期": pd.to_datetime(date_str),
    "類別": category,
    "金額": amount,
    "備註": note
})


df = pd.DataFrame(records)
df.sort_values("日期", inplace=True)


daily_limit = 500

print("\n全部記帳資料：")
display(df)


month_query = input("\n查詢月份（格式 YYYY-MM，留空=全部）: ")
if month_query:
    df['年月'] = df['日期'].dt.to_period("M").astype(str)
    df_month = df[df['年月'] == month_query]
else:
    df_month = df.copy()


total = df_month["金額"].sum()
days = df_month["日期"].nunique()
average = total / days if days else 0

print(f"\n統計資訊（{month_query if month_query else '全部'}）：")
print(f"總金額：{total:.2f}")
print(f"每日平均：{average:.2f}")


print("\n超出每日上限提醒：")
daily_sum = df_month.groupby("日期")["金額"].sum()
for day, total_amt in daily_sum.items():
    if total_amt > daily_limit:
        print(f" {day.date()} 花費 {total_amt} 超過上限 {daily_limit}")

filename = "記帳分析.xlsx"
df.to_excel(filename, index=False)
print(f"\n資料已儲存為：{filename}")
files.download(filename)


print("\n圖表分析：")
cat_sum = df_month.groupby("類別")["金額"].sum()
cat_sum.plot(kind='pie', autopct='%1.1f%%', figsize=(6,6), title="各類別支出比例")
plt.ylabel("")
plt.show()


daily_sum.plot(kind='line', marker='o', title="每日支出金額")
plt.xlabel("日期")
plt.ylabel("金額")
plt.xticks(rotation=45)
plt.grid(True)
plt.show()

```
## 結果與分析
- 圓餅圖：顯示不同類別的總支出比例，可清楚顯示消費重心，例如使用者在「餐飲」或「交通」類別上的花費比例，幫助找出花費最大的類別。

```python

cat_sum = df_month.groupby("類別")["金額"].sum()
cat_sum.plot(kind='pie', autopct='%1.1f%%', figsize=(6,6), title="各類別支出比例")
plt.ylabel("")
plt.show()


```

- 折線圖：每日支出變化趨勢，幫助觀察消費的波動情況，是否有特定日期支出異常偏高，以便進一步檢討當日花費行為。

```python

daily_sum = df_month.groupby("日期")["金額"].sum()
daily_sum.plot(kind='line', marker='o', title="每日支出金額")
plt.xlabel("日期")
plt.ylabel("金額")
plt.xticks(rotation=45)
plt.grid(True)
plt.show()


```

- 每日上限超出檢查 : 當使用者每日總花費超過預設的 500 元上限時，系統會主動提示，協助控制日常開支。

```python

daily_limit = 500
for day, total_amt in daily_sum.items():
    if total_amt > daily_limit:
        print(f" {day.date()} 花費 {total_amt} 超過上限 {daily_limit}")


```

- 統計摘要 : 每月（或整體）總金額與平均花費可幫助掌握整體支出概況，作為是否需要調整消費習慣的依據。

```python

total = df_month["金額"].sum()
days = df_month["日期"].nunique()
average = total / days if days else 0

print(f"▶ 總金額：{total:.2f}")
print(f"▶ 每日平均：{average:.2f}")


```

## 結論與建議
- 結論:
  - 使用者每日記帳資料可有效累積並進行分析，透過簡單的介面即可完成資料輸入、查詢與視覺化。
  
  - 消費總額與每日平均支出可即時回饋，幫助使用者掌握花費規模與趨勢。
  
  - 設定每日支出上限（如：500 元）並進行檢查，可有效提醒超支行為，具備良好的理財輔助功能。
  
  - 類別圓餅圖與每日支出折線圖直觀展現消費分布與變化趨勢，能幫助使用者觀察支出重點與異常消費日。
  
- 建議:
  - 可導入自動儲存功能，將資料每日同步儲存在雲端（如 Google Sheets 或 OneDrive Excel），減少資料遺失風險。
  
  - 增加月統計比較（如：月與月之間的支出變化）、年統計報告，協助用戶進行長期財務規劃。
  
  - 引入視覺化儀表板（如：Dash 或 Streamlit）提供即時動態展示，提升互動性與易用性。
  
  - 未來可擴充預算規劃與支出預測功能，例如透過過去資料進行機器學習預測未來消費模式，提高記帳系統的智慧化與主動提示能力。


 ## 參考資料

- Python 官方文件：https://docs.python.org/
- Pandas 官方手冊：https://pandas.pydata.org/docs/
- Matplotlib 文件：https://matplotlib.org/stable/contents.html
- Google Colab 說明：https://colab.research.google.com/
- OpenPyXL 官方文件：https://openpyxl.readthedocs.io/


