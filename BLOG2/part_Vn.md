![alt text](<Screenshot 2026-03-22 224527.png>)
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

## 2.1. Xây dựng EDA
1. Thư viện
```python
from sklearn.compose import ColumnTransformer
from sklearn.decomposition import PCA
from sklearn.ensemble import RandomForestClassifier
from sklearn.impute import SimpleImputer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, f1_score, precision_score, recall_score, roc_auc_score
from sklearn.model_selection import train_test_split
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler
from sklearn.svm import SVC
from sklearn.tree import DecisionTreeClassifier
```
2. Cấu hình và tải dữ liệu trên Colab
```python
from google.colab import files

TEST_SIZE = 0.2
RANDOM_STATE = 42
PCA_COMPONENTS = 10

MODEL_RESULTS_FILENAME = "model_results.csv"
BEST_MODEL_FILENAME = "best_churn_model.joblib"
PCA_IMPORTANCE_FILENAME = "pca_feature_importance.csv"
RF_IMPORTANCE_FILENAME = "rf_feature_importance.csv"

LOGREG_CONFIG = {"max_iter": 1000, "class_weight": "balanced"}
TREE_CONFIG = {"random_state": RANDOM_STATE}
RF_CONFIG = {"n_estimators": 500, "random_state": RANDOM_STATE, "n_jobs": -1, "class_weight": "balanced"}
SVM_CONFIG = {"kernel": "linear", "probability": True, "random_state": RANDOM_STATE}

def make_one_hot_encoder():
    try:
        return OneHotEncoder(handle_unknown="ignore", sparse_output=False)
    except TypeError:
        return OneHotEncoder(handle_unknown="ignore", sparse=False)

def get_uploaded_path():
    uploaded = files.upload()
    if not uploaded:
        raise ValueError("No file uploaded. Upload the CSV file to continue.")
    return next(iter(uploaded))

DATA_PATH = get_uploaded_path()
```
3. EDA gọn cho các biến quan trọng

Phân tích các yếu tố liên quan đến *Churn*
```python
dataset = pd.read_csv(DATA_PATH)

#Thiết kế biểu đồ 
sns.set_theme(style="whitegrid")
fig, axes = plt.subplots(2, 2, figsize=(14, 9))

#Phân bố Churn
sns.countplot(data=df, x="Churn", ax=axes[0, 0])
axes[0, 0].set_title("Churn Distribution")

#Thống kê loại dịch vụ viễn thông
sns.countplot(data=df, x="Contract", hue="Churn", ax=axes[0, 1])
axes[0, 1].set_title("Churn by Contract")
axes[0, 1].tick_params(axis="x", rotation=15)

#Thống kê dịch vụ Internet
sns.countplot(data=df, x="InternetService", hue="Churn", ax=axes[1, 0])
axes[1, 0].set_title("Churn by InternetService")
axes[1, 0].tick_params(axis="x", rotation=15)

#Thống kê phương thức thanh toán
sns.countplot(data=df, x="PaymentMethod", hue="Churn", ax=axes[1, 1])
axes[1, 1].set_title("Churn by PaymentMethod")
axes[1, 1].tick_params(axis="x", rotation=20)

plt.tight_layout()
plt.show()

#Thống kê số tiền phải trả hằng tháng 
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
sns.boxplot(data=df, x="Churn", y="MonthlyCharges", ax=axes[0])
axes[0].set_title("MonthlyCharges by Churn")

#Thống kê ngừng dịch vụ
sns.boxplot(data=df, x="Churn", y="tenure", ax=axes[1])
axes[1].set_title("Tenure by Churn")
#In kết quả 
plt.tight_layout()
plt.show()
```
<p align = "center">
    <img src = "Screenshot 2026-03-22 214135.png" alt = "Phân bổ Churn"width = "700"/>
    <img src = "Screenshot 2026-03-22 214238.png" alt = "Ngừng sử dụng dịch vụ Internet và phương thức thanh toán"width = "700"/>
    <img src = "Screenshot 2026-03-22 214010.png" alt = "Số tiền phải trả và thời gian gắn bó"width = "700"/>
</p>

## 2.2. Xây dựng biểu đồ
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
<p align = "center">
    <img src = "Screenshot 2026-03-22 210619.png" alt = "In ra kết quả"width = "700"/>
</p>

