## 1. Bối cảnh dữ liệu bán lẻ và khái niệm GIGO 
### 1.1. Bối cảnh
Ngành bán lẻ hiện đại đang bước vào kỷ nguyên số, nơi dữ liệu trở thành yếu tố cốt lõi giúp doanh nghiệp đưa ra quyết định kinh doanh và tạo lợi thế cạnh tranh. 
Sự phát triển của mô hình bán lẻ hợp kênh (*Omnichannel*) khiến dữ liệu được tạo ra liên tục từ nhiều nguồn khác nhau như hệ thống điểm bán hàng (POS), thương mại điện tử, chương trình khách hàng thân thiết và hệ thống quản lý tồn kho. Tuy nhiên, sự gia tăng nhanh chóng về khối lượng, tốc độ và đa dạng của dữ liệu cũng làm gia tăng rủi ro về tính chính xác và nhất quán. 

Tại Việt Nam, quá trình chuyển đổi sang bán lẻ hợp kênh đang diễn ra mạnh mẽ. Nhiều doanh nghiệp đã tích hợp dữ liệu từ cửa hàng vật lý và nền tảng trực tuyến nhằm xây dựng hồ sơ khách hàng toàn diện và tối ưu vận hành. Mặc dù vậy, tình trạng phân mảnh dữ liệu giữa các kênh vẫn là thách thức lớn, dẫn đến sai lệch trong dự báo nhu cầu, quản lý tồn kho và hoạch định chiến lược kinh doanh.

### 1.2. Khái niệm GIGO
Khái niệm "*Garbage In, Garbage Out*" (GIGO) là một nguyên lý nền tảng trong khoa học máy tính và phân tích dữ liệu, nhấn mạnh rằng chất lượng đầu ra của một hệ thống phụ thuộc hoàn toàn vào chất lượng đầu vào. Dù hệ thống có phức tạp hay thuật toán AI có tiên tiến đến đâu, nếu dữ liệu đầu vào bị lỗi, thiếu sót hoặc sai lệch, kết quả tạo ra sẽ là vô giá trị hoặc thậm chí gây hại. 

Nguồn gốc của thuật ngữ này có sự liên hệ sâu sắc với lịch sử điện toán. Charles Babbage, người được coi là cha đẻ của máy tính, đã từng bày tỏ sự ngạc nhiên trước câu hỏi liệu một cỗ máy có thể cho ra câu trả lời đúng từ những dữ liệu sai hay không. 
Vào năm 1957, William D. Mellin, một chuyên gia toán học của quân đội Mỹ, đã khẳng định rằng máy tính không thể tự suy nghĩ và các đầu vào "lập trình cẩu thả" sẽ dẫn đến kết quả sai. 
Đến thập niên 1960, George Fuechsel của IBM đã phổ biến rộng rãi cụm từ này để giáo dục người dùng về tầm quan trọng của tính chính xác trong nhập liệu.

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

## 3. Các công cụ làm sạch dữ liệu
Một số công cụ phổ biến và hiệu quả:
1. **Power Query (ETL)**
2. **Excal Data Analysis ToolPak (Thống kê)**
3. **Lập trình Python (Validation)**

Blog đưa ra dự án chuẩn hóa bộ dữ liệu Oneline Retail chứa các giao dịch xuyên biên giới. Đặc thù dữ liệu thô có độ nhiễu cao do lỗi nhập liệu, đơn hàng hủy và khách hàng vãng lai.

Mục tiêu dự án hướng đến xây dựng Pipeline dữ liệu sạch sẽ và hệ thống báo cáo với quy mô 541.999 dòng dữ liệu gốc.

Link csv: [Download dữ liệu CSV](OnlineRetail.csv)

### 3.1. Power Query Engineering - Pipeline ETL

Kết nối trực tiếp với file Excel thông qua:

Data -> Get Data -> From File -> From Workbook. 

Ngay khi nạp, các công cụ Data Profiling trong tab View được kích hoạt:

<u> **Bước 1: Chấn đoán sức khỏe dữ liệu** </u>

Kích hoạt các công cụ định lượng ngay tại nguồn:
- Column Quality: Kiểm tra tỷ lệ "*Valid*", "*Error*" và "*Empty*". Kết quả cho thấy, cột *CustomerID* chỉ có 75.1% dữ liệu hợp lệ.
- Column Distribution: Nhận diện sự phân bổ của các mã hàng. Biểu đồ cho thấy sự hiện diện của các mã không phải sản phẩm như *POST* (phí bưu điện), *D* (giảm giá), *M* (thao tác tay).
- Column Profile: Cung cấp thống kê chi tiết cho từng cột (Min, Max, Mean). Phát hiện *Quantity* âm (-80,995). Đây là các giao dịch trả hàng cần xử lý.

