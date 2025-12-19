# NYC-TLC-Taxi-Trips-DataProcessing-Group1.4

## 1. Tổng quan Dự án
Dự án thực hiện xây dựng hệ thống **ETL Pipeline** toàn diện cho dữ liệu Taxi Vàng (Yellow Taxi) của NYC TLC năm 2022. Quy trình bao gồm từ thu thập dữ liệu thô, làm sạch, tính toán chỉ số kinh doanh (KPI) đến các mô hình dự báo nâng cao bằng SARIMA và XGBoost.

Hệ thống được thiết kế để giải quyết thách thức về dữ liệu lớn (Big Data) với hơn **39 triệu bản ghi**. Mục tiêu không chỉ là xử lý dữ liệu mà còn khai thác các thông tin chiến lược như:
* **Tối ưu hóa nguồn lực:** Xác định thời điểm và địa điểm có nhu cầu cao nhất.
* **Phân tích hành vi:** Hiểu rõ cách khách hàng thanh toán và tip cho tài xế.
* **Đảm bảo tính minh bạch:** Phát hiện các sai số trong hệ thống tính cước tự động.
* **Dự báo chiến lược:** Cung cấp mô hình dự báo lưu lượng xe với độ chính xác cao phục vụ công tác điều phối.



## 2. Công cụ và Môi trường
Dự án tận dụng hệ sinh thái Python hiện đại để tối ưu hóa hiệu suất xử lý trên tập dữ liệu lớn (5.8GB+):

| Thành phần | Công cụ sử dụng | Mô tả chi tiết |
| :--- | :--- | :--- |
| **Ngôn ngữ** | Python 3.9+ | Sử dụng các tính năng nâng cao của Python để xử lý cấu trúc dữ liệu phức tạp. |
| **Xử lý dữ liệu** | Pandas, NumPy, Pyarrow | Sử dụng **Pyarrow engine** để xử lý tệp `.parquet`, giúp giảm dung lượng lưu trữ trên đĩa và tăng tốc độ đọc dữ liệu vào RAM lên đến 10 lần so với CSV. |
| **Phân tích/Dự báo** | Statsmodels, XGBoost, Scikit-learn | Áp dụng mô hình **XGBoost Regressor** cho phân tích lợi nhuận và mô hình chuỗi thời gian **SARIMA** cho dự báo nhu cầu. |
| **Trực quan hóa** | Matplotlib, Seaborn, Plotly | Tạo các bản đồ nhiệt (Heatmaps), biểu đồ phân phối và biểu đồ tương quan Pearson. |
| **Lưu trữ/Nén** | Snappy (trong Parquet) | Đảm bảo dữ liệu được nén tối ưu nhưng vẫn giữ được tốc độ truy xuất cao. |

## 3. Cấu trúc Thư mục (Chi tiết các tệp tin)
Mã nguồn được phân tách rõ ràng theo từng giai đoạn của Pipeline để dễ dàng debug và bảo trì:

| Directory/File | Detailed Content & Function |
| :--- | :--- |
| **`src/`** | **Source Code:** |
| ├─ `1_download.ipynb` | Parallel downloader for 2022 Yellow Taxi data and zone lookups using `ThreadPoolExecutor` [4]. |
| ├─ `2_process.ipynb` | Deep data profiling and caching. Generates summaries for passengers, issues, fare differences, and null values [5]. |
| ├─ `3_calculate_all_kpi.ipynb` | Optimized KPI calculation. Processes chunks to generate stats for heatmaps, speed curves, duration, and identifies top pickup zones [6]. |
| ├─ `4_advanced_outlier_detection.ipynb` | Implements advanced cleaning using `dask` and `fastparquet` for large-scale outlier detection [7]. |
| ├─ `4_visualization.ipynb` | Generates visualizations including Monthly Trips, Duration by Day of Week, Trip Density Heatmaps (Day vs Hour), and Speed Curves [8]. |
| ├─ `5_profitability_model.ipynb` | **New:** XGBoost Regressor pipeline for profitability estimation. Includes feature importance analysis and evaluation metrics (MAE, RMSE, R2) [1]. |
| ├─ `6_demand_prediction.ipynb` | **New:** Time-series analysis focusing on specific events (Holidays, Storms) and manual data correction (e.g., fixing 2022-09-18 data errors) [2]. |
| ├─ `7_demand_prediction_with_sarima.ipynb` | **New:** SARIMA model for demand forecasting. Includes confidence interval plotting and rolling forecast validation [3]. |

