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