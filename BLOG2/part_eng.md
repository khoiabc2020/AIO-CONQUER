![Churn Problem Overview](<Screenshot 2026-03-22 224527.png>)
# 1. The Churn Problem in Business

## 1.1. What is *Churn*?
Imagine your business as a bucket of water: you keep pouring in new water (new customers), but there are holes at the bottom where water leaks out (customers leaving).
That leakage is **churn**.

In telecom, churn happens when a subscriber stops renewing a plan, cancels a service, or switches to a competitor.
So churn is not just losing one phone number. It is losing recurring revenue and future upsell opportunities.

## 1.2. Why is Churn a critical issue?
- **Direct revenue loss:** each customer who leaves reduces recurring cash flow.
- **High replacement cost:** acquiring a new customer is usually more expensive than retaining an existing one.
- **Long-term growth impact:** a high churn rate often indicates service, pricing, or customer experience problems.

## 1.3. Blog goals
In this blog, we aim to:
- Analyze customer behavior through data.
- Build a Machine Learning model to predict `Churn/No Churn`.
- Propose practical retention actions for business teams.


# 2. Exploratory Data Analysis (EDA)

> Note: in EDA code blocks, use one consistent dataframe name (`df`) to avoid runtime errors.

Quick dataset profile:
- Total rows: **7043**
- Total columns: **21**
- Target variable: **`Churn`**

## 2.1. EDA setup
1. Libraries
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

2. Configuration and data upload in Colab
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

3. Compact EDA for key variables

Analyze factors related to *Churn*
```python
df = pd.read_csv(DATA_PATH)

# Plot layout
sns.set_theme(style="whitegrid")
fig, axes = plt.subplots(2, 2, figsize=(14, 9))

# Churn distribution
sns.countplot(data=df, x="Churn", ax=axes[0, 0])
axes[0, 0].set_title("Churn Distribution")

# Contract type
sns.countplot(data=df, x="Contract", hue="Churn", ax=axes[0, 1])
axes[0, 1].set_title("Churn by Contract")
axes[0, 1].tick_params(axis="x", rotation=15)

# Internet service
sns.countplot(data=df, x="InternetService", hue="Churn", ax=axes[1, 0])
axes[1, 0].set_title("Churn by InternetService")
axes[1, 0].tick_params(axis="x", rotation=15)

# Payment method
sns.countplot(data=df, x="PaymentMethod", hue="Churn", ax=axes[1, 1])
axes[1, 1].set_title("Churn by PaymentMethod")
axes[1, 1].tick_params(axis="x", rotation=20)

plt.tight_layout()
plt.show()

# Monthly charges and tenure
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
sns.boxplot(data=df, x="Churn", y="MonthlyCharges", ax=axes[0])
axes[0].set_title("MonthlyCharges by Churn")

sns.boxplot(data=df, x="Churn", y="tenure", ax=axes[1])
axes[1].set_title("Tenure by Churn")

plt.tight_layout()
plt.show()

# Additional required variables
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
sns.countplot(data=df, x="TechSupport", hue="Churn", ax=axes[0])
axes[0].set_title("Churn by TechSupport")
axes[0].tick_params(axis="x", rotation=15)

sns.countplot(data=df, x="OnlineSecurity", hue="Churn", ax=axes[1])
axes[1].set_title("Churn by OnlineSecurity")
axes[1].tick_params(axis="x", rotation=15)

plt.tight_layout()
plt.show()
```

<p align = "center">
    <img src = "Screenshot 2026-03-22 214135.png" alt = "Churn distribution" width = "700"/>
    <img src = "Screenshot 2026-03-22 214238.png" alt = "Internet service and payment method" width = "700"/>
    <img src = "Screenshot 2026-03-22 214010.png" alt = "Monthly charges and tenure" width = "700"/>
</p>