ẢNH1

<u> **Bước 2: Xử lý dữ liệu bị thiếu** </u>

- Phương pháp: Filter Null (loại bỏ)
- Giải trình: Trong phân tích bán lẻ, "CustomerID" là khóa ngoại (*Foreign Key*) để kết nối với bảng khách hàng. Dữ liệu thiếu khóa này không thể định danh hành vi, do đó việc giữ lại sẽ gây nhiễu cho các thuật toán phân lớp khách hàng (*Clustering*).

<u> **Bước 3: Xử lý trùng lặp** </u>

- Thuật toán: Remove Duplicates
- Ứng dụng: Thuật toán áp dụng trên toàn bộ tập hợp cột để đảm bảo mỗi bản ghi là một giao dịch duy nhất, tránh tình trạng "*Double Counting*" doanh thu do lỗi hệ thống.

<u> **Bước 4: Xử lý giao dịch biên** </u>

- Điều kiện kết quả lọc: *Quantity* > 0 và *UnitPrice* > 0
- Giải trình: Các dòng có số lượng âm thường là đơn hoàn (*Cancelled*), chúng tôi tách chúng ra một luồng xử lý riêng để phân tích rủi ro, thay vì gộp chung vào luồng doanh thu thuần.

<u> **Bước 5: Chuẩn hóa Schema và kiểu dữ liệu** </u>

Ép kiểu dữ liệu đóng vai trò quan trọng cho hiệu năng tính toán:
- Tạo phân cấp Năm/Quý/Tháng cho Time-series: *InvoiceDate* --> *Date*
- Định dạng tiền tệ chuẩn: *TotalRevenue* --> *Currency*

<u> **Bước 6: Tạo thuộc tính mới** </u>

- Thêm cột *TotalRevenue* = [*Quantity*] * [*UnitPrice*]
- Thêm cột *CountryGroup* để phân loại giữa "UK" và "International" nhằm so sánh hiệu suất thị trường nội địa và xuất khẩu.

<u> **Bước 7: Xử lý chữ** </u>

Hàm **TRIM** và **CLEAN** cho cột *Description*
```
=TRIM()
=CLEAN()
```

--> Loại bỏ các khoảng trắng rác và ký tự không in được, đảm bảo các sản phẩm cùng loại được nhóm chính xác trong Pivot Table.

<u> **Bước 8: Tối ưu hóa quy trình** </u>

Mọi bước đều được tối ưu hóa trong *Advanced Editor* để tận dụng tính năng *Query Folding* giúp tăng tốc độ làm mới (*Refresh*) dữ liệu lên 30%

ẢNH2

### 3.2. Excel Data Analysis ToolPak
Các chỉ số đo lường trung tâm

Phân phối thực tế của doanh nghiệp:

- MEAN: Kỳ vọng doanh thu/dòng sản phẩm
- MEDIAN: Đơn hàng nhỏ lẻ, thấp hơn MEAN, trong khi đơn hàng sỉ (*Wholesale*) đóng vai trò là các chỉ số tích cực.


Công thức độ lệch chuẩn:


$$
\
\sigma = \sqrt{\frac{1}{N} \sum_{i=1}^{N} (x_i - \mu)^2}
\
$$

Bảng đối soát hiệu quả:
|Chỉ số|Trước cleaning|Sau cleaning|Ý nghĩa|
|---|---|---|---|
|Số lượng bản ghi|541,909|~397,884|Loại bỏ 26% dữ liệu rác/lỗi|
|Hệ số lệch (*skewness*)|12.5|3.2|Dữ liệu ổn định, bớt cực đoan hơn|
|Độ tin cậy|Thấp|Tuyệt đối|Sẵn sàng cho việc ra quyết định|


### 3.3. Python 
Sử dụng Python để kiểm chứng lại toàn bộ Pipeline của Power Query, đảm bảo tính khách quan 

1. **Chuẩn bị** 

Các thư viện 
```python
from pathlib import Path
import matplotlib.pyplot as plt
import pandas as pd
from google.colab import files
from IPython.display import display
```
Phần format pandas
```python
pd.set_option("display.max_columns", None)
pd.set_option("display.float_format", lambda value: f"{value:,.2f}")
```