## 2.3. Nhận xét
Nhóm khách hàng có nguy cơ rời bỏ cao
- Hợp đồng Month-to-month
- Thanh toán bằng Electronic check
- Chi phí hàng tháng cao (MonthlyCharges lớn)
- Tenure thấp (khách hàng mới)
- Dùng dịch vụ Fiber optic Internet

# 3. Tiền xử lý dữ liệu

## 3.1. Chia dữ liệu và dựng pipeline tiền xử lý
```python
X = dataset.drop(columns=["Churn"])
y = dataset["Churn"]

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=TEST_SIZE, random_state=RANDOM_STATE, stratify=y
)

numeric_cols = X.select_dtypes(include=["number"]).columns.tolist()
categorical_cols = X.select_dtypes(include=["object"]).columns.tolist()

numeric_transformer = Pipeline(steps=[
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler()),
])

categorical_transformer = Pipeline(steps=[
    ("imputer", SimpleImputer(strategy="most_frequent")),
    ("onehot", make_one_hot_encoder()),
])

preprocessor = ColumnTransformer(
    transformers=[
        ("num", numeric_transformer, numeric_cols),
        ("cat", categorical_transformer, categorical_cols),
    ],
    remainder="drop",
)

pca_preprocessor = ColumnTransformer(
    transformers=[
        ("num", SimpleImputer(strategy="median"), numeric_cols),
        ("cat", make_one_hot_encoder(), categorical_cols),
    ],
    remainder="drop",
)

preprocessor
```
<p align = "center">
    <img src = "Screenshot 2026-03-22 220043.png" alt = "Pipeline"width = "700"/>
</p>

## 3.2. Code tiền xử lý dữ liệu

1. Đọc file CSV
```python
doc_file = pd.read_csv(DATA_PATH)
```

2. Khái quát chung của data set trước khi xử lý
```python
doc_file.describe()
```

3. Kiểm tra lỗi
```python
#Kiểm tra cột bị thiếu, sum = 0 -> Đầy đủ
print(doc_file.isnull().sum())
```

4. Chuyển dữ liệu về đúng định dạng
```python
col_nums = ['MonthlyCharges', 'tenure', 'TotalCharges']
#General --> Number
for col in col_nums:
    doc_file[col] = pd.to_numeric(doc_file[col], errors='coerce')
```

5. Xét số lượng kiểu kết quả điền mỗi cột
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

6. Encoding 
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
## 4.1. Xây dựng mô hình hiệu quả nhất

```python
models = {
    "ClassificationTree": DecisionTreeClassifier(**TREE_CONFIG),
    "RandomForest": RandomForestClassifier(**RF_CONFIG),
    "SVM": SVC(**SVM_CONFIG),
    "LogisticRegression": LogisticRegression(**LOGREG_CONFIG),
}

model_display_names = {
    "ClassificationTree": "Decision Tree",
    "RandomForest": "Random Forest",
    "SVM": "Linear SVM",
    "LogisticRegression": "Logistic Regression",
}

results = []
fitted_models = {}

for name, model in models.items():
    clf = Pipeline(steps=[("preprocess", preprocessor), ("model", model)])
    clf.fit(X_train, y_train)
    y_pred = clf.predict(X_test)
    y_proba = clf.predict_proba(X_test)[:, 1]

    results.append({
        "model_code": name,
        "Model": model_display_names[name],
        "Accuracy": accuracy_score(y_test, y_pred),
        "Precision": precision_score(y_test, y_pred),
        "Recall": recall_score(y_test, y_pred),
        "F1": f1_score(y_test, y_pred),
        "ROC AUC": roc_auc_score(y_test, y_proba),
    })

    fitted_models[name] = clf

results_df = pd.DataFrame(results).sort_values(by="ROC AUC", ascending=False)
results_display_df = results_df.drop(columns=["model_code"])
results_display_df

```
So sánh kết quả 
<p align = "center">
    <img src = "Screenshot 2026-03-22 220610.png" alt = "So sánh độ chính xác các mô hình ML"width = "700"/>
</p>

