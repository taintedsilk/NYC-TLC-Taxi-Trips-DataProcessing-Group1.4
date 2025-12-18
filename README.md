# NYC-TLC-Taxi-Trips-DataProcessing-Group1.4

## 1. Tổng quan Dự án
Dự án thực hiện xây dựng hệ thống **ETL Pipeline** toàn diện cho dữ liệu Taxi Vàng (Yellow Taxi) của NYC TLC năm 2022. Quy trình bao gồm từ thu thập dữ liệu thô, làm sạch, tính toán chỉ số kinh doanh (KPI) đến các mô hình dự báo nâng cao bằng SARIMA và XGBoost.

## 2. Công cụ và Môi trường
| Thành phần | Công cụ sử dụng |
| :--- | :--- |
| **Ngôn ngữ** | Python 3.9+ |
| **Xử lý dữ liệu** | Pandas, NumPy, Pyarrow (Xử lý Parquet 5.8GB+) |
| **Phân tích/Dự báo** | Statsmodels (SARIMA), XGBoost, Scikit-learn |
| **Trực quan hóa** | Matplotlib, Seaborn |

## 3. Cấu trúc Thư mục (Chi tiết các tệp tin)
| Thư mục/File | Mô tả nội dung |
| :--- | :--- |
| **`src/`** | **Chứa toàn bộ mã nguồn xử lý (.ipynb):** |
| ├─ `1_download.ipynb` | Tải dữ liệu thô tự động từ nguồn NYC TLC. |
| ├─ `2_process.ipynb` | Làm sạch dữ liệu, xử lý lỗi hệ thống và kiểm tra QA. |
| ├─ `3_calculate_kpi.ipynb` | Tính toán các chỉ số kinh doanh và vận hành cốt lõi. |
| ├─ `3.1_extra_kpis_compat.ipynb` | Xử lý bổ sung các chỉ số tương thích cho báo cáo. |
| ├─ `4_advanced_outlier_detection.ipynb` | Phát hiện điểm dị biệt nâng cao bằng Contextual Z-Score. |
| ├─ `4_visualization.ipynb` | Trực quan hóa mật độ nhu cầu và xu hướng vận hành. |
| ├─ `4_new_visualization.ipynb` | Các phân tích biểu đồ mở rộng về hiệu suất. |
| ├─ `4-1_add_some_kpi_need_for_visualization.ipynb` | Chuẩn bị tập dữ liệu đặc thù cho đồ thị. |
| ├─ `5_profitability_model.ipynb` | Mô hình hóa các yếu tố ảnh hưởng đến lợi nhuận. |
| ├─ `6_demand_prediction.ipynb` | Dự báo nhu cầu khách hàng ngắn hạn. |
| └─ `7_demand_prediction_with_sarima.ipynb` | Dự báo chuỗi thời gian chuyên sâu (MAPE ~5.29%). |
| **`raw/`** | **Dữ liệu thô:** 12 file `.parquet` năm 2022 và `taxi_zone_lookup.csv`. |
| **`processed/`** | **Dữ liệu sạch:** `kpi_daily`, `kpi_monthly`, `kpi_speed_by_hour`,... |
| **`figures/`** | **Kết quả đồ họa:** Lưu trữ các biểu đồ phân tích xuất ra từ code. |
| **`reports/`** | **Tài liệu:** Báo cáo kỹ thuật, tóm tắt QA và Kết luận dự án. |

## 4. Quy trình Tiền xử lý & Làm sạch (QA)
Dữ liệu thô được lọc nghiêm ngặt trong `src/2_process.ipynb` để đảm bảo tính chính xác:
* **Lọc Quãng đường & Hành khách:** Loại bỏ các chuyến đi có quãng đường $d \le 0$ hoặc $d > 100$ miles; số lượng khách không nằm trong khoảng 1-6.
* **Xác thực Tài chính:** Loại bỏ các hóa đơn có `total_amount` hoặc `fare_amount` $\le 0$.
* **Xác thực Thời gian:** Loại bỏ các chuyến đi có thời gian kết thúc trước thời gian bắt đầu.

## 5. Phân tích Nâng cao & Mô hình Dự báo
Điểm nhấn kỹ thuật của dự án nằm ở các phương pháp xử lý chuyên sâu:

### 5.1. Advanced Outlier Detection (`4_advanced_outlier_detection.ipynb`)
Sử dụng kỹ thuật **Contextual Z-Score** thay vì Z-score toàn cục. Phương pháp này tính toán độ lệch chuẩn dựa trên ngữ cảnh (khu vực đón/trả và khung giờ) để xác định chính xác các điểm dị biệt về cước phí mà không làm mất đi các đặc trưng vùng miền.

### 5.2. Demand Prediction & Profitability
* **Mô hình Lợi nhuận:** Sử dụng thuật toán **XGBoost** để phân tích các yếu tố then chốt quyết định thu nhập của tài xế.
* **Dự báo Nhu cầu:** Triển khai mô hình **SARIMA** cho dữ liệu chuỗi thời gian, đạt độ chính xác ấn tượng với sai số **MAPE ~5.29%**.

## 6. Hướng dẫn Cài đặt & Thực thi
Để cài đặt môi trường, mở Terminal tại thư mục gốc và chạy:
```bash
pip install -r requirements.txt

Workflow Pipeline:
Giai đoạn 1 (ETL): Chạy 1_download.ipynb đến 3.1_extra_kpis_compat.ipynb.

Giai đoạn 2 (Analysis): Chạy các file nhóm 4 để kết xuất biểu đồ vào thư mục figures/.

Giai đoạn 3 (Advanced): Thực thi các mô hình tại file 4_advanced..., 5, 6 và 7 để xem kết quả.