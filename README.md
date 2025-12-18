# NYC-TLC-Taxi-Trips-DataProcessing-Group1.4

## 1. Tổng quan Dự án
Dự án thực hiện xây dựng hệ thống **ETL Pipeline** toàn diện cho dữ liệu Taxi Vàng (Yellow Taxi) của NYC TLC năm 2022. Quy trình đi từ nạp dữ liệu thô, làm sạch/chuẩn hóa đến tổng hợp KPI và dự báo nâng cao bằng mô hình SARIMA.

## 2. Công cụ và Môi trường
| Thành phần | Công cụ sử dụng |
| :--- | :--- |
| **Ngôn ngữ** | Python 3.x |
| **Thư viện ETL** | Pandas, NumPy, Pyarrow (xử lý định dạng Parquet) |
| **Phân tích/Dự báo** | Tdigest (P95), Statsmodels (SARIMA), XGBoost (Profitability) |
| **Trực quan hóa** | Matplotlib, Seaborn |

## 3. Cấu trúc Thư mục (Theo yêu cầu đề bài)
| Thư mục/File | Mô tả nội dung |
| :--- | :--- |
| `raw/` | Chứa dữ liệu thô (.parquet 12 tháng) và `taxi_zone_lookup.csv`. |
| `processed/` | Chứa dữ liệu sau khi làm sạch (QA) và các bảng tổng hợp KPI. |
| `reports/` | Chứa các báo cáo kỹ thuật (`report-Group1.4.pdf`) và kết luận. |
| `src/` | Chứa toàn bộ các Jupyter Notebooks (.ipynb) thực hiện quy trình. |
| `README.md` | Hướng dẫn chạy lại toàn bộ quy trình. |
| `requirements.txt` | Danh sách thư viện và phiên bản cần thiết. |

## 4. Cài đặt Dependencies
Để cài đặt môi trường, mở Terminal tại thư mục gốc và chạy câu lệnh:
```bash
pip install -r requirements.txt
```
---
## Hướng dẫn Thực thi Quy trình (Workflow Pipeline)
Vui lòng chạy các tệp Notebook trong thư mục src/ theo đúng trình tự logic sau:

Giai đoạn 1: Pipeline Dữ liệu (ETL & Trực quan cơ bản)
src/1_download.ipynb: Tải dữ liệu thô từ nguồn NYC TLC.

src/2_process.ipynb: Làm sạch dữ liệu và thực hiện kiểm tra QA.

src/3_calculate_kpi.ipynb: Hợp nhất dữ liệu và tính toán các chỉ số KPI cơ bản.

src/3.1_extra_kpis_compat.ipynb: Xử lý bổ sung các KPI tương thích.

src/4_visualization.ipynb: Trực quan hóa dữ liệu tổng thể (Heatmap, P95 Trip Duration).

src/4_new_visualization.ipynb: Các phân tích biểu đồ mở rộng.

src/4-1_add_some_kpi_need_for_visualization.ipynb: Bổ sung các chỉ số cần thiết cho đồ thị.

Giai đoạn 2: Phân tích Nâng cao & Mô hình Dự báo
src/4_advanced_outlier_detection.ipynb: Sử dụng kỹ thuật thống kê nâng cao để phát hiện và xử lý điểm bất thường (Outliers).

src/5_profitability_model.ipynb: Phân tích mô hình lợi nhuận sử dụng XGBoost.

src/6_demand_prediction.ipynb: Dự báo nhu cầu khách hàng ngắn hạn.

src/7_demand_prediction_with_sarima.ipynb: Dự báo chuỗi thời gian nâng cao bằng mô hình SARIMA.

## 6. Ghi chú về Tính Tái Tạo (Reproducibility)
Công bố Seed (Reproducibility Statement)
Mã nguồn trong file 4_advanced_outlier_detection.ipynb và toàn bộ quy trình xử lý dữ liệu bằng Pandas chỉ sử dụng các phép tính thống kê hoàn toàn xác định (Deterministic) như Trung bình, Độ lệch chuẩn, và Z-score.

Mã nguồn không chứa bất kỳ hàm ngẫu nhiên nào. Do đó, kết quả đầu ra sẽ luôn đồng nhất khi chạy lại trên cùng một tập dữ liệu đầu vào. Không cần công bố giá trị seed.