# 1. Bài toán *Churn* trong doanh nghiệp
## 1.1. Khái niệm *Churn*
Hãy hình dung doanh nghiệp của bạn giống như một chiếc xô đang đựng nước. Bạn đổ thêm nước vào (khách hàng mới), nhưng dưới đáy xô lại có những lỗ thủng khiến nước rò rỉ ra ngoài.
Hiện tượng "rò rỉ" đó chính là "*Churn*" (hiện tượng khách hàng rời bỏ - ngừng sử dụng dịch vụ).

Trong ngành viễn thông, *Churn* xảy ra khi một thuê bao quyết định ngừng gia hạn gói cước hoặc chuyển sang sử dụng dịch vụ của nhà mạng đối thủ. 
Đây không đơn thuần là việc mất đi một số điện thoại, mà là mất đi một dòng doanh thu ổn định.

## 1.2. Vì sao Churn là vấn đề sống còn?
- **Mất doanh thu trực tiếp**: Mỗi khách hàng rời đi là mất đi một khoản lợi nhuận.
- **Chi phí đắt đỏ để tìm người** mới: Các nghiên cứu đã chỉ ra rằng chi phí bỏ ra để thu hút khách hàng mới cao gấp 5 - 7 lần so với việc giữ chân khách hàng cũ.
- **Hiệu ứng domino**: Tỉ lệ "*Churn*" cao là dấu hiệu cho thấy sản phẩm/dịch vụ đang có vấn đề, gây ảnh hưởng, tác động tiêu cực đến uy tín và sự phát triển, tăng trưởng bền vững của doanh nghiệp.

## 1.3. Mục tiêu bài Blog
Thông qua bài viết này, chúng ta sẽ cùng nhau:
- Khám phá hành vi khách hàng qua lăng kính dữ liệu.
- Xây dựng mô hình Machine Learning để "gọi tên" những khách hàng có nguy cơ rời đi cao nhất.
- Đề xuất chiến lược thực tế để giữ chân các khách hàng trên trước khi quá muộn.


# 2. Khám phá dữ liệu EDA
## 2.1. Nguồn tham khảo
Website: Kaggle - Telco Customer Churn

Nhóm biến trong dữ liệu

Phân tích tỷ lệ *Churn*

Phân tích các yếu tố liên quan đến *Churn*

## 2.2. Xây dựng biểu đồ

# 3. Tiền xử lý dữ liệu

1. Thư viện
```python
import matplotlib.pyplot as plt
import pandas as pd
import numpy as np
from sklearn.preprocessing import MinMaxScaler
from sklearn.preprocessing import LabelEncoder
```

2. Đọc file CSV
```python
doc_file = pd.read_csv('WA_Fn-UseC_-Telco-Customer-Churn.csv')
```

3. Khái quát chung của data set trước khi xử lý
```python
doc_file.describe()
```

4. Kiểm tra lỗi
```python
#Kiểm tra cột bị thiếu, sum = 0 -> Đầy đủ
print(doc_file.isnull().sum())
```

5. Chuyển dữ liệu về đúng định dạng
```python
col_nums = ['MonthlyCharges', 'tenure', 'TotalCharges']
#General --> Number
for col in col_nums:
    doc_file[col] = pd.to_numeric(doc_file[col], errors='coerce')
```
6. Báo cáo số lượng khách hàng ngừng dịch vụ
```python
tenure_mean = doc_file['tenure'].mean()

doc_file['tenure'].plot(
    kind = "hist",
    bins = 10,
    title = "Bảng thời gian khách hàng ở lại theo Churn",
    xlabel = "Tháng",
    ylabel = "Số lượng",
    figsize=(8, 5),
    alpha = 0.2,
    grid = True,
    edgecolor = 'black'
)

plt.axvline(tenure_mean, color = 'red', linestyle = '--', linewidth = 2)
plt.text(tenure_mean+1, plt.ylim()[1]*0.9, f'Mean = {tenure_mean:.2f}', color = 'red', fontsize = 11)
plt.show()
```

