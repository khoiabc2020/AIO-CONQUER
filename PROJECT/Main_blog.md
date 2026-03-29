![alt text](image.png)

Có những bài viết chỉ cần đọc tiêu đề là đã thấy “có mùi”, nhưng cũng có những bài được viết đủ khéo để đánh lừa cả người đọc cẩn thận. Đó là lúc bài toán phát hiện tin giả trở nên thú vị: ranh giới giữa đúng và sai không còn nằm ở vài từ khóa lộ liễu, mà nằm ở cách dữ liệu được biểu diễn, cách mô hình học ngữ cảnh và cách ta đánh giá nó một cách nghiêm túc.

Project này bắt đầu từ đúng câu hỏi đó: nếu đứng giữa một bài viết trông rất thuyết phục, mô hình nào sẽ nhìn ra sự khác biệt trước, một *baseline tuyến tính* mạnh hay một *transformer* hiểu ngữ cảnh tốt hơn?

## 1. Lời nói đầu 

Tin giả là một kiểu dữ liệu rất khó chịu. Nó không thô đến mức chỉ cần lọc vài từ khóa là xong, nhưng cũng không tinh vi đến mức mọi mô hình đều bất lực. Cái khó nằm ở chỗ nó thường được viết theo cách rất giống tin thật: tiêu đề bắt mắt, nội dung có vẻ hợp lý, giọng điệu đủ thuyết phục để người đọc tin rằng mình đang tiếp nhận một thông tin đáng tin cậy.

Đó cũng là lý do bài toán phát hiện tin giả luôn thú vị trong NLP. Nó nằm đúng ở điểm giao giữa xử lý ngôn ngữ, mô hình hóa dữ liệu và tư duy triển khai thực tế. Một project tốt cho bài toán này không nên chỉ dừng ở việc “train một model cho có”, mà cần đi qua đủ các bước: hiểu dữ liệu, làm sạch văn bản, kiểm tra đặc điểm của tập dữ liệu, so sánh nhiều mô hình, đánh giá đúng cách, rồi lưu lại kết quả để có thể dùng tiếp trong ứng dụng.

Project này được xây dựng theo đúng tinh thần đó trên bộ dữ liệu **WELFake**. Mục tiêu không chỉ là phân loại bài viết thành `real` và `fake`, mà là tạo ra một pipeline đủ gọn, đủ rõ, và đủ thực dụng để vừa dùng cho học thuật, vừa có thể nối sang một giao diện demo.


## 2. Bài toán
|Input|Nội dung bài báo (text, title)|
|----|---|
|**Output**|**Nhãn phân loại (0 hoặc 1)**|

Đây là bài toán "*classification*" (phân loại nhị phân) trong Machine Learning.

Project sử dụng file `WELFake_Dataset.csv` với ba cột chính:

- `title`
- `text`
- `label`

Về mặt xử lý, notebook không làm gì quá rườm rà ở bước đầu. Nó giữ lại đúng ba cột này, loại bỏ các dòng thiếu `label`, loại bỏ dữ liệu trùng lặp, rồi ghép `title` và `text` thành một trường văn bản đầy đủ hơn là `raw_content`.

Một chi tiết nhỏ nhưng rất quan trọng nằm ở nhãn. Trong code hiện tại, nhãn được map lại như sau:

```python
df["label"] = df["label"].astype(int).map({0: 1, 1: 0})
names = {0: "real", 1: "fake"}
```

Điều này có nghĩa là trong toàn bộ pipeline:

- `0` là `real`
- `1` là `fake`

Đây là phần rất dễ bị bỏ qua khi viết báo cáo, nhưng nếu bỏ qua thì toàn bộ *precision*, *recall*, *confusion matrix* và *error analysis* đều có thể bị diễn giải ngược. Với những bài toán nhị phân như thế này, chỉ cần sai một quy ước nhãn là mọi phần nhận xét phía sau sẽ lệch toàn bộ.


## 3. Tiền xử lý dữ liệu
### Các vấn đề cần quan tâm:
- Dữ liệu có cân bằng không?
- Có missing values không?
- Có dữ liệu trùng lặp không?

Notebook tách dữ liệu thành hai dạng văn bản khác nhau:

- `raw_content`: bản gần với dữ liệu gốc, dùng cho DistilBERT
- `content`: bản đã làm sạch, dùng cho baseline TF-IDF

Cách tách này rất hợp lý. Mô hình **transformer** thường hoạt động tốt hơn khi đầu vào vẫn giữ được ngữ cảnh tự nhiên, còn **TF-IDF** lại hưởng lợi rõ rệt khi văn bản đã được chuẩn hóa và giảm nhiễu.

