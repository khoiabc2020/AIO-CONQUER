# Phát hiện tin giả với WELFake: từ huấn luyện trên Kaggle đến suy luận trong ứng dụng

## 1. Giới thiệu

Tin giả là một bài toán khó không phải vì nó hiếm, mà vì nó thường được trình bày theo cách rất giống tin thật. Một bài viết sai lệch có thể mang tiêu đề hấp dẫn, cấu trúc chặt chẽ, thậm chí sử dụng ngôn ngữ “giống báo chí” đến mức đánh lừa cả người đọc cẩn thận. Vì vậy, phát hiện tin giả không thể chỉ dựa vào trực giác hay vài từ khóa bề mặt; nó cần một quy trình phân tích dữ liệu và mô hình hóa được thiết kế nghiêm túc.

Project này xây dựng một pipeline hoàn chỉnh cho bài toán đó trên bộ dữ liệu **WELFake**. Mục tiêu không dừng ở việc huấn luyện một mô hình phân loại `real` và `fake`, mà mở rộng ra toàn bộ vòng đời thực nghiệm: đọc dữ liệu, làm sạch văn bản, phân tích dữ liệu khám phá, so sánh nhiều mô hình, đánh giá bằng các chỉ số phù hợp, lưu artifact, rồi kết nối với ứng dụng giao diện.

Toàn bộ phần trình bày dưới đây bám sát code hiện tại trong repo. Vì vậy, bài viết này không phải một bản tổng quan lý thuyết chung về fake news detection, mà là phần diễn giải trung thực của chính hệ thống đang được cài đặt.

## 2. Mục tiêu của project

Project được xây dựng quanh bốn mục tiêu cốt lõi.

Trước hết, nó cần một pipeline huấn luyện chạy ổn định trên Kaggle, có thể tận dụng GPU khi cần và lưu đầu ra theo cấu trúc rõ ràng. Tiếp theo, nó cần so sánh một nhóm baseline mạnh, nhẹ, dễ triển khai với một mô hình transformer gọn hơn là DistilBERT. Thứ ba, việc chọn mô hình phải dựa trên tiêu chí đánh giá phù hợp, không dựa trên cảm tính hay lựa chọn mặc định. Cuối cùng, đầu ra của quá trình huấn luyện phải đủ sạch và có tổ chức để ứng dụng giao diện có thể sử dụng lại.

Với cách tổ chức này, project đồng thời có giá trị học thuật cho một bài Data Science và giá trị kỹ thuật cho một hệ thống có khả năng mở rộng.

## 3. Dataset: WELFake và cách dữ liệu được dùng trong code

Project sử dụng bộ dữ liệu **WELFake**, đọc từ file `WELFake_Dataset.csv`. Theo notebook train hiện tại, dữ liệu không được tải động từ internet trong lúc chạy, mà được tìm trực tiếp trong `/kaggle/input`. Điều đó cho thấy notebook đang được thiết kế theo hướng **Kaggle-only**: nếu người dùng chưa `Add Data`, notebook sẽ dừng và báo lỗi rõ ràng.

Ba cột được dùng trực tiếp trong code là:

- `title`
- `text`
- `label`

Sau khi đọc dữ liệu, notebook giữ lại đúng ba cột này, loại bỏ các dòng thiếu `label`, loại bỏ bản ghi trùng lặp, rồi ghép `title` và `text` thành một trường văn bản đầy đủ hơn là `raw_content`.

Một chi tiết đặc biệt quan trọng là notebook hiện tại **đảo lại nhãn gốc** theo mapping:

```python
df["label"] = df["label"].astype(int).map({0: 1, 1: 0})
names = {0: "real", 1: "fake"}
```

Nói cách khác, trong pipeline hiện tại:

- `0` tương ứng với `real`
- `1` tương ứng với `fake`

Nếu viết báo cáo hoặc đọc kết quả mà bỏ qua chi tiết này, rất dễ diễn giải ngược toàn bộ precision, recall, confusion matrix và error analysis.

## 4. Môi trường chạy và khả năng tái lập

Notebook train hiện tại được tối ưu cho Kaggle. Phần đầu notebook kiểm tra trực tiếp môi trường bằng `Path("/kaggle").exists()`, đồng thời dò trạng thái GPU qua `torch.cuda.is_available()`.

