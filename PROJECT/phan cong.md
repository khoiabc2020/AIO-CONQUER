# Phát hiện tin giả với WELFake

Có những bài viết chỉ cần đọc tiêu đề là đã thấy “có mùi”, nhưng cũng có những bài được viết đủ khéo để đánh lừa cả người đọc cẩn thận. Đó là lúc bài toán phát hiện tin giả trở nên thú vị: ranh giới giữa đúng và sai không còn nằm ở vài từ khóa lộ liễu, mà nằm ở cách dữ liệu được biểu diễn, cách mô hình học ngữ cảnh và cách ta đánh giá nó một cách nghiêm túc.

Project này bắt đầu từ đúng câu hỏi đó: nếu đứng giữa một bài viết trông rất thuyết phục, mô hình nào sẽ nhìn ra sự khác biệt trước, một baseline tuyến tính mạnh hay một transformer hiểu ngữ cảnh tốt hơn?

## 1. Mở đầu

Tin giả là một kiểu dữ liệu rất khó chịu. Nó không thô đến mức chỉ cần lọc vài từ khóa là xong, nhưng cũng không tinh vi đến mức mọi mô hình đều bất lực. Cái khó nằm ở chỗ nó thường được viết theo cách rất giống tin thật: tiêu đề bắt mắt, nội dung có vẻ hợp lý, giọng điệu đủ thuyết phục để người đọc tin rằng mình đang tiếp nhận một thông tin đáng tin cậy.

Đó cũng là lý do bài toán phát hiện tin giả luôn thú vị trong NLP. Nó nằm đúng ở điểm giao giữa xử lý ngôn ngữ, mô hình hóa dữ liệu và tư duy triển khai thực tế. Một project tốt cho bài toán này không nên chỉ dừng ở việc “train một model cho có”, mà cần đi qua đủ các bước: hiểu dữ liệu, làm sạch văn bản, kiểm tra đặc điểm của tập dữ liệu, so sánh nhiều mô hình, đánh giá đúng cách, rồi lưu lại kết quả để có thể dùng tiếp trong ứng dụng.

Project này được xây dựng theo đúng tinh thần đó trên bộ dữ liệu **WELFake**. Mục tiêu không chỉ là phân loại bài viết thành `real` và `fake`, mà là tạo ra một pipeline đủ gọn, đủ rõ, và đủ thực dụng để vừa dùng cho học thuật, vừa có thể nối sang một giao diện demo.

## 2. Bộ dữ liệu và cách dữ liệu đi vào pipeline

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

Đây là phần rất dễ bị bỏ qua khi viết báo cáo, nhưng nếu bỏ qua thì toàn bộ precision, recall, confusion matrix và error analysis đều có thể bị diễn giải ngược. Với những bài toán nhị phân như thế này, chỉ cần sai một quy ước nhãn là mọi phần nhận xét phía sau sẽ lệch toàn bộ.

## 3. Tiền xử lý: không cầu kỳ, nhưng đúng việc

Điểm tôi thích ở pipeline này là nó không cố “làm sạch thật mạnh” cho mọi mô hình. Thay vào đó, notebook tách dữ liệu thành hai dạng văn bản khác nhau:

- `raw_content`: bản gần với dữ liệu gốc, dùng cho DistilBERT
- `content`: bản đã làm sạch, dùng cho baseline TF-IDF

Cách tách này rất hợp lý. Mô hình transformer thường hoạt động tốt hơn khi đầu vào vẫn giữ được ngữ cảnh tự nhiên, còn TF-IDF lại hưởng lợi rõ rệt khi văn bản đã được chuẩn hóa và giảm nhiễu.

Hàm làm sạch hiện tại khá gọn:

```python
def clean(text: str) -> str:
    text = str(text).lower()
    text = re.sub(r"[^a-z\\s]", " ", text)
    words = re.sub(r"\\s+", " ", text).strip().split()
    words = [word for word in words if word not in stop_words]
    words = [lemmatizer.lemmatize(word) for word in words]
    return " ".join(words)
```

Nhìn vào đó có thể thấy notebook đang làm đúng những việc cần thiết:

- chuyển văn bản về chữ thường
- loại ký tự ngoài chữ cái tiếng Anh
- chuẩn hóa khoảng trắng
- loại stopwords
- lemmatization

Phần thú vị là notebook không thêm các bước hoa mỹ như xử lý HTML hay bóc URL riêng biệt. Tức là pipeline chọn hướng tối giản, nhưng vẫn đủ hiệu quả cho baseline. Đây là kiểu tối giản tốt: bớt thao tác thừa, giữ lại những bước thật sự có ích cho mô hình.