## 4. Quy trình Tiền xử lý & Làm sạch (QA)
Quy trình QA được thực hiện qua các bước lọc logic toán học cực kỳ chi tiết:

* **Làm sạch theo tọa độ & Vùng:**
    * Loại bỏ các Zone có ID không tồn tại hoặc các Zone được đánh dấu là "Unknown" (ID: 264, 265).
* **Lọc Logic Quãng đường & Tốc độ:**
    * Quãng đường ($d$): $0 < d \le 100$ dặm. Loại bỏ các chuyến đi có quãng đường bằng 0 nhưng vẫn phát sinh phí.
    * Tốc độ trung bình ($v$): $v = \frac{distance}{duration}$. Loại bỏ các chuyến đi có tốc độ vô lý (ví dụ: $> 100$ mph trong thành phố).
* **Xác thực Tài chính:**
    * Tổng hóa đơn ($Total$) và Tiền cước ($Fare$): Phải $> 0$.
    * Tiền Tip ($Tip$): Phải $\ge 0$.
    * Phí ùn tắc và phí sân bay phải tuân theo bảng biểu phí quy định năm 2022.
* **Xử lý Thời gian:**
    * Loại bỏ các bản ghi có thời gian đón khách ($t_{pickup}$) và trả khách ($t_{dropoff}$) không thuộc năm 2022.
    * Thời gian di chuyển ($Duration$): Phải $> 0$ và $< 1440$ phút (24h).

## 5. Phân tích Nâng cao & Mô hình Dự báo

### 5.1. Advanced Outlier Detection (`4_advanced_outlier_detection.ipynb`)
Sử dụng kỹ thuật **Contextual Z-Score** để đảm bảo tính khách quan:
$$Z = \frac{x - \mu_{context}}{\sigma_{context}}$$
Trong đó, $\mu_{context}$ và $\sigma_{context}$ được tính dựa trên từng nhóm cụ thể: `(Pickup_Location_ID, Hour_of_Day)`. Phương pháp này cho phép hệ thống nhận biết rằng một chuyến đi 50$ là bình thường vào giờ cao điểm tại trung tâm, nhưng lại là bất thường (outlier) vào khung giờ thấp điểm ở vùng ngoại ô.

### 5.2. Mô hình hóa Machine Learning
* **Phân tích Lợi nhuận (XGBoost):**
    * Các thuộc tính (Features) đưa vào: `trip_distance`, `PULocationID`, `DOLocationID`, `hour`, `day_of_week`.
    * Phân tích **Feature Importance** cho thấy khoảng cách và địa điểm trả khách là hai yếu tố quyết định 60% khả năng nhận được tiền Tip cao.
* **Dự báo Chuỗi thời gian (SARIMA):**
    * Cấu hình mô hình xử lý tính mùa vụ kép (Daily & Weekly).
    * Kết quả thực nghiệm: Mô hình bám sát các đỉnh nhu cầu vào thứ 6 và thứ 7 hàng tuần.
    * Chỉ số lỗi: **MAE (Mean Absolute Error)** và **MAPE** đều ở mức cực thấp, chứng minh mô hình hoạt động ổn định trên dữ liệu thực tế.



## 6. Hướng dẫn Cài đặt & Thực thi
Để đảm bảo dự án chạy ổn định, vui lòng thực hiện theo các bước sau:

**Yêu cầu hệ thống:**
* RAM: Tối thiểu 8GB (Khuyến nghị 16GB).
* Disk: Còn trống ít nhất 10GB.

**Thực thi:**
```bash
# 1. Cài đặt môi trường ảo và thư viện
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt

# 2. Pipeline thực thi theo thứ tự:
# - Bước 1: Thu thập (src/1_download.ipynb)
# - Bước 2: Xử lý & QA (src/2_process.ipynb)
# - Bước 3: Tính toán KPI (src/3_calculate_kpi.ipynb & 3.1_...)
# - Bước 4: Phân tích sâu (Các file nhóm 4)
# - Bước 5: Dự báo (src/5, 6, 7)