### Các bước xử lý:
- Chuyển về chữ thường  
- Loại bỏ URL  
- Loại bỏ HTML  
- Loại bỏ ký tự đặc biệt  
- Chuẩn hóa khoảng trắng  

### (Nâng cao):
- Stopwords removal  
- Lemmatization  

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
Nhận xét: Việc tiền xử lý giúp giảm nhiễu và cải thiện chất lượng feature.

## 4. Phân tích dữ liệu (EDA)

Trước khi xây dựng mô hình, bước quan trọng là khám phá và hiểu rõ đặc điểm của tập dữ liệu.  
Vai trò của EDA:
- Quan sát phân phối độ dài nội dung theo từng lớp.
- Phát hiện các đặc điểm bất thường hoặc thiên lệch trong dữ liệu.

Qua đó, ta có cái nhìn trực quan về dữ liệu, từ đó định hướng các bước tiền xử lý và lựa chọn mô hình phù hợp.

### 4.1. Phân bố nhãn và độ dài văn bản
- Đếm số lượng từng phần tử "real"/"fake" trong cột "label".
- Tin giả và tin thật có độ dài khác nhau không?

Sử dụng **value_counts** 
```python
fig, axes = plt.subplots(1, 2, figsize=(13, 4.5))
count_df = df["label"].value_counts().sort_index().rename(index=names).reset_index()
count_df.columns = ["label_name", "count"]
```
Sau đó tạo biểu đồ bởi seaborn trả kết quả, mỗi cột thêm thông tin rõ ràng.

data = count_df, data = df
![alt text](image5.png)
Nhận xét: Tình trạng số lượng mẫu dữ liệu gần như bằng nhau.

### 4.2. Phân cấp độ dài tiêu đề và chữ 
Tương tự, so sánh độ dài tiêu đề và chữ theo từng lớp.

ax = axes[0], ax = axes[1]

![alt text](image3.png)

### 4.4. Từ vựng phổ biến
- Các từ thường xuất hiện trong tin giả
Vẫn sử dụng hàm **Counter** đếm label 
```python
fake_words = Counter(" ".join(df.loc[df["label"] == 1, "content"]).split()).most_common(15)
real_words = Counter(" ".join(df.loc[df["label"] == 0, "content"]).split()).most_common(15)
```

Tần suất các từ xuất hiện trong **tin giả** và **tin thật**.
![alt text](image4.png)

Nội dung bài báo ghi lại từ lời nói các nhân vật chiếm phần lớn là thông tin sai.

Top word = "*trump*"

Nhận xét: Việc phân tích này giúp hiểu rõ đặc trưng của dữ liệu trước khi modeling.

## 5. Huấn luyện mô hình
### Quy trình:

1. Chia dữ liệu 
2. Huấn luyện model  
3. Điều chỉnh siêu tham số (Tuning hyperparameters)  
4. Sử dụng callbacks để tránh overfitting  

### 5.1. Chia tập train/validation/test
Notebook chia dữ liệu thành ba phần:
- `train`
- `validation`
- `test`

theo cấu hình:
```python
test_size = 0.2
val_size = 0.1
```

Notebook chia song song hai nhánh dữ liệu:
- `X = df["content"]` cho baseline
- `X_raw = df["raw_content"]` cho DistilBERT

Chia train và test 
```python 
X_tv, X_test, y_tv, y_test = train_test_split(
    X,
    y,
    test_size=cfg.test_size,
    stratify=y,
    random_state=SEED,
)
X_raw_tv, X_raw_test = train_test_split(
    X_raw,
    test_size=cfg.test_size,
    stratify=y,
    random_state=SEED,
)
```

Tính tỉ lệ validation trong phần train và val 
```python
val_ratio = cfg.val_size / (1 - cfg.test_size)
```

Chia tiếp train và val 
```python
X_train, X_val, y_train, y_val = train_test_split(
    X_tv,
    y_tv,
    test_size=val_ratio,
    stratify=y_tv,
    random_state=SEED,
)
X_raw_train, X_raw_val = train_test_split(
    X_raw_tv,
    test_size=val_ratio,
    stratify=y_tv,
    random_state=SEED,
)
```


||split|rows|
|---|---|---|
|0|train|44542|
|1|val|6364|
|2|test|12727|

Tỉ lệ cuối cùng:
- train ~ 70%
- val ~ 10%
- test ~ 20%

