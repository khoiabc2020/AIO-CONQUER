![alt text](image.png)
# PHÁT HIỆN TIN GIẢ BẰNG MACHINE LEARNING

## 1. Giới thiệu

Trong thời đại mạng xã hội phát triển, thông tin được lan truyền với tốc độ rất nhanh. Tuy nhiên, điều này cũng kéo theo một vấn đề nghiêm trọng: tin giả (fake news).

Tin giả có thể gây ảnh hưởng lớn đến xã hội, từ việc làm sai lệch nhận thức đến tác động đến chính trị và sức khỏe cộng đồng. Do đó, việc xây dựng hệ thống tự động phát hiện tin giả là một bài toán quan trọng trong lĩnh vực Data Science và NLP.

Theo các nghiên cứu gần đây, việc sử dụng các mô hình học máy có thể giúp phân loại tin giả với độ chính xác cao, thay thế phương pháp kiểm duyệt thủ công vốn không còn khả thi với khối lượng dữ liệu lớn.


## 2. Bài toán

Mục tiêu của bài toán là xây dựng một mô hình có thể phân loại một bài báo thành:

- Tin thật (Real news)
- Tin giả (Fake news)

Input:
- Nội dung bài báo (text, title)

Output:
- Nhãn phân loại (0 hoặc 1)

Đây là bài toán **classification (phân loại nhị phân)** trong Machine Learning.



## 3. Dataset

Trong project này, sử dụng dataset WELFake gồm:

- title
- text
- label

Dữ liệu lấy trong bộ csv: "WELFake_Dataset.csv"

## 3.1. Thư viện
```python
import inspect
import json
import random
import re
from collections import Counter
from dataclasses import asdict, dataclass
from pathlib import Path
from urllib.request import urlretrieve

import joblib
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import seaborn as sns
import torch
import nltk
from datasets import Dataset, DatasetDict
from IPython.display import display
from nltk.corpus import stopwords
from nltk.stem import WordNetLemmatizer
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics import (
    accuracy_score,
    classification_report,
    confusion_matrix,
    f1_score,
    precision_score,
    recall_score,
)
from sklearn.model_selection import GridSearchCV, train_test_split
from sklearn.pipeline import Pipeline
from sklearn.svm import LinearSVC
from transformers import (
    AutoModelForSequenceClassification,
    AutoTokenizer,
    DataCollatorWithPadding,
    Trainer,
    TrainingArguments,
)
# - sklearn de train baseline
# - transformers/datasets de train DistilBERT
```


## 3.2. Load Data
```python

```

## 3.3. Cleaning text
```python
def clean(text: str) -> str:
    # Chuẩn hóa text cho baseline TF-IDF.
    text = str(text).lower()
    text = re.sub(r"[^a-z\\s]", " ", text)
    words = re.sub(r"\\s+", " ", text).strip().split()
    words = [word for word in words if word not in stop_words]
    words = [lemmatizer.lemmatize(word) for word in words]
    return " ".join(words)
```

## 3.4. Đánh giá mô hình trong một dictionary

```python
def metric_row(family: str, model: str, split: str, y_true, y_pred, params=None) -> dict[str, object]:
    #Gom metric thành 1 dòng để so sánh giữa các "model"
    return {
        "family": family,
        "model": model,
        "split": split,
        "accuracy": accuracy_score(y_true, y_pred),
        "precision": precision_score(y_true, y_pred, zero_division=0),
        "recall": recall_score(y_true, y_pred, zero_division=0),
        "f1": f1_score(y_true, y_pred, zero_division=0),
        "params": "" if params is None else json.dumps(params, sort_keys=True),
    }
```

Dữ liệu đã được gán nhãn sẵn, phù hợp cho bài toán supervised learning.

### Các vấn đề cần quan tâm:
- Dữ liệu có cân bằng không?
- Có missing values không?
- Có dữ liệu trùng lặp không?


## 4. Phân tích dữ liệu (EDA)

Trước khi xây dựng mô hình, bước quan trọng là khám phá và hiểu rõ đặc điểm của tập dữ liệu.  
Vai trò của EDA:
- Kiểm tra sự cân bằng giữa các lớp (tin giả và tin thật).
- Quan sát phân phối độ dài nội dung theo từng lớp.
- Phát hiện các đặc điểm bất thường hoặc thiên lệch trong dữ liệu.

Qua đó, ta có cái nhìn trực quan về dữ liệu, từ đó định hướng các bước tiền xử lý và lựa chọn mô hình phù hợp.

### 4.1 Phân bố nhãn
- Tỷ lệ "*fake*" và "*real*"
```python
# Plot class balance and content length by class.
fig, axes = plt.subplots(1, 2, figsize=(13, 4.5))

#Đếm số lượng từng phần tử "real"/"fake" trong cột "label"
count_df = df["label"].value_counts().sort_index().rename(index=names).reset_index()
count_df.columns = ["label_name", "count"]

#Tạo biểu đồ cột
sns.barplot(
    data=count_df,
    x="label_name",
    y="count",
    ax=axes[0],
    hue="label_name",
    dodge=False,
    palette=[palette[0], palette[1]],
    legend=False,
)
axes[0].set_title("Class balance")
axes[0].set_xlabel("")
axes[0].set_ylabel("Rows")
#Thêm số lượng cụ thể mỗi cột 
for idx, row in count_df.iterrows():
    axes[0].text(idx, row["count"], f'{row["count"]:,}', ha="center", va="bottom", fontsize=10)
```
![alt text](image1.png)