7. Xét số lượng kiểu kết quả điền mỗi cột
Trạng thái *unique*
```python
col_all_YN = ['gender', 'Partner', 'Dependents','MultipleLines', 'PhoneService', 
            'InternetService', 'OnlineSecurity', 'OnlineBackup', 'DeviceProtection', 
            'TechSupport', 'StreamingTV','StreamingMovies',  'Contract',
            'PaperlessBilling', 'PaymentMethod', 'Churn']

for x in col_all_YN:
    print(doc_file[x].unique())
```
Kết luận: File đã clean

8. Encoding 
Trong mã hóa dữ liệu, các dạng dữ liệu dưới dạng chữ ('text' - string) cần chuyển sang mã hóa số để mô hình Machine Learning có thể hiểu. 

Ví dụ: 'NO', 'no', 'No' encode sang 0 \
'YES', 'yes', 'Yes' encode sang 1

Ở trong file CSV có xuất hiện '*Female*' và '*Male*' được phân loại ở cột '*gender*'.

Do vậy cần chuẩn hóa dữ liệu chữ để phân loại: \
- '*Female*' = 1
- '*Male*' = 0

```python
#Tạo riêng dataset để chuẩn hóa 
dataset = pd.read_csv('WA_Fn-UseC_-Telco-Customer-Churn.csv')
#Loại bỏ cột ID để chuẩn hóa

#Chuyển hóa dữ liệu của 'TotalCharges' từ object sang dạng number
dataset["TotalCharges"] = pd.to_numeric(dataset["TotalCharges"], errors="coerce")
#Điền dữ liệu thiếu
dataset["TotalCharges"].fillna(dataset["TotalCharges"].median(), inplace=True)

#Chuyển hóa nhị phân với 2 categories
binary_cols = ["gender", "Partner", "Dependents", "PhoneService", "PaperlessBilling", "Churn"]
lab = LabelEncoder()
for col in binary_cols:
    dataset[col] = lab.fit_transform(dataset[col])

#List chứa các cột nhiều hơn 2 categories
multi_cols = ["InternetService", "Contract", "PaymentMethod", 
              "MultipleLines", "OnlineSecurity", "OnlineBackup", 
              "DeviceProtection", "TechSupport", "StreamingTV", "StreamingMovies"]
dataset = pd.get_dummies(dataset, columns = multi_cols)

#Scale dữ liệu number
num_cols = ["tenure", "MonthlyCharges", "TotalCharges"]
scaler = MinMaxScaler()
dataset[num_cols] = scaler.fit_transform(dataset[num_cols])

#In kết quả ra màn hình 
print(dataset.head())
```

<p align = "center">
    <img src = "Screenshot 2026-03-22 201517.png" alt = "After encode"width = "700"/>
    <img src = "Screenshot 2026-03-22 201524.png" alt = "After encode" width = "700"/>
</p>


# 4. Xây dựng và đánh giá mô hình
## 4.1. Các mô hình được sử dụng
Liệt kê

## 4.2. Huấn luyện mô hình
- Train từng mô hình trên tập dữ liệu đã xử lý
- Dự đoán trên tập test

Đánh giá mô hình 
- Accuracy
- Precision
- Recall
- F1-score

Feature Importance
- Xác định các biến quan trọng nhất
- Vẽ biểu đồ top đặc trưng quan trọng
--> Biểu đồ 

## 4.3. So sánh kết quả 
|Tên mô hình|Ưu điểm|Lý do|
|---|---|---|
|Mô hình tuyến tính|Dễ giải thích||
|Mô hình ensemble|||
|Mô hình boosting|||

# 5. Bài toán thực tế
## 5.1. Dự đoán một khách hàng
- Dự đoán `Churn` hay `No Churn`