Để tăng tính tái lập, notebook cố định ngẫu nhiên bằng `SEED = 42` cho:

- `random`
- `numpy`
- `torch`

Nếu có GPU, code còn gọi thêm:

```python
torch.cuda.manual_seed_all(SEED)
torch.backends.cudnn.benchmark = True
```

Đây là một cấu hình hợp lý cho thực nghiệm học máy: đủ để các lần chạy ổn định hơn, nhưng không đánh đổi hiệu năng quá nhiều.

Toàn bộ đầu ra sau huấn luyện được lưu vào:

`/kaggle/working/artifacts/fake_news`

Nhờ đó, quá trình train có cấu trúc rõ ràng và thuận tiện cho việc tải artifact về máy local hoặc chuyển tiếp sang bước triển khai.

## 5. Tiền xử lý dữ liệu: tối giản nhưng đúng mục tiêu

Một điểm đáng giá của project là notebook không “làm sạch quá tay” cho mọi mô hình. Thay vào đó, nó tạo ra **hai dạng văn bản khác nhau** để phục vụ hai họ mô hình khác nhau.

### 5.1. `raw_content`

`raw_content` được tạo bằng cách ghép trực tiếp:

```python
df["raw_content"] = (df["title"].fillna("") + " " + df["text"].fillna("")).str.strip()
```

Đây là dạng văn bản gần với dữ liệu gốc nhất và được dùng cho DistilBERT. Lựa chọn này hợp lý, vì transformer thường hoạt động tốt hơn khi đầu vào vẫn giữ được cấu trúc ngôn ngữ tự nhiên.

### 5.2. `content`

`content` là phiên bản đã được làm sạch từ `raw_content`, thông qua hàm:

```python
def clean(text: str) -> str:
    text = str(text).lower()
    text = re.sub(r"[^a-z\\s]", " ", text)
    words = re.sub(r"\\s+", " ", text).strip().split()
    words = [word for word in words if word not in stop_words]
    words = [lemmatizer.lemmatize(word) for word in words]
    return " ".join(words)
```

Hàm này thực hiện đúng các bước sau:

- đưa toàn bộ văn bản về chữ thường
- loại bỏ ký tự ngoài chữ cái tiếng Anh và khoảng trắng
- chuẩn hóa khoảng trắng
- tách từ
- loại stopwords tiếng Anh
- lemmatization bằng WordNet

Điều cần nhấn mạnh là code hiện tại **không có một bước riêng để loại URL hoặc HTML**. Vì vậy, nếu muốn viết sát với implementation, ta không nên mô tả thêm những bước mà notebook thực tế không làm.

Đối với baseline TF-IDF, `content` là đầu vào phù hợp vì nhiễu đã được giảm, từ vựng được chuẩn hóa hơn, và ma trận đặc trưng sparse thu được sẽ gọn hơn nhưng vẫn giàu tín hiệu.

## 6. Phân tích dữ liệu khám phá (EDA)

Phần EDA trong notebook không chỉ mang tính minh họa. Nó được dùng để kiểm tra những giả định cơ bản trước khi đi vào huấn luyện.

### 6.1. Phân bố nhãn

Notebook dùng `value_counts()` để kiểm tra số lượng mẫu thuộc hai lớp `real` và `fake`, sau đó trực quan hóa bằng bar chart. Mục tiêu của bước này là trả lời câu hỏi quan trọng nhất ở giai đoạn đầu: dữ liệu có mất cân bằng nghiêm trọng hay không.

Nếu lệch lớp quá mạnh, accuracy rất dễ trở thành một chỉ số đánh lừa. Ngược lại, nếu hai lớp tương đối cân bằng, việc diễn giải accuracy, precision, recall và F1 sẽ có giá trị hơn nhiều.

### 6.2. Độ dài nội dung

Notebook tính:

- `title_len`
- `text_len`
- `content_len`

Sau đó dùng histogram để quan sát phân bố độ dài nội dung theo từng lớp. Đây là một bước cần thiết, bởi trong thực tế:

- tin giả có thể ngắn hơn, giật hơn, hoặc giàu cụm từ kích thích cảm xúc
- tin thật thường có xu hướng dài hơn và giàu ngữ cảnh hơn