## 4. EDA: nhìn dữ liệu trước khi tin mô hình

Trước khi huấn luyện, notebook đi qua một vòng EDA tương đối gọn nhưng đúng trọng tâm. Nó kiểm tra ba nhóm tín hiệu chính:

- phân bố nhãn `real` và `fake`
- độ dài của tiêu đề, thân bài và nội dung sau làm sạch
- các từ xuất hiện phổ biến nhất theo từng lớp

Những bước này không chỉ để “cho có biểu đồ”. Chúng giúp trả lời các câu hỏi rất thực tế:

- dữ liệu có mất cân bằng mạnh hay không
- tin giả và tin thật có khác nhau về độ dài hay không
- có những tín hiệu từ vựng nào đang nổi bật trong từng lớp

Với dữ liệu văn bản, trực giác từ EDA thường rất hữu ích. Nó không thay thế mô hình, nhưng nó giúp ta biết mình đang làm việc với loại dữ liệu nào, có gì bất thường, và mô hình nào có nhiều khả năng phù hợp hơn.

## 5. Cách chia dữ liệu: phần nhỏ nhưng quyết định độ tử tế của bài toán

Notebook chia dữ liệu thành ba phần:

- `train`
- `validation`
- `test`

theo cấu hình:

```python
test_size = 0.2
val_size = 0.1
```

Điểm đáng nói không nằm ở con số, mà nằm ở cách dùng ba tập này đúng vai trò của chúng:

- `train` để học
- `validation` để chọn mô hình và siêu tham số
- `test` để đánh giá cuối cùng

Ngoài ra, cả hai bước chia đều dùng `stratify`, giúp tỷ lệ nhãn được giữ ổn định giữa các tập. Đây là một lựa chọn rất cơ bản nhưng nhiều người vẫn làm sai. Nếu thiếu `stratify`, mô hình có thể nhìn đẹp trên một tập chia “may mắn”, nhưng không phản ánh đúng năng lực thực tế.

Một chi tiết khác rất đáng giá là notebook chia song song hai nhánh dữ liệu:

- `X = df["content"]` cho baseline
- `X_raw = df["raw_content"]` cho DistilBERT

Nhờ đó, hai họ mô hình được đánh giá trên cùng một logic chia dữ liệu, nhưng vẫn giữ được kiểu đầu vào phù hợp nhất cho từng bên.

## 6. Baseline: TF-IDF và lý do những mô hình tuyến tính vẫn rất đáng nể

Phần baseline của project dùng `TfidfVectorizer` với cấu hình:

```python
base_tfidf = TfidfVectorizer(
    stop_words="english",
    max_features=cfg.max_features,
    min_df=cfg.min_df,
    max_df=cfg.max_df,
    ngram_range=cfg.ngram_range,
)
```

Theo code hiện tại:

- `max_features = 50000`
- `min_df = 5`
- `max_df = 0.8`
- `ngram_range = (1, 2)`

Đây là một cấu hình rất “đúng sách nhưng không giáo điều”. Nó đủ mạnh để giữ lại tín hiệu từ unigram và bigram, loại bỏ bớt từ quá hiếm, đồng thời giảm ảnh hưởng của những từ quá phổ biến.

Trên nền TF-IDF, notebook so sánh ba baseline:

- `MultinomialNB`
- `LogisticRegression`
- `LinearSVC`

Đây là ba cái tên quen thuộc, nhưng không hề cũ kỹ. Trong rất nhiều bài toán text classification, đặc biệt khi dữ liệu đã được làm sạch tốt, các mô hình tuyến tính vẫn là đối thủ rất đáng gờm. Chúng nhanh, ổn định, dễ giải thích hơn transformer, và đôi khi cho hiệu quả vượt mong đợi.

Việc tuning được thực hiện bằng `RandomizedSearchCV` với:

- `n_iter = 12`
- `cv = 5`
- `scoring = "f1"`
- `n_jobs = -1`

Đây là một lựa chọn hợp lý. Không gian siêu tham số đủ rộng để cần search, nhưng chưa đến mức phải dùng một chiến lược tối ưu hóa quá nặng. Quan trọng hơn, việc dùng **F1-score** làm tiêu chí chính cho thấy project đang nhìn bài toán theo hướng đúng: với phát hiện tin giả, chỉ đúng nhiều thôi là chưa đủ, mà phải cân bằng được precision và recall.

Sau khi chấm trên validation, notebook chọn mô hình tốt nhất một cách động:

```python
val_df = pd.DataFrame(val_rows).sort_values(["f1", "accuracy"], ascending=False)
best_model_name = str(val_df.iloc[0]["model"])
```

Rồi mô hình thắng được train lại trên `train + validation` trước khi đánh giá ở `test`. Đây là một chi tiết nhỏ nhưng nói lên chất lượng của pipeline: validation được dùng đúng vai trò chọn mô hình, còn test được giữ lại cho bước đánh giá cuối cùng.

## 7. DistilBERT: nhánh nâng cao để đo độ sâu ngữ cảnh

Bên cạnh baseline TF-IDF, notebook còn có một nhánh transformer dùng:

`distilbert-base-uncased`

DistilBERT là một lựa chọn rất hợp lý cho một project kiểu này. Nó đủ nhẹ để huấn luyện trong môi trường thực nghiệm, nhưng vẫn giữ được phần lớn tinh thần của BERT: hiểu ngữ cảnh tốt hơn, bớt phụ thuộc vào tín hiệu từ vựng thuần túy.

Nhánh này chỉ chạy khi:

- `cfg.use_transformer = True`
- và GPU khả dụng

Nếu không có GPU, notebook sẽ bỏ qua transformer. Quyết định đó không làm project yếu đi; ngược lại, nó cho thấy pipeline đang được viết theo hướng thực tế, không cố kéo mọi thứ vào một lần chạy chỉ để “cho đủ mô hình”.

DistilBERT dùng `raw_content` làm đầu vào, với cấu hình chính:

- `max_len = 256`
- `epochs = 2`
- `train_bs = 16`
- `eval_bs = 32`
- `lr = 2e-5`
- `wd = 0.01`
- `sample_limit = 30000`

`sample_limit` là một chi tiết rất thực dụng: nếu tập train quá lớn, notebook sẽ lấy mẫu để kiểm soát chi phí huấn luyện. Với một notebook thực nghiệm, đây là lựa chọn hợp lý hơn nhiều so với việc cố train toàn bộ dữ liệu rồi chờ quá lâu hoặc tràn tài nguyên.

## 8. Đánh giá, phân tích lỗi và artifact

Notebook dùng bốn metric chính:

- `accuracy`
- `precision`
- `recall`
- `f1`

Toàn bộ được gom lại bằng hàm `metric_row`, giúp baseline và transformer có cùng schema đầu ra. Thiết kế này làm cho việc so sánh mô hình gọn hơn rất nhiều.

Ngoài các chỉ số tổng hợp, notebook còn hiển thị:

- `classification_report`
- `confusion_matrix`

cho baseline tốt nhất. Đây là phần cần thiết nếu muốn đọc mô hình như một hệ thống thật, thay vì chỉ nhìn một con số rồi kết luận.

Điểm hay là notebook không gắn cứng kết luận kiểu “LinearSVC luôn thắng” hay “DistilBERT chắc chắn tốt hơn”. Sau khi train, nó tạo:

- `val_board`
- `test_board`

và xếp hạng theo `f1` rồi `accuracy`. Nghĩa là kết luận cuối cùng luôn phải đến từ chính kết quả thực nghiệm của lần chạy đó.

Sau khi baseline tốt nhất được train xong, notebook lưu các mẫu dự đoán sai vào `error_samples.csv`. Đây là phần tôi đánh giá rất cao, vì nó chuyển trọng tâm từ “đạt bao nhiêu điểm” sang “mô hình sai ở đâu và vì sao sai”. Với các bài toán như fake news detection, error analysis thường cho nhiều giá trị hơn cả một vài chữ số thập phân chênh lệch trong F1.

Các artifact đầu ra hiện tại gồm:

- `tfidf_pipeline.joblib`
- `error_samples.csv`
- `transformer/`
- `val_results.csv`
- `test_results.csv`
- `manifest.json`

Nhờ đó, notebook không chỉ phục vụ việc quan sát trong quá trình train, mà còn tạo ra đầu ra có thể dùng lại cho ứng dụng.

## 9. Từ notebook đến ứng dụng

Ứng dụng trong `ui.py` hiện ưu tiên dùng **artifact thật của baseline TF-IDF**. Nếu chưa có artifact, app mới fallback về chế độ demo heuristic. Điều này phản ánh rất đúng trạng thái thực tế của project:

- baseline TF-IDF đã đi vào luồng suy luận thật
- DistilBERT hiện vẫn là nhánh thực nghiệm để so sánh