## 2.2. Additional histogram
```python
tenure_mean = doc_file['tenure'].mean()

doc_file['tenure'].plot(
    kind = "hist",
    bins = 10,
    title = "Tenure Distribution",
    xlabel = "Months",
    ylabel = "Count",
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
    <img src = "Screenshot 2026-03-22 210619.png" alt = "Tenure histogram" width = "700"/>
</p>

## 2.3. EDA insights
High-risk churn segments usually have:
- Month-to-month contracts
- Electronic check payments
- Higher monthly charges
- Lower tenure (new customers)
- Fiber optic internet users


# 3. Data Preprocessing

## 3.1. Split data and build preprocessing pipeline
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
    <img src = "Screenshot 2026-03-22 220043.png" alt = "Preprocessing pipeline" width = "700"/>
</p>

## 3.2. Data cleaning code

1. Read CSV
```python
doc_file = pd.read_csv(DATA_PATH)
```

2. Basic summary before cleaning
```python
doc_file.describe()
```

3. Missing check
```python
# Missing values per column
print(doc_file.isnull().sum())
```

4. Convert columns to correct numeric types
```python
col_nums = ['MonthlyCharges', 'tenure', 'TotalCharges']
for col in col_nums:
    doc_file[col] = pd.to_numeric(doc_file[col], errors='coerce')
```

5. Inspect unique categories
```python
col_all_YN = ['gender', 'Partner', 'Dependents','MultipleLines', 'PhoneService', 
            'InternetService', 'OnlineSecurity', 'OnlineBackup', 'DeviceProtection', 
            'TechSupport', 'StreamingTV','StreamingMovies',  'Contract',
            'PaperlessBilling', 'PaymentMethod', 'Churn']

for x in col_all_YN:
    print(doc_file[x].unique())
```
Conclusion: values are consistent for modeling.

6. Encoding
Text/categorical values must be encoded into numeric representations for ML models.

Example:
- `'No'` -> `0`
- `'Yes'` -> `1`

For `gender`, when using `LabelEncoder`, mapping is learned automatically (commonly `Female = 0`, `Male = 1`).

```python
from sklearn.preprocessing import LabelEncoder, MinMaxScaler

# Copy dataset
dataset = pd.read_csv('WA_Fn-UseC_-Telco-Customer-Churn.csv')

# Convert TotalCharges to numeric
dataset["TotalCharges"] = pd.to_numeric(dataset["TotalCharges"], errors="coerce")

# Fill missing values
dataset["TotalCharges"].fillna(dataset["TotalCharges"].median(), inplace=True)

# Binary encoding
binary_cols = ["gender", "Partner", "Dependents", "PhoneService", "PaperlessBilling", "Churn"]
lab = LabelEncoder()
for col in binary_cols:
    dataset[col] = lab.fit_transform(dataset[col])

# Multi-category one-hot
multi_cols = ["InternetService", "Contract", "PaymentMethod", 
              "MultipleLines", "OnlineSecurity", "OnlineBackup", 
              "DeviceProtection", "TechSupport", "StreamingTV", "StreamingMovies"]
dataset = pd.get_dummies(dataset, columns = multi_cols)

# Scale numeric features
num_cols = ["tenure", "MonthlyCharges", "TotalCharges"]
scaler = MinMaxScaler()
dataset[num_cols] = scaler.fit_transform(dataset[num_cols])

print(dataset.head())
```

<p align = "center">
    <img src = "Screenshot 2026-03-22 201517.png" alt = "After encoding" width = "700"/>
    <img src = "Screenshot 2026-03-22 201524.png" alt = "After encoding" width = "700"/>
</p>

Important note:
- Use **one main preprocessing strategy** for training (pipeline-based or manual), and avoid mixing both in one final experiment.
- If `ColumnTransformer + OneHotEncoder` is your official pipeline, the `LabelEncoder + get_dummies` part should be treated as a reference/demo approach.


# 4. Model Building and Evaluation
## 4.1. Build the best-performing model

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
### 4.1.1. Required core model set (as assigned): Logistic, Random Forest, XGBoost
To align with the assignment requirements, the core set can be defined as:

```python
from xgboost import XGBClassifier

