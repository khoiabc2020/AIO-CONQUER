
# PHÂN CÔNG THỰC HIỆN BLOG: DỰ ĐOÁN KHÁCH HÀNG RỜI BỎ (CHURN PREDICTION)

**Dataset:** Telco Customer Churn (Kaggle)  
**Hình thức:** Blog phân tích dữ liệu và mô hình Machine Learning  
**Mục tiêu:** Xây dựng một blog ngắn gọn, logic, có dữ liệu, có mô hình, có ứng dụng thực tế

---

# 1. MỞ ĐẦU: BÀI TOÁN CHURN TRONG DOANH NGHIỆP
**Người phụ trách:** **Lê Nguyễn Hải Giang**

## 1.1. Churn là gì?
- Giải thích khái niệm khách hàng rời bỏ
- Nêu ví dụ ngắn trong doanh nghiệp viễn thông

## 1.2. Vì sao churn là vấn đề quan trọng?
- Mất doanh thu
- Tăng chi phí tìm khách hàng mới
- Ảnh hưởng đến tăng trưởng doanh nghiệp

## 1.3. Mục tiêu của bài blog
- Dùng dữ liệu để tìm hiểu hành vi churn
- Xây dựng mô hình dự đoán khách hàng có khả năng rời bỏ
- Đề xuất hướng ứng dụng thực tế

## 1.4. Đầu ra cần nộp
- Phần viết mở đầu hoàn chỉnh
- Văn phong ngắn gọn, dễ đọc, đúng kiểu blog

---

# 2. GIỚI THIỆU DỮ LIỆU VÀ KHÁM PHÁ DỮ LIỆU (EDA)
**Người phụ trách:** **Phạm Hồng Ngát**

## 2.1. Giới thiệu dataset
- Nguồn dữ liệu Telco Customer Churn
- Số lượng dòng, cột
- Biến mục tiêu `Churn`

## 2.2. Nhóm biến trong dữ liệu
- Nhân khẩu học
- Dịch vụ
- Tài chính
- Hợp đồng

## 2.3. Phân tích tỷ lệ churn
- Tỷ lệ khách hàng churn / không churn
- Nhận xét ngắn về phân bố dữ liệu

## 2.4. Phân tích các yếu tố liên quan đến churn
- `Contract` và churn
- `MonthlyCharges` và churn
- `TechSupport` và churn
- `OnlineSecurity` và churn
- `InternetService` và churn

## 2.5. Trực quan hóa
- Bar chart / countplot
- Histogram / boxplot nếu cần
- Ảnh biểu đồ rõ ràng, dễ nhìn

## 2.6. Insight rút ra từ EDA
- 3 đến 5 nhận xét quan trọng nhất
- Nêu nhóm khách hàng có nguy cơ rời bỏ cao

## 2.7. Đầu ra cần nộp
- Notebook EDA
- Ảnh biểu đồ
- Phần nhận xét ngắn cho từng biểu đồ

---

# 3. TIỀN XỬ LÝ DỮ LIỆU (DATA PREPROCESSING)
**Người phụ trách:** **Dương Khánh Ly**

## 3.1. Kiểm tra dữ liệu
- Kiểm tra kiểu dữ liệu
- Kiểm tra giá trị thiếu
- Kiểm tra cột cần xử lý

## 3.2. Làm sạch dữ liệu
- Xử lý cột `TotalCharges`
- Chuyển dữ liệu về đúng kiểu số

## 3.3. Mã hóa dữ liệu
- Encoding các biến phân loại
- Giải thích ngắn vì sao phải encoding

## 3.4. Chuẩn hóa dữ liệu
- Scaling các biến số nếu cần
- Trình bày ngắn gọn, không dài dòng

## 3.5. Chia tập dữ liệu
- Tách `X` và `y`
- Chia train/test

## 3.6. Đầu ra cần nộp
- File code preprocessing
- Mô tả ngắn các bước xử lý
- Dữ liệu sẵn sàng cho phần modeling

---

# 4. XÂY DỰNG VÀ ĐÁNH GIÁ MÔ HÌNH
**Người phụ trách:** **Nhóm trưởng**

## 4.1. Các mô hình được sử dụng
- Logistic Regression
- Random Forest
- XGBoost

## 4.2. Lý do chọn các mô hình này
- Có mô hình tuyến tính dễ giải thích
- Có mô hình ensemble mạnh
- Có mô hình boosting hiệu quả cao

## 4.3. Huấn luyện mô hình
- Train từng mô hình trên tập dữ liệu đã xử lý
- Dự đoán trên tập test

## 4.4. Đánh giá mô hình
- Accuracy
- Precision
- Recall
- F1-score

## 4.5. So sánh kết quả
- Lập bảng so sánh 3 mô hình
- Chọn mô hình tốt nhất
- Nêu lý do chọn

## 4.6. Feature Importance
- Xác định các biến quan trọng nhất
- Vẽ biểu đồ top đặc trưng quan trọng