Không phải lúc nào sự khác biệt này cũng đủ mạnh để tách lớp, nhưng việc quan sát nó giúp người làm bài hiểu bản chất dữ liệu trước khi mô hình hóa.

### 6.3. Độ dài tiêu đề và thân bài theo lớp

Notebook còn dùng boxplot để so sánh riêng:

- độ dài tiêu đề
- độ dài phần thân bài

theo từng lớp. So với việc chỉ nhìn trung bình, boxplot cho thấy rõ hơn độ phân tán và sự hiện diện của outlier. Đây là một cách EDA tốt hơn hẳn việc chỉ in vài thống kê mô tả đơn giản.

### 6.4. Từ phổ biến theo từng lớp

Phần EDA tiếp theo dùng `Counter` để lấy 15 từ xuất hiện nhiều nhất trong:

- nhóm `fake`
- nhóm `real`

Mục tiêu không phải để kết luận vội rằng “cứ thấy từ X là tin giả”, mà để kiểm tra:

- dữ liệu có đang lộ ra các tín hiệu từ vựng đặc trưng hay không
- có bias hiển nhiên nào trong bộ dữ liệu hay không
- mô hình tuyến tính sẽ có khả năng khai thác những tín hiệu nào

Đây là bước nối rất tự nhiên giữa EDA và phần feature engineering phía sau.

## 7. Chia dữ liệu và lý do chia như vậy

Notebook hiện tại chia dữ liệu thành ba tập:

- train
- validation
- test

theo cấu hình:

```python
test_size = 0.2
val_size = 0.1
```

Đây là thiết kế đúng cho thực nghiệm học máy:

- `train` dùng để học tham số
- `validation` dùng để chọn mô hình và siêu tham số
- `test` dùng để đánh giá cuối cùng

Quá trình chia được thực hiện theo hai tầng. Đầu tiên, notebook tách `test` khỏi toàn bộ dữ liệu. Sau đó, từ phần còn lại, notebook tính:

```python
val_ratio = cfg.val_size / (1 - cfg.test_size)
```

rồi tách tiếp `validation`.

Cả hai bước chia đều dùng `stratify`, giúp giữ tỷ lệ nhãn tương đối ổn định giữa các tập. Đây là một chi tiết phương pháp luận rất quan trọng trong bài toán phân loại nhị phân.

Một điểm đáng giá khác là notebook không chỉ chia một nhánh dữ liệu duy nhất. Nó chia song song:

- `X = df["content"]` cho baseline
- `X_raw = df["raw_content"]` cho DistilBERT

Nhờ vậy, cả hai họ mô hình đều được huấn luyện và đánh giá trên cùng logic phân chia dữ liệu, nhưng vẫn giữ được kiểu đầu vào phù hợp với bản chất của từng mô hình.

## 8. Biểu diễn đặc trưng cho baseline: TF-IDF

Nhánh baseline của project dựa trên `TfidfVectorizer` với cấu hình:

```python
base_tfidf = TfidfVectorizer(
    stop_words="english",
    max_features=cfg.max_features,
    min_df=cfg.min_df,
    max_df=cfg.max_df,
    ngram_range=cfg.ngram_range,
)
```

Theo notebook hiện tại:

- `max_features = 50000`
- `min_df = 5`
- `max_df = 0.8`
- `ngram_range = (1, 2)`

Đây là một cấu hình rất thực dụng cho bài toán phân loại văn bản:

- `min_df=5` giúp loại bỏ những từ quá hiếm
- `max_df=0.8` giảm ảnh hưởng của các từ quá phổ biến
- `ngram_range=(1,2)` cho phép mô hình học cả unigram lẫn bigram

Trong nhiều bài toán text classification truyền thống, TF-IDF là một baseline rất khó đánh bại nếu dữ liệu đủ sạch và mô hình phân lớp phía sau đủ mạnh.

## 9. Ba baseline được so sánh trong notebook

Notebook hiện tại không chỉ train một baseline duy nhất. Nó so sánh ba mô hình trên cùng một không gian TF-IDF.

### 9.1. Multinomial Naive Bayes

Đây là baseline cổ điển cho dữ liệu văn bản. Nó rất nhanh, đơn giản và thường cho kết quả khởi đầu tốt. Dù giả định độc lập điều kiện giữa các từ khá mạnh và không hoàn toàn đúng trong ngôn ngữ tự nhiên, Naive Bayes vẫn là một cột mốc cần có trong hầu hết các bài toán phân loại văn bản.