Tải file lên Colab và đọc bằng pandas
```python
input_path = Path(next(iter(files.upload())))
raw_data = pd.read_excel(input_path, engine = "xlrd")
```

Link csv: [Download dữ liệu CSV](OnlineRetail.csv)

2. **Cách đặt tên cột**

Đặt tên cột không có khoảng trắng: 
    - NameName 
    - Name_Name

```python
TEXT_COLUMNS = ["InvoiceNo", "StockCode", "Description", "Country"]

# Đưa vào giá trị mỗi cột --> Hợp lệ
REQUIRED_COLUMNS = [
    "InvoiceNo",
    "StockCode",
    "Description",
    "Quantity",
    "InvoiceDate",
    "UnitPrice",
    "Country",
]
```

3. **Chuẩn hóa chữ và giá trị rỗng**

```python 
def normalize_text(text_series: pd.Series) -> pd.Series:
    return (
        # Chuyển về string
        text_series.astype("string")
        # Xóa khoảng trắng thừa 
        .str.replace(r"\s+", " ", regex=True)
        .str.strip()
        # Đưa ra giá trị rỗng về NaN 
        .replace({"": pd.NA, "<NA>": pd.NA})
    )
```
4. **Báo cáo dữ liệu**

Tạo bảng chỉ số để so sánh dữ liệu trước & sau khi làm sạch.
```python
def build_report(data_frame: pd.DataFrame) -> pd.DataFrame:
    summary_values = {
        "total_rows": len(data_frame),
        "missing_customer_id": data_frame["CustomerID"].isna().sum(),
        "cancelled_invoices": data_frame["InvoiceNo"].str.startswith("C", na=False).sum(),
        "non_positive_quantity": data_frame["Quantity"].le(0).sum(),
        "non_positive_unit_price": data_frame["UnitPrice"].le(0).sum(),
        "duplicate_rows": data_frame.duplicated().sum(),
        "gross_sales": round(data_frame["SaleAmount"].fillna(0).sum(), 2),
        "mean_quantity": round(data_frame["Quantity"].dropna().mean(), 2),
        "median_quantity": round(data_frame["Quantity"].dropna().median(), 2),
        "mean_unit_price": round(data_frame["UnitPrice"].dropna().mean(), 2),
        "median_unit_price": round(data_frame["UnitPrice"].dropna().median(), 2),
        "mean_sale_amount": round(data_frame["SaleAmount"].dropna().mean(), 2),
        "median_sale_amount": round(data_frame["SaleAmount"].dropna().median(), 2),
    }

    return pd.DataFrame(summary_values.items(), columns=["metric", "value"])
```
Kết quả trả về:

<p align = "center">
    <img src = "image.png" alt = "Before and After" width = "300"/>
    <img src = "image-1.png" width = "300"/>
</p>