## 4.7. Đầu ra cần nộp
- Notebook modeling
- Bảng kết quả so sánh mô hình
- Biểu đồ feature importance
- Nhận xét ngắn phần kết quả

---

# 5. DEMO DỰ ĐOÁN VÀ ỨNG DỤNG THỰC TẾ
**Người phụ trách:** **Lê Minh Ngọc**

## 5.1. Demo dự đoán 1 khách hàng
- Nhập dữ liệu 1 khách hàng
- Tiền xử lý đầu vào
- Dự đoán `Churn / No Churn`

## 5.2. Trình bày kết quả demo
- In kết quả rõ ràng
- Có thể ghi thêm xác suất dự đoán nếu làm được

## 5.3. Ứng dụng thực tế trong doanh nghiệp
- Xác định nhóm khách hàng nguy cơ cao
- Gửi ưu đãi giữ chân
- Cải thiện dịch vụ hỗ trợ kỹ thuật
- Hỗ trợ bộ phận CSKH ưu tiên chăm sóc

## 5.4. Ý nghĩa kinh doanh
- Mô hình không chỉ để dự đoán
- Mà còn giúp doanh nghiệp ra quyết định giữ chân khách hàng

## 5.5. Đầu ra cần nộp
- File code demo
- Phần viết ứng dụng thực tế
- Nội dung ngắn gọn, thiên về business

---

# 6. KẾT LUẬN, TỔNG HỢP VÀ HOÀN THIỆN BLOG
**Người phụ trách:** **Lê Nguyễn Hải Giang**  
**Phối hợp:** **Toàn nhóm**

## 6.1. Kết luận chung
- Tóm tắt bài toán
- Tóm tắt quy trình thực hiện
- Nêu mô hình tốt nhất
- Nêu yếu tố ảnh hưởng mạnh đến churn

## 6.2. Hạn chế ngắn
- Dataset mẫu
- Chưa tối ưu quá sâu
- Chưa triển khai thành hệ thống thực tế

## 6.3. Hướng phát triển
- Thử thêm mô hình khác
- Tối ưu hyperparameter
- Xây dựng dashboard hoặc app demo

## 6.4. Tổng hợp bài blog
- Ghép các phần từ 1 đến 5
- Chuẩn hóa tiêu đề
- Kiểm tra chính tả
- Kiểm tra hình ảnh, bảng biểu, code minh họa

## 6.5. Đầu ra cần nộp
- File blog hoàn chỉnh
- Format đồng nhất
- Nội dung mạch lạc, ngắn gọn, dễ đọc

---

# 7. BẢNG PHÂN CÔNG TÓM TẮT

| Mục | Nội dung | Người phụ trách | Đầu ra |
|---|---|---|---|
| **1** | Mở đầu: khái niệm churn, tầm quan trọng, mục tiêu blog | **Lê Nguyễn Hải Giang** | Phần viết mở đầu |
| **2** | Giới thiệu dữ liệu và EDA | **Phạm Hồng Ngát** | Notebook EDA + biểu đồ + nhận xét |
| **3** | Tiền xử lý dữ liệu | **Dương Khánh Ly** | Code preprocessing |
| **4** | Xây dựng và đánh giá mô hình | **Nhóm trưởng** | Notebook modeling + bảng kết quả |
| **5** | Demo dự đoán và ứng dụng thực tế | **Lê Minh Ngọc** | Code demo + phần business |
| **6** | Kết luận, tổng hợp, hoàn thiện blog | **Lê Nguyễn Hải Giang** | Blog hoàn chỉnh |

---

# 8. LỘ TRÌNH THỰC HIỆN

## Ngày 1
- Hoàn thành mục 1
- Hoàn thành giới thiệu dataset trong mục 2

## Ngày 2
- Hoàn thành EDA trong mục 2
- Hoàn thành mục 3

## Ngày 3
- Hoàn thành mục 4
- Xuất bảng kết quả mô hình

## Ngày 4
- Hoàn thành mục 5
- Viết phần ứng dụng thực tế

## Ngày 5
- Hoàn thành mục 6
- Ghép bài, sửa format, kiểm tra lần cuối

---

# 9. TIÊU CHÍ HOÀN THÀNH

## 9.1. Nội dung
- Đúng trọng tâm bài toán churn
- Không viết lan man
- Có số liệu, biểu đồ, mô hình và kết quả

## 9.2. Hình thức
- Các đề mục rõ ràng
- Văn phong ngắn gọn kiểu blog
- Bảng biểu và hình ảnh dễ nhìn

## 9.3. Kỹ thuật
- Code chạy được
- Có kết quả rõ ràng
- Có phần demo dự đoán khách hàng

---

# 10. SẢN PHẨM CUỐI CÙNG
- 01 bài blog hoàn chỉnh
- 01 notebook code tổng
- Bộ biểu đồ EDA
- Bảng so sánh mô hình
- Demo dự đoán churn cho 1 khách hàng