### 4.2 Độ dài văn bản
- Tin giả và tin thật có độ dài khác nhau không?

```python 
#Biểu đồ histogram: Độ dài nội dung 
sns.histplot(
    data=df,
    x="content_len",
    hue="label",
    bins=40,
    kde=True,
    stat="density",
    common_norm=False,
    palette=palette,
    ax=axes[1],
)
axes[1].set_title("Content length by class")
axes[1].set_xlabel("Words")

plt.tight_layout()
plt.show()
```
![alt text](image2.png)

### 4.3 Từ vựng phổ biến
- Các từ thường xuất hiện trong fake news

```python
# Show the most frequent words in fake and real samples.
fake_words = Counter(" ".join(df.loc[df["label"] == 1, "content"]).split()).most_common(15)
real_words = Counter(" ".join(df.loc[df["label"] == 0, "content"]).split()).most_common(15)

fig, axes = plt.subplots(1, 2, figsize=(14, 5))

sns.barplot(
    data=pd.DataFrame(fake_words, columns=["word", "count"]),
    x="count",
    y="word",
    ax=axes[0],
    color=palette[1],
)
axes[0].set_title("Top words in fake news")

sns.barplot(
    data=pd.DataFrame(real_words, columns=["word", "count"]),
    x="count",
    y="word",
    ax=axes[1],
    color=palette[0],
)
axes[1].set_title("Top words in real news")

plt.tight_layout()
plt.show()
```

Nhận xét: Việc phân tích này giúp hiểu rõ đặc trưng của dữ liệu trước khi modeling.


## 5. Tiền xử lý dữ liệu

Dữ liệu văn bản thường chứa nhiều nhiễu, do đó cần làm sạch.

### Các bước xử lý:

- Chuyển về chữ thường  
- Loại bỏ URL  
- Loại bỏ HTML  
- Loại bỏ ký tự đặc biệt  
- Chuẩn hóa khoảng trắng  

### (Nâng cao):
- Stopwords removal  
- Lemmatization  

-->> Việc tiền xử lý giúp giảm nhiễu và cải thiện chất lượng feature.

## 6. Biểu diễn dữ liệu (Feature Engineering)

Máy học không hiểu text → cần chuyển sang dạng số.

### 6.1 TF-IDF

TF-IDF (Term Frequency – Inverse Document Frequency) là phương pháp biểu diễn văn bản phổ biến, cho phép đánh giá mức độ quan trọng của một từ trong một tài liệu so với toàn bộ tập dữ liệu.
Cụ thể, TF-IDF tăng trọng số cho những từ xuất hiện nhiều trong một văn bản nhưng hiếm trong các văn bản khác, từ đó giúp mô hình học máy tập trung vào các đặc trưng có giá trị phân biệt cao. Phương pháp này được sử dụng rộng rãi trong các hệ thống phân loại văn bản và phát hiện tin giả.


### 6.2 Tokenization (Deep Learning)

- Chuyển text → sequence
- Padding để cùng độ dài


## 7. Mô hình

### 7.1 Baseline (TF-IDF + Linear Model)

- Nhanh
- Dễ triển khai
- Hiệu quả cao trong text classification


### 7.2 Deep Learning (LSTM / BERT)

- Hiểu ngữ cảnh tốt hơn
- Phù hợp với dữ liệu phức tạp

Các nghiên cứu cho thấy các mô hình học sâu và SVM có thể đạt độ chính xác cao hơn so với các phương pháp truyền thống :contentReference[oaicite:2]{index=2}.


## 8. Huấn luyện mô hình

### Quy trình:

- Chia dữ liệu: train / test  
- Huấn luyện model  
- Tuning hyperparameters  
- Sử dụng callbacks để tránh overfitting  


## 9. Đánh giá mô hình

Không chỉ dùng accuracy, cần sử dụng:

- Precision  
- Recall  
- F1-score  
- Confusion Matrix  

-->> F1-score đặc biệt quan trọng trong bài toán này vì cần cân bằng giữa false positive và false negative.


## 10. Kết quả và phân tích

- Model đạt độ chính xác: ...  
- So sánh giữa các model  

### Nhận xét:
- Model nào tốt hơn?
- Vì sao?

## 11. Phân tích lỗi (Error Analysis)

- Các trường hợp model dự đoán sai  
- Nguyên nhân:
  - ngôn ngữ mơ hồ
  - thiếu context
  - tiêu đề gây hiểu nhầm  

-->> Đây là phần thể hiện tư duy Data Science rõ nhất.


## 12. Hạn chế

- Dataset chưa đủ đa dạng  
- Chỉ dựa vào text  
- Không sử dụng context từ social media  


## 13. Hướng phát triển

- Sử dụng BERT / Transformer  
- Kết hợp dữ liệu mạng xã hội  
- Xây dựng hệ thống realtime  


## 14. Kết luận

Bài toán phát hiện tin giả có thể được giải quyết hiệu quả bằng các phương pháp Machine Learning và NLP.

Project này không chỉ dừng ở việc xây dựng mô hình, mà còn thể hiện một pipeline hoàn chỉnh từ xử lý dữ liệu, xây dựng model đến đánh giá và phân tích.

Trong tương lai, việc kết hợp các mô hình nâng cao và dữ liệu đa nguồn sẽ giúp cải thiện độ chính xác và tính ứng dụng thực tế.

## 15. Nguồn tham khảo
[1] [Thumbnail - Canva Education](https://www.canva.com/)

[2] [Website Kaggle - Fake news](https://www.kaggle.com/datasets/saurabhshahane/fake-news-classification/)