### 9.2. Logistic Regression

Logistic Regression là mô hình tuyến tính phân biệt, hoạt động rất tốt trên đặc trưng sparse như TF-IDF. So với Naive Bayes, Logistic Regression linh hoạt hơn về biên quyết định và thường cho chất lượng tốt hơn nếu dữ liệu đủ phong phú.

### 9.3. LinearSVC

LinearSVC là một lựa chọn đặc biệt mạnh trong các bài toán văn bản chiều cao. Với TF-IDF, mô hình này thường tạo được biên phân lớp tốt và có chi phí tính toán hợp lý. Không phải ngẫu nhiên mà trong nhiều bài toán NLP cổ điển, LinearSVC là một baseline rất khó bị thay thế hoàn toàn.

Việc đặt cả ba mô hình vào cùng một quy trình so sánh là một quyết định tốt về mặt khoa học dữ liệu: thay vì khóa mình vào một lựa chọn ban đầu, notebook để cho chính dữ liệu quyết định mô hình nào phù hợp nhất.

## 10. Tuning siêu tham số bằng RandomizedSearchCV

Thay vì cố định siêu tham số thủ công, notebook dùng `RandomizedSearchCV` cho từng baseline với:

- `n_iter = 12`
- `cv = 5`
- `scoring = "f1"`
- `n_jobs = -1`

Đây là một lựa chọn hợp lý vì ba lý do.

Thứ nhất, không gian siêu tham số của các mô hình tuyến tính đủ lớn để việc thử thủ công dễ thiếu công bằng. Thứ hai, `RandomizedSearchCV` tiết kiệm thời gian hơn Grid Search khi không gian tìm kiếm rộng. Thứ ba, việc dùng `F1-score` làm tiêu chí tối ưu phù hợp hơn accuracy trong bối cảnh phát hiện tin giả, nơi precision và recall đều quan trọng.

Trong code hiện tại, các miền siêu tham số tiêu biểu gồm:

- `alpha` cho Naive Bayes
- `C` và `class_weight` cho Logistic Regression
- `C` và `class_weight` cho LinearSVC

Sau khi search xong, notebook đánh giá mô hình trên tập validation và lưu kết quả vào `val_df`.

## 11. Quy trình chọn baseline tốt nhất

Một điểm rất đáng khen của notebook là việc chọn mô hình không hề hard-code. Toàn bộ baseline được xếp hạng động bằng:

```python
val_df = pd.DataFrame(val_rows).sort_values(["f1", "accuracy"], ascending=False)
best_model_name = str(val_df.iloc[0]["model"])
```

Nói cách khác, mô hình chiến thắng là mô hình có:

1. `F1-score` cao nhất trên validation
2. nếu bằng nhau thì xét tiếp `accuracy`

Sau khi xác định baseline thắng trên validation, notebook **train lại mô hình tốt nhất trên `train + validation`**, rồi mới đánh giá trên `test`.

Đó là một lựa chọn chặt chẽ về phương pháp luận:

- validation hoàn thành vai trò chọn mô hình
- dữ liệu validation sau đó được tận dụng lại để tăng lượng dữ liệu huấn luyện
- test vẫn được giữ riêng cho đánh giá cuối cùng

Rất nhiều bài làm sinh viên bỏ qua bước này, nhưng đây mới là cách làm chuẩn hơn.

## 12. DistilBERT: nhánh transformer để so sánh

Bên cạnh baseline TF-IDF, notebook còn có một nhánh DistilBERT với checkpoint:

`distilbert-base-uncased`

DistilBERT được chọn thay vì các mô hình transformer lớn hơn vì nó cân bằng khá tốt giữa:

- chất lượng ngữ cảnh
- chi phí huấn luyện
- tính khả thi trên môi trường Kaggle

Tuy nhiên, notebook hiện tại không ép buộc lúc nào cũng phải train DistilBERT. Nhánh này chỉ được chạy khi:

- `cfg.use_transformer = True`
- và GPU khả dụng

Nếu không có GPU, notebook sẽ bỏ qua nhánh này và hiển thị trạng thái `skipped`. Đây là một thiết kế thực tế, bởi transformer trên CPU sẽ quá chậm và không phù hợp với một notebook thực nghiệm thông thường.

