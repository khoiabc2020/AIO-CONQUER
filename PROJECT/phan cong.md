# PHÁT HIỆN TIN GIẢ BẰNG MACHINE LEARNING

---

## 1. Giới thiệu

Trong thời đại mạng xã hội phát triển, thông tin được lan truyền với tốc độ rất nhanh. Tuy nhiên, điều này cũng kéo theo một vấn đề nghiêm trọng: tin giả (fake news).

Tin giả có thể gây ảnh hưởng lớn đến xã hội, từ việc làm sai lệch nhận thức đến tác động đến chính trị và sức khỏe cộng đồng. Do đó, việc xây dựng hệ thống tự động phát hiện tin giả là một bài toán quan trọng trong lĩnh vực Data Science và NLP.

Theo các nghiên cứu gần đây, việc sử dụng các mô hình học máy có thể giúp phân loại tin giả với độ chính xác cao, thay thế phương pháp kiểm duyệt thủ công vốn không còn khả thi với khối lượng dữ liệu lớn :contentReference[oaicite:0]{index=0}.

---

## 2. Bài toán

Mục tiêu của bài toán là xây dựng một mô hình có thể phân loại một bài báo thành:

- Tin thật (Real)
- Tin giả (Fake)

### Input:
- Nội dung bài báo (text, title)

### Output:
- Nhãn phân loại (0 hoặc 1)

Đây là bài toán **classification (phân loại nhị phân)** trong Machine Learning.

---

## 3. Dataset

Trong project này, sử dụng dataset WELFake gồm:

- title
- text
- label

Dữ liệu đã được gán nhãn sẵn, phù hợp cho bài toán supervised learning.

### Các vấn đề cần quan tâm:
- Dữ liệu có cân bằng không?
- Có missing values không?
- Có dữ liệu trùng lặp không?

---

## 4. Phân tích dữ liệu (EDA)

Trước khi huấn luyện mô hình, cần hiểu dữ liệu:

### 4.1 Phân bố nhãn
- Tỷ lệ fake vs real

### 4.2 Độ dài văn bản
- Tin giả và tin thật có độ dài khác nhau không?

### 4.3 Từ vựng phổ biến
- Các từ thường xuất hiện trong fake news

-->> Việc phân tích này giúp hiểu rõ đặc trưng của dữ liệu trước khi modeling.

---

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

TF-IDF giúp biểu diễn mức độ quan trọng của từ trong văn bản.

Theo nghiên cứu, TF-IDF là một trong những phương pháp phổ biến để vector hóa dữ liệu văn bản trước khi đưa vào mô hình học máy :contentReference[oaicite:1]{index=1}.

---

### 6.2 Tokenization (Deep Learning)

- Chuyển text → sequence
- Padding để cùng độ dài

---

## 7. Mô hình

### 7.1 Baseline (TF-IDF + Linear Model)

- Nhanh
- Dễ triển khai
- Hiệu quả cao trong text classification

---

### 7.2 Deep Learning (LSTM / BERT)

- Hiểu ngữ cảnh tốt hơn
- Phù hợp với dữ liệu phức tạp

Các nghiên cứu cho thấy các mô hình học sâu và SVM có thể đạt độ chính xác cao hơn so với các phương pháp truyền thống :contentReference[oaicite:2]{index=2}.

---

## 8. Huấn luyện mô hình

### Quy trình:

- Chia dữ liệu: train / test  
- Huấn luyện model  
- Tuning hyperparameters  
- Sử dụng callbacks để tránh overfitting  

---

## 9. Đánh giá mô hình

Không chỉ dùng accuracy, cần sử dụng:

- Precision  
- Recall  
- F1-score  
- Confusion Matrix  

-->> F1-score đặc biệt quan trọng trong bài toán này vì cần cân bằng giữa false positive và false negative.

---

## 10. Kết quả và phân tích

- Model đạt độ chính xác: ...  
- So sánh giữa các model  

### Nhận xét:

- Model nào tốt hơn?
- Vì sao?

---

## 11. Phân tích lỗi (Error Analysis)

- Các trường hợp model dự đoán sai  
- Nguyên nhân:
  - ngôn ngữ mơ hồ
  - thiếu context
  - tiêu đề gây hiểu nhầm  

-->> Đây là phần thể hiện tư duy Data Science rõ nhất.

---

## 12. Hạn chế

- Dataset chưa đủ đa dạng  
- Chỉ dựa vào text  
- Không sử dụng context từ social media  

---

## 13. Hướng phát triển

- Sử dụng BERT / Transformer  
- Kết hợp dữ liệu mạng xã hội  
- Xây dựng hệ thống realtime  

---

## 14. Kết luận

Bài toán phát hiện tin giả có thể được giải quyết hiệu quả bằng các phương pháp Machine Learning và NLP.

Project này không chỉ dừng ở việc xây dựng mô hình, mà còn thể hiện một pipeline hoàn chỉnh từ xử lý dữ liệu, xây dựng model đến đánh giá và phân tích.

Trong tương lai, việc kết hợp các mô hình nâng cao và dữ liệu đa nguồn sẽ giúp cải thiện độ chính xác và tính ứng dụng thực tế.