models_core = {
    "LogisticRegression": LogisticRegression(max_iter=1000, class_weight="balanced"),
    "RandomForest": RandomForestClassifier(
        n_estimators=500, random_state=RANDOM_STATE, n_jobs=-1, class_weight="balanced"
    ),
    "XGBoost": XGBClassifier(
        n_estimators=400,
        learning_rate=0.05,
        max_depth=4,
        subsample=0.9,
        colsample_bytree=0.9,
        objective="binary:logistic",
        eval_metric="logloss",
        random_state=RANDOM_STATE,
    ),
}

```

Model comparison
<p align = "center">
    <img src = "Screenshot 2026-03-22 220610.png" alt = "Model comparison" width = "700"/>
</p>

Save results and best model
```python
results_display_df.to_csv(MODEL_RESULTS_FILENAME, index=False)

best_model_code = results_df.iloc[0]["model_code"]
best_model_name = results_df.iloc[0]["Model"]
best_clf = fitted_models[best_model_code]
joblib.dump(best_clf, BEST_MODEL_FILENAME)

print("Saved model comparison to:", MODEL_RESULTS_FILENAME)
print("Saved best model:", best_model_name, "->", BEST_MODEL_FILENAME)
```
Result:
- Saved model comparison to: `model_results.csv`
- Saved best model: **Logistic Regression** -> `best_churn_model.joblib`

## 4.2. Feature importance
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
Feature importance output:
![Feature importance](<Screenshot 2026-03-22 221331.png>)


# 5. Practical Demo
## 5.1. Full-dataset churn prediction scenario

- Goal: predict `Churn` or `No Churn`

```python
# Setup
TARGET = "Churn"
ID_COL = "customerID" 

# Build train data from full dataset
train = dataset.copy()
y = train[TARGET]
X = train.drop(columns=[TARGET, ID_COL])
X_test = X.copy()
test_ids = dataset[ID_COL]

# FEATURE ENGINEERING
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

# SUBMISSION
submission = pd.DataFrame({
    "customerID": test_ids,
    "Churn_Prob": test_preds,
    "Prediction": np.where(test_preds >= 0.5, "Churn", "No Churn")
})

submission.to_csv("submission.csv", index=False)
print("Submission created successfully!")
print(submission.head())
```

<p align = "center">
    <img src = "Screenshot 2026-03-22 205500.png" alt = "Fold-by-fold quality" width = "300"/>
    <img src = "Screenshot 2026-03-22 205527.png" alt = "Churn probability output" width = "300"/>
</p>

## 5.2. Single-customer prediction

Input any customer profile:
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
Result:
- Churn probability: `0.8576`
- Prediction: `Churn`

## 5.3. Business meaning
1. Real business application:
   - Identify high-risk customer groups
   - Send targeted retention offers
   - Improve technical support quality
   - Help CS teams prioritize outreach
2. Strategic value:
   - The model is not only predictive; it supports data-driven retention decisions.

# 6. Conclusion
The churn prediction problem helps identify customers likely to leave, enabling proactive retention strategies.

Proposed workflow:
1. Data preprocessing
2. Model building and training
3. Evaluation with standard metrics

Key churn drivers include tenure, contract type, internet-service-related factors, and monthly charges.

The Telco sample dataset on Kaggle does not fully reflect all real-world business dynamics; churn is also affected by competition, market trends, and macroeconomic conditions.

Operational conclusion:
- Based on the comparison table, the best model is selected by `ROC AUC`.
- This model can rank high-risk customers and support retention campaign prioritization.

# 7. References

1. Kaggle: [Telco Customer Churn](https://www.kaggle.com/datasets/blastchar/telco-customer-churn/data)

2. Scikit-learn Documentation: [https://scikit-learn.org/stable/](https://scikit-learn.org/stable/)

3. XGBoost Documentation: [https://xgboost.readthedocs.io/](https://xgboost.readthedocs.io/)

4. Canva Education (Design resource): [https://www.canva.com/](https://www.canva.com/)