## 13. Đầu vào và cấu hình của DistilBERT

DistilBERT dùng `raw_content` chứ không dùng văn bản đã clean mạnh. Lựa chọn này đúng với bản chất của transformer: tokenizer và embedding pre-trained hoạt động tốt hơn khi đầu vào vẫn gần với ngôn ngữ tự nhiên.

Cấu hình hiện tại của notebook là:

- `max_len = 256`
- `epochs = 2`
- `train_bs = 16`
- `eval_bs = 32`
- `lr = 2e-5`
- `wd = 0.01`
- `sample_limit = 30000`

`sample_limit` là một chi tiết quan trọng. Nếu tập train lớn hơn 30.000 mẫu, notebook sẽ lấy mẫu ngẫu nhiên để giới hạn chi phí huấn luyện. Đây là một quyết định thực dụng và đủ hợp lý cho mục tiêu so sánh mô hình trong môi trường tài nguyên hữu hạn.

DistilBERT được huấn luyện qua `Trainer` của Hugging Face, với `DataCollatorWithPadding` và `compute_metrics` riêng cho accuracy, precision, recall và F1.

Một điểm kỹ thuật rất tốt trong code là notebook cố viết tương thích nhiều phiên bản thư viện bằng cách kiểm tra chữ ký hàm trước khi truyền tham số như:

- `evaluation_strategy` hoặc `eval_strategy`
- `processing_class` hoặc `tokenizer`

Chi tiết này giúp notebook bền hơn trước khác biệt phiên bản của `transformers`.

## 14. Hệ metric và cách đánh giá

Project hiện tại đánh giá mô hình bằng bốn chỉ số chính:

- `accuracy`
- `precision`
- `recall`
- `f1`

Toàn bộ được gom lại bởi hàm `metric_row`, giúp mọi mô hình, dù là baseline hay transformer, đều xuất kết quả về cùng một schema. Đây là một chi tiết nhỏ nhưng rất có giá trị trong thiết kế pipeline, vì nó làm cho việc so sánh giữa các mô hình trở nên gọn, sạch và ít lỗi hơn.

Ngoài các metric tổng hợp, notebook còn hiển thị:

- `classification_report`
- `confusion_matrix`

cho baseline tốt nhất. Điều đó cho phép phân tích sâu hơn loại lỗi mà mô hình đang mắc phải.

Trong bối cảnh phát hiện tin giả, **F1-score** là chỉ số trung tâm hơn accuracy, bởi accuracy có thể đẹp trong khi precision hoặc recall của lớp `fake` vẫn chưa đủ tốt. Một mô hình hữu dụng không chỉ cần “đoán đúng nhiều”, mà còn phải cân bằng giữa:

- tránh bỏ sót tin giả
- và tránh gán nhầm quá nhiều tin thật thành tin giả

## 15. Kết quả trong notebook được tổ chức như thế nào

Notebook hiện tại không gắn cứng một kết luận kiểu “LinearSVC luôn thắng” hay “DistilBERT luôn tốt hơn”. Thay vào đó, toàn bộ kết quả được để cho chính lần chạy notebook quyết định.

Sau khi train xong, notebook tạo:

- `val_board`
- `test_board`

và sắp xếp theo:

- `f1`
- rồi `accuracy`

Điều đó có nghĩa là kết luận cuối cùng về mô hình tốt nhất chỉ nên được đưa ra sau khi đã chạy notebook thực tế. Cách viết đúng trong báo cáo là:

- baseline tốt nhất được chọn động từ `val_df`
- mô hình tốt nhất cuối cùng được lấy từ `test_board`

Nếu notebook đã chạy và artifact đã được sinh ra, toàn bộ số liệu thật sẽ nằm trong:

- `val_results.csv`
- `test_results.csv`

Vì vậy, phần kết quả định lượng trong báo cáo nên được lấy trực tiếp từ hai file này thay vì điền tay.

## 16. Phân tích lỗi: phần thể hiện rõ nhất tư duy Data Science

Sau khi baseline tốt nhất được huấn luyện trên `train + validation`, notebook lưu các mẫu dự đoán sai vào `error_samples.csv`. Đây là một trong những phần có giá trị nhất của project, vì nó chuyển trọng tâm từ “mô hình đạt điểm bao nhiêu” sang câu hỏi quan trọng hơn:

- mô hình sai ở đâu
- sai với loại bài viết nào
- vì sao lại sai

Với bài toán tin giả, các lỗi phổ biến thường đến từ:

- tiêu đề giật gân nhưng nội dung không đủ bằng chứng
- bài viết ngắn, thiếu ngữ cảnh
- văn bản có nhiều từ khóa giống tin giả nhưng thực ra là trích dẫn hoặc phân tích
- trường hợp nội dung thật nhưng cách diễn đạt cảm tính và khuếch đại

Việc lưu `error_samples.csv` cho phép người làm project quay lại đọc trực tiếp những mẫu thất bại này. Đây là nơi thể hiện rõ nhất tư duy phân tích, thay vì chỉ dừng ở việc báo cáo điểm số.

## 17. Artifact được lưu sau huấn luyện

Notebook hiện tại xuất ra một bộ artifact khá đầy đủ:

- `tfidf_pipeline.joblib`
- `error_samples.csv`
- thư mục `transformer/`
- `val_results.csv`
- `test_results.csv`
- `manifest.json`

Mỗi artifact giữ một vai trò riêng.

`tfidf_pipeline.joblib` là pipeline baseline đã fit xong và có thể load lại để suy luận. `error_samples.csv` phục vụ phân tích lỗi. Thư mục `transformer/` chứa model và tokenizer đã lưu lại. `val_results.csv` và `test_results.csv` là hai bảng tổng hợp kết quả. Cuối cùng, `manifest.json` đóng vai trò như một bản chỉ mục giúp ứng dụng biết file nào đang ở đâu.

Về mặt kỹ thuật, đây là một điểm rất mạnh của project. Nó cho thấy notebook không chỉ train để quan sát, mà train để tạo ra đầu ra có thể tái sử dụng.

## 18. Mối liên hệ giữa notebook train và ứng dụng giao diện

Ứng dụng giao diện trong `ui.py` hiện tại ưu tiên dùng **artifact thật của baseline TF-IDF**. Nếu không tìm thấy artifact, app mới fallback về một chế độ demo heuristic.

Chi tiết này rất quan trọng khi viết báo cáo, vì nó phản ánh đúng trạng thái hiện tại của hệ thống:

- baseline TF-IDF đã đi vào luồng suy luận thật của app
- DistilBERT hiện chủ yếu đóng vai trò nhánh thực nghiệm và so sánh

Đây là một quyết định hợp lý. Trong nhiều bài toán ứng dụng, mô hình tốt nhất về mặt điểm số chưa chắc là mô hình tốt nhất cho triển khai. Một baseline tuyến tính như TF-IDF + LinearSVC có những ưu điểm rất rõ:

- nhẹ
- nhanh
- dễ đóng gói
- dễ nạp lại bằng `joblib`

Trong khi đó, transformer tuy mạnh hơn về ngữ cảnh nhưng nặng hơn đáng kể trong suy luận thời gian thực.

Một chi tiết cần lưu ý là notebook trên Kaggle lưu artifact ở `/kaggle/working/artifacts/fake_news`, còn ứng dụng local lại đọc từ `artifacts/fake_news`. Vì vậy, nếu muốn dùng kết quả train từ Kaggle trong app local, cần chuyển các artifact cần thiết về đúng thư mục mà app đang kỳ vọng.

## 19. Điểm mạnh của thiết kế hiện tại

Nhìn tổng thể, pipeline hiện tại có nhiều điểm mạnh đáng ghi nhận.

Trước hết, nó phân biệt rõ dữ liệu đầu vào cho baseline và transformer, thay vì cố dùng một biểu diễn chung cho mọi mô hình. Tiếp theo, nó chọn đúng nhóm baseline mạnh cho bài toán văn bản sparse. Nó cũng dùng F1-score làm tiêu chí chính, thể hiện sự nghiêm túc trong lựa chọn metric. Quan trọng hơn, notebook có validation độc lập, test độc lập, quy trình chọn mô hình rõ ràng, phân tích lỗi và quản lý artifact. Cuối cùng, nó đã kết nối được với ứng dụng giao diện, tức là đã tiến gần hơn nhiều đến một hệ thống hoàn chỉnh thay vì chỉ là notebook thử nghiệm.