Đây là một quyết định hợp lý. Trong môi trường triển khai, mô hình có điểm cao nhất chưa chắc là mô hình phù hợp nhất. Một baseline tuyến tính như TF-IDF + LinearSVC có nhiều ưu điểm thực dụng:

- nhẹ
- nhanh
- dễ đóng gói
- dễ nạp lại bằng `joblib`

Trong khi đó, transformer mạnh hơn về ngữ cảnh nhưng đắt hơn trong suy luận thời gian thực.

Nhìn tổng thể, đây là điểm mạnh lớn của project: nó không chỉ dừng ở notebook, mà đã có cầu nối sang một ứng dụng thực sự.

## 10. Nhìn lại toàn bộ pipeline

Điều đáng giá nhất của project này không nằm ở việc dùng TF-IDF hay DistilBERT, mà nằm ở cách toàn bộ pipeline được tổ chức.

Nó có dữ liệu đủ rõ, tiền xử lý đúng mức, EDA đúng trọng tâm, baseline đủ mạnh, transformer đủ hợp lý, chiến lược đánh giá tương đối chặt, có phân tích lỗi, có artifact, và có đường đi sang ứng dụng. Đó là những thứ làm nên chất lượng của một project Data Science tốt.

Nếu nhìn thẳng vào code hiện tại, có vài điểm cải tiến khá rõ:

- Logic train hiện vẫn nằm chủ yếu trong notebook. Nếu tách phần tiền xử lý, train, evaluate và save artifact ra các module Python riêng, code sẽ dễ test và dễ tái sử dụng hơn nhiều.
- Ứng dụng hiện chỉ dùng artifact thật của TF-IDF. DistilBERT đã được train và lưu, nhưng chưa đi vào luồng suy luận thực trong `ui.py`.
- Phần xác suất của baseline trong app đang được suy ra từ `decision_function` qua một hàm sigmoid tự chế. Cách này đủ để demo, nhưng chưa phải một bước calibration đúng nghĩa.
- Notebook đang lưu `error_samples.csv` cho baseline, nhưng chưa có error analysis riêng cho DistilBERT. Nếu muốn so sánh sâu hai nhánh, đây là phần nên bổ sung.
- Đường đi artifact giữa notebook và app vẫn là một điểm cần dọn. Notebook train và app đọc artifact theo hai ngữ cảnh chạy khác nhau, nên việc đồng bộ file vẫn cần thao tác thủ công.
- Cấu trúc file hiện còn hơi trùng ý, ví dụ `train_kaggle.ipynb`, `train_colab.ipynb`, `build_train_kaggle_notebook.ps1`, `build_train_colab_notebook.ps1`. Nếu tiếp tục phát triển, nên gom lại cho tên gọi nhất quán hơn.
- Project hiện chưa có test tự động cho các phần quan trọng như load artifact, predict pipeline hay schema của `manifest.json`. Với một project muốn đi xa hơn mức demo, đây là phần nên có.

Tất nhiên, hệ thống vẫn còn những giới hạn quen thuộc:

- mới chỉ dựa vào văn bản
- chưa khai thác metadata
- DistilBERT chưa đi vào suy luận thật trong app
- chưa có đánh giá ngoài miền dữ liệu
- khả năng giải thích mô hình còn cơ bản

Từ đó, hướng phát triển tiếp theo cũng hiện ra khá rõ:

- đưa DistilBERT vào pipeline suy luận thật
- mở rộng dữ liệu hoặc đánh giá trên tập khác
- tăng cường explainability
- tách train pipeline và serving pipeline rõ hơn
- thêm calibration cho xác suất đầu ra
- thêm test tự động và chuẩn hóa cấu trúc project

Nếu phải tóm gọn bằng một câu, tôi sẽ nói thế này: điểm mạnh nhất của project không phải là “mô hình nào thắng”, mà là việc nó đã đi xa hơn mức thử nghiệm đơn lẻ để trở thành một pipeline có cấu trúc, có thể đọc, có thể đánh giá, và có thể tiếp tục phát triển.

## 11. Tài liệu tham khảo

1. WELFake Dataset trên Kaggle  
2. Tài liệu `scikit-learn` về `TfidfVectorizer`, `RandomizedSearchCV`, `LogisticRegression`, `LinearSVC`, `MultinomialNB`  
3. Tài liệu Hugging Face về `DistilBERT`, `Trainer`, `TrainingArguments`  
4. Mã nguồn hiện tại trong repo:
   - `train_kaggle.ipynb`
   - `train_colab.ipynb`
   - `build_train_kaggle_notebook.ps1`
   - `ui.py`
