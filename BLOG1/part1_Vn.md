## 1. Bối cảnh dữ liệu bán lẻ và khái niệm GIGO 
### 1.1. Bối cảnh

### 1.2. Khái niệm GIGO

## 2. Vai trò của Data cleaning
### 2.1. Khái niệm của Data cleaning
Thuật ngữ *Data cleaning* chỉ quá trình xử lý dữ liệu để làm sạch bộ dữ liệu thô. Các thao tác *cleaning* hướng đến cải thiện chất lượng và khả năng sử dụng của dữ liệu một cách nhất quán.

Tầm quan trọng của làm sạch dữ liệu trong các lĩnh vực khác nhau:
1. Phân tích dữ liệu:
    - Đảm bảo tính chính xác trong đưa ra các quyết định quan trọng: Tính toàn vẹn của kết luận phụ thuộc vào độ sạch của dữ liệu.
    - Giảm thiểu sai lệch trong quá trình phân tích, đặc biệt trong nghiên cứu và phát triển khoa học, doanh nghiệp,...
2. Khoa học dữ liệu:
    - Chiếm 60-80% thời gian, công sức của dự án 
    - Giúp các nhà khoa học xây dựng mô hình có độ tin cậy cao. 
    - Tối ưu các thuật toán: cả về chi phí, thời gian.
3. Học máy:
    - Nền tảng xây dựng mô hình học máy (*Machine Learning*) hiệu quả.
    - Cải thiện khả năng tổng quan trên dữ liệu mới.
    - Giảm thiểu rủi ro quá khớp (*overfitting*). 

Phân biệt *cleaning* và *transformation*
|Đặc điểm|Cleaning|Transformation|
|---|---|---|
|Mục tiêu|Đảm bảo dữ liệu đầy đủ, chính xác|Chuyển cấu trúc dữ liệu sang dạng phù hợp hơn cho phân tích|
|Hoạt động chính|- Sửa lỗi nhập liệu: sai chính tả, thiếu dữ liệu - Loại bỏ sự trùng lặp - Loại bỏ số liệu không hợp lệ (NaN, số âm) - Loại bỏ dữ liệu nhiễu (không liên quan)|- Chuẩn hóa đơn vị đo - Dùng Pivot/Unipivot để thay đổi cấu trúc - Áp dụng các phép toán chuẩn hóa dữ liệu về cùng thang đo - Tạo biến mới từ dữ liệu gốc|
|Ví dụ|||

### 2.2. Tư duy xử lý dữ liệu giữa GUI và Code

## 3. Cách sử dụng Power Query

## 4. Thống kê chỉ số 
Một vài hàm đặc trưng trong excel

Ứng dụng Excel được sử dụng rộng rãi trong các bài toán *Data Analyst*. Ta ví dụ cho một bộ data set sau (định dạng csv)
|Hàm|Ứng dụng|Cấu trúc|Ví dụ|
|---|---|---|---|
|Average|Tính giá trị trung bình |=AVERAGE(block:block)|=AVERAGE(A1:A10) tính giá trị từ ô A1 đên ô A2|
|Meadian|Tìm trung vị|=MEDIAN(block:block)|=MEDIAN(C2:C6)| 
|Standard Deviation |Độ lệch chuẩn của mẫu |=STDEV.S(block:block)|=STDEV.S(G6:G13)|
|Skewness|Độ lệch |=SKEW(block:block)|=SKEW(A4:A30)|

Ví dụ với data set: "Giá sản phẩm" \
![alt text](image.png)

```excel
Trung vị của giá mặt hàng
=MEDIAN(B2:B10)=45.000 
```
Sự thay đổi sau dọn dẹp 

## 5. Luồng dữ liệu Pipeline