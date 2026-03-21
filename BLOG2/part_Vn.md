# 1. Bài toán *Churn* trong doanh nghiệp
## 1.1. Khái niệm *Telco Customer Churn*
Thuật ngữ *Telco Customer Churn* trong doanh nghiệp chỉ hiện tượng khách hàng ngừng sử dụng dịch vụ hoặc chuyển sang nhà cung cấp/công ty khác.

Vai trò của bài toán *Churn* 

## 1.2. Mục tiêu bài Blog

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
Trong mã hóa dữ liệu, các dạng dữ liệu dưới dạng chữ ('text' - string) cần chuyển sang mã hóa số để mô hình Machine Learning có thể hiểu. 

Ví dụ: 'NO', 'no', 'No' encode sang 0 \
'YES', 'yes', 'Yes' encode sang 1

Ở trong file CSV có xuất hiện '*Female*' và '*Male*' được phân loại ở cột '*gender*'.

Do vậy cần chuẩn hóa dữ liệu chữ để phân loại: \
- '*Female*' = 1
- '*Male*' = 0


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
- Nhập dữ liệu 1 khách hàng
- Tiền xử lý đầu vào
- Dự đoán `Churn / No Churn`

Trình bày kết quả demo
- In kết quả rõ ràng
- Có thể ghi thêm xác suất dự đoán nếu làm được

## 5.2. Ứng dụng thực tế trong doanh nghiệp 

Ý nghĩa:

# 6. Kết luận

# 7. Nguồn tham khảo 