1. Thư viện
```python
# Đặt
TARGET = "Churn"
ID_COL = "customerID" 

# Tạo 'train' từ toàn bộ dataset
train = dataset.copy()
y = train[TARGET]
X = train.drop(columns=[TARGET, ID_COL])
X_test = X.copy()
test_ids = dataset[ID_COL]

# FEATURE ENGINEERING
# Tạo interaction features
X["Tenure_Monthly"] = X["tenure"] * X["MonthlyCharges"]
X_test["Tenure_Monthly"] = X_test["tenure"] * X_test["MonthlyCharges"]

X["Charge_Ratio"] = X["TotalCharges"] / (X["tenure"] + 1)
X_test["Charge_Ratio"] = X_test["TotalCharges"] / (X_test["tenure"] + 1)

X = X.astype("float32")
X_test = X_test.astype("float32")

gc.collect()

# STACKING
skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)

oof = np.zeros(len(X), dtype="float32")
test_preds = np.zeros(len(X_test), dtype="float32")

for fold, (train_idx, val_idx) in enumerate(skf.split(X, y)):
    print(f"\n===== Fold {fold+1} =====")

    X_train, X_val = X.iloc[train_idx], X.iloc[val_idx]
    y_train, y_val = y.iloc[train_idx], y.iloc[val_idx]

    # LEVEL 1: XGBOOST
    xgb = XGBClassifier(
        n_estimators=50000,
        learning_rate=0.02,
        max_depth=6,
        subsample=0.85,
        colsample_bytree=0.85,
        reg_lambda=5,
        reg_alpha=1,
        min_child_weight=3,
        gamma=0.2,
        objective="binary:logistic",
        eval_metric="auc",
        tree_method="hist",   
        random_state=fold * 99,
        early_stopping_rounds=1000
    )

    xgb.fit(
        X_train,
        y_train,
        eval_set=[(X_val, y_val)],
        verbose=False
    )

    train_pred = xgb.predict_proba(X_train)[:,1]
    val_pred = xgb.predict_proba(X_val)[:,1]
    test_pred = xgb.predict_proba(X_test)[:,1]

    # LEVEL 2: RESIDUAL RF
    residual = y_train - train_pred
    rf = RandomForestRegressor(
        n_estimators=400,
        max_depth=8,
        min_samples_leaf=10,
        random_state=fold * 111,
        n_jobs=-1
    )
    rf.fit(X_train, residual)

    val_res = rf.predict(X_val)
    test_res = rf.predict(X_test)

    # LOGIT BLENDING
    val_logit = logit(np.clip(val_pred, 1e-6, 1-1e-6))
    test_logit = logit(np.clip(test_pred, 1e-6, 1-1e-6))

    val_final = expit(val_logit + val_res)
    test_final = expit(test_logit + test_res)

    # Rank sharpening
    val_final = rankdata(val_final) / len(val_final)
    test_final = rankdata(test_final) / len(test_final)

    oof[val_idx] = val_final
    test_preds += test_final / skf.n_splits

    print("Fold AUC:", roc_auc_score(y_val, val_final))

print("\nFINAL CV AUC:", roc_auc_score(y, oof))

#SUBMISSION
submission = pd.DataFrame({
    "customerID": test_ids,
    "Churn_Prob": test_preds,
    "Prediction": np.where(test_preds >= 0.5, "Churn", "No Churn")
})

submission.to_csv("submission.csv", index=False)
print("Tạo submission thành công!")

#In kết quả ra màn hình
print(submission.head())
```
<p align = "center">
    <img src = "Screenshot 2026-03-22 205500.png" alt = "After encode"width = "300"/>
    <img src = "Screenshot 2026-03-22 205527.png" alt = "After encode" width = "300"/>
</p>

## 5.2. Ý nghĩa:
1. Ứng dụng thực tế trong doanh nghiệp
    - Xác định nhóm khách hàng có nguy cơ cao
    - Gửi ưu đãi giữ chân
    - Cải thiện dịch vụ hỗ trợ kỹ thuật
    - Hỗ trợ bộ phận CSKH ưu tiên chăm sóc
2. Ý nghĩa kinh doanh:
    - Mô hình không chỉ để dự đoán
    - Mà còn giúp doanh nghiệp ra quyết định giữ chân khách hàng


# 6. Kết luận

# 7. Nguồn tham khảo 