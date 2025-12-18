# NYC-TLC-Taxi-Trips-DataProcessing-Group1.4

## 1. Tổng quan Dự án
Dự án thực hiện xây dựng hệ thống **ETL Pipeline** toàn diện cho dữ liệu Taxi Vàng (Yellow Taxi) của NYC TLC năm 2022. Quy trình đi từ nạp dữ liệu thô, làm sạch/chuẩn hóa đến tổng hợp KPI và dự báo nâng cao bằng mô hình SARIMA và XGBoost.

## 2. Công cụ và Môi trường
| Thành phần | Công cụ sử dụng |
| :--- | :--- |
| **Ngôn ngữ** | Python 3.x |
| **Thư viện ETL** | Pandas, NumPy, Pyarrow |
| **Phân tích/Dự báo** | Tdigest (P95), Statsmodels (SARIMA), XGBoost (Profitability) |
| **Trực quan hóa** | Matplotlib, Seaborn |

## 3. Cấu trúc Thư mục (Chi tiết các tệp tin)
| Thư mục/File | Mô tả nội dung |
| :--- | :--- |
| **`src/`** | **Chứa toàn bộ mã nguồn xử lý (.ipynb):** |
| ├─ `1_download.ipynb` | Tải dữ liệu thô từ nguồn NYC TLC. |
| ├─ `2_process.ipynb` | Làm sạch dữ liệu và thực hiện kiểm tra QA. |
| ├─ `3_calculate_kpi.ipynb` | Tính toán các chỉ số KPI cơ bản. |
| ├─ `3.1_extra_kpis_compat.ipynb` | Xử lý bổ sung các KPI tương thích. |
| ├─ `4_advanced_outlier_detection.ipynb` | Phát hiện điểm dị biệt bằng Contextual Z-Score. |
| ├─ `4_new_visualization.ipynb` | Các phân tích biểu đồ mở rộng. |
| ├─ `4_visualization.ipynb` | Trực quan hóa dữ liệu tổng thể (Heatmap, P95). |
| ├─ `4-1_add_some_kpi_need_for_visualization.ipynb` | Bổ sung chỉ số cho đồ thị. |
| ├─ `5_profitability_model.ipynb` | Mô hình đánh giá lợi nhuận bằng XGBoost. |
| ├─ `6_demand_prediction.ipynb` | Dự báo nhu cầu khách hàng ngắn hạn. |
| └─ `7_demand_prediction_with_sarima.ipynb` | Dự báo chuỗi thời gian nâng cao bằng SARIMA. |
| **`raw/`** | **Dữ liệu thô đầu vào:** |
| ├─ `yellow_tripdata_2022-01.parquet` -> `12.parquet` | Dữ liệu 12 tháng năm 2022. |
| └─ `taxi_zone_lookup.csv` | File tra cứu thông tin khu vực. |
| **`processed/`** | **Dữ liệu đã qua xử lý & KPI:** |
| ├─ `kpi_daily_2022.csv`, `kpi_weekly_2022.csv` | KPI tổng hợp theo ngày và tuần. |
| ├─ `kpi_monthly_2022.csv`, `kpi_speed_by_hour.csv` | KPI theo tháng và vận tốc theo giờ. |
| └─ `kpi_dow_p95_trip_duration.csv` | Chỉ số P95 thời gian chuyến đi. |
| **`figures/`** | **Kết quả trực quan hóa (.png):** |
| └─ `average_speed_by_hour_2022.png`, `hour_weekday_heatmap.png`, `top10_zones_2022.png`,... | Biểu đồ phục vụ báo cáo. |
| **`reports/`** | **Báo cáo & Tài liệu:** |
| ├─ `report Group1.4.pdf` | Báo cáo kỹ thuật chính. |
| ├─ `conclusion.pdf`, `data_dictionary.pdf` | Kết luận và từ điển dữ liệu. |
| └─ `qa_summary_2022.md`, `qa_summary.csv` | Tóm tắt kiểm tra chất lượng. |
| **File gốc** | `LICENSE`, `README.md`, `requirements.txt`, `.gitignore`. |

## 4. Cài đặt Dependencies
Để cài đặt môi trường, mở Terminal tại thư mục gốc và chạy câu lệnh:
```bash
pip install -r requirements.txt
```
---
## 5. Hướng dẫn Thực thi Quy trình (Workflow Pipeline)
Vui lòng chạy các tệp Notebook trong thư mục src/ theo đúng trình tự logic sau:

Giai đoạn 1: Pipeline Dữ liệu (ETL & Trực quan cơ bản)

src/1_download.ipynb: Tải dữ liệu thô.

src/2_process.ipynb: Làm sạch và QA (Loại bỏ quãng đường ≤ 0, giá tiền âm, tốc độ bất thường).

src/3_calculate_kpi.ipynb: Tính toán KPI doanh thu, vận tốc, P95 Duration.

src/3.1_extra_kpis_compat.ipynb: Đồng bộ hóa các chỉ số bổ sung.

src/4_visualization.ipynb, 4_new_visualization.ipynb, 4-1_add_some_kpi_...: Thực hiện trực quan hóa xu hướng và mật độ dữ liệu.

Giai đoạn 2: Phân tích Nâng cao & Mô hình Dự báo

src/4_advanced_outlier_detection.ipynb: Xử lý điểm bất thường bằng Contextual Z-Score.

src/5_profitability_model.ipynb: Mô hình lợi nhuận XGBoost.

src/6_demand_prediction.ipynb: Dự báo nhu cầu ngắn hạn.

src/7_demand_prediction_with_sarima.ipynb: Dự báo nâng cao SARIMA (đạt MAPE ~5.29%).