Nếu đánh giá như một đồ án Data Science, đây là một bộ khung khá chặt chẽ.

## 20. Hạn chế của hệ thống hiện tại

Dù pipeline đã tốt ở mức đồ án hoặc project kỹ thuật, hệ thống hiện tại vẫn còn một số giới hạn rõ ràng.

Giới hạn đầu tiên là mô hình hiện mới chỉ dựa vào văn bản. Nó chưa khai thác metadata như nguồn đăng, thời gian đăng, tác giả, tín hiệu lan truyền hay mức độ tương tác xã hội. Trong thực tế, các tín hiệu ngoài văn bản thường rất hữu ích cho fake news detection.

Giới hạn thứ hai là DistilBERT chưa được dùng trong suy luận thực của ứng dụng. Điều này không sai, nhưng cho thấy nhánh transformer vẫn chưa đi trọn con đường từ thực nghiệm đến triển khai.

Giới hạn thứ ba là chưa có đánh giá ngoài miền dữ liệu. Một mô hình tốt trên WELFake chưa chắc sẽ giữ được chất lượng tương tự trên một tập dữ liệu tin tức khác.

Giới hạn thứ tư là khả năng giải thích mô hình mới dừng ở mức cơ bản. Error analysis đã có, nhưng project chưa đi sâu vào việc giải thích đặc trưng nào thực sự đẩy mô hình đến quyết định cuối cùng.

## 21. Hướng phát triển tiếp theo

Từ nền tảng hiện tại, có một số hướng mở rộng rất tự nhiên.

Hướng thứ nhất là đưa DistilBERT vào pipeline suy luận thật trong ứng dụng, ít nhất như một tùy chọn thứ hai bên cạnh baseline TF-IDF. Hướng thứ hai là mở rộng dữ liệu hoặc đánh giá trên thêm các tập dữ liệu khác để kiểm tra khả năng tổng quát hóa. Hướng thứ ba là bổ sung các kỹ thuật giải thích mô hình, chẳng hạn phân tích n-gram quan trọng đối với LinearSVC hoặc phân tích attention/saliency đối với transformer. Hướng thứ tư là tách train pipeline và serving pipeline thành các thành phần rõ ràng hơn, từ đó đưa project tiến gần hơn đến một cấu trúc production.

## 22. Kết luận

Project phát hiện tin giả với WELFake trong repo hiện tại cho thấy một điều quan trọng: một hệ thống tốt không chỉ được đo bằng một con số accuracy hay F1, mà bằng độ chặt chẽ của toàn bộ quy trình.

Ở đây, quy trình đó đã bao gồm:

- chuẩn hóa đầu vào
- tách hai nhánh dữ liệu phù hợp cho hai họ mô hình
- so sánh ba baseline mạnh trên TF-IDF
- bổ sung DistilBERT như một nhánh ngữ cảnh sâu hơn
- đánh giá bằng metric phù hợp
- lưu artifact có thể dùng lại
- và kết nối được với ứng dụng giao diện

Từ góc nhìn học thuật, đây là một pipeline có cấu trúc tốt. Từ góc nhìn kỹ thuật, đây cũng là một project có giá trị vì đã đi qua nhiều bước mà các bài làm mô hình đơn thuần thường bỏ sót: validation nghiêm túc, error analysis, artifact management và integration với app.

Nếu tiếp tục bổ sung số liệu thực nghiệm sau khi chạy notebook và đẩy sâu hơn phần phân tích lỗi, bài toán này hoàn toàn có thể trở thành một case study rất mạnh cho cả Data Science lẫn NLP ứng dụng.

## 23. Tài liệu tham khảo

1. WELFake Dataset trên Kaggle  
2. Tài liệu `scikit-learn` về `TfidfVectorizer`, `RandomizedSearchCV`, `LogisticRegression`, `LinearSVC`, `MultinomialNB`  
3. Tài liệu Hugging Face về `DistilBERT`, `Trainer`, `TrainingArguments`  
4. Mã nguồn hiện tại trong repo:
   - `train_kaggle.ipynb`
   - `train_colab.ipynb`
   - `build_train_kaggle_notebook.ps1`
   - `ui.py`