### 5.2. Train các baseline 
Sau khi huấn luyện sẽ so sánh trên tập validation.

Phần baseline của project dùng `TfidfVectorizer` với cấu hình:
```python
# Khối tiền xử lý đặc trưng của văn bản: biến dữ liệu text thành vector TF-IDF.
base_tfidf = TfidfVectorizer(
    stop_words="english",
    max_features=cfg.max_features,
    min_df=cfg.min_df,
    max_df=cfg.max_df,
    ngram_range=cfg.ngram_range,
)
```
Trên nền TF-IDF, notebook so sánh ba baseline:
- `MultinomialNB`
- `LogisticRegression`
- `LinearSVC`

Mỗi baseline bao gồm:
- pipeline: nối bước TF-IDF với classifier.
- params: tập hyperpaprameters.
```python
model_specs = {
    # Mô hình 1: Naive Bayes (MultinomialNB) - phù hợp cho dạng dữ liệu text rời rạc, nhanh, thường là baseline mạnh mẽ.
    "naive_bayes": {
        "pipe": Pipeline([("tfidf", base_tfidf), ("clf", MultinomialNB())]),
        "params": {"clf__alpha": np.logspace(-3, 1, 30)},
    },

    # Mô hình 2: Logistic Regression - mô hình tuyến tính ân bằng class imbalance qua class_weight.
    "logistic_regression": {
        "pipe": Pipeline([("tfidf", base_tfidf), ("clf", LogisticRegression(max_iter=2000))]),
        "params": {
            "clf__C": np.logspace(-3, 2, 40),
            "clf__class_weight": [None, "balanced"],
        },
    },

    # Mô hình 3: Linear SVC - tối ưu margin 
    "linear_svc": {
        "pipe": Pipeline([("tfidf", base_tfidf), ("clf", LinearSVC())]),
        "params": {
            "clf__C": np.logspace(-3, 2, 40),
            "clf__class_weight": [None, "balanced"],
        },
    },
}
```
Trong rất nhiều bài toán text classification, đặc biệt khi dữ liệu đã được làm sạch tốt, các mô hình tuyến tính vẫn ổn định, dễ giải thích hơn *transformer*, và đôi khi cho hiệu quả vượt mong đợi.

#### **Tuning hyperparameters**

`RandomizedSearchCV` chọn ngẫu nhiên một số lượng tổ hợp để thử nghiệm.

Ưu điểm:
- Nhanh hơn GridSearchCV: Khi không gian siêu tham số lớn, *RandomizedSearchCV* tiết kiệm thời gian và tài nguyên.
- Có thể chỉ định số lần thử `n_iter` để kiểm soát độ rộng tìm kiếm.
```python
search = RandomizedSearchCV(
        estimator=spec["pipe"],
        param_distributions=spec["params"],
        n_iter=12,
        scoring="f1",
        cv=cfg.cv,
        n_jobs=-1,
        random_state=SEED,
        refit=True,
    )
    search.fit(X_train, y_train)
```

#### Huấn luyện lại train và validation trước khi đánh giá 

```python
# Train lại baseline tốt nhất trên `train + validation`.
best_spec = model_specs[best_model_name]
best_search = RandomizedSearchCV(
    estimator=best_spec["pipe"],
    param_distributions=best_spec["params"],
    n_iter=12,
    scoring="f1",
    cv=cfg.cv,
    n_jobs=-1,
    random_state=SEED,
    refit=True,
)

#Fit trên tập train & validation 
best_search.fit(pd.concat([X_train, X_val]), pd.concat([y_train, y_val]))

# Dự đoán trên test 
tfidf_pred = best_search.best_estimator_.predict(X_test)
```
Sau đó cần lưu các mẫu baseline dự đoán sai vào file csv mới để phân tích sau: "error_samples.csv".

### 5.3. Baseline tốt nhất 

## 6. DistilBERT
DistilBERT là một phiên bản rút gọn của mô hình BERT. Nó được huấn luyện bằng kỹ thuật *knowledge distillation*, tức là một mô hình nhỏ hơn (student) học cách bắt chước mô hình lớn hơn (teacher – BERT).

### 6.1. Chuẩn bị dữ liệu cho DistilBERT
```python
# DistilBERT dùng model public, nên không cần Hugging Face token.
# Tokenizer biến text thành token id để model đọc được.
tok = AutoTokenizer.from_pretrained(cfg.model_ckpt, token=None)
# Chuẩn bị dataset từ pandas 
dset = DatasetDict(
    {
        "train": Dataset.from_pandas(pd.DataFrame({"text": X_tr, "labels": y_tr}), preserve_index=False),
        "validation": Dataset.from_pandas(pd.DataFrame({"text": X_raw_val, "labels": y_val}), preserve_index=False),
        "test": Dataset.from_pandas(pd.DataFrame({"text": X_raw_test, "labels": y_test}), preserve_index=False),
    }
)
```

