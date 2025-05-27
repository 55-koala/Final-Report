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

- 數據來源 : 
使用來自使用者每日輸入的個人支出資料，包含消費日期、類別、金額與備註等欄位，並以 Google Colab 為平台進行輸入與分析。資料會依照使用者輸入持續累積，並可選擇特定月份進行查詢與分析。

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
!pip install pandas seaborn matplotlib scikit-learn --quiet

import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
from sklearn.datasets import load_iris
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split
from sklearn.metrics import classification_report

iris = load_iris()
df = pd.DataFrame(iris.data, columns=iris.feature_names)
df['species'] = iris.target
df['species'] = df['species'].map({i: name for i, name in enumerate(iris.target_names)})

print("前五筆資料：")
print(df.head())

print("\n資料摘要：")
print(df.describe())

```
## 結果與分析
- 箱型圖：顯示不同品種下各特徵的分布

```python

for col in df.columns[:-1]:
    plt.figure(figsize=(6, 4))
    sns.boxplot(x='species', y=col, data=df)
    plt.title(f'{col} 不同品種的分布')
    plt.tight_layout()
    plt.show()

```

- 折線圖：每個品種的特徵平均值趨勢

```python

mean_features = df.groupby('species').mean()
plt.figure(figsize=(8, 5))
for species in mean_features.index:
    plt.plot(mean_features.columns, mean_features.loc[species], marker='o', label=species)

plt.title('各品種平均特徵值折線圖')
plt.xlabel('特徵')
plt.ylabel('平均值')
plt.xticks(rotation=45)
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()

```

- 長條圖：展示各特徵在不同品種的平均值

```python

mean_features.T.plot(kind='bar', figsize=(10, 6))
plt.title('不同品種特徵平均值長條圖')
plt.ylabel('平均值')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()

```

- 建立模型並輸出分類報告

```python

X = df.iloc[:, :-1]
y = df['species']
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

model = LogisticRegression(max_iter=200)
model.fit(X_train, y_train)
y_pred = model.predict(X_test)

print("\n🔹 分類報告：")
print(classification_report(y_test, y_pred))

```
- 分析結果
  - petal 長度與寬度 是最具鑑別力的特徵，setosa 幾乎完全可依此分離。

  - sepal 特徵 在三個品種中有部分重疊，辨識效果相對較差。

  - versicolor 與 virginica 最易混淆，因特徵範圍較接近。

## 結論與建議
- 結論:
  - 花瓣長度與寬度（petal length, petal width） 為最具辨識力的特徵，尤其是對 Setosa，其特徵分布明顯與其他兩類區隔。

  - 萼片特徵（sepal length, sepal width） 區別能力較弱，三種花之間的重疊較多，特別是 Versicolor 與 Virginica。
   
  - 平均值圖（折線圖與長條圖） 顯示出三個品種在各特徵上有穩定的趨勢差異，尤其在花瓣相關特徵上分布差異明顯。

  - 使用邏輯迴歸模型進行分類，整體準確率高，尤其對 Setosa 分類準確率接近 100%。
  
- 建議:
  - 導入 PCA（主成分分析），以視覺化高維特徵在二維空間中的分布，可進一步驗證特徵可分性。
    
  - 除分類報告外，可加入 混淆矩陣、ROC 曲線、F1-score 等指標，以更全面評估模型性能。
  
  - 可將模型部署為簡單 Web 應用，供用戶輸入特徵資料並即時預測品種，提升互動性與應用價值。

