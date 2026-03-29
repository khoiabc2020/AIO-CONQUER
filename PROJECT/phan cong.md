# BÁO CÁO TÓM TẮT ĐỀ TÀI PHÁT HIỆN TIN GIẢ VỚI WELFAKE

## 1. Lời nói đầu

Trong bối cảnh thông tin số phát triển mạnh mẽ, tốc độ lan truyền của tin tức trên các nền tảng trực tuyến ngày càng cao. Song song với lợi ích về khả năng tiếp cận thông tin nhanh chóng là sự gia tăng của các nội dung sai lệch, thiếu kiểm chứng hoặc cố ý gây hiểu lầm. Tin giả không chỉ ảnh hưởng đến nhận thức của người dùng, mà còn tác động trực tiếp đến niềm tin xã hội, chất lượng truyền thông và quá trình ra quyết định trong nhiều lĩnh vực quan trọng.

Xuất phát từ thực tế đó, đề tài phát hiện tin giả mang ý nghĩa rõ rệt cả về mặt học thuật lẫn ứng dụng. Về mặt học thuật, đây là bài toán thuộc lĩnh vực xử lý ngôn ngữ tự nhiên và học máy, đòi hỏi sự kết hợp giữa phân tích dữ liệu, biểu diễn văn bản và lựa chọn mô hình phù hợp. Về mặt thực tiễn, kết quả của đề tài có thể được sử dụng làm nền tảng cho các công cụ hỗ trợ kiểm chứng thông tin, cảnh báo nội dung sai lệch và xây dựng các hệ thống lọc tin.

Project hiện tại được xây dựng bám sát file `train_colab.ipynb`, với định hướng không chỉ huấn luyện mô hình mà còn hình thành một pipeline hoàn chỉnh: chuẩn bị dữ liệu, tiền xử lý, phân tích dữ liệu khám phá, huấn luyện nhiều mô hình, đánh giá kết quả, lưu artifact và kết nối với ứng dụng giao diện.

## 2. Mục tiêu của đề tài

Mục tiêu chính của đề tài là xây dựng một hệ thống phát hiện tin giả trên dữ liệu WELFake theo hướng có thể tái sử dụng và mở rộng. Cụ thể, project hiện tại tập trung vào các mục tiêu sau:

- xây dựng pipeline huấn luyện ổn định trong notebook
- so sánh một nhánh baseline nhẹ, mạnh và dễ triển khai với một nhánh transformer
- lựa chọn mô hình bằng tiêu chí đánh giá phù hợp
- lưu artifact để phục vụ tái sử dụng trong ứng dụng

Ngay từ cell cài đặt thư viện đầu tiên, notebook đã cho thấy định hướng thực dụng. File này chủ động cài lại các package quan trọng như `numpy`, `scipy`, `scikit-learn`, `pandas`, `matplotlib`, `seaborn`, `datasets`, `transformers`, `accelerate` và `nltk`, đồng thời nhắc người dùng restart runtime nếu vừa thay đổi stack. Chi tiết này cho thấy notebook không chỉ quan tâm đến mô hình, mà còn quan tâm đến tính ổn định của môi trường thực thi.

## 3. Dữ liệu và cách tổ chức dữ liệu trong pipeline

Project sử dụng file `WELFake_Dataset.csv` với ba cột đầu vào chính:

- `title`
- `text`
- `label`

Hàm `get_csv()` trong notebook được viết theo hướng linh hoạt. Trước hết, nó ưu tiên tìm file trong `/kaggle/input`. Nếu chưa tìm thấy, notebook fallback sang tải dữ liệu từ Zenodo về `cfg.data_dir`. Nhờ đó, notebook có thể thích nghi với nhiều cách tổ chức dữ liệu khác nhau thay vì bị khóa vào một nguồn duy nhất.

Sau khi đọc dữ liệu, notebook thực hiện các bước:

- giữ lại đúng ba cột cần thiết
- loại bỏ các dòng thiếu `label`
- loại bỏ bản ghi trùng lặp
- ghép `title` và `text` thành `raw_content`

Một chi tiết rất quan trọng trong project là phần ánh xạ nhãn:

```python
df["label"] = df["label"].astype(int).map({0: 1, 1: 0})
names = {0: "real", 1: "fake"}
```

Theo đó:

- `0` tương ứng với `real`
- `1` tương ứng với `fake`

Việc nêu rõ quy ước này là bắt buộc, bởi nếu diễn giải sai nhãn thì toàn bộ phần nhận xét về precision, recall, confusion matrix và error analysis đều sẽ sai theo.

## 4. Tiền xử lý dữ liệu

Notebook không áp dụng một pipeline tiền xử lý giống nhau cho mọi mô hình. Thay vào đó, dữ liệu được tách thành hai dạng đầu vào:

- `raw_content`: văn bản gần với dữ liệu gốc, dùng cho DistilBERT
- `content`: văn bản đã làm sạch, dùng cho TF-IDF

Hàm `clean()` trong notebook hiện tại là:

```python
def clean(text: str) -> str:
    text = str(text).lower()
    text = re.sub(r"http\\S+|www\\.\\S+", " ", text)
    text = re.sub(r"[^a-z\\s]", " ", text)
    words = re.sub(r"\\s+", " ", text).strip().split()
    words = [word for word in words if word not in stop_words]
    words = [lemmatizer.lemmatize(word) for word in words]
    return " ".join(words)
```

Các bước xử lý có thể tóm tắt như sau:

- chuyển văn bản về chữ thường
- loại bỏ URL
- loại bỏ ký tự ngoài chữ cái tiếng Anh
- chuẩn hóa khoảng trắng
- loại stopwords
- lemmatization

Đây là một pipeline vừa đủ gọn nhưng vẫn hiệu quả cho nhánh baseline TF-IDF.

## 5. Phân tích dữ liệu khám phá

Trước khi huấn luyện, notebook thực hiện một phần EDA tương đối rõ ràng. Ba nhóm quan sát chính được kiểm tra là:

- phân bố nhãn `real` và `fake`
- độ dài tiêu đề, thân bài và nội dung sau làm sạch
- các từ xuất hiện phổ biến nhất trong từng lớp

Phần EDA này giúp trả lời những câu hỏi quan trọng trước khi modeling:

- dữ liệu có cân bằng hay không
- tin giả và tin thật có khác nhau về độ dài không
- tín hiệu từ vựng nào đang nổi bật trong từng lớp

Đây là bước cần thiết để hiểu dữ liệu trước khi đưa vào mô hình.

## 6. Chia dữ liệu và nguyên tắc đánh giá

Notebook chia dữ liệu thành ba phần:

- `train`
- `validation`
- `test`

theo cấu hình:

```python
test_size = 0.2
val_size = 0.1
```

Hai lần chia đều dùng `stratify`, nhờ đó tỷ lệ nhãn được giữ ổn định giữa các tập. Đồng thời, notebook chia song song hai nhánh đầu vào:

- `X = df["content"]` cho baseline
- `X_raw = df["raw_content"]` cho DistilBERT

Thiết kế này bảo đảm hai họ mô hình được đánh giá trên cùng một logic chia dữ liệu, nhưng vẫn giữ được đầu vào phù hợp với bản chất của từng mô hình.

## 7. Nhánh baseline: TF-IDF và các mô hình tuyến tính

Phần baseline sử dụng `TfidfVectorizer` với cấu hình:

- `max_features = 50000`
- `min_df = 5`
- `max_df = 0.8`
- `ngram_range = (1, 2)`

Trên nền TF-IDF, notebook so sánh ba mô hình:

- `MultinomialNB`
- `LogisticRegression`
- `LinearSVC`

Việc tuning được thực hiện bằng `RandomizedSearchCV` với:

- `n_iter = 12`
- `cv = 5`
- `scoring = "f1"`
- `n_jobs = -1`

Notebook không gắn cứng mô hình thắng. Sau khi đánh giá trên validation, nó xếp hạng kết quả theo `f1` rồi `accuracy`, chọn mô hình tốt nhất, sau đó train lại trên `train + validation` trước khi đánh giá trên `test`. Đây là một điểm mạnh rõ ràng về phương pháp.

## 8. Nhánh DistilBERT

Notebook còn có một nhánh transformer dùng checkpoint:

`distilbert-base-uncased`

Điểm đáng chú ý là file `train_colab.ipynb` không chỉ kiểm tra sự hiện diện của GPU, mà còn xây dựng thêm nhiều lớp bảo vệ runtime:

- kiểm tra `gpu_ready` bằng một smoke test nhỏ trên CUDA
- ghi lại `gpu_note` để hỗ trợ chẩn đoán lỗi
- dùng `ensure_hf_stack()` để sửa stack Hugging Face khi lệch phiên bản
- dùng `get_hf_token()` để ưu tiên lấy token từ biến môi trường hoặc Colab Secret `HF_TOKEN`

Nhánh DistilBERT hiện được cấu hình với:

- `max_len = 256`
- `epochs = 2`
- `train_bs = 12`
- `eval_bs = 24`
- `lr = 2e-5`
- `wd = 0.01`
- `sample_limit = 30000`

Nếu GPU không ổn hoặc stack Hugging Face gặp lỗi, notebook sẽ không cố train bằng mọi giá. Thay vào đó, nó phản hồi bằng các trạng thái `ready`, `skipped` hoặc `failed`, kèm theo nguyên nhân tương ứng.

## 9. Đánh giá kết quả và phân tích lỗi

Notebook sử dụng bốn chỉ số đánh giá chính:

- `accuracy`
- `precision`
- `recall`
- `f1`

Các kết quả được gom vào cùng một schema bằng hàm `metric_row`, giúp baseline và transformer có thể so sánh trong cùng một bảng.

Ngoài các chỉ số tổng hợp, notebook còn:

- in `classification_report`
- vẽ `confusion_matrix`
- lưu `error_samples.csv` cho baseline tốt nhất

Đây là điểm có giá trị lớn về mặt Data Science, bởi nó cho phép người làm bài không chỉ nhìn vào điểm số, mà còn quay lại xem mô hình đang sai ở đâu.

## 10. Artifact và khả năng tái sử dụng

Sau khi huấn luyện, notebook lưu các artifact sau:

- `tfidf_pipeline.joblib`
- `error_samples.csv`
- `transformer/`
- `val_results.csv`
- `test_results.csv`
- `manifest.json`

Trong đó, `manifest.json` đóng vai trò như một bản chỉ mục, lưu các đường dẫn quan trọng tới pipeline baseline, file lỗi, thư mục transformer và các bảng kết quả.

Nhờ đó, đầu ra của notebook không dừng ở việc “xem trong lúc train”, mà có thể chuyển tiếp sang giai đoạn ứng dụng.

## 11. Liên hệ với ứng dụng giao diện

Theo code trong `ui.py`, ứng dụng hiện ưu tiên sử dụng artifact thật của baseline TF-IDF. Nếu chưa có artifact, app sẽ fallback sang chế độ demo heuristic.

Điều đó cho thấy trạng thái hiện tại của project như sau:

- baseline TF-IDF đã đi vào suy luận thật
- DistilBERT đã được huấn luyện và lưu, nhưng chưa đi vào luồng dự đoán thực trong ứng dụng

Từ góc nhìn triển khai, đây là một lựa chọn hợp lý. Mô hình tốt nhất về điểm số chưa chắc là mô hình phù hợp nhất cho suy luận thời gian thực. Một pipeline như `TF-IDF + LinearSVC` có ưu điểm rõ ràng về tốc độ, độ gọn và khả năng đóng gói.

## 12. Đánh giá chung và hướng cải tiến

Nhìn tổng thể, pipeline hiện tại có nhiều điểm mạnh:

- dữ liệu được tổ chức rõ ràng
- tiền xử lý vừa đủ và có chủ đích
- EDA đúng trọng tâm
- baseline mạnh và ổn định
- transformer được tích hợp theo hướng thực dụng
- có đánh giá, phân tích lỗi và lưu artifact
- đã có cầu nối sang ứng dụng

Tuy nhiên, nếu nhìn trực tiếp từ code, vẫn còn một số điểm cần cải thiện:

- logic train vẫn chủ yếu nằm trong notebook
- DistilBERT chưa đi vào luồng suy luận thực của app
- xác suất đầu ra của baseline trong app chưa được calibration đúng nghĩa
- chưa có error analysis riêng cho DistilBERT
- cấu trúc file còn hơi trùng ý giữa các notebook và script build
- chưa có test tự động cho các phần quan trọng

Nói cách khác, đây đã là một pipeline tốt để học, nghiên cứu và demo, nhưng vẫn còn dư địa rõ ràng để tiến gần hơn tới một codebase sạch và sẵn sàng mở rộng.

## 13. Kết luận

Giá trị lớn nhất của project này không nằm ở việc một mô hình cụ thể chiến thắng, mà nằm ở việc toàn bộ quy trình đã được tổ chức tương đối chặt chẽ. Notebook không chỉ huấn luyện mô hình, mà còn đi qua đầy đủ các bước quan trọng của một bài toán Data Science và NLP ứng dụng: hiểu dữ liệu, làm sạch văn bản, khám phá dữ liệu, so sánh mô hình, đánh giá đúng vai trò của validation và test, lưu artifact và chuẩn bị đầu ra cho ứng dụng.

Nếu tiếp tục bổ sung số liệu thực nghiệm cuối cùng, làm sâu hơn phần error analysis và tách bớt logic ra khỏi notebook, project này hoàn toàn có thể trở thành một case study rất tốt cho cả học máy ứng dụng lẫn xử lý ngôn ngữ tự nhiên.

## 14. Tài liệu tham khảo

1. WELFake Dataset trên Kaggle  
2. Tài liệu `scikit-learn` về `TfidfVectorizer`, `RandomizedSearchCV`, `LogisticRegression`, `LinearSVC`, `MultinomialNB`  
3. Tài liệu Hugging Face về `DistilBERT`, `Trainer`, `TrainingArguments`  
4. Mã nguồn hiện tại trong repo:
   - `train_colab.ipynb`
   - `train_kaggle.ipynb`
   - `build_train_colab_notebook.ps1`
   - `ui.py`