Biến văn bản thành *token ID* và *attention mask*.
- Bước chuẩn hóa dữ liệu để mô hình hiểu được. DistilBERT vốn đã học ngữ nghĩa từ corpora lớn, nên chỉ cần token hóa là có thể fine-tune cho bài toán phân loại tin giả.
```python
dset = dset.map(
    # Đảm bảo văn bản dài được cắt gọn, tránh vượt giới hạn mô hình
    lambda batch: tok(batch["text"], truncation=True, max_length=cfg.max_len),
    batched=True,
).remove_columns(["text"]) #Chỉ giữ dữ liệu đã mã hóa
```

Tải model pre-trained rồi đổi head cho bài toán 2 lớp (fake và real)
```python
model = AutoModelForSequenceClassification.from_pretrained(
    cfg.model_ckpt,
    num_labels=2,
    id2label=names,
    label2id={v: k for k, v in names.items()},
    token=None,
)
```
DistilBERT được chọn vì *nhẹ hơn* BERT nhưng vẫn giữ hiệu năng cao (~97%), phù hợp cho bài toán phân loại tin giả.

Pipeline này cho phép *fine-tune* nhanh trên WelFake dataset, đảm bảo mô hình vừa chính xác vừa hiệu quả triển khai.

### 6.2. Train DistilBERT

Tạo hàm *make_args* cấu hình toàn bộ quá trình huấn luyện DistilBERT.
```python
def make_args():
    # Tạo dictionary cấu hình chung
    common = {
        "output_dir": str(cfg.out_dir / "transformer_run"),
        "learning_rate": cfg.lr,
        "per_device_train_batch_size": cfg.train_bs,
        "per_device_eval_batch_size": cfg.eval_bs,
        "num_train_epochs": cfg.epochs,
        "weight_decay": cfg.wd, # Hệ số regularization để tránh overfitting.
        "save_strategy": "epoch",
        "load_best_model_at_end": True,
        "metric_for_best_model": "f1",
        "report_to": "none",
        "fp16": gpu_ready,
        "dataloader_pin_memory": gpu_ready,
        "dataloader_num_workers": 2,
    }

    # Kiểm tra tên tham số trong TrainingArguments
    arg_names = set(inspect.signature(TrainingArguments.__init__).parameters)
    if "evaluation_strategy" in arg_names:
        common["evaluation_strategy"] = "epoch"
    elif "eval_strategy" in arg_names:
        common["eval_strategy"] = "epoch"
    common = {key: value for key, value in common.items() if key in arg_names}
    return TrainingArguments(**common)
```
--> Tối ưu GPU dùng fp16, pin_memory, num_workers tăng tốc độ huấn luyện.


Hàm tiện ích để khởi tạo Trainer một cách linh hoạt.
```python
def make_trainer(model, args, train_dataset, eval_dataset, tokenizer):
    trainer_kwargs = {
        "model": model,
        "args": args,
        "train_dataset": train_dataset,
        "eval_dataset": eval_dataset,
        "data_collator": DataCollatorWithPadding(tokenizer=tokenizer),
        "compute_metrics": trf_metrics,
    }
    trainer_signature = inspect.signature(Trainer.__init__)

    # Tương thích ngược với nhiều phiên bản Transformers.
    if "processing_class" in trainer_signature.parameters:
        trainer_kwargs["processing_class"] = tokenizer
    elif "tokenizer" in trainer_signature.parameters:
        trainer_kwargs["tokenizer"] = tokenizer
    return Trainer(**trainer_kwargs)
```


Trainer là bộ khung train/eval có sẵn của Hugging Face, tự động lo việc:
- train
- eval
- logging
- checkpoint 
```python
trainer = make_trainer(
    model=model,
    args=make_args(), # Chứa hyperparameters
    train_dataset=dset["train"],
    eval_dataset=dset["validation"],
    tokenizer=tok,
)
```

## 7. Đánh giá và phân tích lỗi 
Đánh giá DistilBERT trên *validation* và *test*.

Không chỉ dùng *accuracy*, cần sử dụng:
- Precision  
- Recall  
- F1-score  
- Confusion Matrix  


