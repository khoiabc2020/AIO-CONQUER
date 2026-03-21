# 1. Bài toán *Churn* trong doanh nghiệp
## 1.1. Khái niệm *Churn*
Hãy hình dung doanh nghiệp của bạn giống như một chiếc xô đang đựng nước. Bạn đổ thêm nước vào (khách hàng mới), nhưng dưới đáy xô lại có những lỗ thủng khiến nước rò rỉ ra ngoài.
Hiện tượng "rò rỉ" đó chính là "*Churn*" (hiện tượng khách hàng rời bỏ - ngừng sử dụng dịch vụ).

Trong ngành viễn thông, *Churn* xảy ra khi một thuê bao quyết định ngừng gia hạn gói cước hoặc chuyển sang sử dụng dịch vụ của nhà mạng đối thủ. 
Đây không đơn thuần là việc mất đi một số điện thoại, mà là mất đi một dòng doanh thu ổn định.

## 1.2. Vì sao Churn là vấn đề sống còn?
- **Mất doanh thu trực tiếp**: Mỗi khách hàng rời đi là mất đi một khoản lợi nhuận.
- **Chi phí đắt đỏ để tìm người** mới: Các nghiên cứu đã chỉ ra rằng chi phí bỏ ra để thu hút khách hàng mới cao gấp 5 - 7 lần so với việc giữ chân khách hàng cũ.
- **Hiệu ứng domino**: Tỉ lệ "Churn" cao là dấu hiệu cho thấy sản phẩm/dịch vụ đang có vấn đề, gây ảnh hưởng, tác động tiêu cực đến uy tín và sự phát triển, tăng trưởng bền vững của doanh nghiệp.

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