Lưu kết quả và mô hình tốt nhất
```python
results_display_df.to_csv(MODEL_RESULTS_FILENAME, index=False)

best_model_code = results_df.iloc[0]["model_code"]
best_model_name = results_df.iloc[0]["Model"]
best_clf = fitted_models[best_model_code]
joblib.dump(best_clf, BEST_MODEL_FILENAME)

print("Saved model comparison to:", MODEL_RESULTS_FILENAME)
print("Saved best model:", best_model_name, "->", BEST_MODEL_FILENAME)
```
Kết quả: 
- Saved model comparison to: model_results.csv 
- Saved best model: **Logistic Regression** -> best_churn_model.joblib

## 4.2. Các biến đặc trưng
```python
def get_feature_names(preprocessor):
    output_features = []
    for name, transformer, cols in preprocessor.transformers_:
        if name == "num":
            output_features += cols
        elif name == "cat":
            ohe = transformer.named_steps["onehot"]
            output_features += list(ohe.get_feature_names_out(cols))
    return output_features

feature_names = get_feature_names(preprocessor)

X_full = pca_preprocessor.fit_transform(X)
X_scaled = StandardScaler().fit_transform(X_full)
pca = PCA(n_components=PCA_COMPONENTS, random_state=RANDOM_STATE)
pca.fit(X_scaled)
feature_scores = np.abs(pca.components_).T @ pca.explained_variance_ratio_

pca_importance = pd.DataFrame({"feature": feature_names, "score": feature_scores}).sort_values(
    by="score", ascending=False
)

rf_model = fitted_models["RandomForest"].named_steps["model"]
rf_importance = pd.DataFrame({
    "feature": feature_names,
    "importance": rf_model.feature_importances_,
}).sort_values(by="importance", ascending=False)

pca_importance.to_csv(PCA_IMPORTANCE_FILENAME, index=False)
rf_importance.to_csv(RF_IMPORTANCE_FILENAME, index=False)

print("Saved:", PCA_IMPORTANCE_FILENAME, RF_IMPORTANCE_FILENAME)
pca_importance.head(10)
```
Kết quả phân bố: \
![Xác suất quan trọng](<Screenshot 2026-03-22 221331.png>)



# 5. Bài toán thực tế
## 5.1. Dự đoán cho dataset *Telco Customer Churn*

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
    <img src = "Screenshot 2026-03-22 205500.png" alt = "Đánh giá chất lượng mô hình trên mỗi lần chia"width = "300"/>
    <img src = "Screenshot 2026-03-22 205527.png" alt = "Xác suất đạt Churn hoặc No Churn" width = "300"/>
</p>

## 5.2. Dự đoán một khách hàng

Nhập vào input data bất kì 
```python
SAMPLE_INPUT = {
    "gender": "Female",
    "SeniorCitizen": 0,
    "Partner": "Yes",
    "Dependents": "No",
    "tenure": 5,
    "PhoneService": "Yes",
    "MultipleLines": "No",
    "InternetService": "Fiber optic",
    "OnlineSecurity": "No",
    "OnlineBackup": "Yes",
    "DeviceProtection": "No",
    "TechSupport": "No",
    "StreamingTV": "Yes",
    "StreamingMovies": "Yes",
    "Contract": "Month-to-month",
    "PaperlessBilling": "Yes",
    "PaymentMethod": "Electronic check",
    "MonthlyCharges": 85.0,
    "TotalCharges": 425.0,
}

sample = pd.DataFrame([SAMPLE_INPUT])
proba = best_clf.predict_proba(sample)[0, 1]
pred = best_clf.predict(sample)[0]

print("Churn probability:", round(proba, 4))
print("Prediction:", "Churn" if pred == 1 else "No Churn")
```
Kết quả:
- Churn probability: 0.8576
- Prediction: Churn

## 5.2. Ý nghĩa:
1. Ứng dụng thực tế trong doanh nghiệp
    - Xác định nhóm khách hàng có nguy cơ cao
    - Gửi ưu đãi giữ chân
    - Cải thiện dịch vụ hỗ trợ kỹ thuật
    - Hỗ trợ bộ phận CSKH ưu tiên chăm sóc
2. Ý nghĩa kinh doanh:
    - Mô hình không chỉ để dự đoán mà còn giúp doanh nghiệp ra quyết định giữ chân khách hàng

# 6. Kết luận

# 7. Nguồn tham khảo 

1. Website Kaggle: [Telco Customer Churn](https://www.kaggle.com/datasets/blastchar/telco-customer-churn/data)

2. Website Canva Education - Key word *Data*: (https://www.canva.com/)