```python
# trf_val là kết quả validation
trf_val = pd.DataFrame(
    [
        {
            "family": "transformer",
            "model": cfg.model_ckpt,
            "split": "val",
            "accuracy": float(val_out["eval_accuracy"]),
            "precision": float(val_out["eval_precision"]),
            "recall": float(val_out["eval_recall"]),
            "f1": float(val_out["eval_f1"]),
            "params": "",
        }
    ]
)

# trf_test là kết quả test
trf_test = pd.DataFrame([metric_row("transformer", cfg.model_ckpt, "test", y_test, trf_pred)])
```
Nhận xét: F1-score đặc biệt quan trọng trong bài toán này vì cần cân bằng giữa false positive và false negative.

Mô hình được đánh giá trên tập *validation* và *test*, kết quả như sau:

### Kết quả và phân tích
- Model đạt độ chính xác: ...  
- So sánh giữa các model  

### Nhận xét:
- Model nào tốt hơn?
- Vì sao?

### Phân tích lỗi (Error Analysis)
- Các trường hợp model dự đoán sai  
- Nguyên nhân:
  - ngôn ngữ mơ hồ
  - thiếu context
  - tiêu đề gây hiểu nhầm  

Tạo bảng xếp hạng 
- "*f1*" và "*accuracy*" giảm dần -> mô hình tốt nhất sẽ nằm trên đầu bảng.
```python
val_frames = [val_df] + ([trf_val] if not trf_val.empty else [])
test_frames = [tfidf_test] + ([trf_test] if not trf_test.empty else [])

val_board = pd.concat(val_frames, ignore_index=True).sort_values(["f1", "accuracy"], ascending=False)
test_board = pd.concat(test_frames, ignore_index=True).sort_values(["f1", "accuracy"], ascending=False)
```

Lưu bảng kết quả validation và test ra file "*.csv*"

Đường dẫn được lấy từ "*cfg.out_dir*".
```python
val_path = cfg.out_dir / "val_results.csv"
test_path = cfg.out_dir / "test_results.csv"
manifest_path = cfg.out_dir / "manifest.json"

val_board.to_csv(val_path, index=False)
test_board.to_csv(test_path, index=False)
```

Tạo file manifest chứa mô hình tốt nhất, các pipeline, mẫu dữ liệu lỗi, kết quả test.
```python 
manifest = {
    "csv_path": str(get_csv()),
    "out_dir": str(cfg.out_dir),
    "best_family": test_board.iloc[0]["family"],
    "best_model": test_board.iloc[0]["model"],
    "tfidf_pipeline": str(pipe_path),
    "error_samples": str(err_path),
    "transformer_dir": None if trf_path is None else str(trf_path),
    "val_results": str(val_path),
    "test_results": str(test_path),
}
manifest_path.write_text(json.dumps(manifest, indent=2), encoding="utf-8")
```
Các artifact đầu ra hiện tại gồm:
- tfidf_pipeline.joblib
- error_samples.csv
- transformer/
- val_results.csv
- test_results.csv
- manifest.json

## 11. Nhận xét

Tất nhiên, hệ thống vẫn còn những giới hạn quen thuộc:
- Phụ thuộc vào văn bản
- Chưa khai thác metadata
- DistilBERT chưa đi vào suy luận thật trong app
- Chưa có đánh giá ngoài miền dữ liệu
- Khả năng giải thích mô hình còn cơ bản

Hướng phát triển tiếp theo:
- Đưa DistilBERT vào pipeline suy luận thật
- Mở rộng dữ liệu hoặc đánh giá trên tập khác
- Tăng cường explainability
- Tách train pipeline và serving pipeline rõ hơn
- Thêm calibration cho xác suất đầu ra
- Thêm test tự động và chuẩn hóa cấu trúc project


## 13. Kết luận

Bài toán phát hiện tin giả có thể được giải quyết hiệu quả bằng các phương pháp Machine Learning và NLP.

Project này không chỉ dừng ở việc xây dựng mô hình, mà còn thể hiện một pipeline hoàn chỉnh từ xử lý dữ liệu, xây dựng model đến đánh giá và phân tích.

Trong tương lai, việc kết hợp các mô hình nâng cao và dữ liệu đa nguồn sẽ giúp cải thiện độ chính xác và tính ứng dụng thực tế.

## 14. Nguồn tham khảo
[1] [Thumbnail - Canva Education](https://www.canva.com/)

[2] [Website Kaggle - Fake news](https://www.kaggle.com/datasets/saurabhshahane/fake-news-classification/)
