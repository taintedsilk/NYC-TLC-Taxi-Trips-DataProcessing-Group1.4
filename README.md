# NYC-TLC-Taxi-Trips-DataProcessing-Group1.4

## Mục lục
1.  Tổng quan Dự án
2.  Công cụ và Môi trường
3.  Cấu trúc Thư mục
4.  Cài đặt Dependencies
5.  Hướng dẫn Thực thi Quy trình (Workflow)
6.  Ghi chú về Tính Tái Tạo (Reproducibility)

---

## 1. Tổng quan Dự án

Dự án này thực hiện quy trình **ETL (Extract, Transform, Load)** hoàn chỉnh cho dữ liệu chuyến đi Taxi Vàng (Yellow Taxi) của NYC TLC trong năm 2022.

**Mục tiêu chính:**
1.  **Làm sạch & Hợp nhất:** Tải về và hợp nhất 12 file dữ liệu thô hàng tháng. Áp dụng các kiểm tra QA nghiêm ngặt trên `trip_duration`, `trip_distance`, `fare_amount`, và `zone_codes`.
2.  **Tính toán KPI:** Tính toán các chỉ số cơ bản và tổng hợp các KPI chi tiết theo thời gian/khu vực (ví dụ: Tốc độ Trung vị theo giờ, Top Zones).
3.  **Phân tích Nâng cao:** Thực hiện phân tích Z-score theo bối cảnh (Hour x Day of Week) để xác định các chuyến đi bất thường về tốc độ/thời lượng.

## 2. Công cụ và Môi trường

| Loại | Công cụ | Mục đích |
| :--- | :--- | :--- |
| **Ngôn ngữ** | Python 3.x | Ngôn ngữ lập trình chính. |
| **Thư viện** | Pandas, NumPy | Xử lý dữ liệu in-memory. |
| **Hỗ trợ I/O** | Pyarrow | Đọc và ghi dữ liệu ở định dạng Parquet. |
| **Tính toán** | Tdigest | Hỗ trợ tính toán chính xác các bách phân vị (percentiles/median). |
| **Môi trường** | Jupyter Notebooks / Python Script | Môi trường phát triển và thực thi code. |

## 3. Cấu trúc Thư mục

| Thư mục/File | Mục đích |
| :--- | :--- |
| `raw/` | Chứa dữ liệu thô ban đầu (file Parquet 12 tháng) và file lookup khu vực (`taxi_zone_lookup.csv`). |
| `processed/` | **ĐẦU RA:** Chứa các file dữ liệu trung gian và file hợp nhất cuối cùng (`all_cleaned_yellow_tripdata_2022.parquet`). |
| `src/` | Chứa các Jupyter Notebooks (.ipynb) thực hiện các bước trong quy trình. |
| `04_advanced_analysis.py` | Script Python chạy phân tích Z-score nâng cao, độc lập với Notebook. |
| `requirements.txt` | Liệt kê tất cả các thư viện Python cần thiết. |

## 4. Cài đặt Dependencies

Để chạy lại mã nguồn, bạn cần có Python 3.x. Sử dụng `pip` để cài đặt tất cả các thư viện cần thiết, được liệt kê trong `requirements.txt`:

```bash
pip install -r requirements.txt
```
---

## 5. Hướng dẫn Thực thi Quy trình (Workflow)

Quy trình được chia thành 5 giai đoạn chính. Vui lòng chạy các Notebooks theo thứ tự sau để đảm bảo dữ liệu được xử lý tuần tự:

| Giai đoạn | Tên Notebook/Script | Mục tiêu Chính |
| :--- | :--- | :--- |
| **1. Tải về** | `1_download.ipynb` | Tải về dữ liệu thô 12 tháng và file lookup khu vực. |
| **2. Làm sạch (QA)** | `2_process.ipynb` | Áp dụng các quy tắc làm sạch, tính toán thời lượng/tốc độ cơ bản, và lưu file sạch từng tháng. |
| **3. Hợp nhất & KPI** | `3_calculate_kpi.ipynb` | Hợp nhất dữ liệu đã làm sạch (tạo `all_cleaned_yellow_tripdata_2022.parquet`) và tính toán KPI cơ bản. |
| **4. KPI Chi tiết & Trực quan** | `3.1_extra_kpis_compat.ipynb`, `4-1_add_some_kpi_need_for_visualization (1).ipynb`, `4_visualization.ipynb` | Tính toán các KPI chi tiết theo giờ/khu vực (Heatmap data, Median Speed) và tạo các biểu đồ trực quan hóa. |
| **5. Phân tích Nâng cao** | `04_advanced_analysis.py` | **(Chạy bằng lệnh Terminal)** Thực hiện phân tích Z-score theo bối cảnh để tạo số liệu báo cáo về Outliers. |

### Thực thi Giai đoạn 5 (Phân tích Nâng cao)

Sau khi hoàn thành các Notebook (Giai đoạn 1-4) và file dữ liệu hợp nhất đã được tạo, bạn chạy script phân tích cuối cùng bằng lệnh Terminal:

```bash
python 4_advanced_analysis.ipynb
```

## 6. Ghi chú về Tính Tái Tạo (Reproducibility)

### Công bố Seed (Reproducibility Statement)

Mã nguồn trong file `04_advanced_analysis.py` và toàn bộ quy trình xử lý dữ liệu bằng Pandas **chỉ sử dụng** các phép tính thống kê hoàn toàn xác định (Deterministic) như Trung bình, Độ lệch chuẩn, và Z-score.

**Mã nguồn không chứa bất kỳ hàm ngẫu nhiên nào.** Do đó, kết quả đầu ra sẽ luôn đồng nhất khi chạy lại trên cùng một tập dữ liệu đầu vào. **Không cần công bố giá trị seed.**