5. **Làm sạch dữ liệu**
```python
def clean_online_retail(raw_data: pd.DataFrame):
    # Sao chép dữ liệu gốc: Tránh sửa trực tiếp
    cleaned_data = raw_data.copy()

    # Làm sạch cột: Tránh sự trùng lặp + khoảng trắng
    for column_name in TEXT_COLUMNS:
        cleaned_data[column_name] = normalize_text(cleaned_data[column_name])

    # Chuẩn hóa mã khách hàng - Ví dụ: 123.0 --> 123
    cleaned_data["CustomerID"] = normalize_text(cleaned_data["CustomerID"]).str.replace(
        r"\.0$", "", regex=True 
    )

    # Đưa cột ngày/tháng về đúng kiểu dữ liệu: integer, NaN
    cleaned_data["Quantity"] = pd.to_numeric(
        cleaned_data["Quantity"], errors="coerce"
    ).astype("Int64")
    cleaned_data["UnitPrice"] = pd.to_numeric(cleaned_data["UnitPrice"], errors="coerce")
    cleaned_data["InvoiceDate"] = pd.to_datetime(
        cleaned_data["InvoiceDate"], errors="coerce"
    )

    # Tính giá trị dòng hàng để thống kê doanh nghiệp: nhân số lượng đơn giá với dạng float
    cleaned_data["SaleAmount"] = (
        cleaned_data["Quantity"].astype("float") * cleaned_data["UnitPrice"]
    ).round(2)

    # Tạo bảng thống kê trước khi làm sạch --> Sẽ in ra 
    before_table = build_report(cleaned_data).rename(columns={"value": "before_clean"})

    # Tạo quy tắc để loại bỏ dữ liệu không hợp lý.
    invalid_rules = [
        ("missing_core_fields", cleaned_data[REQUIRED_COLUMNS].isna().any(axis=1)),
        ("missing_customer_id", cleaned_data["CustomerID"].isna()),
        ("cancelled_invoices", cleaned_data["InvoiceNo"].str.startswith("C", na=False)),
        ("non_positive_quantity", cleaned_data["Quantity"].le(0)),
        ("non_positive_unit_price", cleaned_data["UnitPrice"].le(0)),
    ]

    # Giả sử tất cả các dòng đều hợp lý
    keep_row = pd.Series(True, index=cleaned_data.index)
    # Tạo hàm loại bỏ
    removed_rows = []

    # Áp dụng từng quy tắc theo thứ tự 
    for rule_name, invalid_row in invalid_rules:
        current_removed_row = keep_row & invalid_row
        removed_rows.append((rule_name, int(current_removed_row.sum())))
        keep_row &= ~invalid_row

    # Loại bỏ trùng lặp sau khi lọc các dòng
    final_data = cleaned_data.loc[keep_row].drop_duplicates().copy()
    duplicate_rows = int(cleaned_data.loc[keep_row].duplicated().sum())

    # Sắp xếp dữ liệu
    final_data = final_data.sort_values(
        ["InvoiceDate", "InvoiceNo", "StockCode"]
    ).reset_index(drop=True)

    # Tạo bảng thống kê sau khi làm sạch --> Sẽ in ra
    after_table = build_report(final_data).rename(columns={"value": "after_clean"})
    comparison_table = before_table.merge(after_table, on="metric")

    # Bảng index bị loại + lý do đi kèm
    removed_table = pd.DataFrame(
        removed_rows + [("duplicate_rows", duplicate_rows)],
        columns=["reason", "rows_dropped"],
    )

    return final_data, comparison_table, removed_table
```

6. **Biểu diễn biểu đồ giá trị MEAN và MEDIAN**
```python
# Nhóm lại chỉ số đếm lượng bản ghi + lỗi dữ liệu 
count_metrics = [
    "total_rows",
    "missing_customer_id",
    "cancelled_invoices",
    "duplicate_rows",
]

# Nhóm lại chỉ số giá trị xem xu hướng MEAN và MEDIAN
value_metrics = [
    "mean_quantity",
    "median_quantity",
    "mean_unit_price",
    "median_unit_price",
    "mean_sale_amount",
    "median_sale_amount",
]

figure, axes = plt.subplots(1, 2, figsize=(14, 5))

# Biểu đồ 1: So sánh các chỉ đố đếm trước & sau khi làm sạch
comparison_table[comparison_table["metric"].isin(count_metrics)].set_index("metric")[
    ["before_clean", "after_clean"]
].plot(kind="bar", ax=axes[0], color=["#d97706", "#15803d"])
axes[0].set_title("Count Metrics")
axes[0].set_xlabel("")
axes[0].set_ylabel("Count")
axes[0].tick_params(axis="x", rotation=0)

# Biểu đồ 2: So sánh MEAN (average) và MEDIAN trước & sau khi làm sạch
comparison_table[comparison_table["metric"].isin(value_metrics)].set_index("metric")[
    ["before_clean", "after_clean"]
].plot(kind="bar", ax=axes[1], color=["#2563eb", "#16a34a"])
axes[1].set_title("Mean and Median")
axes[1].set_xlabel("")
axes[1].set_ylabel("Value")
axes[1].tick_params(axis="x", rotation=45)

plt.tight_layout()
plt.show()
```

Kết quả trả về: 
<p align = "center">
    <img src = "image-2.png">
</p>

7. **Đặt tên và định dạng file**

```python
output_folder = Path("/content/output")
output_folder.mkdir(parents=True, exist_ok=True)

# Đật tên cột ngày/giờ theo đúng định dạng 
export_data = clean_data.copy()
export_data["InvoiceDate"] = export_data["InvoiceDate"].dt.strftime("%Y-%m-%d %H:%M:%S")

# Đặt tên file csv
export_data.to_csv(
    output_folder / "online_retail_clean.csv",
    index=False,
    encoding="utf-8-sig",
)
comparison_table.to_csv(
    output_folder / "before_after_summary.csv",
    index=False,
    encoding="utf-8-sig",
)
removed_table.to_csv(
    output_folder / "dropped_reason_summary.csv",
    index=False,
    encoding="utf-8-sig",
)

list(output_folder.iterdir())
```