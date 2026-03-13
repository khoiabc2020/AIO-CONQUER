## 1. Bối cảnh dữ liệu bán lẻ và khái niệm GIGO 
### 1.1. Bối cảnh
Ngành bán lẻ hiện đại đang bước vào kỷ nguyên dữ liệu và trí tuệ nhân tạo (AI), nơi dữ liệu trở thành yếu tố cốt lõi giúp doanh nghiệp đưa ra quyết định kinh doanh và tạo lợi thế cạnh tranh. Sự phát triển của mô hình bán lẻ hợp kênh (Omnichannel) khiến dữ liệu được tạo ra liên tục từ nhiều nguồn khác nhau như hệ thống điểm bán hàng (POS), thương mại điện tử, chương trình khách hàng thân thiết và hệ thống quản lý tồn kho. Tuy nhiên, sự gia tăng nhanh chóng về khối lượng, tốc độ và đa dạng của dữ liệu cũng làm gia tăng rủi ro về tính chính xác và nhất quán. 

Tại Việt Nam, quá trình chuyển đổi sang bán lẻ hợp kênh đang diễn ra mạnh mẽ. Nhiều doanh nghiệp đã tích hợp dữ liệu từ cửa hàng vật lý và nền tảng trực tuyến nhằm xây dựng hồ sơ khách hàng toàn diện và tối ưu vận hành. Tuy nhiên, tình trạng phân mảnh dữ liệu giữa các kênh vẫn là thách thức lớn, dẫn đến sai lệch trong dự báo nhu cầu, quản lý tồn kho và hoạch định chiến lược kinh doanh.

### 1.2. Khái niệm GIGO
Khái niệm "Garbage In, Garbage Out" (GIGO) là một nguyên lý nền tảng trong khoa học máy tính và phân tích dữ liệu, nhấn mạnh rằng chất lượng đầu ra của một hệ thống phụ thuộc hoàn toàn vào chất lượng đầu vào. Dù hệ thống có phức tạp hay thuật toán AI có tiên tiến đến đâu, nếu dữ liệu đầu vào bị lỗi, thiếu sót hoặc sai lệch, kết quả tạo ra sẽ là vô giá trị hoặc thậm chí gây hại. 

Nguồn gốc của thuật ngữ này có sự liên hệ sâu sắc với lịch sử điện toán. Charles Babbage, người được coi là cha đẻ của máy tính, đã từng bày tỏ sự ngạc nhiên trước câu hỏi liệu một cỗ máy có thể cho ra câu trả lời đúng từ những dữ liệu sai hay không. Vào năm 1957, William D. Mellin, một chuyên gia toán học của quân đội Mỹ, đã khẳng định rằng máy tính không thể tự suy nghĩ và các đầu vào "lập trình cẩu thả" sẽ dẫn đến kết quả sai. Đến thập niên 1960, George Fuechsel của IBM đã phổ biến rộng rãi cụm từ này để giáo dục người dùng về tầm quan trọng của tính chính xác trong nhập liệu.

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
